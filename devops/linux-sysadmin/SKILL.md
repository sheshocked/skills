---
name: linux-sysadmin
description: 
category: devops
tags: [linux-sysadmin]
---

## When to Use
Linux system administration: user management, service control, disk management, networking, security hardening, log analysis, and performance troubleshooting on production servers.

## Core Concepts
- **systemd**: Service management (unit files, timers, journald)
- **iptables/nftables**: Firewall rules and NAT
- **LVM**: Logical Volume Management for flexible disk allocation
- **SELinux/AppArmor**: Mandatory access control
- **sysctl**: Kernel parameter tuning
- **cron/systemd timers**: Scheduled task execution

## Workflow
1. Harden SSH (key-only auth, non-standard port, fail2ban)
2. Configure firewall (deny all, allow specific ports)
3. Set up log rotation and remote syslog
4. Configure monitoring (node_exporter, netdata)
5. Automate maintenance with cron/timers

## Key Patterns
```bash
# SSH hardening — /etc/ssh/sshd_config
PermitRootLogin prohibit-password
PasswordAuthentication no
MaxAuthTries 3
AllowUsers deploy admin
Protocol 2
ClientAliveInterval 300
ClientAliveCountMax 2

# Apply changes
systemctl restart sshd
```

```bash
# Firewall — iptables
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4

# nftables equivalent
nft add table inet filter
nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
nft add rule inet filter input ct state established,related accept
nft add rule inet filter input iif lo accept
nft add rule inet filter input tcp dport {22,80,443} accept
```

```bash
# LVM — Expand logical volume
lvextend -L +10G /dev/vg0/lv_data
resize2fs /dev/vg0/lv_data        # ext4
xfs_growfs /dev/vg0/lv_data       # xfs

# Disk monitoring
df -h
du -sh /var/log/*
ncdu /var/log
```

```bash
# Performance troubleshooting
top -bn1 | head -20                    # CPU overview
vmstat 1 5                             # memory and I/O
iostat -xz 1 5                         # disk I/O
ss -tlnp                               # listening ports
ss -s                                   # socket stats
journalctl -u nginx --since "1 hour ago"  # service logs
dmesg -T | tail -50                    # kernel messages
sar -u 1 10                            # CPU over time
```

```bash
# Systemd service management
systemctl status nginx
systemctl enable --now nginx
systemctl edit nginx                     # drop-in override
systemctl list-timers --all             # scheduled tasks

# Systemd timer (cron alternative)
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

## Pitfalls
- **SELinux**: Don't disable it — use `setsebool` and `semanage` for proper configuration
- **Log rotation**: Configure `/etc/logrotate.d/` for custom applications
- **Swap**: Use `swappiness=10` for servers (not desktops)
- **NTP**: Always run chrony/ntpd — time drift breaks TLS, databases, logs
- **File descriptors**: Increase `nofile` limits for high-traffic services
- **Core dumps**: Configure `core_pattern` and `ulimit -c` for debugging

## Verification
```bash
# Verify SSH hardening
sshd -T | grep -E "permitroot|passwordauth|maxauth"

# Verify firewall
iptables -L -n -v
nft list ruleset

# Verify disk space and health
smartctl -a /dev/sda
lsblk -f
```