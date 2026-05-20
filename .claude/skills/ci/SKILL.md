---
name: ci
description: GitHub Actions CI conventions — pipeline structure, Docker-first testing, Docker image publishing, Codecov integration, and weekly rebuilds. Use when creating or modifying CI workflows.
---

# CI — GitHub Actions

## Philosophy

The same Docker image that runs locally also runs in CI. No native language runtimes on the runner — no `setup-python`, no `npm install`, no `pip`. Every test and lint command goes through `docker compose -f dev/docker-compose.yml`, exactly as it does on the developer's machine. This eliminates the "works on my machine" class of CI failures.

Coverage files written by the test command appear on the host automatically because the dev compose bind-mounts the source directories (`backend/` and `frontend/`). No extraction step needed.

---

## Workflow Structure

Three workflows. Each has a single responsibility.

### 1. `ci.yml` — test and build on every push and PR

**Triggers**: push to `main`, PR targeting `main`, release published.

**Job graph**:

```
test-backend ──┐
               ├──→ build-backend
test-frontend ─┤
               └──→ build-frontend
```

- Test jobs run in parallel on every push and PR.
- Build/push jobs run only after both test jobs pass, and only on `push` or `release` (not on PRs).

```yaml
build-backend:
  needs: [test-backend, test-frontend]
  if: github.event_name == 'push' || github.event_name == 'release'
```

### 2. `weekly-rebuild.yml` — rebuild images without cache

**Trigger**: cron, every Monday at 06:00 UTC.

Rebuilds production Docker images with `no-cache: true` to pick up base image patches and transitive dependency fixes. Does not run tests — the code has not changed.

### 3. `site-deploy.yml` _(if applicable)_

Deploys a docs or landing site to GitHub Pages on push to `main`. Keep it separate from CI to isolate concerns.

---

## Test Jobs

### Pattern

Each test job follows the same sequence:

```
1. checkout
2. create .env from .env.example (CI overrides via env:)
3. docker compose build <service>
4. docker compose run --rm --no-deps <service> <lint-command>
5. docker compose run --rm <service> <test-with-coverage-command>
6. upload coverage to Codecov
```

**`--no-deps` for lint**: lint does not need infrastructure (db, redis). Pass `--no-deps` to skip starting dependency services and speed up the step.

**No `--no-deps` for tests**: `docker compose run` starts services declared in `depends_on`. If those services have a `healthcheck`, use `condition: service_healthy` in `depends_on` so the runner waits for them to be ready before executing the test command.

### Environment variables in CI

The dev compose references `env_file: ../.env`. In CI, create the file from `.env.example` and override with job-level `env:`:

```yaml
- name: Create env file
  run: cp .env.example .env

- name: Test
  run: docker compose -f dev/docker-compose.yml run --rm backend <test-command>
  env:
    DATABASE_URL: postgres://<user>:<password>@db:5432/<db>
    REDIS_URL: redis://redis:6379/0
```

Secrets go in GitHub Actions secrets (Settings → Secrets → Actions), not in `.env.example`.

### Coverage extraction

Because `dev/docker-compose.yml` bind-mounts the source directory into the container, any file the test command writes to the working directory appears on the host automatically:

- A test command writing `coverage.xml` to `/app/` inside the container → file appears at `backend/coverage.xml` on the host.
- Upload that file directly with `codecov/codecov-action`.

If paths in the coverage file are relative to the service directory but Codecov needs repo-root-relative paths, apply the sed fix before uploading:

```bash
sed -i 's|filename="|filename="backend/|g' backend/coverage.xml
```

---

## Build / Push Jobs

Build jobs produce production images and push them to Docker Hub. They use `docker/build-push-action` (not dev compose) with multi-platform builds and GHA layer caching.

```yaml
build-backend:
  needs: [test-backend, test-frontend]
  if: github.event_name == 'push' || github.event_name == 'release'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: docker/setup-qemu-action@v3
    - uses: docker/setup-buildx-action@v3
    - uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - name: Docker meta
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: <dockerhub-org>/<project>-backend
        tags: |
          type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}
          type=raw,value=stable,enable=${{ github.event_name == 'release' }}
          type=semver,pattern={{version}},enable=${{ github.event_name == 'release' }}
    - uses: docker/build-push-action@v6
      with:
        context: ./backend
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    - name: Update Docker Hub description
      uses: peter-evans/dockerhub-description@v4
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
        repository: <dockerhub-org>/<project>-backend
        readme-filepath: ./README.md
```

### Image tagging strategy

| Event | Tags applied |
|-------|-------------|
| Push to `main` | `latest` |
| Release published | `stable`, `x.y.z` (semver) |
| Weekly rebuild | `latest` (no new tag) |

---

## Codecov Integration

### Flags

Use `flags:` to separate backend and frontend coverage in the Codecov dashboard:

```yaml
- uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: backend/coverage.xml      # path on the host after bind-mount extraction
    flags: backend
    fail_ci_if_error: false          # Codecov outage must not block merges
    verbose: true
```

### `fail_ci_if_error: false`

Coverage reporting is observability, not a gate. A Codecov outage should never block a merge.

---

## Required Secrets

| Secret | Where to get it |
|--------|----------------|
| `CODECOV_TOKEN` | codecov.io → repository settings |
| `DOCKERHUB_USERNAME` | Docker Hub account username |
| `DOCKERHUB_TOKEN` | Docker Hub → Account Settings → Personal Access Tokens |
