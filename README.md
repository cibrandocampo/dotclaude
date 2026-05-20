# dotclaude

> **For Claude Code only.** This template is designed specifically for [Claude Code](https://claude.ai/code) (the CLI and IDE extension). It will not work as-is with other AI assistants or generic LLM setups.

---

## The problem with raw AI

AI coding assistants are extraordinarily powerful — and completely undirected out of the box. Ask a raw model to build a feature and it will produce *something*: it might be overengineered, inconsistently structured, missing tests, or simply not what you meant. The model is not wrong; it just has no idea how *you* work.

In bowling, the **coverstock** is the outer shell of the ball. It is what actually contacts the lane. Change the coverstock and you change everything: the hook potential, the friction, the trajectory. The same throw produces a completely different result. A skilled bowler does not just throw harder — they choose the right coverstock for the lane conditions and trust the physics to do the rest.

This template is the coverstock for Claude Code.

From experience building production software with AI, the conclusion is the same every time: the model's raw capability is not the bottleneck. **The bottleneck is control.** Without structure, an AI assistant is a powerful engine with no steering wheel — fast, but not necessarily going where you need. With the right guidelines baked in from day one, Claude becomes a disciplined engineering partner that plans before coding, asks before assuming, tests what it ships, and never commits without a green pipeline.

This template encodes those guidelines. Drop it into a project and Claude immediately knows your workflow, your conventions, and your standards — without you having to explain them in every conversation.

This project was developed with Claude Code.

---

## What is this

A set of commands, skills, and behavioral guidelines that shape how Claude Code operates in a project. Instead of explaining your conventions in every conversation, you define them once here and Claude follows them consistently across all sessions and developers.

The template covers:

- **Behavioral guidelines** — how Claude thinks before coding, when to ask, when to stop.
- **Development workflow** — a structured pipeline from idea to shipped PR.
- **Project conventions** — architecture, testing, security, API design, git, Docker.
- **Memory system** — what Claude should remember between sessions.

---

## Repository structure

```
.claude/
  commands/          # Slash commands — the development workflow
    dev-1-plan.md    # Plan a new feature
    dev-2-tasks.md   # Break the plan into executable tasks
    dev-3-run.md     # Execute a task
    dev-4-qa.md      # QA review of a completed task
    fix.md           # Quick bug fix (no pipeline)
    audit.md         # Consistency audit of a code area
    push.md          # Commit, PR, and CI verification
    pr-create.md     # Create a structured GitHub PR
  skills/            # Reference documents Claude consults during work
    architecture/    # Layer structure and design validation
    api-design/      # REST conventions
    backend-patterns/  # Backend stack conventions (customize per project)
    frontend-patterns/ # Frontend stack conventions (customize per project)
    dev-workflow/    # Docker commands and environment setup (customize per project)
    git-conventions/ # Commit format, branch naming, pre-commit sequence
    security/        # Security checklist for every commit
    test-discipline/ # How to write and debug tests
    ci/              # GitHub Actions conventions — pipeline, Codecov, Docker publishing
    memory-conventions/ # What goes in MEMORY.md vs Claude's auto-memory
CLAUDE.md            # Behavioral guidelines — the foundation of everything
```

---

## How to use this as a template

### 1. Copy into your project

```bash
cp -r dotclaude/.claude your-project/
cp dotclaude/CLAUDE.md your-project/
```

Or use this repository as a GitHub template to start a new repo with everything in place.

### 2. Customize the two placeholder skills

These skills are intentionally left as templates — fill them in with your project's actual stack:

**`.claude/skills/dev-workflow/SKILL.md`**
Replace the placeholder commands with the real ones for your services (test runner, linter, migration command, etc.).

**`.claude/skills/backend-patterns/SKILL.md`** and **`.claude/skills/frontend-patterns/SKILL.md`**
Fill in your framework, project structure, naming conventions, and testing approach.

Everything else works out of the box.

### 3. Create a MEMORY.md in the project root

```markdown
# MEMORY — Your Project Name

## Architecture decisions

## Gotchas

## Recurring patterns
```

Claude will populate it as the project evolves. See `memory-conventions` skill for what belongs here.

---

## Project setup checklist

Everything that needs to be filled in before starting development. Claude will warn you if it hits an unfilled placeholder, but going through this list upfront saves time.

**Skills — fill these in first, before any development work**

- [ ] `.claude/skills/dev-workflow/SKILL.md` — service names, test, lint, build, and migration commands
- [ ] `.claude/skills/backend-patterns/SKILL.md` — language, framework, directory structure, naming conventions
- [ ] `.claude/skills/frontend-patterns/SKILL.md` — framework, routing, state, styling, design tokens

**Docker**

- [ ] `docker-compose.yml` — replace `<dockerhub-org>/<project>` with real image names
- [ ] `dev/docker-compose.yml` — replace placeholder services with real ones (Dockerfiles, volumes, commands)

**Environment**

- [ ] `.env.example` — add all variables the project needs (no secrets, just keys)
- [ ] `.env` — copy from `.env.example` and fill in real values (never commit this file)

**GitHub Actions**

- [ ] `.github/workflows/ci.yml` — replace all `<placeholder>` values (commands, image names, coverage files)
- [ ] `.github/workflows/weekly-rebuild.yml` — replace `<dockerhub-org>/<project>` with real image names

**Project memory**

- [ ] `MEMORY.md` — replace `<Project Name>` with the actual project name

**Optional**

- [ ] `CLAUDE.md` — merge any project-specific behavioral rules that override or extend the defaults

---

## Development workflow

There are three paths. Choose the one that fits the work.

### Full feature pipeline

For new features with multiple layers or non-trivial design:

```
/dev-1-plan   →   /dev-2-tasks   →   /dev-3-run   →   /dev-4-qa   →   /push
```

| Step | What happens |
|------|-------------|
| `/dev-1-plan <feature>` | Claude explores the codebase, asks clarifying questions, and produces a design document in `docs/plans/`. No code is written. |
| `/dev-2-tasks <plan>` | Claude splits the plan into self-contained tasks with explicit DoD. Creates files in `docs/tasks/` and an `INDEX.md`. |
| `/dev-3-run <task-id>` | Claude implements one task, saves evidence of every DoD check, and documents decisions. Creates the feature branch on first run. |
| `/dev-4-qa <task-id>` | Independent re-execution of all verifications. Produces APPROVED or RETURNED with specific blockers. |
| `/push` | Documentation check, pre-commit validation, commit, PR creation, CI monitoring. |

One commit per feature — at the end of the full cycle, not after each task.

### Quick fix

For bug fixes and small isolated changes:

```
/fix   →   /push
```

No plan document, no task files. Claude reads the affected code, applies the minimal change, runs relevant tests, and hands off to `/push`.

If the fix grows beyond ~3 files or reveals unexpected complexity, Claude stops and suggests escalating to `/dev-1-plan`.

### Consistency audit

For reviewing a code area against project conventions:

```
/audit <area>   →   /push
```

Claude explores the target area, produces a numbered findings table grouped by severity, and waits for approval before applying any change.

---

## Key design decisions

**Evidence-based QA.** `/dev-4-qa` re-executes every verification from scratch. If the command was not run and the output not saved to a file, there is no evidence. "The code looks correct" is not a passing condition.

**One commit per feature.** The commit happens once — when all tasks are QA-approved and `/push` runs. This keeps history clean and ensures nothing ships without passing the full cycle.

**Docker-first.** No language runtimes on the host machine. Every project command goes through `docker compose -f dev/docker-compose.yml exec <service> <command>`. See `CLAUDE.md §9` for the full rule.

**Total ownership.** If Claude encounters a broken test or a failing lint check during a task — even if unrelated to the task — it is responsible for fixing it before continuing. There is no "pre-existing error".

**Right-sized architecture.** The architecture skill adapts to project complexity: scripts get flat modules, CRUD APIs get service functions, complex domains get full layering. No cathedrals for village chapels.

---

## Skills reference

| Skill | When Claude uses it |
|-------|-------------------|
| `architecture` | Before implementing any feature or fix — validates layer placement and scope |
| `git-conventions` | Before every commit and branch creation |
| `security` | Before every commit — full checklist against the diff |
| `test-discipline` | When writing or debugging any test |
| `ci` | When creating or modifying GitHub Actions workflows — Docker-first testing, pipeline structure, Codecov, Docker publishing |
| `api-design` | When designing or reviewing REST endpoints |
| `dev-workflow` | For all Docker commands during development |
| `backend-patterns` | When writing backend code — conventions and structure |
| `frontend-patterns` | When writing frontend code — conventions and structure |
| `memory-conventions` | When deciding what to write to `MEMORY.md` |

