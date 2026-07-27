---
name: ssh-security
description: 
category: security
tags: [ssh-security]
---

## When to Use
Use this skill when implementing SSH key-only authentication, configuring 2FA, setting up jump hosts/bastion servers, implementing port knocking, or hardening SSH for production environments.

## Core Concepts
- **Key-only auth**: Disable password authentication entirely, use Ed25519 or RSA-4096 keys
- **Multi-factor**: Combine public key with TOTP (Google Authenticator PAM)
- **Jump hosts**: ProxyJump for bastion-based access to internal networks
- **Port knocking**: Sequential port access to hide SSH from scanners
- **Certificate authority**: SSH CA for short-lived host/user certificates

## Workflow
1. Generate strong key pair: `ssh-keygen -t ed25519 -a 100 -C "$(whoami)@$(hostname)"`
2. Copy public key to server: `ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host`
3. Harden sshd_config: disable passwords, set protocol 2, limit ciphers
4. Optionally add TOTP 2FA via libpam-google-authenticator
5. Set up jump host config for internal servers
6. Implement port knocking with knockd
7. Distribute keys securely; never use shared keys

## Key Patterns

### Key Generation and Distribution
```bash
# Generate Ed25519 key (preferred over RSA for speed and security)
ssh-keygen -t ed25519 -a 100 -C "ops@prod-server" -f ~/.ssh/prod_key

# Copy to server
ssh-copy-id -i ~/.ssh/prod_key.pub deploy@10.0.1.50

# Verify on server
cat ~/.ssh/authorized_keys
# Should show: ssh-ed25519 AAAA... ops@prod-server

# Set proper permissions on server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/id_ed25519.pub
```

### SSH 2FA with Google Authenticator
```bash
# Install on server
apt install libpam-google-authenticator

# Add to /etc/pam.d/sshd (top of file):
auth required pam_google_authenticator.so

# Configure sshd
# /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
UsePAM yes

# For each user: run google-authenticator and scan QR code
# Save backup codes securely
```

### Jump Host / Bastion Configuration
```bash
# ~/.ssh/config — connect through bastion to internal server
Host bastion
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/bastion_key

Host internal-*
    ProxyJump bastion
    User deploy
    IdentityFile ~/.ssh/internal_key

Host internal-db
    HostName 10.0.1.50

# Connect: ssh internal-db
# Traffic flows: you → bastion → internal-db
```

### Port Knocking with knockd
```bash
# Install
apt install knockd

# Configure /etc/knockd.conf
[options]
    UseSyslog
    logfile = /var/log/knockd.log

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 10
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 10
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

# Start knockd
knockd -d -i eth0

# Client: knock before SSH
knock -v 203.0.113.10 7000 8000 9000
ssh -p 2222 deploy@203.0.113.10
```

### Hardened sshd_config (Complete)
```bash
# /etc/ssh/sshd_config — production hardened
Port 2222
AddressFamily inet
ListenAddress 0.0.0.0
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 30
AuthenticationMethods publickey

# Access control
AllowUsers deploy admin
DenyUsers root

# Security
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no
GatewayPorts no
ClientAliveInterval 300
ClientAliveCountMax 2

# Ciphers and MACs (strong only)
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512

# Logging
LogLevel VERBOSE
SyslogFacility AUTH

# Banner
Banner /etc/issue.net
PrintMotd no

# Misc
PermitUserEnvironment no
```

## Pitfalls
- **Key permissions**: `~/.ssh` must be 700, `authorized_keys` must be 600 — otherwise sshd refuses to use them
- **Ed25519 vs RSA**: Ed25519 is faster and more secure; use RSA-4096 only for compatibility
- **2FA + key**: Both factors must be configured — don't rely on just one
- **Jump host failure**: If bastion is unreachable, you lose access to internal servers — ensure reliable connectivity
- **Port knocking security**: Not truly secure — susceptible to replay attacks and network monitoring; combine with key auth
- **Agent forwarding risk**: Never `ForwardAgent yes` on untrusted hosts — can be hijacked for key theft

## Verification
- `ssh -o PreferredAuthentications=publickey -i ~/.ssh/prod_key deploy@host` — should connect without password prompt
- `ssh -v host 2>&1 | grep -i "auth"` — verify only pubkey is attempted
- `sshd -T | grep -E '(passwordauth|challenge|permitroot)'` — confirm all settings
- Test 2FA: attempt login, should prompt for TOTP after key verification
- `ssh -J bastion internal-db` — verify jump host works
- Check logs: `journalctl -u sshd | grep "Accepted"` — confirm key-only acceptance