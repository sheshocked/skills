---
name: trojan-gfw
description: 
category: protocols
tags: [trojan-gfw]
---

## When to Use
Set up Trojan-GFW TLS proxy with fallback web server for stealth.

## Config
```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": ["YOUR_PASSWORD"],
    "log_level": 1,
    "ssl": {
        "cert": "/path/fullchain.pem",
        "key": "/path/privkey.pem",
        "alpn": ["h2", "http/1.1"]
    },
    "fallbacks": [{
        "dest": "127.0.0.1:80",
        "path": "/trojan"
    }]
}
```

## Pitfalls
- **Certificate**: Use valid cert from Let's Encrypt
- **Fallback**: Configure web server on port 80 for camouflage
- **Password**: Use strong password, not default

## Verification
- Connect with Trojan client
- Verify fallback serves web content
- Check TLS handshake matches web server