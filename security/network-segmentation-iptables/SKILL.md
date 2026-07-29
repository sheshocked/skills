---
name: network-segmentation-iptables
description: Establish UFW/nftables firewalls, limit open ports, block scanner active probing, and restrict subnets.
category: security
tags: [firewall, iptables, ufw, nftables, network-security]
---

# Network Segmentation Iptables

## When to Use
Use when deploying multi-service servers to restrict communication lanes and prevent active port scanning.

## Prerequisites
- Root privileges.

## Workflow
1. Disable standard public access to administrative ports.
2. Establish nftables/UFW rule structures.
3. Permit only specific proxy traffic, drop scanner packets.

## Key Patterns
```bash
# Reset UFW rules
ufw default deny incoming
ufw default allow outgoing

# Allow only standard ports
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 443/udp

# Restrict SSH (assuming port 2222) to specific admin IP
ufw allow from 203.0.113.50 to any port 2222 proto tcp

# Enable firewall
ufw enable
```

## Pitfalls
- **Locking yourself out:** Ensure your current IP is whitelisted for the custom SSH port before enabling.
- **Docker bypass:** Docker bypasses UFW rules by writing directly to iptables chains. Use `ufw-docker` or configure Docker daemon setting `{"iptables": false}`.

## Verification
- Run `ufw status verbose` to check active policies.
- Run Nmap scan from external network to confirm blocked ports.
