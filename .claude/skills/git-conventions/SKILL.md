---
name: git-conventions
description: Git commit message conventions and branch naming standards. Use when creating commits, branches, or preparing code for version control.
---

# Git Conventions

## Commit Message Format

```
<type>: <subject>

<bullet points explaining what changed and why>
```

### Rules

1. **Type**: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`
2. **Subject**: imperative mood, lowercase, no period at end
3. **Body**: bullet points grouped by area (backend, frontend, infra, etc.)
4. **Co-Authored-By**: NEVER include Co-Authored-By lines
5. **Language**: always English
6. **Author/Committer**: always use the git config from the current PC (never hardcode or use other identities). New commits automatically use `git config user.name` and `git config user.email`. When amending, use `--reset-author` to update to current PC config.

## Branch Naming

```
feat/<slug>    # new feature
fix/<slug>     # bug fix
chore/<slug>   # maintenance, refactor, tooling
```

Examples: `feat/user-notifications`, `fix/token-refresh`, `chore/ci-hardening`

## PR Workflow

`main` is protected — direct push is rejected. Always:

```bash
git checkout -b <type>/<slug>
git push -u origin <type>/<slug>
gh pr create --title "<concise title>" --body "..."
```

Never `push --force`. If there are conflicts, resolve with merge.

### Pre-commit Sequence

Before every commit — regardless of whether you are using `/push` or committing directly — run these steps in order:

1. **Lint & format** — run the project's linter and formatter (see `dev-workflow` skill). Fix all errors.
2. **Tests** — run the relevant test suite. Do not commit with failing tests.
3. **Security review** — consult the `security` skill and scan `git diff --staged`. Fix any issue before committing.
4. **Commit** — apply the format and rules below.

If the project has a pre-commit hook and it fails, **fix the issue and create a NEW commit** (never `--amend`).

### Example

```
feat: add user notification preferences

Notifications:
- Add NotificationPreference model with per-type opt-in flags
- Add API endpoint to update preferences
- Add frontend settings page with toggle controls
```
