---
name: docker-production
description: 
category: devops
tags: [docker-production]
---

## When to Use
Run containers in production with proper resource limits, health checks, logging, networking, and multi-stage builds. Covers Docker Compose for multi-service stacks and Docker Swarm for clustering.

## Core Concepts
- **Multi-stage builds**: Separate build and runtime stages to minimize image size
- **Resource limits**: CPU/memory constraints prevent noisy-neighbor problems
- **Health checks**: Container-level liveness/readiness probes
- **Named volumes**: Persistent data across container restarts
- **Overlay networks**: Cross-host container networking in Swarm mode
- **Docker Scout/Trivy**: Image vulnerability scanning before deployment

## Workflow
1. Write multi-stage Dockerfile (builder → runtime)
2. Add HEALTHCHECK instruction
3. Set resource limits in docker-compose.yml or run command
4. Configure logging driver (json-file with max-size/max-file)
5. Scan image for CVEs before pushing
6. Deploy with rolling update strategy

## Key Patterns
```dockerfile
# Multi-stage build — Go example
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/server .

FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/server /server
USER nonroot:nonroot
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:8080/health || exit 1
ENTRYPOINT ["/server"]
```

```yaml
# docker-compose.prod.yml
services:
  app:
    image: registry.example.com/app:1.2.3
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 128M
      restart_policy:
        condition: on-failure
        max_attempts: 5
        delay: 5s
    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "5"
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
    networks:
      - backend
  postgres:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    deploy:
      resources:
        limits:
          memory: 1G

volumes:
  pgdata:

networks:
  backend:
    driver: overlay

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

```bash
# Image scanning with Trivy
trivy image --severity HIGH,CRITICAL registry.example.com/app:1.2.3

# Docker logging and monitoring
docker stats --no-stream
docker system df
docker system prune -af --volumes  # careful in prod
```

## Pitfalls
- **Running as root**: Always use USER directive or nonroot base image
- **Latest tag**: Never use `:latest` in production — pin exact versions
- **Ephemeral storage**: Containers lose data on restart without named volumes
- **Log explosion**: Set max-size/max-file on json-file logging driver
- **Health check timing**: start_period gives container time to initialize
- **Build cache**: Use BuildKit (`DOCKER_BUILDKIT=1`) for faster builds

## Verification
```bash
# Verify resource limits
docker inspect --format '{{.HostConfig.Memory}}' <container>
docker stats --no-stream <container>

# Verify health check is running
docker inspect --format '{{json .State.Health}}' <container> | jq

# Verify image size
docker images registry.example.com/app:1.2.3 --format '{{.Size}}'

# Verify no CVEs
trivy image --exit-code 1 --severity HIGH,CRITICAL <image>
```