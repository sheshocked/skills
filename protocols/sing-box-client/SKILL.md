---
name: sing-box-client
description: Advanced Sing-box client JSON configuration with TUN interface, strict DNS hijacking, and domain/IP routing rules.
category: protocols
tags: [singbox, client-config, tun-mode, dns-hijacking, custom-routing, json]
---

# Sing-box Client Configuration Masterclass

## When to Use
Use when configuring cross-platform Sing-box clients (Windows, Android, macOS) for direct IP routing, DNS leak protection, and automated split-tunneling.

## Prerequisites
- Sing-box executable or core client application.

## Workflow
1. Configure a local TUN inbound with system integration and packet sniffing.
2. Build two DNS servers: one secure over DOH (for general queries) and one local (for bypass domains).
3. Set up outbounds: Proxy (VLESS/Reality), Direct (bypass), DNS-out (capturing internal DNS queries).
4. Configure route rules to direct domestic (e.g. `.ir` domains) to direct and international to proxy.

## Key Patterns

### Production Client JSON Configuration (config.json)
```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "dns": {
    "servers": [
      {
        "tag": "cloudflare-dns",
        "address": "https://1.1.1.1/dns-query",
        "detour": "proxy"
      },
      {
        "tag": "local-dns",
        "address": "8.8.8.8",
        "detour": "direct"
      }
    ],
    "rules": [
      {
        "outbound": "any",
        "server": "cloudflare-dns"
      },
      {
        "geosite": ["ir"],
        "server": "local-dns"
      }
    ],
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "tag": "tun-in",
      "interface_name": "singtun0",
      "inet4_address": "172.19.0.1/30",
      "auto_route": true,
      "strict_route": true,
      "stack": "system",
      "sniff": true,
      "sniff_override_destination": true
    }
  ],
  "outbounds": [
    {
      "type": "vless",
      "tag": "proxy",
      "server": "185.71.219.72",
      "port": 443,
      "uuid": "e44d32a0-43df-40ab-829d-4e92bf180da1",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "www.gstatic.com",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "PUBLIC_KEY_GENERATED_BY_X25519",
          "short_id": "81b8672d2cbf1c16"
        }
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": "dns",
        "outbound": "dns-out"
      },
      {
        "geosite": ["ir"],
        "geoip": ["ir", "private"],
        "outbound": "direct"
      }
    ],
    "auto_detect_interface": true
  }
}
```

## Pitfalls
- **Infinite DNS detour loops:** If your primary DNS server (`cloudflare-dns`) is not set with `"detour": "proxy"`, it will attempt to query directly, fail on censored networks, or create an infinite loop. Always detour DNS queries to their respective outbounds.
- **Android battery drain in TUN mode:** The `gvisor` TUN stack is resource-heavy. On mobile targets, prefer the `system` stack (`"stack": "system"`) to reduce CPU utilization.

## Verification
- Test syntax: `sing-box check -c config.json` should exit with status 0.
- Start client: `sing-box run -c config.json` and verify the TUN interface `singtun0` binds to the system routing table.
- Verify DNS security: Run `nslookup google.com` and verify the resolver IP is your secure TUN gateway (`172.19.0.1`).
