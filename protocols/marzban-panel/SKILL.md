---
name: marzban-panel
description: Deploy Marzban multi-node management panel, configure subscription endpoints and node TLS.
category: protocols
tags: [marzban, vless-sub, multi-node, python, docker-compose]
---

# Marzban Panel

## When to Use
Use when managing high-scale proxy setups with multiple backend VPS nodes and client subscription endpoints.

## Prerequisites
- Docker & Docker Compose installed.

## Workflow
1. Clone Marzban repo and generate env keys.
2. Launch core dashboard containers.
3. Establish node link connections using certificate handshakes.

## Key Patterns
```yaml
# docker-compose.yml for Marzban
version: "3"
services:
  marzban:
    image: gozargah/marzban:latest
    restart: always
    env_file: .env
    volumes:
      - /var/lib/marzban:/var/lib/marzban
    ports:
      - "8000:8000"
```

## Pitfalls
- **Database lockouts:** Use external Postgres containers for scaling; local SQLite file locking blocks connections under high load.
- **Node sync failure:** Verify security keys are identical in nodes env profiles.

## Verification
- Verify containers run with `docker compose ps`.
- Query sub path using HTTP client and inspect returned configurations.
