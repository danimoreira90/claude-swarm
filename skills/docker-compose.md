---
name: docker-compose
description: >
  Multi-service Docker Compose configurations. Healthchecks, networking, volumes,
  development vs production configs. Use for local dev setup and staging environments.
---

# Docker Compose Skill

> One command to start your entire stack.

## File Structure

```
docker-compose.yml          ← base config (shared between envs)
docker-compose.override.yml ← dev overrides (auto-loaded in dev)
docker-compose.prod.yml     ← production overrides
.env                        ← local secrets (gitignored)
.env.example               ← template (committed)
```

## Service Template

See `agents/tier2/cloud-engineer.md` for full Docker Compose example.

## Key Patterns

### Service Dependencies
```yaml
services:
  api:
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
```

### Healthchecks (Required for All Services)
```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "app"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s
```

### Volume Mounts (Dev vs Prod)
```yaml
# docker-compose.override.yml (dev)
services:
  api:
    volumes:
      - ./src:/app/src  # hot reload
    environment:
      - NODE_ENV=development

# docker-compose.prod.yml
services:
  api:
    # No volume mounts — use the built image
    environment:
      - NODE_ENV=production
```

### Networking
```yaml
networks:
  backend:     # internal — services communicate
    internal: true
  frontend:    # exposed — proxy reaches here
    internal: false

services:
  postgres:
    networks:
      - backend    # only backend can reach DB

  api:
    networks:
      - backend
      - frontend

  nginx:
    networks:
      - frontend
    ports:
      - "80:80"
      - "443:443"
```

## Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api

# Run migrations
docker-compose exec api npx prisma migrate deploy

# Rebuild after Dockerfile changes
docker-compose up -d --build api

# Stop and remove containers (keep volumes)
docker-compose down

# Stop and remove everything including volumes (dangerous!)
docker-compose down -v
```
