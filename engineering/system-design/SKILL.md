---
name: system-design
description: - Designing a new system from scratch or major architectural overhaul
category: engineering
tags: [system-design]
---

## When to Use

- Designing a new system from scratch or major architectural overhaul
- Evaluating trade-offs between scalability, reliability, and cost
- Preparing for system design interviews or architecture review meetings
- Onboarding engineers to understand existing system topology

## Core Concepts

- **CAP Theorem**: Distributed systems sacrifice one of Consistency, Availability, Partition tolerance. Choose AP for high-availability (Cassandra), CP for correctness (ZooKeeper), CA for single-node (PostgreSQL).
- **Horizontal vs Vertical Scaling**: Vertical = bigger box (simple, limited). Horizontal = more boxes (complex, elastic). Start vertical, graduate horizontal when you hit limits.
- **Back-of-Envelope Estimation**: QPS = daily_active_users / 86400. Storage = QPS × avg_record_size × retention_days × 86400. Bandwidth = QPS × response_size_bytes.
- **Single Points of Failure (SPOF)**: Every component must fail gracefully. Load balancers, databases, caches, DNS — all need redundancy.

## Workflow

1. **Clarify requirements** — functional (what it does) vs non-functional (latency, throughput, availability, consistency)
2. **Estimate scale** — QPS, storage, bandwidth numbers drive all downstream decisions
3. **Define API contract** — before designing internals, define the interface
4. **High-level architecture** — draw major components and data flow
5. **Deep dive** — scale the bottleneck component, add caching, sharding, replication
6. **Address failure modes** — what breaks, how to detect, how to recover

## Key Patterns

```yaml
# Typical web application architecture
services:
  client:
    - CDN (CloudFront/Cloudflare)
    - Load Balancer (ALB/NLB)
  application:
    - API Gateway (rate limiting, auth, routing)
    - Stateless app servers (auto-scaling group)
    - Worker pool (async jobs via SQS/Redis)
  data:
    - Primary DB (PostgreSQL with read replicas)
    - Cache layer (Redis cluster, 3-node)
    - Search index (Elasticsearch)
    - Object storage (S3)
  observability:
    - Metrics (Prometheus + Grafana)
    - Logs (ELK or Loki)
    - Traces (Jaeger/OpenTelemetry)
```

```python
# Back-of-envelope estimation helper
def estimate_system(
    dau: int,
    avg_requests_per_user: float,
    avg_response_bytes: int,
    storage_days: int,
    read_write_ratio: float = 10,
):
    rps = (dau * avg_requests_per_user) / 86400
    bandwidth_bps = rps * avg_response_bytes * 8
    storage_bytes = rps * avg_response_bytes * 86400 * storage_days
    write_rps = rps / (1 + read_write_ratio)
    read_rps = rps - write_rps
    return {
        "rps": round(rps),
        "write_rps": round(write_rps),
        "read_rps": round(read_rps),
        "bandwidth_mbps": round(bandwidth_bps / 1e6, 2),
        "storage_gb": round(storage_bytes / 1e9, 2),
    }

# Example: 1M DAU, 10 requests/user, 5KB avg response, 30 day retention
result = estimate_system(1_000_000, 10, 5000, 30)
# {'rps': 116, 'write_rps': 11, 'read_rps': 105, 'bandwidth_mbps': 4.63, 'storage_gb': 150.0}
```

```python
# Circuit breaker pattern for external dependencies
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30, half_open_max=1):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.half_open_max = half_open_max
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.last_failure_time = 0
        self.half_open_count = 0

    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
                self.half_open_count = 0
            else:
                raise RuntimeError("Circuit is OPEN — request rejected")

        try:
            result = func(*args, **kwargs)
            if self.state == CircuitState.HALF_OPEN:
                self.half_open_count += 1
                if self.half_open_count >= self.half_open_max:
                    self.state = CircuitState.CLOSED
                    self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = CircuitState.OPEN
            raise
```

## Pitfalls

- **Over-engineering early**: Don't build for 10x scale when you need 1x. Premature optimization adds complexity that slows you down.
- **Ignoring the write path**: Everyone designs for reads (caching, CDNs). Writes are harder — you can't cache them away.
- **Single-region assumptions**: If you need global availability, design for multi-region from day one. Retrofitting is painful.
- **Forgetting idempotency**: Network retries happen. Your API must handle duplicate requests gracefully.

## Verification

- Run load tests (k6, Locust) against your design estimates
- Chaos engineer: kill random instances, verify no data loss
- Map every request path end-to-end, identify the bottleneck
- Ask: "What happens when this component fails at 3 AM?"
- Verify SLOs: 99.9% availability = 8.76 hours downtime/year