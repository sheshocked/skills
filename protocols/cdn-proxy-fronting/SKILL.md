---
name: cdn-proxy-fronting
description: 
category: protocols
tags: [cdn-proxy-fronting]
---

## When to Use
Route proxy traffic through CDN (Cloudflare, ArvanCloud) for censorship resistance and IP masking.

## Cloudflare WebSocket Setup
```
Client → CDN (Cloudflare:443) → Your Server (nginx:443)
                                  ↓
                            Xray/V2Ray WS inbound
```

## Nginx Config
```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    ssl_certificate /path/fullchain.pem;
    ssl_certificate_key /path/privkey.pem;

    location /ws-path {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

## Pitfalls
- **CDN IP visibility**: CDN masks your real IP but CDN IP is public
- **WebSocket only**: CDN only supports HTTP-based transports (WS, gRPC)
- **Free tier limits**: Cloudflare free has WebSocket support but rate limits
- **ArvanCloud**: Better for Iran; supports WS and gRPC

## Verification
- Test through CDN with correct host header
- Verify origin server receives proxied traffic
- Check IP shows CDN, not your server