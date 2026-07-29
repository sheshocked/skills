---
name: hysteria2-quic
description: Deploy Hysteria2 QUIC-based VPN protocol, tuning congestion control for high packet loss networks.
category: protocols
tags: [hysteria2, quic, congestion-control, bbr, udp-proxy]
---

# Hysteria2 Quic

## When to Use
Use Hysteria2 to achieve high-speed connections on networks experiencing heavy packet loss (e.g. mobile networks during throttling).

## Prerequisites
- Port 443 UDP free on server.
- Valid SSL certificate.

## Workflow
1. Install Hysteria2 server engine.
2. Tune system UDP buffer limits using sysctl.
3. Configure server with SSL cert, port, and authentication password.

## Key Patterns
```yaml
# /etc/hysteria/config.yaml
listen: :443

tls:
  cert: /etc/letsencrypt/live/proxy.domain.com/fullchain.pem
  key: /etc/letsencrypt/live/proxy.domain.com/privkey.pem

auth:
  type: password
  password: mysecretpassword

masquerade:
  type: proxy
  proxy:
    url: https://www.bing.com
```

## Pitfalls
- **UDP throttling:** Some providers block high UDP flows. Set up port hopping on client and server to bypass blocks.
- **Low UDP buffer warning:** Always increase kernel UDP socket limits to avoid dropped packets.

## Verification
- Verify server runs: `systemctl status hysteria-server`.
- Test speed and bandwidth performance with Hysteria client console.
