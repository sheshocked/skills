---
name: dns-anti-censorship
description: 
category: protocols
tags: [dns-anti-censorship]
---

## When to Use
Deploy anti-censorship DNS: DoH/DoT, DNS leak prevention, ECH, fragmented TLS.

## DNS Setup
```bash
# DNS over HTTPS (DoH)
# /etc/dnscrypt-proxy/dnscrypt-proxy.toml
listen_addresses = ['127.0.0.1:53']
server_names = ['cloudflare', 'google']
dnscrypt_servers = true
doh_servers = true
```

## ECH (Encrypted Client Hello)
- TLS extension that encrypts SNI in ClientHello
- Requires DNS HTTPS record with ECH config
- Supported by Cloudflare, Firefox, Chrome 105+

## Pitfalls
- **DNS leaks**: Ensure all DNS goes through encrypted resolver
- **ECH adoption**: Not all servers support ECH yet
- **Fragmentation**: TLS fragmentation helps bypass some DPI

## Verification
- Test DNS resolution through encrypted resolver
- Check for DNS leaks at dnsleaktest.com
- Verify ECH works with supported browsers