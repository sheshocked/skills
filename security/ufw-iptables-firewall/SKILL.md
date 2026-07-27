---
name: ufw-iptables-firewall
description: 
category: security
tags: [ufw-iptables-firewall]
---

## When to Use
Use this skill when designing and implementing firewall rules with UFW or iptables/nftables — default-deny policies, rate limiting, port whitelisting, Docker interaction, and stateful packet filtering.

## Core Concepts
- **Default deny**: Block all inbound by default, explicitly allow needed ports
- **Stateful filtering**: Track connection state (NEW, ESTABLISHED, RELATED)
- **Rate limiting**: Protect against port scanning and brute-force
- **Docker interaction**: Docker manipulates iptables directly — UFW rules can be bypassed
- **Ingress vs egress**: Control both inbound AND outbound traffic
- **Logging**: Log dropped packets for troubleshooting and threat detection

## Workflow
1. Set default policy: deny inbound, allow outbound
2. Allow SSH and management ports first (avoid lockout)
3. Add service-specific rules with rate limiting where needed
4. Configure Docker to respect host firewall (if applicable)
5. Enable logging and set up log rotation
6. Test all rules before closing management session
7. Create firewall backup/restore script

## Key Patterns

### UFW — Complete Setup
```bash
# Reset and set defaults
ufw reset
ufw default deny incoming
ufw default allow outgoing
ufw default deny routed

# Allow SSH with rate limit
ufw limit 2222/tcp comment "SSH rate-limited"

# Allow web traffic
ufw allow 80/tcp comment "HTTP"
ufw allow 443/tcp comment "HTTPS"

# Allow specific IPs only (management)
ufw allow from 10.0.0.0/24 to any port 22 comment "SSH from LAN"

# Allow specific service from specific subnet
ufw allow from 172.16.0.0/16 to any port 5432 comment "Postgres from Docker"

# Enable logging
ufw logging on
ufw logging medium

# Enable firewall
ufw enable

# Show status
ufw status verbose numbered
```

### iptables — Production Ruleset
```bash
#!/bin/bash
# /etc/iptables/rules.v4 — production firewall

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t mangle -F

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established/related connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Drop invalid packets
iptables -A INPUT -m state --state INVALID -j DROP

# Rate limit SSH (port 2222)
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping (limited)
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT

# Log dropped packets (rate limited)
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-drop: " --log-level 4

# Save rules
iptables-save > /etc/iptables/rules.v4

# Restore on boot
iptables-restore < /etc/iptables/rules.v4
```

### Docker + Host Firewall Integration
```bash
# Problem: Docker bypasses UFW by modifying iptables directly
# Solution: Use DOCKER-USER chain or iptables interface

# Option 1: Use DOCKER-USER chain (recommended)
iptables -I DOCKER-USER -i eth0 -j DROP
iptables -I DOCKER-USER -i eth0 -p tcp --dport 443 -j ACCEPT
iptables -I DOCKER-USER -i eth0 -p tcp --dport 80 -j ACCEPT

# Option 2: Restrict Docker's iptables manipulation
# /etc/docker/daemon.json
{
  "iptables": false,
  "bridge": "none"
}
# Then manually configure Docker networking

# Option 3: Use ufw-docker helper
# Install: https://github.com/chaifeng/ufw-docker
ufw-docker allow 80/tcp
ufw-docker allow 443/tcp
```

### Rate Limiting Patterns
```bash
# Rate limit HTTP (100 req/min per IP)
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 100 -j DROP

# SYN flood protection
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Port scan protection
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL FIN,URG,PSH -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP
iptables -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP

# Log and drop repeated offenders
iptables -A INPUT -m recent --name portscan --rcheck --seconds 86400 -j DROP
iptables -A INPUT -m recent --name portscan --remove
```

## Pitfalls
- **Docker bypasses UFW**: Docker writes directly to iptables — UFW rules don't apply to container traffic unless you use DOCKER-USER chain
- **Locking yourself out**: Always allow SSH BEFORE enabling firewall; test rules in a secondary session
- **IPv6**: UFW handles IPv6 separately — `ufw enable` applies to both; verify with `ufw status numbered`
- **Rule persistence**: UFW persists automatically; raw iptables does not — use iptables-persistent or systemd service
- **Stateful vs stateless**: Without `-m state --state` rules, legitimate return traffic gets blocked
- **Logging volume**: `LOG` target generates massive logs — always rate-limit with `-m limit`

## Verification
- `ufw status verbose` or `iptables -L -n -v` — confirm rules are active
- `nmap -sT -p 1-65535 host` — from external: only allowed ports should be open
- `nc -zv host 22` — should work; `nc -zv host 3306` — should be blocked
- `journalctl -k | grep iptables` — check kernel log for drops
- Test from Docker container: `docker run --rm alpine ping -c 1 host` — verify Docker respects host firewall
- `ufw logging on && tail -f /var/log/ufw.log` — watch for blocked attempts