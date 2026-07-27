---
name: feature-flags
description: - Gradual rollout of new features without full deployment
category: engineering
tags: [feature-flags]
---

## When to Use

- Gradual rollout of new features without full deployment
- A/B testing different implementations or UIs
- Kill switch for problematic features in production
- Enabling features per-tenant, per-region, or per-user-segment

## Core Concepts

- **Boolean Flags**: Simple on/off. Good for kill switches, simple toggles.
- **Percentage Rollout**: Gradually increase traffic to new code (1% → 10% → 50% → 100%).
- **User Segmentation**: Enable by user attributes (plan, region, signup date).
- **Operational Flags**: Downtime windows, maintenance mode, rate limit adjustments.
- **Flag Lifecycle**: Create → active → permanently on → archived. Clean up stale flags regularly.

## Workflow

1. **Define the flag** — name, description, owner, expected cleanup date
2. **Implement flag checks** — wrap new code paths behind the flag
3. **Configure targeting** — who sees the new behavior
4. **Monitor metrics** — compare old vs new behavior in production
5. **Graduate rollout** — 1% → 10% → 50% → 100%
6. **Clean up** — remove flag code once 100% and stable

## Key Patterns

```python
# Feature flag client with local evaluation
import json
from typing import Optional

class FeatureFlag:
    def __init__(self, flags_config: dict):
        self.flags = flags_config

    def is_enabled(
        self,
        flag_name: str,
        user_id: Optional[str] = None,
        default: bool = False,
    ) -> bool:
        flag = self.flags.get(flag_name, {})
        if not flag.get("enabled", False):
            return default

        # Percentage rollout based on deterministic hashing
        if "rollout_percentage" in flag:
            if user_id:
                bucket = hash(f"{flag_name}:{user_id}") % 100
                return bucket < flag["rollout_percentage"]
            return hash(flag_name) % 100 < flag["rollout_percentage"]

        # User segment targeting
        if "segments" in flag and user_id:
            return user_id in flag["segments"]

        return flag.get("enabled", default)

# Configuration
FEATURE_FLAGS = {
    "new_checkout_flow": {
        "enabled": True,
        "rollout_percentage": 25,  # 25% of users see new checkout
        "owner": "payments-team",
        "cleanup_date": "2024-03-01",
    },
    "dark_mode": {
        "enabled": True,
        "segments": ["beta_testers", "enterprise"],
        "owner": "frontend-team",
    },
    "maintenance_mode": {
        "enabled": False,
        "owner": "platform-team",
    },
}

flags = FeatureFlag(FEATURE_FLAGS)

# Usage in code
async def checkout(user_id: str, cart):
    if flags.is_enabled("new_checkout_flow", user_id=user_id):
        return await new_checkout(user_id, cart)
    return await legacy_checkout(user_id, cart)
```

```python
# Feature flag middleware for HTTP routes
from fastapi import Request

@app.middleware("http")
async def feature_flags_middleware(request: Request, call_next):
    user_id = get_user_from_request(request)
    request.state.flags = flags
    request.state.user_id = user_id

    # Maintenance mode check
    if flags.is_enabled("maintenance_mode"):
        return JSONResponse(
            status_code=503,
            content={"error": {"code": "MAINTENANCE", "message": "System under maintenance"}},
        )

    return await call_next(request)
```

```python
# Flag lifecycle cleanup tool
import json
from datetime import datetime, timedelta

def find_stale_flags(flags_config: dict, grace_days: int = 30) -> list[str]: