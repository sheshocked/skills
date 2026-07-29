---
name: api-design-best-practices
description: Establish RESTful standard routing, cursor pagination, OpenAPI documentation, and idempotency structures.
category: engineering
tags: [api-design, restful, pagination, openapi, idempotency]
---

# Api Design Best Practices

## When to Use
Use when developing scalable backend APIs to ensure standards compliance and data resilience.

## Prerequisites
- API development environment.

## Workflow
1. Define resource-centric paths using standard HTTP verbs.
2. Implement cursor-based pagination for large datasets.
3. Validate idempotency keys for mutable state requests (e.g. payments).

## Key Patterns
```python
# Cursor Pagination pattern in Python
def get_paginated_users(cursor: str = None, limit: int = 20):
    query = User.select()
    if cursor:
        query = query.where(User.id > decode_cursor(cursor))
    
    users = list(query.order_by(User.id).limit(limit + 1))
    
    has_more = len(users) > limit
    next_cursor = encode_cursor(users[-1].id) if has_more else None
    
    return {
        "data": users[:limit],
        "pagination": {"next_cursor": next_cursor, "has_more": has_more}
    }
```

## Pitfalls
- **Offset pagination issues:** Offset pagination (`LIMIT 20 OFFSET 10000`) becomes slow as database scales. Always use cursor-based queries.
- **Mutable actions without Idempotency:** Failing to enforce idempotency keys on payment transactions leads to double billing on network timeout retries.

## Verification
- Run OpenAPI validator tools to check specs.
- Test duplicate POST requests with identical idempotency keys; verify only one transaction executes.
