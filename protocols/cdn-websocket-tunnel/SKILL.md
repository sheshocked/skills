---
name: cdn-websocket-tunnel
description: Establish VLESS + WebSocket + TLS tunnels behind CDN edge proxies (Cloudflare/ArvanCloud).
category: protocols
tags: [cdn-tunnel, websocket-tls, cloudflare-cdn, arvancloud, vless]
---

# Cdn Websocket Tunnel

## When to Use
Use to build durable fallback connections using ArvanCloud or Cloudflare CDN edges when direct IP connections are blocked.

## Prerequisites
- Domain mapped in Cloudflare or ArvanCloud.
- Nginx reverse proxy on origin.

## Workflow
1. Setup local VLESS WebSocket inbound on server (non-TLS).
2. Configure Nginx HTTPS block matching incoming paths and upgrading to WebSockets.
3. Enable DNS proxy (orange cloud) in CDN panel.

## Key Patterns
```nginx
# Nginx Proxy Block
server {
    listen 8443 ssl http2;
    server_name sub.mydomain.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location /mysecretws {
        if ($http_upgrade != "websocket") { return 404; }
        proxy_pass http://127.0.0.1:10085;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

## Pitfalls
- **ALPN configuration:** Never enforce HTTP/2 on WebSockets paths inside Nginx blocks; WebSockets require HTTP/1.1 to switch protocols.
- **CDN port limits:** Ensure Nginx listens on standard HTTPS ports (443, 8443, 2053) allowed by Cloudflare/ArvanCloud CDN.

## Verification
- Query WS endpoint using curl and verify `101 Switching Protocols` response.
- Inspect CDN headers in response to confirm proxies status.
