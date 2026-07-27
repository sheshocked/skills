---
name: mihomo-clash-config
description: 
category: protocols
tags: [mihomo-clash-config]
---

## When to Use
Configure Mihomo (Clash.Meta) client: proxy-groups, rulesets, fake-ip DNS, TUN stack.

## Config Template
```yaml
mixed-port: 7890
allow-lan: true
mode: rule
log-level: info

dns:
  enable: true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query

proxies:
  - name: "VLESS-Proxy"
    type: vless
    server: SERVER
    port: 443
    uuid: UUID
    network: ws
    tls: true
    ws-opts:
      path: /ws
      headers:
        Host: domain.com

proxy-groups:
  - name: "Auto"
    type: url-test
    proxies:
      - VLESS-Proxy
    url: http://www.gstatic.com/generate_204
    interval: 300

rules:
  - GEOIP,CN,DIRECT
  - MATCH,Auto
```

## Pitfalls
- **fake-ip**: May interfere with local services
- **Ruleset updates**: Must update regularly
- **TUN mode**: Requires special permissions

## Verification
- Test proxy switching works
- Verify fake-ip DNS resolves correctly
- Check rules route traffic as expected