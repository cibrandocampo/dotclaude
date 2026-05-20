---
name: dev-workflow
description: Development commands and environment setup for this project. Customize this skill with the project's actual services and commands. All commands run inside Docker — never on the host.
---

# Dev Workflow

> **Customize this skill for your project.** Replace service names and commands with the actual ones for your stack. The Docker-first rule is non-negotiable — see CLAUDE.md section 9.

## Project Layout

```
project/
  docker-compose.yml        # production (pre-built images, no bind mounts)
  dev/
    docker-compose.yml      # development (local builds, bind mounts)
    Dockerfile.backend
    Dockerfile.frontend
  .env                      # shared by both compose files
  .env.example
  docs/
  frontend/
  backend/
```

**Always use `dev/docker-compose.yml` for development.** Never use the root `docker-compose.yml` locally — it is for production only.

## Services

> _Fill in: list the services defined in dev/docker-compose.yml and their dev ports._

| Service | Description | Dev port | Internal port |
|---------|-------------|----------|---------------|
| `backend` | Backend API | `18000` | `8000` |
| `frontend` | Frontend dev server | `15173` | `5173` |
| `db` | Database | `15432` | `5432` |
| `redis` | Cache / queue | `16379` | `6379` |
| | _add more as needed_ | | |

## Start / Stop

```bash
# Start all services
docker compose -f dev/docker-compose.yml up -d

# Stop all services
docker compose -f dev/docker-compose.yml down

# View logs
docker compose -f dev/docker-compose.yml logs -f <service>

# Restart a service (e.g. after changing a singleton or env var)
docker compose -f dev/docker-compose.yml restart <service>
```

## Run Tests

```bash
# Backend
docker compose -f dev/docker-compose.yml exec backend <test-command>
# e.g.: pytest, python manage.py test, go test ./...

# Frontend
docker compose -f dev/docker-compose.yml exec frontend <test-command>
# e.g.: npx vitest run, npm test

# E2E
# <e2e-command>
# e.g.: docker run --rm --network host <e2e-image> npx playwright test
```

## Lint & Format

```bash
# Backend
docker compose -f dev/docker-compose.yml exec backend <lint-command>
docker compose -f dev/docker-compose.yml exec backend <format-command>

# Frontend
docker compose -f dev/docker-compose.yml exec frontend <lint-command>
docker compose -f dev/docker-compose.yml exec frontend <format-command>
```

## Build

```bash
# Frontend production build (verify before pushing)
docker compose -f dev/docker-compose.yml exec frontend <build-command>

# Rebuild a service image after Dockerfile changes
docker compose -f dev/docker-compose.yml build <service>
```

## Coverage

```bash
# Backend
docker compose -f dev/docker-compose.yml exec backend <coverage-command>

# Frontend
docker compose -f dev/docker-compose.yml exec frontend <coverage-command>
```

## Database

```bash
# Run migrations
docker compose -f dev/docker-compose.yml exec backend <migrate-command>

# Open DB shell
docker compose -f dev/docker-compose.yml exec db <db-shell-command>
# e.g.: psql -U <user> <db>, mysql -u <user> -p
```

## Environment Variables

Both compose files share `.env` at the project root.
- `docker-compose.yml` (prod): `env_file: .env`
- `dev/docker-compose.yml` (dev): `env_file: ../.env`

Never commit `.env`. Commit `.env.example` with all keys but no secrets.

## Host-Allowed Tools

The following tools may be run directly on the host (no Docker needed):

- `docker` / `docker compose`
- `git`
- `gh` (GitHub CLI)

Everything else goes through Docker. If an exception is needed, document it here with a justification.
