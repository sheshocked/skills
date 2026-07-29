---
name: amneziawg-obfuscation
description: Deploy AmneziaWG (obfuscated WireGuard) to defeat Deep Packet Inspection on restrictive networks.
category: protocols
tags: [amneziawg, wireguard, obfuscation, dpi-bypass, networking]
---

# Amneziawg Obfuscation

## When to Use
Use when WireGuard connections are blocked by MCI or Irancell DPI engines detecting standard WireGuard handshakes.

## Prerequisites
- AmneziaWG Go/Kernel module installed on server.

## Workflow
1. Set up standard WireGuard peer configurations.
2. Apply obfuscation parameter keys (`Jc`, `Jmin`, `Jmax`, `S1`, `S2`, `H1`, `H2`, `H3`, `H4`).
3. Deploy configured peer details to client apps.

## Key Patterns
```ini
# Client configuration parameters
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/32
DNS = 1.1.1.1

# AmneziaWG obfuscated variables
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 15
S2 = 23
H1 = 129038102
H2 = 981273918
H3 = 102938102
H4 = 827361823

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = 185.71.219.72:51820
AllowedIPs = 0.0.0.0/0
```

## Pitfalls
- **Parameter mismatch:** All junk packet parameters must match server parameters exactly; otherwise handshakes fail instantly.
- **High latency overhead:** Over-large Jmax values slow transmission due to overhead. Keep `Jmax` below `1200`.

## Verification
- Monitor handshakes: `awg show` on server.
- Run tcpdump to verify packet payloads lack standard WireGuard headers.
