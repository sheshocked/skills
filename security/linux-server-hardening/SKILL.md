---
name: linux-server-hardening
description: 
category: security
tags: [linux-server-hardening]
---

## When to Use
Use this skill when hardening a new Linux server or auditing an existing one against baseline security controls — SSH lockdown, firewall defaults, kernel sysctl hardening, audit logging, and automated intrusion prevention.

## Core Concepts
- **CIS Benchmark alignment**: Follow CIS Level 1/2 for consistent hardening
- **Defense in depth**: Multiple layers — kernel params, firewall, file permissions, audit logging
- **Principle of least privilege**: Minimal packages, minimal users, minimal services
- **Audit trail**: auditd captures file access, privilege escalation, and auth events
- **Automated enforcement**: Ansible/sysctl.d for idempotent configuration

## Workflow
1. Run baseline audit: `lynis audit system` to identify gaps
2. Harden SSH: key-only auth, disable root, change port
3. Configure firewall: default-deny, allow only necessary ports
4. Apply kernel sysctl hardening: disable IP forwarding, ignore ICMP redirects
5. Set up auditd with rules for critical files and commands
6. Install and configure fail2ban for brute-force protection
7. Lock down file permissions: `/etc/shadow`, `/etc/gshadow`, world-writable dirs
8. Remove unnecessary packages and disable unused services
9. Schedule recurring `lynis` scans

## Key Patterns

### SSH Hardening (/etc/ssh/sshd_config)
```bash
# /etc/ssh/sshd_config — hardened configuration
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers deploy admin
Protocol 2
X11Forwarding no
AllowTcpForwarding no
Banner /etc/issue.net
LoginGraceTime 60

# Apply changes
sshd -t && systemctl restart sshd
```

### Kernel Sysctl Hardening (/etc/sysctl.d/99-hardening.conf)
```bash
# /etc/sysctl.d/99-hardening.conf
# Network hardening
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_forward = 0

# Kernel hardening
kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
fs.suid_dumpable = 0
fs.protected_hardlinks = 1
fs.protected_symlinks = 1

# Apply immediately
sysctl -p /etc/sysctl.d/99-hardening.conf
```

### Auditd Rules (/etc/audit/rules.d/hardening.rules)
```bash
# /etc/audit/rules.d/hardening.rules
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/ssh/sshd_config -p wa -k sshd_config
-w /var/log/auth.log -p wa -k auth_log
-a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0 -k privilege_escalation
-a always,exit -F arch=b64 -S setuid -S setgid -k privs
-a always,exit -F arch=b64 -S mount -S umount2 -k mounts
```

### Fail2ban Configuration
```bash
# /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
backend = systemd

[sshd]
enabled = true
port = 2222
filter = sshd
maxretry = 3
bantime = 86400

# Custom rule for repeated failed logins
[sshd-aggressive]
enabled = true
port = 2222
filter = sshd[mode=aggressive]
maxretry = 2
bantime = 604800

# Start and enable
systemctl enable --now fail2ban
fail2ban-client status sshd
```

## Pitfalls
- **Don't lock yourself out**: Always test SSH config changes in a secondary session before closing the primary one — `sshd -t` validates config
- **SSH port change**: Update firewall rules BEFORE restarting sshd, or you lose access
- **fail2ban loops**: Ban ranges too broadly can lock out legitimate users — use `ignorip` in jail.local
- **Sysctl vs sysctl.d**: Don't mix runtime sysctl with persistent config — use drop-in files
- **Auditd noise**: Overly broad audit rules create log spam — tune rules before enabling
- **Root access**: Don't disable root login via sudo first — ensure you have a working sudo user

## Verification
- `lynis audit system --quick` — aim for score > 75
- `sshd -T | grep -E '(permitrootlogin|passwordauthentication|protocol)'` — confirm hardened settings
- `sysctl net.ipv4.conf.all.accept_redirects` — should return 0
- `auditctl -l` — confirm audit rules are loaded
- `fail2ban-client status` — verify jails are active
- `cat /etc/passwd | awk -F: '$3==0 {print}'` — should show only root for UID 0
- Test SSH: `ssh -p 2222 user@host` — verify key-only auth works