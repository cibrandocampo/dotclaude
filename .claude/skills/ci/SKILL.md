---
name: ci
description: GitHub Actions CI conventions — pipeline structure, triggers, Docker image publishing, Codecov integration, dependency management, and weekly rebuilds. Use when creating or modifying CI workflows.
---

# CI — GitHub Actions

## Pipeline Philosophy

Tests gate everything. Images are never built from untested code. Code coverage is tracked on every PR. Images rebuild weekly to pick up base image security patches.

---

## Workflow Structure

Three workflows. Each has a single responsibility.

### 1. `ci.yml` — Test and build on every push/PR

**Triggers**:
- `push` to `main`
- `pull_request` targeting `main`
- `release` published (to also build and tag stable images)

**Jobs and their order**:

```
test-backend ──┐
               ├──→ build-backend
test-frontend ─┤
               └──→ build-frontend
```

- Test jobs run in parallel, unconditionally (every push and PR).
- Build jobs run only after both test jobs pass, and only on `push` or `release` (not on PRs).

```yaml
build-backend:
  needs: [test-backend, test-frontend]
  if: github.event_name == 'push' || github.event_name == 'release'
```

### 2. `weekly-rebuild.yml` — Rebuild images without cache

**Trigger**: cron, every Monday at 06:00 UTC.

Rebuilds Docker images with `--no-cache: true` to pick up base image patches and transitive dependency bugfixes. Does **not** run tests — the code hasn't changed, only dependencies. The CI pipeline already gates every code change.

### 3. `site-deploy.yml` _(if applicable)_

Deploys a landing/docs site to GitHub Pages on every push to `main`. Separate from the main CI to keep concerns isolated.

---

## Test Jobs

Each test job uses native runners (not Docker) with service containers for infrastructure dependencies.

### Backend (Python)

```yaml
test-backend:
  runs-on: ubuntu-latest
  services:
    postgres:
      image: postgres:17-alpine
      env:
        POSTGRES_DB: <db>
        POSTGRES_USER: <user>
        POSTGRES_PASSWORD: <password>
      ports:
        - 5432:5432
      options: >-
        --health-cmd pg_isready
        --health-interval 10s
        --health-timeout 5s
        --health-retries 5
    redis:
      image: redis:8-alpine
      ports:
        - 6379:6379
      options: >-
        --health-cmd "redis-cli ping"
        --health-interval 10s
        --health-timeout 5s
        --health-retries 5
  defaults:
    run:
      working-directory: backend
  steps:
    - uses: actions/checkout@v6
    - uses: actions/setup-python@v6
      with:
        python-version: '3.13'
        cache: pip
        cache-dependency-path: backend/requirements.txt
    - run: pip install -r requirements.txt -r ../dev/requirements.txt
    - run: ruff check .
    - run: ruff format --check .
    - run: coverage run manage.py test
      env:
        DATABASE_URL: postgres://<user>:<password>@localhost:5432/<db>
        REDIS_URL: redis://localhost:6379/0
        DJANGO_SECRET_KEY: ci-secret-key-not-used-in-production
        DJANGO_DEBUG: 'True'
    - run: coverage xml
    # Monorepo path fix: coverage.xml paths are relative to backend/ but
    # Codecov needs them relative to the repo root (backend/apps/...).
    - run: sed -i 's|filename="|filename="backend/|g' coverage.xml
    - uses: codecov/codecov-action@v5
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: backend/coverage.xml
        flags: backend
        fail_ci_if_error: false
        verbose: true
```

### Frontend (Node)

```yaml
test-frontend:
  runs-on: ubuntu-latest
  defaults:
    run:
      working-directory: frontend
  steps:
    - uses: actions/checkout@v6
    - uses: actions/setup-node@v5
      with:
        node-version: '22'
        cache: npm
        cache-dependency-path: frontend/package-lock.json
    - run: corepack enable npm && corepack prepare npm@11 --activate
    - run: npm ci
    - run: npm run lint -- --max-warnings 0
    - run: npm run test:coverage
    - uses: codecov/codecov-action@v5
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: frontend/coverage/coverage-final.json
        flags: frontend
        fail_ci_if_error: false
```

---

## Docker Build Jobs

Build jobs publish multi-platform images to Docker Hub.

```yaml
build-backend:
  needs: [test-backend, test-frontend]
  if: github.event_name == 'push' || github.event_name == 'release'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v6
    - uses: docker/setup-qemu-action@v4
    - uses: docker/setup-buildx-action@v4
    - uses: docker/login-action@v4
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - name: Docker meta
      id: meta
      uses: docker/metadata-action@v6
      with:
        images: <dockerhub-org>/<project>-backend
        tags: |
          type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}
          type=raw,value=stable,enable=${{ github.event_name == 'release' }}
          type=semver,pattern={{version}},enable=${{ github.event_name == 'release' }}
    - uses: docker/build-push-action@v7
      with:
        context: ./backend
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    - name: Update Docker Hub description
      uses: peter-evans/dockerhub-description@v5
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

### Required secret

Add `CODECOV_TOKEN` to the repository secrets (Settings → Secrets → Actions). Get it from [codecov.io](https://codecov.io) after linking the repository.

### Flags

Use `flags:` to separate backend and frontend coverage in the Codecov dashboard:
- `flags: backend`
- `flags: frontend`

### Monorepo path fix

`coverage xml` generates paths relative to the working directory (e.g. `apps/models.py`). Codecov needs them relative to the repo root (`backend/apps/models.py`). Always apply the sed fix before uploading:

```bash
sed -i 's|filename="|filename="backend/|g' coverage.xml
```

### `fail_ci_if_error: false`

Codecov upload failures do not block CI. Coverage reporting is observability — a Codecov outage should not prevent merging.

---

## Python Dependency Conventions

### Production (`backend/requirements.txt`)

Use **compatible release** (`~=`) for all production dependencies:

```
django~=5.2
djangorestframework~=3.16
celery[redis]~=5.5
```

`~=5.2` means `>=5.2, <6.0` — gets patch and minor updates automatically, protects against breaking major version changes. This is the right balance between staying current on security patches and stability.

Never use unpinned (`django`) or overly strict (`django==5.2.1`) in production requirements — unpinned breaks reproducibility, overly strict blocks security patches.

### Dev-only (`dev/requirements.txt`)

Use **minimum version** (`>=`) for dev tools:

```
coverage>=7.4
ruff>=0.8
```

Dev tools can update freely. Strict pinning on linters creates unnecessary friction with no safety benefit.

### Updating dependencies

- Review changelogs before bumping a major version.
- `~=` handles minor/patch updates automatically via `pip install -r requirements.txt`.
- For a deliberate major bump: update the version specifier, test locally, then commit.

---

## Required Secrets

| Secret | Where to get it |
|--------|----------------|
| `CODECOV_TOKEN` | codecov.io → repository settings |
| `DOCKERHUB_USERNAME` | Docker Hub account username |
| `DOCKERHUB_TOKEN` | Docker Hub → Account Settings → Personal Access Tokens |

---

## Caching

- **Python**: `actions/setup-python` with `cache: pip` and `cache-dependency-path: backend/requirements.txt`
- **Node**: `actions/setup-node` with `cache: npm` and `cache-dependency-path: frontend/package-lock.json`
- **Docker layers**: `cache-from: type=gha` / `cache-to: type=gha,mode=max` on all build jobs and weekly rebuild
