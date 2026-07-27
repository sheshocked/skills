---
name: event-sourcing-cqrs
description: - Domain requires complete audit trail (finance, healthcare, legal)
category: engineering
tags: [event-sourcing-cqrs]
---

## When to Use

- Domain requires complete audit trail (finance, healthcare, legal)
- Complex business logic where current state is hard to reconstruct
- Multiple read models needed from the same data (reporting, analytics, search)
- Temporal queries: "What was the state at time T?" or "How did this evolve?"

## Core Concepts

- **Event Sourcing**: Store every state change as an immutable event. State = replay all events from beginning.
- **CQRS**: Command Query Responsibility Segregation. Separate write model (commands) from read model (queries).
- **Projections**: Derived read models built by consuming events. Can be rebuilt from scratch at any time.
- **Aggregate**: Consistency boundary. All events for one aggregate are ordered. Cross-aggregate events are eventually consistent.
- **Event Store**: Append-only log of events. Never delete or modify. The source of truth.

## Workflow

1. **Identify aggregates** — what entities have lifecycle (Order, Account, Inventory)
2. **Design events** — past-tense verbs: `OrderPlaced`, `PaymentReceived`, `ItemShipped`
3. **Implement command handlers** — validate business rules, emit events
4. **Build projections** — one per read model (order summary, user history, analytics)
5. **Handle versioning** — events must be forward-compatible (new fields optional, old fields never removed)
6. **Snapshot optimization** — periodically snapshot aggregate state to avoid replaying 10000 events

## Key Patterns

```python
# Event store implementation
from dataclasses import dataclass, field
from typing import List
import json
import time

@dataclass
class Event:
    event_type: str
    aggregate_id: str
    aggregate_type: str
    data: dict
    metadata: dict = field(default_factory=dict)
    version: int = 0
    timestamp: float = field(default_factory=time.time)

class EventStore:
    def __init__(self, db):
        self.db = db

    async def append(self, events: List[Event], expected_version: int = None) -> None: