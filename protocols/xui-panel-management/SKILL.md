---
name: xui-panel-management
description: 
category: protocols
tags: [xui-panel-management]
---

## When to Use
Install, secure, and manage x-ui / 3x-ui panels for multi-protocol proxy management.

## Installation
```bash
bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
```

## Security Hardening
1. Change default port: Panel Settings > Port
2. Set strong username/password
3. Enable SSL for panel access
4. Configure IP whitelist
5. Set up backup cron

## API Usage
```bash
# Get client traffic
curl -X POST http://127.0.0.1:PORT/api/traffic -H "Content-Type: application/json" -d '{"user":"USERNAME"}'
```

## Pitfalls
- **Default credentials**: Always change after install
- **Backup**: Regular SQLite database backups
- **Updates**: Keep panel updated for security patches

## Verification
- Test panel access with new credentials
- Verify inbounds are active
- Check client subscriptions work