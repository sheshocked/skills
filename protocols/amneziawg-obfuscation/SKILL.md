---
name: amneziawg-obfuscation
description: Complete deployment and parameter tuning for AmneziaWG obfuscation, bypassing WireGuard protocol detection.
category: protocols
tags: [amneziawg, wireguard, obfuscation, packet-headers, dpi-bypass, kernel]
---

# AmneziaWG Protocol Obfuscation Masterclass

## When to Use
Use AmneziaWG when standard WireGuard connections are blocked by the ISP (MCI/Irancell) using Deep Packet Inspection (DPI) to identify standard WireGuard handshake patterns (initiations and responses).

## Prerequisites
- Linux Kernel headers installed on the server.
- AmneziaWG Go or dynamic kernel module compiled.

## Workflow
1. Generate standard WireGuard private and public key pairs.
2. Edit peer and interface files to add AmneziaWG header parameters.
3. Configure identical obfuscation keys on both server and client configurations.

## Key Patterns

### Obfuscation Variables Explained
AmneziaWG adds random packet size additions and replaces standard protocol header magic bytes:
- `Jc` (Junk packet count): Packets sent on startup to trick DPI.
- `Jmin` / `Jmax`: Range of random bytes added to handshake packets.
- `S1` / `S2` (Initiation / Response packet size additions).
- `H1` / `H2` / `H3` / `H4`: Custom magic bytes replacing standard WireGuard protocol header bytes.

### Server Interface Configuration (/etc/amnezia/awg0.conf)
```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY

# Obfuscated parameters - Must match client exactly!
Jc = 4
Jmin = 40
Jmax = 1000
S1 = 15
S2 = 23
H1 = 129038102
H2 = 981273918
H3 = 102938102
H4 = 827361823

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.8.0.2/32
```

### Client Configuration (awg-client.conf)
```ini
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.8.0.2/24
DNS = 1.1.1.1

# Obfuscation blocks
Jc = 4
Jmin = 40
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
PersistentKeepalive = 25
```

## Pitfalls
- **Parameter mismatch failures:** If a single H-byte or J-size parameter differs by 1 value between server and client, the server will silently drop incoming handshakes. Double-check config values before restarting.
- **DPI fingerprinting pattern replication:** Do not use default Amnezia values (`Jc = 4`, `Jmin = 20`, `Jmax = 500`) since some DPI profiles block known defaults. Alter values slightly.

## Verification
- Start interface: `wg-quick up awg0` on server.
- Run `awg show` to inspect established sessions and transfer rates.
