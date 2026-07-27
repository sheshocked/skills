---
name: error-handling-resilience
description: - External dependencies are unreliable (APIs, databases, file systems)
category: engineering
tags: [error-handling-resilience]
---

## When to Use

- External dependencies are unreliable (APIs, databases, file systems)
- Errors need to be actionable (not just "something went wrong")
- System must survive partial failures without full outages
- Error responses must not leak internal details to clients

## Core Concepts

- **Structured Errors**: Every error has a code, message, and context. Never return raw exceptions to clients.
- **Error Hierarchy**: Domain errors (business rules), Infrastructure errors (DB, network), Programming errors (bugs).
- **Retry with Backoff**: Exponential backoff + jitter prevents thundering herd on recovery.
- **Bulkhead Pattern**: Isolate failure domains. One slow service shouldn't exhaust connection pools for everything.
- **Graceful Degradation**: Serve degraded results when dependencies fail (cached data, default values).

## Workflow

1. **Classify errors** — transient (retryable) vs permanent (don't retry)
2. **Define error contracts** — structured responses with codes for clients
3. **Implement retry logic** — exponential backoff with jitter
4. **Add circuit breakers** — stop calling failing services
5. **Log with context** — correlation IDs, request context, stack traces
6. **Alert on patterns** — error rate spikes, not individual errors

## Key Patterns

```python
# Structured error hierarchy
from enum import Enum
from dataclasses import dataclass, field
from typing import Any, Optional

class ErrorCode(Enum):
    # Client errors (4xx)
    VALIDATION_ERROR = "VALIDATION_ERROR"
    NOT_FOUND = "NOT_FOUND"
    UNAUTHORIZED = "UNAUTHORIZED"
    RATE_LIMITED = "RATE_LIMITED"

    # Infrastructure errors (5xx, retryable)
    DB_TIMEOUT = "DB_TIMEOUT"
    EXTERNAL_API_ERROR = "EXTERNAL_API_ERROR"
    SERVICE_UNAVAILABLE = "SERVICE_UNAVAILABLE"

    # Programming errors (5xx, not retryable)
    INTERNAL_ERROR = "INTERNAL_ERROR"

@dataclass
class AppError(Exception):
    code: ErrorCode
    message: str
    details: dict = field(default_factory=dict)
    retryable: bool = False
    cause: Optional[Exception] = None

    def to_response(self) -> dict:
        return {
            "error": {
                "code": self.code.value,
                "message": self.message,
                "details": self.details,
            }
        }

def raise_not_found(resource: str, resource_id: str):
    raise AppError(
        code=ErrorCode.NOT_FOUND,
        message=f"{resource} with id '{resource_id}' not found",
    )

def raise_validation(field: str, message: str):
    raise AppError(
        code=ErrorCode.VALIDATION_ERROR,
        message=f"Validation failed: {field}",
        details={"field": field, "reason": message},
    )
```

```python
# Retry with exponential backoff + jitter
import asyncio
import random

async def retry_with_backoff(
    func,
    max_retries: int = 3,
    base_delay: float = 0.5,
    max_delay: float = 30.0,
    retryable_exceptions: tuple = (ConnectionError, TimeoutError),
):
    for attempt in range(max_retries + 1):
        try:
            return await func()
        except retryable_exceptions as e:
            if attempt == max_retries:
                raise
            # Exponential backoff with jitter
            delay = min(base_delay * (2 ** attempt), max_delay)
            jitter = random.uniform(0, delay * 0.5)
            logger.warning(
                f"Retry {attempt + 1}/{max_retries} after {delay + jitter:.2f}s",
                extra={"error": str(e), "attempt": attempt + 1},
            )
            await asyncio.sleep(delay + jitter)
```

```python
# Error logging with correlation context
import structlog
import uuid
from contextvars import ContextVar

request_id_var: ContextVar[str] = ContextVar("request_id", default="")

def setup_logging():
    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.add_log_level,
            structlog.processors.JSONRenderer(),
        ],
    )

async def api_handler(request):
    rid = request.headers.get("X-Request-ID", str(uuid.uuid4())[:12])
    request_id_var.set(rid)
    log = structlog.get_logger()
    try:
        result = await process_request(request)
        log.info("request.success", path=request.url.path, status=200)
        return JSONResponse(result, status_code=200)
    except AppError as e:
        log.warning(
            "request.app_error",
            error_code=e.code.value,
            message=e.message,
            path=request.url.path,
        )
        return JSONResponse(e.to_response(), status_code=error_to_http(e.code))
    except Exception as e:
        log.error("request.unhandled", error=str(e), exc_info=True)
        return JSONResponse(
            {"error": {"code": "INTERNAL_ERROR", "message": "An error occurred"}},
            status_code=500,
        )

def error_to_http(code: ErrorCode) -> int:
    mapping = {
        ErrorCode.VALIDATION_ERROR: 422,
        ErrorCode.NOT_FOUND: 404,
        ErrorCode.UNAUTHORIZED: 401,
        ErrorCode.RATE_LIMITED: 429,
        ErrorCode.DB_TIMEOUT: 503,
        ErrorCode.EXTERNAL_API_ERROR: 502,
        ErrorCode.SERVICE_UNAVAILABLE: 503,
        ErrorCode.INTERNAL_ERROR: 500,
    }
    return mapping.get(code, 500)
```

## Pitfalls

- **Catching broad exceptions**: `except Exception` hides bugs. Catch specific exceptions. Let programming errors propagate.
- **Silent failures**: Logging an error but returning success. Clients think it worked, data is corrupted.
- **Retry everything**: Don't retry validation errors, authentication failures, or 404s. Only retry transient failures.
- **Swallowing exceptions**: `except: pass` is a bug factory. Every exception must be handled or propagated.
- **Leaking stack traces**: Never return internal error details to clients. Log them server-side.

## Verification

- Inject failures (network timeout, DB error) → verify retry behavior
- Check error logs have correlation IDs and enough context to debug
- Verify client never sees internal error details or stack traces
- Test circuit breaker: after N failures, calls should stop and return fallback
- Load test with degraded dependency → verify graceful degradation, not crash