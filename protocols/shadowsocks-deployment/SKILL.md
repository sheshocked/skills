---
name: shadowsocks-deployment
description: 
category: protocols
tags: [shadowsocks-deployment]
---

## When to Use
Deploy Shadowsocks 2022 with modern ciphers, plugin obfuscation, and multi-user management.

## Server Setup
```bash
# Install
apt install shadowsocks-rust

# Config /etc/shadowsocks-rust/config.json
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "method": "2022-blake3-aes-128-gcm",
    "password": "BASE64_16BYTE_KEY",
    "fast_open": true,
    "mode": "tcp_and_udp",
    "users": [
        {
            "user": "user1",
            "password": "BASE64_16BYTE_KEY_2"
        }
    ]
}
```

## Plugin Obfuscation
```bash
# With v2ray-plugin
{
    "plugin": "v2ray-plugin",
    "plugin_opts": "tls;host=example.com"
}
```

## Pitfalls
- **2022 ciphers**: Use 2022-blake3 variants, not legacy ciphers
- **Key format**: Password must be base64-encoded 16-byte key
- **Multi-user**: Requires Shadowsocks 2022 format

## Verification
- Test with sslocal client
- Check for traffic obfuscation
- Verify UDP relay works