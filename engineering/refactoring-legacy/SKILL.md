---
name: refactoring-legacy
description: - Legacy code is too risky to modify for new features
category: engineering
tags: [refactoring-legacy]
---

## When to Use

- Legacy code is too risky to modify for new features
- Test coverage is too low to safely change behavior
- Architecture has drifted and needs realignment
- Performance degrades and root cause is in legacy layers

## Core Concepts

- **Strangler Fig Pattern**: Gradually replace legacy components by routing traffic to new implementations. Never big-bang rewrite.
- **Characterization Tests**: Write tests that capture CURRENT behavior before changing anything. These tests define the contract.
- **Extract and Isolate**: Pull legacy code behind an interface, then implement new versions behind the same interface.
- **Feature Flags**: Wrap new implementations behind flags. Roll out incrementally, rollback instantly.
- **Boy Scout Rule**: Leave code cleaner than you found it. Small improvements every PR, not giant refactoring PRs.

## Workflow

1. **Write characterization tests** — capture current behavior as tests
2. **Create interface boundary** — abstract the legacy code behind a clean interface
3. **Implement new version** — alongside the old, behind a feature flag
4. **Gradually migrate traffic** — shadow testing → canary → full rollout
5. **Remove old code** — once new code is stable and fully migrated
6. **Celebrate** — refactoring done right is incremental, not heroic

## Key Patterns

```python
# Strangler fig: interface extraction
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    async def charge(self, amount_cents: int, customer_id: str) -> dict:
        pass

class LegacyStripeProcessor(PaymentProcessor):