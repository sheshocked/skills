---
name: database-schema-design
description: Build safe migrations, partial indexes, multi-tenant row level security (RLS), and GIN/JSONB indexes.
category: engineering
tags: [database, postgresql, migrations, RLS, indexes, sql]
---

# Database Schema & Migrations Masterclass

## When to Use
Use when designing database schemas and writing migrations for production databases where table locks can freeze APIs.

## Prerequisites
- PostgreSQL database environment.

## Workflow
1. Write backward-compatible migrations (avoid dropping columns immediately).
2. Establish partial indexes to reduce storage sizes.
3. Configure Row Level Security (RLS) to isolate multi-tenant data.

## Key Patterns

### Safe PostgreSQL Migration & Indexing
```sql
-- 1. Create index concurrently to avoid locking table writes
CREATE INDEX CONCURRENTLY idx_users_active_email 
ON users(email) 
WHERE status = 'active';

-- 2. Configure Row Level Security (RLS)
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON user_profiles
FOR ALL
USING (tenant_id = current_setting('app.current_tenant_id', true));
```

## Pitfalls
- **Running migrations without timeouts:** Adding column defaults on large tables without `lock_timeout` locks the database, causing API timeouts. Always set `SET lock_timeout = '2s';` before migration blocks.
- **Over-indexing tables:** Adding indexes to write-heavy tables slows insertions significantly. Optimize only active query parameters.

## Verification
- Run `EXPLAIN ANALYZE SELECT * FROM users WHERE status = 'active' AND email = 'user@example.com';` to confirm execution hits the partial index.
- Verify security policies fail queries executed without active tenant contexts.
