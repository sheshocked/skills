---
name: reverse-proxy-nginx
description: 
category: devops
tags: [reverse-proxy-nginx]
---

## When to Use
Configure Nginx as a reverse proxy for load balancing, SSL termination, caching, rate limiting, and request routing. Covers upstream definitions, proxy buffering, WebSocket support, and security headers.

## Core Concepts
- **upstream**: Backend server pool with load balancing algorithms
- **proxy_pass**: Forward requests to upstream or specific backends
- **SSL termination**: Handle TLS at Nginx, plain HTTP to backends
- **rate limiting**: Limit requests per IP to prevent abuse
- **proxy_buffering**: Control response buffering for performance
- **location blocks**: Route by path, method, or headers

## Workflow
1. Configure upstream with load balancing (least_conn, ip_hash, round-robin)
2. Set up SSL with Let's Encrypt (certbot)
3. Add proxy headers (Host, X-Real-IP, X-Forwarded-For)
4. Configure rate limiting and connection limits
5. Enable caching for static assets
6. Add security headers (HSTS, CSP, X-Frame-Options)

## Key Patterns
```nginx
# /etc/nginx/conf.d/upstream.conf
upstream api_backend {
    least_conn;
    server 10.0.1.10:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:8080 weight=2 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:8080 backup;

    keepalive 32;
}

# Main site config
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        limit_req zone=api_limit burst=20 nodelay;

        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";

        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # WebSocket support
    location /ws {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }

    # Static assets with caching
    location /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
}

# HTTP → HTTPS redirect
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}
```

```nginx
# Load balancing methods
# Round-robin (default)
upstream backend_rr { server 10.0.1.10:8080; server 10.0.1.11:8080; }

# IP hash (sticky sessions)
upstream backend_iphash { ip_hash; server 10.0.1.10:8080; server 10.0.1.11:8080; }

# Least connections
upstream backend_lc { least_conn; server 10.0.1.10:8080; server 10.0.1.11:8080; }

# Weighted
upstream backend_w { server 10.0.1.10:8080 weight=3; server 10.0.1.11:8080 weight=1; }
```

## Pitfalls
- **Missing proxy headers**: X-Real-IP and X-Forwarded-For needed for logging/rate limiting
- **keepalive connections**: Enable `Connection ""` header and `keepalive` in upstream
- **SSL session reuse**: Use `ssl_session_cache shared` for performance
- **Buffering**: Disable for streaming endpoints (`proxy_buffering off`)
- **WebSocket timeout**: Set high `proxy_read_timeout` for WebSocket connections
- **Certbot auto-renewal**: Set up systemd timer or cron for cert renewal

## Verification
```bash
# Test configuration
nginx -t

# Reload without downtime
nginx -s reload

# Verify upstream health
curl -I http://api_backend/health

# Check SSL
openssl s_client -connect api.example.com:443 -servername api.example.com
```