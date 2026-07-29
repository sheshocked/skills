---
name: nginx-reverse-proxy
description: Establish HTTP/2, rate limiting, secure headers, and WebSocket upgrades on Nginx.
category: devops
tags: [nginx, reverse-proxy, ssl, rate-limit, websocket]
---

# Nginx Reverse Proxy

## When to Use
Use when fronting applications or panels to configure SSL, route endpoints, and restrict request spamming.

## Prerequisites
- Nginx installed.

## Workflow
1. Setup rate limiting zones in `nginx.conf`.
2. Configure SSL parameters and TLS ciphers.
3. Set WebSockets headers.

## Key Patterns
```nginx
# /etc/nginx/sites-available/default
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    listen 443 ssl http2;
    server_name api.eltemas.fun;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Pitfalls
- **Duplicate Header definitions:** Adding CORS headers in both application and proxy blocks throws errors.
- **Incorrect resolver configurations:** Check internal proxy routes mapping local addresses.

## Verification
- Run `nginx -t` to confirm syntax validity.
- Run `curl -I https://api.eltemas.fun/api/` and inspect returned headers.
