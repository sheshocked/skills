---
name: sing-box-client
description: Construct advanced Sing-box client JSON configs with TUN interfaces and DNS routing rules.
category: protocols
tags: [sing-box, client-config, tun-mode, dns-rules, route]
---

# Sing Box Client

## When to Use
Use when configuring cross-platform Sing-box clients (Windows, Android, macOS) for direct IP split-tunneling and DNS leak avoidance.

## Prerequisites
- Sing-box binary or client app.

## Workflow
1. Declare a TUN inbound with platform configurations.
2. Establish DNS inbounds and rule blocks.
3. Configure the outbounds (direct, proxy, dns-out).
4. Assign route rules mapping local/ir domains to direct, and everything else to proxy.

## Key Patterns
```json
{
  "dns": {
    "servers": [
      { "tag": "cf", "address": "https://1.1.1.1/dns-query" },
      { "tag": "local", "address": "8.8.8.8", "detour": "direct" }
    ],
    "rules": [
      { "geosite": ["ir"], "server": "local" },
      { "outbound": ["any"], "server": "cf" }
    ]
  },
  "inbounds": [{
    "type": "tun", "tag": "tun-in", "inet4_address": "172.19.0.1/30",
    "auto_route": true, "strict_route": true, "sniff": true
  }],
  "outbounds": [
    { "type": "vless", "tag": "proxy", "server": "185.71.219.72", "port": 443, "uuid": "UUID", "flow": "xtls-rprx-vision" },
    { "type": "direct", "tag": "direct" },
    { "type": "dns", "tag": "dns-out" }
  ],
  "route": {
    "rules": [
      { "protocol": "dns", "outbound": "dns-out" },
      { "geosite": ["ir"], "geoip": ["ir", "private"], "outbound": "direct" }
    ]
  }
}
```

## Pitfalls
- **DNS Leak Loop:** Always force DNS traffic through `dns-out` outbound; otherwise recursive loops freeze routing.
- **Auto Route conflicts:** Conflicting routing rules from system services cause disconnects. Enable `strict_route` to override.

## Verification
- Run `sing-box run -c config.json` and watch for start confirmation logs.
- Perform a dnsleaktest.com lookup to confirm non-leakage.
