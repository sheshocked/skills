---
name: database-admin
description: 
category: devops
tags: [database-admin]
---

## When to Use
Administer PostgreSQL, MySQL/MariaDB, and Redis databases: installation, configuration, backup, performance tuning, replication, user management, and monitoring.

## Core Concepts
- **Connection pooling**: PgBouncer for PostgreSQL, ProxySQL for MySQL
- **WAL archiving**: Write-Ahead Log for point-in-time recovery
- **Replication**: Streaming replication for read replicas
- **Vacuum**: PostgreSQL garbage collection (prevent table bloat)
- **Slow query log**: Identify and optimize expensive queries
- **索引策略**: Index design for query performance (B-tree, GIN, GiST)

## Workflow
1. Install database with production-ready configuration
2. Configure connection pooling
3. Set up automated backups with WAL archiving
4. Tune shared_buffers, work_mem, effective_cache_size
5. Configure replication for read replicas
6. Monitor with pg_stat_statements and exporters

## Key Patterns
```ini
# PostgreSQL production config — postgresql.conf
listen_addresses = '*'
port = 5432
max_connections = 200

# Memory — for dedicated DB server with 16GB RAM
shared_buffers = 4GB              # 25% of RAM
effective_cache_size = 12GB       # 75% of RAM
work_mem = 64MB                   # per-sort/hash operation
maintenance_work_mem = 1GB        # VACUUM, CREATE INDEX

# WAL and backup
wal_level = replica
archive_mode = on
archive_command = 'pgbackrest --stanza=main archive-push %p'
max_wal_senders = 5
wal_keep_size = '1GB'

# Query performance
random_page_cost = 1.1            # SSD storage
effective_io_concurrency = 200    # SSD storage
log_min_duration_statement = 500  # Log slow queries (>500ms)
log_statement = 'ddl'             # Log schema changes
log_checkpoints = on

# Connection pooling with PgBouncer
[databases]
myapp = host=127.0.0.1 port=5432 dbname=myapp

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction           # release connection after transaction
max_client_conn = 1000
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
```

```bash
# PostgreSQL user management
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;

# Index strategy
CREATE INDEX CONCURRENTLY idx_orders_user_created ON orders (user_id, created_at DESC);
CREATE INDEX CONCURRENTLY idx_orders_status ON orders (status) WHERE status IN ('pending', 'processing');
CREATE INDEX CONCURRENTLY idx_events_payload ON events USING GIN (payload jsonb_path_ops);

# Monitor slow queries
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

# Table bloat check
SELECT schemaname, relname, n_dead_tup, last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 10;
```

```yaml
# Redis production config
# redis.conf
bind 0.0.0.0
port 6379
requirepass YOUR_PASSWORD
maxmemory 2gb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
```

```bash
# Backup with pgbackrest
pgbackrest --stanza=main --type=full backup
pgbackrest --stanza=main --type=diff backup

# Verify backup
pgbackrest --stanza=main info

# Restore to specific point in time
pgbackrest --stanza=main --target="2024-01-15 14:30:00" \
  --type=time --target-action=promote restore
```

## Pitfalls
- **Autovacuum tuning**: Don't disable it — tune autovacuum_work_mem for large tables
- **Connection exhaustion**: Use PgBouncer; don't let apps open connections directly
- **Missing indexes**: Run `EXPLAIN ANALYZE` on slow queries before optimizing
- **WAL bloat**: Set wal_keep_size appropriately; monitor pg_replication_slots
- **Connection string leaks**: Never commit credentials to code — use secrets management
- **Disk space**: WAL and replication slots can consume disk if not managed

## Verification
```bash
# Check database health
psql -c "SELECT version();"
psql -c "SELECT * FROM pg_stat_activity WHERE state != 'idle';"
psql -c "SELECT pg_size_pretty(pg_database_size('myapp'));"

# Check replication lag
psql -c "SELECT now() - pg_last_xact_replay_timestamp() AS replication_lag;"

# Check index usage
psql -c "SELECT indexrelname, idx_scan FROM pg_stat_user_indexes ORDER BY idx_scan;"
```