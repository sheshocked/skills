---
name: caching-strategies
description: - Response times exceed SLAs due to repeated expensive computations
category: engineering
tags: [caching-strategies]
---

## When to Use

- Response times exceed SLAs due to repeated expensive computations
- Database is the bottleneck — reads far exceed writes
- Expensive external API calls need local caching
- Static or slowly-changing data is being re-fetched repeatedly

## Core Concepts

- **Cache-Aside (Lazy Loading)**: App checks cache first, misses hit DB, writes to cache. Most common pattern.
- **Write-Through**: Writes go to cache + DB simultaneously. Data always consistent but writes are slower.
- **Write-Behind (Write-Back)**: Writes go to cache, async batch flush to DB. Fast writes, risk of data loss on crash.
- **TTL (Time-to-Live)**: Maximum staleness you can tolerate. Short TTL = fresh but more DB load. Long TTL = fast but stale.
- **Cache Invalidation**: Harder than caching itself. Version your keys, use TTL as safety net, never trust invalidation alone.

## Workflow

1. **Profile before caching** — identify the expensive operation (CPU, DB, network)
2. **Choose cache layer** — L1 (in-process), L2 (Redis/Memcached), L3 (CDN)
3. **Define cache key strategy** — deterministic, versioned, with namespace
4. **Set TTL** — based on freshness requirements, not "forever"
5. **Handle stampede** — singleflight/lock to prevent thundering herd
6. **Monitor hit rate** — <80% hit rate means something is wrong with key design or TTL

## Key Patterns

```python
# Cache-aside with stampede protection (singleflight)
import hashlib
import json
import time
from functools import wraps

class CacheManager:
    def __init__(self, redis_client):
        self.redis = redis_client
        self._inflight = {}  # singleflight lock

    async def get_or_set(self, key: str, ttl: int, fetch_fn):
        # Check cache first
        cached = await self.redis.get(key)
        if cached is not None:
            return json.loads(cached)

        # Stampede protection: only one concurrent fetch per key
        if key in self._inflight:
            # Wait for inflight request
            while key in self._inflight:
                await asyncio.sleep(0.05)
            return json.loads(await self.redis.get(key))

        try:
            self._inflight[key] = True
            result = await fetch_fn()
            await self.redis.setex(key, ttl, json.dumps(result))
            return result
        finally:
            self._inflight.pop(key, None)

# Usage
async def get_user_profile(user_id: str):
    return await cache.get_or_set(
        key=f"user:profile:{user_id}:v1",
        ttl=300,  # 5 minutes
        fetch_fn=lambda: db.get_user_profile(user_id),
    )
```

```python
# Write-behind cache with batch flush
import asyncio
from collections import defaultdict

class WriteBehindCache:
    def __init__(self, redis_client, db_client, flush_interval=5, batch_size=100):
        self.redis = redis_client
        self.db = db_client
        self.flush_interval = flush_interval
        self.batch_size = batch_size
        self._buffer = defaultdict(dict)
        self._lock = asyncio.Lock()

    async def set(self, key: str, value: dict):
        async with self._lock:
            self._buffer[key] = value
        # Also write to Redis for immediate reads
        await self.redis.setex(key, 300, json.dumps(value))

    async def flush_loop(self):