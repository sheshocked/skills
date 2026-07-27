---
name: sing-box-config
description: 
category: protocols
tags: [sing-box-config]
---

## When to Use
Configure sing-box as unified proxy core: VLESS/Trojan/Hysteria/TUIC in one binary with TUN mode.

## Config Template
```json
{
    "inbounds": [{
        "type": "tun",
        "tag": "tun-in",
        "inet4_address": "172.19.0.1/30",
        "auto_route": true,
        "strict_route": true,
        "stack": "system"
    }],
    "outbounds": [
        {"type": "vless", "tag": "proxy", "server": "SERVER", "server_port": 443, ...},
        {"type": "direct", "tag": "direct"}
    ],
    "route": {
        "rules": [
            {"ip_is_private": true, "outbound": "direct"},
            {"geosite": "cn", "outbound": "direct"}
        ],
        "final": "proxy"
    }
}
```

## Pitfalls
- **TUN mode**: Requires root or system-level permissions
- **Auto route**: May conflict with other VPNs
- **Rule priority**: First match wins

## Verification
- Test TUN mode captures all traffic
- Verify routing rules work
- Check DNS resolution through tunnel