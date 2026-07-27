---
name: performance-profiling
description: - API response times exceed SLA targets
category: engineering
tags: [performance-profiling]
---

## When to Use

- API response times exceed SLA targets
- CPU usage spikes unexpectedly under load
- Memory usage grows continuously (potential leak)
- Database queries are slow but you can't identify the bottleneck

## Core Concepts

- **Profile Before Optimizing**: Never guess. Measure. The bottleneck is rarely where you think.
- **CPU Profiling**: Sample call stacks at high frequency. Flame graphs show where time is spent. Use `py-spy`, `perf`, `pprof`.
- **Memory Profiling**: Track allocations over time. Look for growing heaps, large object counts. Use `tracemalloc`, `memory_profiler`.
- **I/O Profiling**: Trace blocking calls (DB, HTTP, file). Use OpenTelemetry spans or `strace`/`dtrace`.
- **Amdahl's Law**: Speedup is limited by the serial fraction. Optimizing 95% parallel code gives at most 20x speedup.

## Workflow

1. **Establish baseline** — measure current performance under realistic load
2. **Identify bottleneck** — CPU-bound? I/O-bound? Memory-bound?
3. **Profile in production-like environment** — dev results differ from prod
4. **Analyze flame graph** — find the hottest code path
5. **Optimize the bottleneck** — only the bottleneck, not everything
6. **Verify improvement** — re-run baseline comparison

## Key Patterns

```bash
# Python CPU profiling with py-spy (no code changes needed)
pip install py-spy

# Profile a running process
py-spy record --pid 12345 --output profile.svg --duration 30

# Profile a script
py-spy record -- python -m myapp.server

# Top consumers without full profile
py-spy top --pid 12345
```

```python
# Python memory profiling with tracemalloc
import tracemalloc

def profile_memory():
    tracemalloc.start()
    # ... your code here ...
    snapshot = tracemalloc.take_snapshot()
    top_stats = snapshot.statistics("lineno")
    print("[ Top 10 memory consumers ]")
    for stat in top_stats[:10]:
        print(f"  {stat}")
    # Example output:
    #   /app/models/user.py:42 — 12.3 MiB: user = User(name=name)
    #   /app/cache.py:88 — 8.7 MiB: self._cache = {}

# Track memory over time
import tracemalloc
import time

tracemalloc.start()
snapshots = []
for i in range(10):
    time.sleep(1)
    snapshots.append(tracemalloc.take_snapshot())

# Compare first and last to find growth
first, last = snapshots[0], snapshots[-1]
stats = last.compare_to(first, "lineno")
print("Memory growth:")
for stat in stats[:10]:
    if stat.size_diff > 0:
        print(f"  +{stat.size_diff / 1024:.1f} KB: {stat}")
```

```python
# OpenTelemetry tracing for I/O bottleneck detection
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter

provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("myapp")

@tracer.start_as_current_span("get_user_orders")
async def get_user_orders(user_id: str):
    span = trace.get_current_span()
    # Each sub-operation becomes a child span
    with tracer.start_as_current_span("db_query"):
        orders = await db.query("SELECT * FROM orders WHERE user_id = $1", user_id)
    with tracer.start_as_current_span("enrich_orders"):
        enriched = await asyncio.gather(*[enrich(o) for o in orders])
    span.set_attribute("order_count", len(enriched))
    return enriched
```

```bash
# Database query analysis
# PostgreSQL: enable slow query logging
ALTER SYSTEM SET log_min_duration_statement = 100;  # Log queries > 100ms
SELECT pg_reload_conf();

# Analyze specific query
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.*, COUNT(o.id) as order_count
FROM users u LEFT JOIN orders o ON o.user_id = u.id
WHERE u.org_id = 'org_123'
GROUP BY u.id ORDER BY order_count DESC LIMIT 20;

# Check for missing indexes
SELECT schemaname, relname, seq_scan, seq_tup_read,
       idx_scan, idx_tup_fetch, n_live_tup
FROM pg_stat_user_tables
WHERE seq_scan > 100 AND seq_tup_read > 10000
ORDER BY seq_tup_read DESC;
```

## Pitfalls

- **Premature optimization**: Don't optimize code that isn't the bottleneck. Profile first.
- **Dev environment profiling**: Your laptop isn't production. Profile with production-scale data and traffic patterns.
- **Ignoring GC pauses**: Python's GC can cause latency spikes. Profile GC activity: `gc.set_debug(gc.DEBUG_STATS)`.
- **Caching the wrong thing**: If the bottleneck is compute, cache results. If it's DB, cache queries. Don't cache what's already fast.
- **Single-threaded assumptions**: Python's GIL means CPU-bound code doesn't benefit from threads. Use multiprocessing.

## Verification

- Compare before/after metrics: p50, p95, p99 latency
- Load test with 2x expected traffic — verify headroom exists
- Check CPU utilization stays under 70% at peak — leave room for spikes
- Memory should be flat over 24h — no upward trend means no leak
- Profile each deployment — catch regressions before they hit production