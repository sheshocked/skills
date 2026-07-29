---
name: ssh-security-hardening
description: Disable password authentication, enforce key pairs, secure ports, and configure fail2ban.
category: devops
tags: [ssh, fail2ban, port-hardening, keys, security]
---

# Ssh Security Hardening

## When to Use
Use immediately upon server provisioning to stop brute force attempts.

## Prerequisites
- Configured SSH keypair.

## Workflow
1. Add authorized public key to `~/.ssh/authorized_keys`.
2. Edit `/etc/ssh/sshd_config` to secure settings.
3. Install and start fail2ban.

## Key Patterns
```bash
# /etc/ssh/sshd_config variables
Port 2222
PermitRootLogin prohibit-password
PasswordAuthentication no
X11Forwarding no
MaxAuthTries 3

# Restart SSH daemon
systemctl restart sshd
```

## Pitfalls
- **Password lockout:** Never disable password auth until verifying key access in another terminal window.
- **Fail2ban jail conflicts:** Ensure fail2ban SSH port matches your changed SSH port.

## Verification
- Test password login; verify it yields `Permission denied (publickey)`.
- Inspect fail2ban logs: `fail2ban-client status sshd`.
