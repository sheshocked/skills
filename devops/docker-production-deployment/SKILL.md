---
name: docker-production-deployment
description: Establish multi-stage builds, non-root runtimes, health checks, and Docker Compose configurations.
category: devops
tags: [docker, docker-compose, multi-stage, non-root, hardening]
---

# Docker Production Deployment

## When to Use
Use when containerizing applications for production runtimes.

## Prerequisites
- Docker Engine.

## Workflow
1. Design multi-stage Dockerfiles to minimize image sizes.
2. Inject a custom system user; do not run as root.
3. Configure Compose health checks.

## Key Patterns
```dockerfile
# Multi-stage Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## Pitfalls
- **Orphan processes:** Use correct entrypoint array configurations to prevent processes from ignoring shutdown signals.
- **Leaked secrets:** Never write secrets inside build arguments or variables; use runtime file mounts.

## Verification
- Inspect image size: `docker images` shows minimal build.
- Run `docker inspect <container>` and verify `User` field is non-root.
