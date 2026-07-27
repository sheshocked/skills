---
name: network-segmentation
description: 
category: security
tags: [network-segmentation]
---

## When to Use
Use this skill when designing and implementing network segmentation — VLAN architecture, microsegmentation, zero-trust access policies, east-west traffic filtering, and firewall zone design.

## Core Concepts
- **Macrosegmentation**: Divide network into VLANs by function (web, app, db, management)
- **Microsegmentation**: Per-workload firewall policies for granular east-west control
- **Zero trust**: Never trust, always verify — authenticate every connection regardless of network location
- **Defense in depth**: Multiple segmentation layers (VLAN, firewall, host-based)
- **East-west vs north-south**: Internal traffic filtering is often more critical than perimeter
- **DMZ**: Demilitarized zone for public-facing services

## Workflow
1. **Asset inventory**: Map all systems, data flows, and trust boundaries
2. **Zone design**: Define network zones (DMZ, internal, management, database)
3. **Firewall rules**: Create inter-zone rules with default-deny
4. **VLAN configuration**: Assign ports/VLANs based on zone design
5. **Host-based rules**: iptables/nftables for per-host microsegmentation
6. **Monitoring**: Log all cross-zone traffic for anomaly detection
7. **Testing**: Verify segmentation with connectivity tests
8. **Documentation**: Maintain network diagrams and rule documentation

## Key Patterns

### VLAN Architecture
```bash
# /etc/network/interfaces — VLAN configuration
auto eth0.10
iface eth0.10 inet static
    address 10.10.10.1
    netmask 255.255.255.0
    vlan-raw-device eth0

auto eth0.20
iface eth0.20 inet static
    address 10.20.20.1
    netmask 255.255.255.0
    vlan-raw-device eth0

auto eth0.30
iface eth0.30 inet static
    address 10.30.30.1
    netmask 255.255.255.0
    vlan-raw-device eth0

# VLAN assignment on switch (Open vSwitch example)
ovs-vsctl add-port br0 eth0 tag=10  # Management VLAN
ovs-vsctl add-port br0 eth1 tag=20  # Web VLAN
ovs-vsctl add-port br0 eth2 tag=30  # Database VLAN
```

### Inter-VLAN Firewall Rules
```bash
#!/bin/bash
# /etc/iptables/rules.v4 — inter-VLAN firewall
# Zone definitions:
# VLAN 10 (eth0.10): Management — 10.10.10.0/24
# VLAN 20 (eth0.20): Web — 10.20.20.0/24
# VLAN 30 (eth0.30): Database — 10.30.30.0/24

# Flush existing rules
iptables -F
iptables -X

# Default policies
iptables -P FORWARD DROP

# Allow established connections
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Web → Database (only port 5432)
iptables -A FORWARD -i eth0.20 -o eth0.30 -s 10.20.20.0/24 \
  -d 10.30.30.0/24 -p tcp --dport 5432 -m state --state NEW -j ACCEPT

# Management → All (SSH only)
iptables -A FORWARD -i eth0.10 -s 10.10.10.0/24 \
  -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Database → Web: NO (initiate from web side only)

# Block all other inter-VLAN traffic
iptables -A FORWARD -j LOG --log-prefix "inter-vlan-drop: "
iptables -A FORWARD -j DROP

# Save
iptables-save > /etc/iptables/rules.v4
```

### Zero Trust with WireGuard
```bash
# WireGuard per-service tunnel — no implicit trust
# Each service gets its own WireGuard interface

# /etc/wireguard/wg-web.conf
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.100.0.1/24
ListenPort = 51821

[Peer]
PublicKey = WEB_CLIENT_KEY
AllowedIPs = 10.100.0.2/32

# /etc/wireguard/wg-db.conf
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.200.0.1/24
ListenPort = 51822

[Peer]
PublicKey = DB_CLIENT_KEY
AllowedIPs = 10.200.0.2/32

# Firewall rules — only allow traffic through WireGuard interfaces
iptables -A FORWARD -i wg-web -o eth0.30 -d 10.30.30.0/24 -p tcp --dport 5432 -j ACCEPT
iptables -A FORWARD -i wg-db -j DROP  # DB client cannot reach anything else
```

### Microsegmentation with nftables
```bash
#!/usr/sbin/nft -f
# /etc/nftables.conf — per-host microsegmentation

flush ruleset

table inet filter {
    # Define sets of allowed sources per service
    set web_allowed {
        type ipv4_addr
        elements = { 10.10.10.0/24 }  # Only management can reach web
    }

    set db_allowed {
        type ipv4_addr
        elements = { 10.20.20.0/24 }  # Only web tier can reach db
    }

    chain input {
        type filter hook input priority 0; policy drop;

        # Allow established
        ct state established,related accept

        # Allow loopback
        iif "lo" accept

        # SSH from management only
        ip saddr 10.10.10.0/24 tcp dport 22 ct state new accept

        # HTTP/HTTPS from web tier
        ip saddr @web_allowed tcp dport { 80, 443 } ct state new accept

        # ICMP
        icmp type echo-request limit rate 1/second accept

        # Log and drop everything else
        log prefix "nft-drop: " drop
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        ct state established,related accept
        drop
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

### Network Diagram (ASCII)
```
                    ┌─────────────────────────────────────┐
                    │           INTERNET                   │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────┴──────────────────────┐
                    │        DMZ (VLAN 100)                │
                    │   Web Server    Load Balancer         │
                    │   10.100.0.10  10.100.0.1            │
                    └──────────────┬──────────────────────┘
                                   │ :443 only
                    ┌──────────────┴──────────────────────┐
                    │     Application (VLAN 200)           │
                    │   App Server 1  App Server 2         │
                    │   10.200.0.10  10.200.0.11           │
                    └──────────────┬──────────────────────┘
                                   │ :8080 only
                    ┌──────────────┴──────────────────────┐
                    │     Database (VLAN 300)              │
                    │   DB Primary    DB Replica           │
                    │   10.300.0.10  10.300.0.11           │
                    └─────────────────────────────────────┘

Management (VLAN 50): 10.50.0.0/24 — SSH access to all zones
```

## Pitfalls
- **VLAN hopping**: Ensure trunk ports are properly configured — disable DTP on access ports
- **Management access**: Lock down management VLAN — no direct internet access
- **DNS resolution**: Cross-zone DNS may break — configure split DNS or zone-specific resolvers
- **Logging gaps**: Without logging cross-zone traffic, you can't detect breaches
- **Overly restrictive**: Too-tight rules break legitimate traffic — test thoroughly before enforcement
- **Documentation drift**: Network diagrams become outdated — review quarterly

## Verification
- `nmap -sT 10.200.0.0/24` from outside — only expected ports open
- `traceroute` between zones shows firewall hops
- `iptables -L -v` — rules match intended policy
- Cross-zone connectivity tests: web→db works, db→web blocked
- Management can SSH to all zones
- No unauthorized inter-zone traffic in firewall logs
- Network diagram matches actual configuration