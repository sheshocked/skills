---
name: vless-reality-nginx
description: Masterclass configuration for VLESS + Reality cohosted on port 443 with Nginx stream module using ssl_preread for SNI-based routing.
category: protocols
tags: [xray, vless, reality, nginx-stream, ssl_preread, multi-core, reality-ghost]
---

# VLESS Reality Behind Nginx Stream sni-filter Masterclass

## When to Use
Use this protocol pattern when you want to cohost an administrative website, a client subscription server, and VLESS/REALITY active connections on port 443 simultaneously. This defeats Active Probing scanners by falling back to a decoy site or standard Google endpoints when unmatched SNIs or direct IPs hit port 443.

## Prerequisites
- Nginx compiled with `--with-stream` and `--with-stream_ssl_preread_module`.
- Xray-core installed on the VPS.
- Decoy website running on port 8080.
- A subdomain configured on a DNS provider (e.g., `sub.eltemas.fun`).

## Workflow
1. Route incoming port 443 TCP traffic through Nginx's stream module.
2. Enable `ssl_preread on` to sniff the Server Name Indication (SNI) from the TLS ClientHello without decrypting the stream.
3. Map the SNI: Allowed Reality targets (e.g., Google SNIs) go to Xray (port 11443), your subscription domain goes to Nginx HTTP (port 8080), and direct IPs or scanners go to a decoy site (port 8082).
4. Configure Xray Reality inbound to listen on 127.0.0.1:11443 and decrypt only traffic passing the Reality handshake.

## Key Patterns

### Nginx Stream Configuration (/etc/nginx/nginx.conf)
```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 4096;
    multi_accept on;
}

stream {
    # Sniff Server Name Indication (SNI) and map to backend ports
    map $ssl_preread_server_name $rg_backend {
        # Reality SNIs mapped directly to Xray reality port
        www.gstatic.com            127.0.0.1:11443;
        ajax.googleapis.com        127.0.0.1:11443;
        fonts.gstatic.com          127.0.0.1:11443;
        fonts.googleapis.com       127.0.0.1:11443;
        
        # User subscription/management panels SNI
        sub.eltemas.fun            127.0.0.1:8443;
        
        # Default fallback (Scanners/Blocked requests) -> Decoy
        default                    127.0.0.1:8080;
    }

    server {
        listen 443 reuseport;
        listen [::]:443 reuseport;
        ssl_preread on;
        proxy_protocol on; # Pass client IP to Xray
        proxy_pass $rg_backend;
        proxy_buffer_size 16k;
    }
}
```

### Xray Configuration (/usr/local/etc/xray/config.json)
```json
{
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 11443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "e44d32a0-43df-40ab-829d-4e92bf180da1",
            "flow": "xtls-rprx-vision",
            "level": 0
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.gstatic.com:443",
          "xver": 1,
          "serverNames": [
            "www.gstatic.com",
            "ajax.googleapis.com",
            "fonts.gstatic.com",
            "fonts.googleapis.com"
          ],
          "privateKey": "PRIVATE_KEY_GENERATED_BY_X25519",
          "shortIds": [
            "81b8672d2cbf1c16",
            "23871959d88c4cea"
          ]
        },
        "sockopt": {
          "acceptProxyProtocol": true,
          "tcpFastOpen": true
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls", "quic"],
        "routeOnly": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag": "direct"
    }
  ]
}
```

## Pitfalls
- **Nginx duplicate stream blocks:** Appending stream blocks when Nginx modular structure loads `/etc/nginx/modules-enabled/50-mod-stream.conf` dynamically can trigger a duplicate stream module load error. Ensure `load_module` statements are only declared once.
- **Xray Xver config mismatch:** When passing `proxy_protocol on` from Nginx, you must specify `"xver": 1` in the Xray streamSettings. Failing to do so causes connection resets.

## Verification
- Test Nginx configuration: `nginx -t` should pass.
- Test active probing behavior using curl: `curl -v -k https://<server-ip>` should display the decoy site (running on port 8080) instead of closing connection or exposing Xray TLS.
