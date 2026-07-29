---
name: system-design-patterns
description: Implement robust system patterns (Circuit Breaker, Singleflight, Rate Limiting) in production APIs.
category: engineering
tags: [system-design, circuit-breaker, singleflight, concurrency, engineering]
---

# System Design Patterns

## When to Use
Use when building APIs that communicate with external microservices or third-party resources, ensuring network failures do not cascade and exhaust server resources.

## Prerequisites
- Python 3.10+.

## Workflow
1. Wrap network calls inside a Circuit Breaker object.
2. Configure threshold counters for consecutive timeouts.
3. Automatically switch state to OPEN (failing fast) during downtime.
4. Implement Singleflight to deduplicate concurrent requests.

## Key Patterns

### Circuit Breaker Implementation (circuit.py)
```python
import time

class CircuitBreakerOpenException(Exception): pass

class CircuitBreaker:
    def __init__(self, failure_threshold=3, recovery_time=30):
        self.failure_threshold = failure_threshold
        self.recovery_time = recovery_time
        
        self.state = "CLOSED" # CLOSED, OPEN, HALF-OPEN
        self.failures = 0
        self.last_failure_time = 0

    def __call__(self, func, *args, **kwargs):
        current_time = time.time()
        
        # Check recovery transition
        if self.state == "OPEN":
            if current_time - self.last_failure_time > self.recovery_time:
                self.state = "HALF-OPEN"
            else:
                raise CircuitBreakerOpenException("Circuit is open. Fail fast.")

        try:
            result = func(*args, **kwargs)
            if self.state == "HALF-OPEN":
                self.state = "CLOSED"
                self.failures = 0
            return result
        except Exception as e:
            self.failures += 1
            self.last_failure_time = current_time
            if self.failures >= self.failure_threshold:
                self.state = "OPEN"
            raise e
```

## Pitfalls
- **Incorrect recovery timers:** Setting recovery time too short causes thrashing states under persistent downtime. Set generous recovery delays.
- **Dropping context logs:** Always trigger alert logs when circuit transitions state.

## Verification
- Run load tests mocking downstream delays; verify requests fail fast immediately after the failure threshold is crossed.

