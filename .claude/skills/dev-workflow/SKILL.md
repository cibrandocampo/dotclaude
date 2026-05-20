---
name: dev-workflow
description: Development commands and environment setup for this project. Customize this skill with the project's actual services and commands. All commands run inside Docker — never on the host.
---

# Dev Workflow

> **Customize this skill for your project.** Replace service names and commands with the actual ones for your stack. The Docker-first rule is non-negotiable — see CLAUDE.md section 9.

## Monorepo Structure

```
project/
  docs/          # documentation, plans, tasks
  frontend/      # frontend application
  backend/       # backend application
  infra/         # Docker Compose, CI/CD, scripts
    docker-compose.yml        # production
    dev/
      docker-compose.yml      # development (bind mounts)
```

## Services

> _Fill in: list the services defined in infra/dev/docker-compose.yml and their ports._

| Service | Description | Port |
|---------|-------------|------|
| `backend` | Backend API | `<port>` |
| `frontend` | Frontend dev server | `<port>` |
| `db` | Database | `<port>` |
| | _add more as needed_ | |

## Start / Stop

```bash
# Start all services
docker compose -f infra/dev/docker-compose.yml up -d

# Stop all services
docker compose -f infra/dev/docker-compose.yml down

# View logs
docker compose -f infra/dev/docker-compose.yml logs -f <service>

# Restart a service
docker compose -f infra/dev/docker-compose.yml restart <service>
```

## Run Tests

```bash
# Backend
docker compose -f infra/dev/docker-compose.yml exec backend <test-command>
# e.g.: pytest, python manage.py test, go test ./...

# Frontend
docker compose -f infra/dev/docker-compose.yml exec frontend <test-command>
# e.g.: npx vitest run, npm test

# E2E
# <e2e-command>
# e.g.: docker run --rm --network host <e2e-image> npx playwright test
```

## Lint & Format

```bash
# Backend
docker compose -f infra/dev/docker-compose.yml exec backend <lint-command>
docker compose -f infra/dev/docker-compose.yml exec backend <format-command>

# Frontend
docker compose -f infra/dev/docker-compose.yml exec frontend <lint-command>
docker compose -f infra/dev/docker-compose.yml exec frontend <format-command>
```

## Build

```bash
# Frontend production build
docker compose -f infra/dev/docker-compose.yml exec frontend <build-command>

# Backend (if applicable)
docker compose -f infra/dev/docker-compose.yml exec backend <build-command>
```

## Coverage

```bash
# Backend
docker compose -f infra/dev/docker-compose.yml exec backend <coverage-command>

# Frontend
docker compose -f infra/dev/docker-compose.yml exec frontend <coverage-command>
```

## Database

```bash
# Run migrations
docker compose -f infra/dev/docker-compose.yml exec backend <migrate-command>

# Open DB shell
docker compose -f infra/dev/docker-compose.yml exec db <db-shell-command>
# e.g.: psql -U <user> <db>, mysql -u <user> -p
```

## Host-Allowed Tools

The following tools may be run directly on the host (no Docker needed):

- `docker` / `docker compose`
- `git`
- `gh` (GitHub CLI)

Everything else goes through Docker. If an exception is needed, document it here with a justification.
