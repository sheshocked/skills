---
name: concurrency-patterns
description: - Multiple independent tasks need to run simultaneously
category: engineering
tags: [concurrency-patterns]
---

## When to Use

- Multiple independent tasks need to run simultaneously
- API needs to handle many concurrent requests efficiently
- Background processing (jobs, workers) alongside request handling
- Combining results from multiple parallel operations

## Core Concepts

- **GIL (Global Interpreter Lock)**: Python's GIL prevents true parallel execution of Python bytecode. Use multiprocessing for CPU-bound, asyncio for I/O-bound.
- **asyncio**: Cooperative multitasking. Tasks yield control at `await` points. Great for I/O-bound, terrible for CPU-bound.
- **Thread Pool**: `concurrent.futures.ThreadPoolExecutor` for mixing sync/async I/O-bound work.
- **Process Pool**: `ProcessPoolExecutor` for CPU-bound work that can't use asyncio.
- **Semaphore**: Limit concurrency. Don't fire 1000 DB connections — use a semaphore to cap at 50.

## Workflow

1. **Identify task type** — CPU-bound vs I/O-bound vs mixed
2. **Choose concurrency model** — asyncio, threads, processes, or combination
3. **Set resource limits** — semaphores, pool sizes, timeouts
4. **Handle errors** — individual task failures shouldn't crash the batch
5. **Implement backpressure** — don't queue unlimited work
6. **Monitor** — active tasks, queue depth, error rate

## Key Patterns

```python
# AsyncIO: parallel independent I/O operations
import asyncio
import aiohttp

async def fetch_user_data(user_id: str, session: aiohttp.ClientSession) -> dict:
    async with session.get(f"https://api.example.com/users/{user_id}") as resp:
        return await resp.json()

async def fetch_all_users(user_ids: list[str]) -> list[dict]:
    connector = aiohttp.TCPConnector(limit=50)  # Max 50 concurrent connections
    semaphore = asyncio.Semaphore(20)  # Max 20 in-flight requests

    async def limited_fetch(uid):
        async with semaphore:
            return await fetch_user_data(uid, session)

    async with aiohttp.ClientSession(connector=connector) as session:
        tasks = [limited_fetch(uid) for uid in user_ids]
        results = await asyncio.gather(*tasks, return_exceptions=True)

    # Separate successes from failures
    successful = [r for r in results if not isinstance(r, Exception)]
    failed = [r for r in results if isinstance(r, Exception)]
    if failed:
        logger.warning(f"{len(failed)} fetches failed: {failed[0]}")
    return successful
```

```python
# CPU-bound parallel processing with ProcessPoolExecutor
from concurrent.futures import ProcessPoolExecutor, as_completed
import hashlib

def cpu_intensive_hash(data: bytes) -> str: