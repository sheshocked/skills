---
name: nginx-stream-multiplexing
description: 
category: protocols
tags: [nginx-stream-multiplexing]
---

## When to Use
Cohost multiple TLS services on port 443 using nginx ssl_preread SNI routing — no need for multiple IPs.

## Config
```nginx
stream {
    map $ssl_preread_server_name $backend {
        vless.yourdomain.com    vless_backend;
        trojan.yourdomain.com   trojan_backend;
        default                  default_backend;
    }

    upstream vless_backend { server 127.0.0.1:8443; }
    upstream trojan_backend { server 127.0.0.1:8444; }
    upstream default_backend { server 127.0.0.1:443; }

    server {
        listen 443;
        listen [::]:443;
        ssl_preread on;
        proxy_pass $backend;
        proxy_protocol off;
    }
}
```

## Pitfalls
- **SNI required**: Client must send SNI (most modern clients do)
- **SSL termination**: stream block doesn't terminate SSL — just routes
- **No http directives**: stream block uses different config syntax

## Verification
- Test each SNI routes to correct backend
- Verify all services work through single :443
- Check nginx error logs for routing issues