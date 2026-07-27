---
name: vless-reality-setup
description: 
category: protocols
tags: [vless-reality-setup]
---

## When to Use
Deploy VLESS with REALITY transport for censorship-resistant proxy connections. REALITY mimics legitimate TLS handshakes to defeat DPI without needing a real domain or certificate.

## Server Config (Xray)
```json
{
  "inbounds": [{
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{"id": "UUID_HERE", "flow": "xtls-rprx-vision"}]
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.microsoft.com:443",
        "xver": 0,
        "serverNames": ["www.microsoft.com"],
        "privateKey": "PRIVATE_KEY_HERE",
        "shortIds": [""]
      }
    }
  }],
  "outbounds": [{"protocol": "freedom"}]
}
```

## Client Config (VLESS+Reality)
```json
{
  "outbounds": [{
    "protocol": "vless",
    "settings": {
      "vnext": [{
        "address": "YOUR_IP",
        "port": 443,
        "users": [{
          "id": "UUID_HERE",
          "encryption": "none",
          "flow": "xtls-rprx-vision"
        }]
      }]
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "fingerprint": "chrome",
        "serverName": "www.microsoft.com",
        "publicKey": "PUBLIC_KEY_HERE",
        "shortId": "",
        "spiderX": ""
      }
    }
  }]
}
```

## Workflow
1. Generate key pair: `xray x25519`
2. Pick a dest (SNI) that has valid TLS and high traffic (microsoft.com, apple.com, etc.)
3. Configure server with dest, private key, serverNames
4. Configure client with public key, serverName matching dest
5. Test connection and verify real TLS handshake

## Pitfalls
- **Dest selection**: Must support TLS 1.3 and have valid certificate
- **Fingerprint**: Use "chrome" or "firefox" — not "random"
- **UUID**: Generate proper v4 UUID, don't use placeholder
- **shortId**: Leave empty unless you need multiplexing
- **Flow**: Must use "xtls-rprx-vision" for Vision mode

## Verification
- `curl -v https://YOUR_IP` should show handshake to dest server
- Wireshark should show TLS ClientHello matching real browser
- No visible REALITY signature in packet capture