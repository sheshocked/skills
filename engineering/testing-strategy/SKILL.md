---
name: testing-strategy
description: - Setting up test infrastructure for a new project
category: engineering
tags: [testing-strategy]
---

## When to Use

- Setting up test infrastructure for a new project
- Inconsistent test quality across the team
- Too many slow/flaky tests that nobody trusts
- Need to define what should be tested at each level

## Core Concepts

- **Testing Pyramid**: Unit (fast, many) → Integration (moderate, some) → E2E (slow, few). More units, fewer E2E.
- **Test Isolation**: Each test runs independently. No shared state, no ordering dependency, no external service calls.
- **Arrange-Act-Assert (AAA)**: Three distinct phases. Setup, exercise, verify. Clear and readable.
- **Test Doubles**: Mock (verify interactions) vs Stub (provide canned data). Use stubs by default; mock only for side effects.
- **Mutation Testing**: Modify code (kill a condition), verify tests fail. Proves tests actually catch bugs.

## Workflow

1. **Write unit tests first** — for business logic, algorithms, data transformations
2. **Add integration tests** — for DB queries, API endpoints, external service calls
3. **Add E2E tests** — critical user journeys only (login, checkout, payment)
4. **Set up CI gates** — tests must pass before merge
5. **Monitor flakiness** — track and fix flaky tests immediately
6. **Review test coverage** — not just %, but coverage of critical paths

## Key Patterns

```python
# Unit test with pytest fixtures (no external dependencies)
import pytest
from unittest.mock import AsyncMock, patch

@pytest.fixture
def order_service(db_mock):
    return OrderService(db=db_mock)

@pytest.fixture
def db_mock():
    mock = AsyncMock()
    mock.fetch.return_value = [
        {"id": "ord_1", "status": "pending", "total": 1000},
        {"id": "ord_2", "status": "shipped", "total": 2500},
    ]
    return mock

async def test_list_pending_orders(order_service, db_mock):
    # Arrange
    db_mock.fetch.return_value = [
        {"id": "ord_1", "status": "pending", "total": 1000},
    ]
    # Act
    orders = await order_service.list_orders(status="pending")
    # Assert
    assert len(orders) == 1
    assert orders[0].status == "pending"
    db_mock.fetch.assert_called_once_with(
        "SELECT * FROM orders WHERE status = $1", "pending"
    )

async def test_create_order_validates_quantity():
    # Test validation logic without touching DB
    with pytest.raises(ValidationError, match="Quantity must be positive"):
        create_order(product_id="p1", quantity=-1)
```

```python
# Integration test with test database
import pytest
from httpx import AsyncClient

@pytest.fixture
async def test_app(test_db):