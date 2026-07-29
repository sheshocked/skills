---
name: linux-server-hardening
description: Hardening sysctl.conf, limits.conf, kernel security, and resource limits on production servers.
category: devops
tags: [sysctl, limits, security, sysadmin, linux]
---

# Linux Server Hardening

## When to Use
Use when deploying a new server to block network attacks, optimize TCP throughput, and protect system memory under load.

## Prerequisites
- Root access to server.

## Workflow
1. Update `/etc/sysctl.conf` with networking parameters.
2. Edit `/etc/security/limits.conf` to increase file descriptor constraints.
3. Reload kernel parameters.

## Key Patterns
```bash
# Append to /etc/sysctl.conf
# Enable BBR congestion control
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr

# TCP buffer optimization for high load
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216

# Protect against SYN flood attacks
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_max_syn_backlog=8192

# Apply variables
sysctl -p
```

## Pitfalls
- **Enabling BBR on old kernels:** BBR requires Linux kernel 4.9+. Check kernel version: `uname -r` before applying.

## Verification
- Run `sysctl net.ipv4.tcp_congestion_control` to verify output is `bbr`.
- Test descriptor limits: `ulimit -n` returns new values.
