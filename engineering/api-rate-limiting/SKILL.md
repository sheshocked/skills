---
name: api-rate-limiting
description: - Public-facing APIs that need protection from abuse
category: engineering
tags: [api-rate-limiting]
---

## When to Use

- Public-facing APIs that need protection from abuse
- Internal APIs where noisy-neighbor affects other tenants
- Third-party API integrations where you must respect their limits
- Cost control: preventing runaway compute/storage from excessive requests

## Core Concepts

- **Rate Limits**: Maximum requests per time window (e.g., 1000/hour). Prevent abuse, control costs.
- **Throttling vs Rejection**: Throttle (slow down) vs reject (429 immediately). Choose based on use case.
- **Token Bucket**: Tokens added at fixed rate, consumed per request. Allows bursts up to bucket capacity.
- **Sliding Window**: More accurate than fixed window. Counts requests in rolling window, not calendar window.
- **Per-User vs Global**: Limit per API key, per IP, or globally. Per-user prevents noisy-neighbor; global prevents overload.

## Workflow

1. **Define limits** — per-tier (free: 100/hr, paid: 1000/hr, enterprise: 10k/hr)
2. **Choose algorithm** — token bucket (allows bursts) vs sliding window (smoother)
3. **Implement at gateway** — centralized rate limiting, not per-service
4. **Return standard headers** — `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
5. **Handle 429 gracefully** — clients should retry with backoff
6. **Monitor** — track limit hits, identify patterns of abuse

## Key Patterns

```python
# Token bucket rate limiter (Redis-backed)
import time
import redis

class TokenBucketLimiter:
    def __init__(self, redis_client, max_tokens: int, refill_rate: float):
        self.redis = redis_client
        self.max_tokens = max_tokens
        self.refill_rate = refill_rate  # tokens per second

    def _get_bucket_key(self, identifier: str) -> str:
        return f"ratelimit:{identifier}"

    def allow(self, identifier: str) -> dict:
        key = self._get_bucket_key(identifier)
        now = time.time()
        bucket = self.redis.hgetall(key)

        if not bucket:
            tokens = self.max_tokens - 1
            self.redis.hset(key, mapping={"tokens": tokens, "last_refill": now})
            self.redis.expire(key, int(self.max_tokens / self.refill_rate) + 1)
            return {"allowed": True, "remaining": tokens}

        tokens = float(bucket[b"tokens"])
        last_refill = float(bucket[b"last_refill"])

        # Refill tokens based on elapsed time
        elapsed = now - last_refill
        tokens = min(self.max_tokens, tokens + elapsed * self.refill_rate)

        if tokens < 1:
            retry_after = (1 - tokens) / self.refill_rate
            return {"allowed": False, "remaining": 0, "retry_after": retry_after}

        tokens -= 1
        self.redis.hset(key, mapping={"tokens": tokens, "last_refill": now})
        return {"allowed": True, "remaining": int(tokens)}

# FastAPI middleware
from fastapi import Request, Response

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    api_key = request.headers.get("X-API-Key", request.client.host)
    limiter = TokenBucketLimiter(redis_client, max_tokens=100, refill_rate=10)

    result = limiter.allow(api_key)
    response = await call_next(request)

    response.headers["X-RateLimit-Limit"] = "100"
    response.headers["X-RateLimit-Remaining"] = str(result["remaining"])

    if not result["allowed"]:
        response.headers["Retry-After"] = str(int(result["retry_after"]))
        return JSONResponse(
            status_code=429,
            content={"error": {"code": "RATE_LIMITED", "message": "Too many requests"}},
        )
    return response
```

```python
# Sliding window counter (more accurate than fixed window)
class SlidingWindowLimiter: