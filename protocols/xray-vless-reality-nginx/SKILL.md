---
name: xray-vless-reality-nginx
description: State-of-the-art configuration for Xray VLESS + Reality cohosted on port 443 with Nginx stream module using ssl_preread for SNI-based routing, including direct-IP bypass fallback.
category: protocols
tags: [masterclass, networking, 3d, pro]
---
# Xray VLESS Reality Behind Nginx Stream sni-filter Masterclass

## Overview
State-of-the-art configuration for Xray VLESS + Reality cohosted on port 443 with Nginx stream module using ssl_preread for SNI-based routing, including direct-IP bypass fallback.

## Core Instructions

1. **Host Cohosting Architecture:** Route all incoming port 443 traffic via Nginx Stream. Use `ssl_preread` to inspect the SNI.
2. **Reality Fallback Configuration:** Configure Xray to serve as the default destination for unknown SNIs or direct IP access to avoid detection by active probing.
3. **Optimizing Reality Parameters:** Implement shortIds and Snis dynamically matching Google SNIs or local high-trust domains.


## Proven Recipes
```kotlin

# Nginx Stream Config Block:
stream {
    map $ssl_preread_server_name $backend {
        default local_xray_reality;
        eltemas.fun flask_app;
    }
    server {
        listen 443;
        proxy_pass $backend;
        ssl_preread on;
    }
}

```

## Potential Pitfalls
1. Avoid mismatched SNI headers on client routing tables.
2. Ensure Deno websocket limits are respected during high-concurrency relays.
