---
name: xui-panel-tuning
description: Optimize x-ui/3x-ui panels: secure listening ports, automate cert renewals, and clean SQLite storage.
category: protocols
tags: [x-ui, 3x-ui, sqlite, port-hardening, database]
---

# Xui Panel Tuning

## When to Use
Use when managing x-ui / 3x-ui panels to secure admin access and clean up database overhead.

## Prerequisites
- x-ui / 3x-ui panel installed.
- SQLite3 CLI utility.

## Workflow
1. Bind panel to localhost and proxy through Nginx reverse proxy.
2. Clear orphaned database clients using SQLite scripts.
3. Optimize SQLite configuration parameters.

## Key Patterns
```bash
# Hardening x-ui SQLite storage size
sqlite3 /etc/x-ui-english/x-ui.db "VACUUM;"
sqlite3 /etc/x-ui-english/x-ui.db "PRAGMA journal_mode = WAL;"

# Check current listening ports safely
sqlite3 /etc/x-ui-english/x-ui.db "SELECT port, protocol, settings FROM inbounds;"
```

## Pitfalls
- **x-ui overwrite loop:** Never modify `config.json` manually; x-ui overwrites it on start from database values. Perform all additions via SQLite table scripts.
- **Port Conflict on start:** Keep panel port away from standard VPN port ranges.

## Verification
- Test db access: `sqlite3 /etc/x-ui-english/x-ui.db "SELECT * FROM users;"`.
- Monitor active panel resources with `systemctl status x-ui`.
