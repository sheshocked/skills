---
name: proxy-chain-routing
description: Chain outbound proxy protocols (VLESS -> SOCKS5 -> Residential IP) to hide server IPs.
category: protocols
tags: [proxy-chain, socks5-outbound, residential-proxy, xray]
---

# Proxy Chain Routing

## When to Use
Use when you want to connect using an Xray server, but force the actual internet requests to exit through a Turkish residential IP to avoid VPN blocks.

## Prerequisites
- Active SOCKS5/HTTP residential proxy subscription.

## Workflow
1. Set up standard VLESS inbound.
2. Create SOCKS5 outbound pointing to residential proxy endpoint.
3. Build routing rules forwarding user connections to residential outbound.

## Key Patterns
```json
{
  "inbounds": [{ "port": 10080, "protocol": "vless", "settings": { "clients": [{"id": "UUID"}] } }],
  "outbounds": [
    {
      "protocol": "socks",
      "tag": "residential",
      "settings": {
        "servers": [{
          "address": "res.proxyprovider.com",
          "port": 5000,
          "users": [{ "user": "username", "pass": "password" }]
        }]
      }
    },
    { "protocol": "freedom", "tag": "direct" }
  ],
  "routing": {
    "rules": [
      { "type": "field", "outboundTag": "residential", "network": "tcp,udp" }
    ]
  }
}
```

## Pitfalls
- **High Latency:** Double proxies slow connections. Route only sensitive domains through residential outbounds; direct others through standard outbounds.
- **DNS Leakage:** Ensure client DNS matches destination rules.

## Verification
- Connect via client and visit `ipinfo.io` to check IP and ISP match residential specifications.
- Check Xray logs for output tag routing confirmations.
