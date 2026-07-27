---
name: wireguard-server-setup
description: 
category: protocols
tags: [wireguard-server-setup]
---

## When to Use
Set up WireGuard VPN servers: key management, peer configuration, NAT, persistent keepalive, firewall rules.

## Server Setup
```bash
# Install WireGuard
apt install wireguard

# Generate keys
wg genkey | tee server_private.key | wg pubkey > server_public.key
chmod 600 server_private.key

# Server config /etc/wireguard/wg0.conf
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
PersistentKeepalive = 25
```

## Peer Key Generation
```bash
# Generate client key pair
wg genkey | tee client_private.key | wg pubkey > client_public.key

# Generate preshared key (optional but recommended)
wg genpsk > preshared.key
```

## Client Config
```ini
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
PresharedKey = PRESHARED_KEY
Endpoint = SERVER_IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Pitfalls
- **Firewall**: Must open UDP port 51820
- **NAT**: Server must have IP forwarding enabled
- **Key security**: Never share private keys
- **AllowedIPs**: 0.0.0.0/0 routes all traffic through VPN

## Verification
- `wg show` to verify interface
- `ping 10.0.0.1` from client
- Check external IP changes
- Test DNS resolution through tunnel