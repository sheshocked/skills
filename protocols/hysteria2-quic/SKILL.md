---
name: hysteria2-quic
description: 
category: protocols
tags: [hysteria2-quic]
---

## When to Use
Deploy Hysteria2 QUIC proxy for high-throughput, low-latency connections with masquerade support.

## Server Config
```yaml
listen: :443
tls:
  cert: /path/fullchain.pem
  key: /path/privkey.pem
auth:
  type: password
  password: YOUR_PASSWORD
masquerade:
  type: proxy
  proxy:
    url: https://www.bing.com
    rewriteHost: true
```

## Client Config
```
server: YOUR_IP:443
auth: YOUR_PASSWORD
tls:
  sni: YOUR_DOMAIN
  insecure: false
```

## Pitfalls
- **Bandwidth**: Set correct bandwidth for BBR congestion control
- **UDP**: QUIC requires UDP — ensure firewall allows it
- **Certificate**: Need valid TLS cert for server

## Verification
- Test with hysteria client
- Check bandwidth limits
- Verify masquerade serves web content