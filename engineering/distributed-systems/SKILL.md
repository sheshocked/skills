---
name: distributed-systems
description: - Building systems spanning multiple machines, data centers, or cloud regions
category: engineering
tags: [distributed-systems]
---

## When to Use

- Building systems spanning multiple machines, data centers, or cloud regions
- Debugging issues that only manifest under partial failures
- Designing for availability when individual components fail regularly
- Implementing consensus, leader election, or distributed transactions

## Core Concepts

- **Partial Failure**: In distributed systems, components fail independently. Network partitions mean node A can't reach B, but both think they're alive.
- **Consensus**: Nodes must agree on a value. Raft/Paxos are the standards. Don't roll your own — use etcd, ZooKeeper, or Consul.
- **Vector Clocks**: Track causality across nodes. If VC(A) > VC(B), A happened after B. If neither dominates, they're concurrent.
- **Gossip Protocol**: Nodes periodically exchange state with random peers. Eventually consistent but highly available (Cassandra, SWIM).
- **Split Brain**: Two leaders emerge during network partition. Prevent with fencing tokens, quorum-based decisions.

## Workflow

1. **Identify failure modes** — every network call can fail, timeout, or return partial data
2. **Design for idempotency** — duplicate messages are inevitable; handlers must tolerate them
3. **Choose consistency model** — strong (Paxos/Raft) vs eventual (gossip/CQRS)
4. **Implement failure detection** — health checks, heartbeats, gossip
5. **Add circuit breakers** — prevent cascading failures across service boundaries
6. **Chaos test** — inject failures systematically to validate resilience

## Key Patterns

```python
# Consistent hashing for distributed data placement
import hashlib
from bisect import bisect_right

class ConsistentHash:
    def __init__(self, nodes: list[str], virtual_nodes: int = 150):
        self.ring = {}
        self.sorted_keys = []
        for node in nodes:
            for i in range(virtual_nodes):
                key = self._hash(f"{node}:{i}")
                self.ring[key] = node
                self.sorted_keys.append(key)
        self.sorted_keys.sort()

    def _hash(self, key: str) -> int:
        return int(hashlib.md5(key.encode()).hexdigest(), 16)

    def get_node(self, item: str) -> str:
        h = self._hash(item)
        idx = bisect_right(self.sorted_keys, h) % len(self.sorted_keys)
        return self.ring[self.sorted_keys[idx]]

    def get_distribution(self, items: list[str]) -> dict[str, int]:
        dist = {}
        for item in items:
            node = self.get_node(item)
            dist[node] = dist.get(node, 0) + 1
        return dist

# Usage
hash_ring = ConsistentHash(["node-1", "node-2", "node-3"])
# Adding a node only remaps ~1/N of keys
hash_ring.add_node("node-4")
```

```python
# Leader election using etcd (simplified)
import etcd3
import time
import os

class LeaderElection:
    def __init__(self, etcd_client, service_name: str):
        self.client = etcd_client
        self.key = f"/leaders/{service_name}"
        self.lease = None
        self.is_leader = False

    def try_acquire(self, ttl: int = 15) -> bool:
        self.lease = self.client.lease(ttl)
        success = self.client.transaction(
            compare=[self.client.transactions.create(self.key) == 0],
            success=[self.client.transactions.put(self.key, os.getenv("HOSTNAME"), lease=self.lease)],
            failure=[],
        )
        self.is_leader = success[0]
        return self.is_leader

    def run(self, on_leader, on_follower):
        while True:
            if not self.is_leader:
                if self.try_acquire():
                    on_leader()
            else:
                # Keepalive to renew lease
                self.lease.refresh()
            time.sleep(5)
```

```python
# Failure injection for chaos testing
import random
import functools

def chaos_fraud(failure_rate: float = 0.1, exception_cls=ConnectionError):