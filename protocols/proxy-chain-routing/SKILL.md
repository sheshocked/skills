---
name: proxy-chain-routing
description: 
category: protocols
tags: [proxy-chain-routing]
---

## When to Use
Chain proxy upstreams, implement geo-routing, split tunneling, and load balancing.

## Xray Routing Example
```json
{
    "routing": {
        "rules": [
            {"type": "field", "ip": ["geoip:cn"], "outboundTag": "direct"},
            {"type": "field", "domain": ["geosite:google"], "outboundTag": "proxy"},
            {"type": "field", "domain": ["geosite:category-ads-all"], "outboundTag": "block"}
        ],
        "strategy": "rules"
    }
}
```

## Pitfalls
- **Rule order**: First matching rule wins
- **GeoIP/Geosite**: Must update databases regularly
- **DNS routing**: DNS queries must be routed correctly too

## Verification
- Test domain-based routing works
- Verify geoIP rules correctly match
- Check load balancing distributes traffic