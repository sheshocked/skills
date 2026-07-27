---
name: amneziawg-obfuscation
description: 
category: protocols
tags: [amneziawg-obfuscation]
---

## When to Use
Deploy AmneziaWG to bypass DPI on WireGuard connections using junk packets and magic headers that defeat traffic analysis.

## What AmneziaWG Adds
- **Jc** (Junk packet Count): Number of junk packets sent after handshake
- **Jmin/Jmax**: Min/max size of junk packets (bytes)
- **S1/S2**: Handshake init/padding size
- **H1-H4**: Magic header values for packet identification

## Config Comparison
```ini
# Standard WireGuard
[Interface]
PrivateKey = ...
Address = 10.0.0.1/24
ListenPort = 51820

# AmneziaWG (added fields)
[Interface]
PrivateKey = ...
Address = 10.0.0.1/24
ListenPort = 51820
Jc = 4
Jmin = 40
Jmax = 70
S1 = 0
S2 = 0
H1 = 0
H2 = 0
H3 = 0
H4 = 0

[Peer]
PublicKey = ...
AllowedIPs = 0.0.0.0/0
Endpoint = ...
Jc = 4
Jmin = 40
Jmax = 70
S1 = 0
S2 = 0
H1 = 0
H2 = 0
H3 = 0
H4 = 0
```

## DPI Evasion Strategy
1. Set Jc=3-5, Jmin=40, Jmax=70 for realistic traffic patterns
2. Use H1-H4 values that match common WireGuard packet headers
3. Adjust S1/S2 for handshake padding to look like regular TLS
4. Test against DPI systems (GFW, Iran DPI, etc.)

## Pitfalls
- **Both sides must match**: Server and client must have identical J/H/S values
- **Not bulletproof**: Advanced DPI may still detect patterns
- **Performance overhead**: Junk packets add bandwidth usage

## Verification
- Compare packet sizes with/without AmneziaWG
- Test against known DPI systems
- Verify connection stability under load