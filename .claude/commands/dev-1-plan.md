---
description: Plan a new feature — conceptual design, scope, and proposal
argument-hint: <brief description of the feature>
---

# Plan Feature: $1

**Goal**: produce a design document approved by the user.
**Behaviour**: you only plan and ask questions. No code, no tasks, no files beyond the plan document itself.

---

## Phase 1 — Understand the Goal

1. Read `CLAUDE.md` and `MEMORY.md` for project context.
2. Read `docs/` for existing documentation (ARCHITECTURE.md, configuration.md, etc.).
3. If previous plans exist in `docs/plans/`, read them to understand decisions already made.
4. Ask the user everything needed with `AskUserQuestion`:
   - What problem does this feature solve?
   - What stack layers does it affect?
   - Are there open design decisions?
   - Are there constraints or preferences?

**Never assume. If anything is unclear, ask with `AskUserQuestion` before continuing. Five questions now beats a full rewrite later.**

**Do not advance to Phase 2 until you have clear answers.**

---

## Phase 2 — Explore the Affected Code

1. Use `Explore` agents to understand the current state of the code in affected areas.
2. **Consult the `architecture` skill** and validate that the proposed approach fits the project's layer structure. Surface any red flags before continuing.
3. Identify:
   - Files that will be modified or created.
   - Existing patterns that must be followed (consult `backend-patterns` and `frontend-patterns` skills).
   - Opportunities to reuse existing code — search before proposing new abstractions.
   - Dependencies between components.
   - Known risks or pitfalls (consult MEMORY.md).
4. Present the user with a summary of findings and validate your understanding.

---

## Phase 3 — Write the Proposal

Create the plan document at `docs/plans/<feature-name>.md` with this structure:

```markdown
# Feature Name

## Branch

`feat/<feature-name>`

> Branch will be created from `develop` if it exists, otherwise from `main`/`master`.
> All tasks in this feature will be implemented on this branch.

## Context

What problem it solves. Why it is needed now.

## Decisions Confirmed with User

| Topic | Decision |
|-------|----------|
| ... | ... |

## Design Proposal

Conceptual design (not code). Components involved, data flow, architecture decisions.

## Scope

### What Is Included
- ...

### What Is NOT Included
- ...

## Affected Layers

| Layer | Impact |
|-------|--------|
| ... | ... |

## Implementation Order

Numbered list of high-level steps.

## Critical Files

| File | Changes |
|------|---------|
| ... | ... |

## Risks and Considerations

- ...

## Open Design Decisions

(If there are decisions yet to be made, list them here. They must be closed before creating tasks.)
```

Confirm the branch name with the user before saving the document — it will be used for the entire feature.

Present the document to the user and ask for feedback.

---

## Phase 4 — Iterate Until Approval

- If the user has feedback, adjust the document.
- If there are open decisions, close them with `AskUserQuestion`.
- Once approved, tell the user: **"Plan saved. Next step: `/dev-2-tasks docs/plans/<feature-name>.md`"**

---

## Unbreakable Rules

- **Do not implement anything**: no code, no tasks, no files beyond the plan document.
- **Ask before assuming**: if in doubt, use `AskUserQuestion`. Always.
- **Read the actual code** before proposing changes to existing areas.
- **The document must be self-contained**: someone reading it without prior context must understand the proposal.
