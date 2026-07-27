---
name: api-design-rest
description: - Designing new REST APIs for web/mobile/backend services
category: engineering
tags: [api-design-rest]
---

## When to Use

- Designing new REST APIs for web/mobile/backend services
- Reviewing API contracts before implementation begins
- Migrating from SOAP/XML-RPC to RESTful services
- Standardizing inconsistent API conventions across teams

## Core Concepts

- **Resource-Oriented Design**: URLs represent nouns (resources), HTTP methods represent verbs (actions). `/users/{id}` not `/getUser?id=1`.
- **HTTP Semantics**: GET (safe, idempotent), POST (create), PUT (idempotent replace), PATCH (partial update), DELETE (idempotent remove).
- **HATEOAS**: Responses include links to related resources, enabling clients to navigate without hardcoding URLs.
- **Content Negotiation**: Use `Accept` and `Content-Type` headers. Support JSON by default, XML if contractually required.
- **Pagination**: Always paginate list endpoints. Cursor-based > offset-based for real-time data.

## Workflow

1. **Identify resources** — nouns that represent your domain entities
2. **Define URL structure** — consistent, hierarchical, lowercase, plural nouns
3. **Assign HTTP methods** — map CRUD to GET/POST/PUT/PATCH/DELETE
4. **Design request/response schemas** — OpenAPI 3.0 spec
5. **Add error contracts** — structured error responses with codes
6. **Version the API** — URL path versioning (`/v1/`) or header-based

## Key Patterns

```yaml
# OpenAPI 3.0 spec skeleton
openapi: "3.0.3"
info:
  title: "Order Service API"
  version: "1.2.0"
paths:
  /v1/orders:
    get:
      summary: "List orders"
      parameters:
        - name: cursor
          in: query
          schema: { type: string }
        - name: limit
          in: query
          schema: { type: integer, default: 20, maximum: 100 }
      responses:
        "200":
          description: "Paginated order list"
          content:
            application/json:
              schema:
                type: object
                properties:
                  data: { type: array, items: { $ref: "#/components/schemas/Order" } }
                  next_cursor: { type: string, nullable: true }
                  has_more: { type: boolean }
    post:
      summary: "Create order"
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CreateOrder" }
      responses:
        "201": { description: "Order created" }
        "422": { description: "Validation error", content:
          application/json:
            schema: { $ref: "#/components/schemas/ValidationError" } }
```

```python
# FastAPI with proper REST patterns
from fastapi import FastAPI, Query, HTTPException, status
from pydantic import BaseModel, Field
from typing import Optional
import uuid

app = FastAPI()

class CreateOrder(BaseModel):
    product_id: str
    quantity: int = Field(..., gt=0, le=1000)
    idempotency_key: str = Field(..., min_length=16, max_length=64)

class OrderResponse(BaseModel):
    id: str
    status: str
    product_id: str
    quantity: int
    created_at: str

class PaginatedOrders(BaseModel):
    data: list[OrderResponse]
    next_cursor: Optional[str] = None
    has_more: bool

@app.post("/v1/orders", response_model=OrderResponse, status_code=201)
async def create_order(order: CreateOrder):
    # Idempotency check — prevent duplicate creates on retry
    existing = await db.find_by_idempotency_key(order.idempotency_key)
    if existing:
        return existing
    result = await db.create_order(order.dict())
    return result

@app.get("/v1/orders", response_model=PaginatedOrders)
async def list_orders(
    cursor: Optional[str] = Query(None),
    limit: int = Query(20, ge=1, le=100),
):
    rows = await db.list_orders(cursor=cursor, limit=limit + 1)
    has_more = len(rows) > limit
    data = rows[:limit]
    next_cursor = data[-1].id if has_more else None
    return PaginatedOrders(data=data, next_cursor=next_cursor, has_more=has_more)

# Error response format
@app.exception_handler(HTTPException)
async def api_error_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.detail.get("code", "UNKNOWN"),
                "message": exc.detail.get("message", str(exc)),
                "details": exc.detail.get("details", []),
            }
        },
    )
```

## Pitfalls

- **Noun-verb URLs**: `/getUsers`, `/createOrder` — these are RPC endpoints pretending to be REST. Use HTTP methods for verbs.
- **Leaking implementation details**: Don't expose DB table names, internal IDs, or ORM models in your API.
- **Missing idempotency**: POST without idempotency keys means retries create duplicates. Always support them for critical writes.
- **Over-fetching**: Returning entire DB rows when clients need 3 fields. Use field selection (`?fields=id,name`).
- **Ignoring HTTP status codes**: Returning 200 with error in body. Use 4xx for client errors, 5xx for server errors.

## Verification

- Validate OpenAPI spec: `npx @openapitools/openapi-generator-cli validate`
- Run contract tests (Pact, Schemathesis) against implementation
- Test with `curl -v` to verify status codes, headers, and body format
- Load test pagination with realistic data volumes
- Review every error path returns structured, actionable error responses