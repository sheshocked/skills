---
name: xray-core-config
description: 
category: protocols
tags: [xray-core-config]
---

## When to Use
Use this skill when configuring Xray core: inbounds, outbounds, routing rules, DNS, sniffing, and protocol combinations.

## Full Server Config Template
```json
{
  "log": {"loglevel": "warning"},
  "dns": {
    "servers": ["https://1.1.1.1/dns-query", "localhost"]
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [{"id": "UUID", "flow": "xtls-rprx-vision"}],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.microsoft.com:443",
          "privateKey": "KEY",
          "serverNames": ["www.microsoft.com"],
          "shortIds": [""]
        }
      }
    }
  ],
  "outbounds": [
    {"protocol": "freedom", "tag": "direct"},
    {"protocol": "blackhole", "tag": "block"}
  ],
  "routing": {
    "rules": [
      {"type": "field", "ip": ["geoip:private"], "outboundTag": "block"},
      {"type": "field", "domain": ["geosite:category-ads-all"], "outboundTag": "block"}
    ]
  }
}
```

## Key Concepts
- **Inbounds**: Where clients connect (VLESS, VMess, Shadowsocks, Trojan)
- **Outbounds**: Where traffic goes (freedom, blackhole, other proxies)
- **Routing**: Rules to direct traffic based on domain/IP/protocol
- **Sniffing**: Extract real destination from captured packets
- **DNS**: Built-in DNS resolver with DoH/DoT support

## Pitfalls
- **Sniffing**: Enable "sniffing": {"enabled": true} for proper routing
- **Flow control**: VLESS needs "decryption": "none" and proper flow setting
- **Routing conflicts**: Order matters — first matching rule wins

## Verification
- Test with xray run -c config.json
- Check logs for errors
- Verify routing works as expected