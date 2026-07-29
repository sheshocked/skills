---
name: vless-reality-nginx
description: Setup VLESS + Reality cohosting with Nginx Stream multiplexing and ssl_preread on port 443.
category: protocols
tags: [xray, vless, reality, nginx-stream, ssl_preread, reality-ghost, proxy]
---

# Vless Reality Nginx

## When to Use
Use this protocol architecture when cohosting administrative websites, subscription endpoints, and VLESS/REALITY active proxies on port 443 simultaneously, bypassing Active Probing scanners.

## Prerequisites
- Nginx with stream and ssl_preread modules.
- Xray-core installed on server.
- Decoy webserver running locally on port 8080.

## Workflow
1. Route incoming port 443 TCP traffic through Nginx Stream.
2. Enable `ssl_preread` to map incoming SNIs.
3. Route allowed Reality SNIs to Xray (port 11443), subscription domains to Nginx HTTP, and unmatched/scanners requests to the decoy.
4. Enable PROXY protocol on Nginx to pass client IPs to Xray.

## Key Patterns

### Nginx Stream multiplexing (/etc/nginx/nginx.conf)
```nginx
load_module modules/ngx_stream_module.so;

stream {
    map $ssl_preread_server_name $rg_backend {
        # Reality SNIs
        www.gstatic.com            127.0.0.1:11443;
        ajax.googleapis.com        127.0.0.1:11443;
        fonts.gstatic.com          127.0.0.1:11443;
        
        # User subscription panel
        sub.eltemas.fun            127.0.0.1:8443;
        
        # Fallback for direct IP and scanners
        default                    127.0.0.1:8080;
    }

    server {
        listen 443 reuseport;
        listen [::]:443 reuseport;
        ssl_preread on;
        proxy_protocol on; # Pass client IP to Xray
        proxy_pass $rg_backend;
    }
}
```

### Xray Reality Inbound Config
```json
{
  "inbounds": [{
    "listen": "127.0.0.1",
    "port": 11443,
    "protocol": "vless",
    "settings": {
      "clients": [{"id": "e44d32a0-43df-40ab-829d-4e92bf180da1", "flow": "xtls-rprx-vision"}],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.gstatic.com:443",
        "xver": 1,
        "serverNames": ["www.gstatic.com", "ajax.googleapis.com", "fonts.gstatic.com"],
        "privateKey": "PRIVATE_KEY",
        "shortIds": ["81b8672d2cbf1c16"]
      }
    }
  }]
}
```

## Pitfalls
- **Duplicate Stream declarations:** Never nest `stream {}` inside `http {}` configurations.
- **Xray Xver Mismatch:** Ensure `"xver": 1` is configured on Xray when `proxy_protocol on` is set on Nginx.

## Verification
- Verify `nginx -t` passes.
- Test endpoint: `curl -v -k https://<server-ip>` loads decoy site contents.

