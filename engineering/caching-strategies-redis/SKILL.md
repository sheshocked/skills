---
name: caching-strategies-redis
description: Establish cache-aside pattern, cache invalidation, singleflight execution, and redis storage.
category: engineering
tags: [redis, caching, cache-aside, concurrency, performance]
---

# Caching Strategies Redis

## When to Use
Use to decrease query latencies and protect relational databases from excess load.

## Prerequisites
- Running Redis instance.

## Workflow
1. Read from cache first.
2. If cache miss, pull from database, update cache, and return data.
3. Enforce TTL values on all cached records.
4. Implement Singleflight to prevent cache stampedes.

## Key Patterns
```python
import redis
import json

cache = redis.Redis(host='localhost', port=6379, db=0)

def get_user_profile(user_id: int):
    cache_key = f"user:{user_id}"
    cached_data = cache.get(cache_key)
    
    if cached_data:
        return json.loads(cached_data)
        
    # Cache Miss -> Pull database
    profile = db_fetch_user_profile(user_id)
    
    # Store to cache with 10 minutes expiration TTL
    cache.setex(cache_key, 600, json.dumps(profile))
    return profile
```

## Pitfalls
- **Cache stampede (Thundering Herd):** When cached entries expire under high load, multiple threads query DB at the same time. Use singleflight/locks.
- **Cache invalidation delays:** Inconsistent updates cause users to read stale data. Invalidate cache keys immediately upon database mutations.

## Verification
- Monitor Redis queries: `redis-cli monitor`.
- Audit cache hit rates and database query load under pressure.
