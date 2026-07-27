---
name: database-schema-design
description: - Designing a new database schema for an application
category: engineering
tags: [database-schema-design]
---

## When to Use

- Designing a new database schema for an application
- Refactoring an existing schema that's become unwieldy
- Planning data migrations for zero-downtime deployments
- Evaluating normalization levels (3NF vs denormalization)

## Core Concepts

- **Normalization**: 1NF (atomic values), 2NF (partial dependency removal), 3NF (transitive dependency removal). Normalize for write-heavy, denormalize for read-heavy.
- **Indexing Strategy**: B-tree for equality/range, GIN for full-text/JSONB, partial indexes for filtered queries. Every WHERE clause should hit an index.
- **Migration Safety**: DDL changes must be backward-compatible. `ALTER TABLE ADD COLUMN` is safe; `ALTER TABLE DROP COLUMN` is not until all code stops reading it.
- **Referential Integrity**: Foreign keys are documentation + enforcement. Use them unless you have a performance-critical reason not to.
- **Partitioning**: Range (time-series), Hash (even distribution), List (multi-tenant). Partition before you need it — retrofitting is painful.

## Workflow

1. **Model entities** — identify business objects and their relationships
2. **Define constraints** — primary keys, foreign keys, unique, check constraints
3. **Choose indexes** — based on actual query patterns, not theoretical ones
4. **Plan migrations** — every schema change needs a reversible migration
5. **Load test** — validate query performance with production-scale data
6. **Document** — ER diagrams, index purpose, migration runbooks

## Key Patterns

```sql
-- Multi-tenant schema with row-level security
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    plan TEXT NOT NULL DEFAULT 'free',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    org_id UUID NOT NULL REFERENCES organizations(id),
    email TEXT NOT NULL,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(org_id, email)
);

-- Partial index: only index active users (saves space)
CREATE INDEX idx_users_active ON users(org_id, email)
    WHERE deleted_at IS NULL;

-- Composite index matching query pattern: lookup by org + status + date range
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    org_id UUID NOT NULL REFERENCES organizations(id),
    status TEXT NOT NULL DEFAULT 'pending',
    total_cents BIGINT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_orders_org_status_date
    ON orders(org_id, status, created_at DESC);

-- JSONB for flexible attributes without schema changes
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    org_id UUID NOT NULL,
    name TEXT NOT NULL,
    attributes JSONB NOT NULL DEFAULT '{}',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- GIN index for JSONB containment queries
CREATE INDEX idx_products_attrs ON products USING GIN(attributes);
-- Query: SELECT * FROM products WHERE attributes @> '{"color": "red"}';

-- Row-level security policy
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY org_isolation ON orders
    USING (org_id = current_setting('app.current_org_id')::UUID);

-- Safe migration pattern: add column with default, backfill, then use
ALTER TABLE orders ADD COLUMN IF NOT EXISTS tax_cents BIGINT DEFAULT 0;
-- Backfill in batches (not all at once!)
UPDATE orders SET tax_cents = 0
WHERE id IN (SELECT id FROM orders WHERE tax_cents IS NULL LIMIT 10000);
```

```python
# Migration safety checker
import re

UNSAFE_PATTERNS = [
    (r"ALTER\s+TABLE.*DROP\s+COLUMN", "Dropping column — ensure no code reads it first"),
    (r"ALTER\s+TABLE.*DROP\s+CONSTRAINT", "Dropping constraint — verify no依赖"),
    (r"UPDATE\s+\w+\s+SET", "Bulk UPDATE — use batched approach for large tables"),
    (r"DELETE\s+FROM\s+\w+\s+WHERE", "Bulk DELETE — use batched approach"),
]

def check_migration_safety(sql: str) -> list[str]:
    warnings = []
    for pattern, message in UNSAFE_PATTERNS:
        if re.search(pattern, sql, re.IGNORECASE):
            warnings.append(message)
    return warnings
```

## Pitfalls

- **Over-indexing**: Every index slows writes. 10 indexes on a hot table = 10x slower inserts. Profile, don't guess.
- **Using UUIDs as primary keys**: Random UUIDs destroy B-tree page locality. Use UUIDv7 (time-ordered) or bigserial.
- **Ignoring NULL semantics**: `NULL != NULL` in SQL. `WHERE col = NULL` never matches. Use `IS NULL`.
- **Premature denormalization**: Duplicate data before you need to. Normalization is free; denormalization adds sync bugs.
- **N+1 at the query level**: ORMs that lazy-load relationships. Eager-load or batch explicitly.

## Verification

- `EXPLAIN ANALYZE` every slow query — check for sequential scans on large tables
- Run `pg_badger` on slow query log — find missing indexes
- Load test with 10x production data volume
- Verify migration rollback works: `pg_dump` before, compare after
- Check index usage: `SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0` — unused indexes cost write performance