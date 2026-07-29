---
name: vless-reality-nginx
description: Configure VLESS + REALITY cohosted behind Nginx stream multiplexing using ssl_preread on port 443.
category: protocols
tags: [vless, reality, nginx-stream, ssl_preread, reality-ghost, proxy]
---

# Vless Reality Nginx

## When to Use
Use this protocol pattern when you want to cohost your administrative website, subscription page, and VLESS/REALITY active proxy connection on port 443 simultaneously, hiding the proxy behind legitimate web services.

## Prerequisites
- Linux VPS with Docker / Nginx and Xray installed.
- Public domain pointing to the server IP.

## Workflow
1. Route all incoming port 443 traffic via Nginx Stream.
2. Enable `ssl_preread` on Nginx Stream to peak at the client TLS ClientHello SNI.
3. Configure SNI routing: match allowed reality SNIs to local Xray port, otherwise fallback to decoy webserver.
4. Set up local Xray reality inbound on loopback port.

## Key Patterns
```nginx
# Nginx Stream Block (/etc/nginx/nginx.conf)
load_module modules/ngx_stream_module.so;

stream {
    map $ssl_preread_server_name $rg_backend {
        # Reality SNIs
        www.gstatic.com            127.0.0.1:11443;
        ajax.googleapis.com        127.0.0.1:11443;
        fonts.gstatic.com          127.0.0.1:11443;
        
        # Admin / Sub site SNI
        sub.eltemas.fun            127.0.0.1:8080;
        
        # Scanners fall to decoy
        default                    127.0.0.1:8082;
    }

    server {
        listen 443 reuseport;
        ssl_preread on;
        proxy_pass $rg_backend;
    }
}
```

```json
// Xray Reality Inbound Config
{
  "inbounds": [{
    "listen": "127.0.0.1",
    "port": 11443,
    "protocol": "vless",
    "settings": {
      "clients": [{"id": "CLIENT_UUID", "flow": "xtls-rprx-vision"}],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.gstatic.com:443",
        "xver": 0,
        "serverNames": ["www.gstatic.com", "ajax.googleapis.com", "fonts.gstatic.com"],
        "privateKey": "PRIVATE_KEY",
        "shortIds": ["81b8672d2cbf1c16"]
      }
    }
  }]
}
```

## Pitfalls
- **Port Conflict:** Xray or Nginx HTTP trying to bind directly to 0.0.0.0:443 will crash the service. Ensure only Nginx Stream binds to port 443; bind Xray and Nginx HTTP to `127.0.0.1`.
- **Active Probing Detection:** Unmatched SNIs must lead to a valid TLS decoy site (like google or a normal blogs site) to pass scanning tests.

## Verification
- Test connection: `curl -v --resolve sub.eltemas.fun:443:127.0.0.1 https://sub.eltemas.fun` should load the admin site.
- Verify proxy connection using client (e.g. v2rayNG) connecting over REALITY parameters.
