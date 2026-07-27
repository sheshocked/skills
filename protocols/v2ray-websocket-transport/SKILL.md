---
name: v2ray-websocket-transport
description: 
category: protocols
tags: [v2ray-websocket-transport]
---

## When to Use
Configure WebSocket and gRPC transports for V2Ray/Xray: path design, nginx reverse proxy, keepalive.

## Nginx WebSocket Config
```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    ssl_certificate /path/cert.pem;
    ssl_certificate_key /path/key.pem;

    location /ws-path {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 300s;
    }
}
```

## gRPC Config
```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    location /grpc-service {
        grpc_pass grpc://127.0.0.1:8080;
        grpc_read_timeout 300s;
    }
}
```

## Pitfalls
- **WebSocket path**: Use unique path to avoid detection
- **gRPC requires HTTP/2**: nginx must have http2 enabled
- **Keepalive**: Set proper timeouts for long connections

## Verification
- Test WebSocket upgrade with curl
- Verify gRPC connection works
- Check nginx logs for errors