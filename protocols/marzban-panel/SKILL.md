---
name: marzban-panel
description: 
category: protocols
tags: [marzban-panel]
---

## When to Use
Set up Marzban multi-protocol panel with node management, user management, and Telegram bot integration.

## Installation
```bash
git clone https://github.com/Gozargah/Marzban
cd Marzban
docker compose up -d
```

## Key Features
- Multi-protocol: VLESS, VMess, Trojan, Shadowsocks
- Node management for multi-server
- User expiration and data limits
- Telegram bot for user management

## Pitfalls
- **Database**: Use MySQL/PostgreSQL for production
- **SSL**: Configure reverse proxy with SSL
- **Backups**: Regular database and config backups

## Verification
- Test user creation and subscription
- Verify multi-protocol inbounds work
- Check node connectivity