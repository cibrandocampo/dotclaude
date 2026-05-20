---
description: Structured audit of a code area — find inconsistencies, propose fixes, apply with approval
argument-hint: <area to audit, e.g.: "CSS design system", "API endpoints", "data models">
---

# Audit: $1

**Goal**: systematically review a code area, surface inconsistencies against project conventions, and apply approved fixes in a single clean pass.
**Behaviour**: you explore and propose before touching anything. No changes until the user approves the full list.

---

## Step 1 — Load Context

1. Read `CLAUDE.md` and `MEMORY.md`.
2. Read the relevant project skills for the area being audited (see `.claude/skills/`).
3. Identify the canonical conventions that apply to `$1`.

---

## Step 2 — Explore the Area

Use `Explore` agents to read all files in the target area.
Do not read the entire codebase — scope to what `$1` describes.

---

## Step 3 — Build the Findings List

Compare what you found against the conventions from Step 1.
Produce a numbered findings table:

| # | File | Issue | Convention violated | Proposed fix |
|---|------|-------|--------------------|--------------|
| 1 | `src/components/Foo.js` | ... | ... | ... |

Group findings by severity:
- **Must fix**: contradicts an explicit convention
- **Should fix**: inconsistent with the rest of the codebase
- **Minor**: style preference, low impact

Present the table to the user and ask:
- Is anything missing from the list?
- Is anything out of scope?
- Are there findings you want to skip?

**Do not apply any changes until the user approves.**

---

## Step 4 — Apply Approved Fixes

Once the user approves (all or a subset):

1. Apply each fix in order.
2. For each fix: read the file, apply the change, re-read to confirm correctness.
3. Do not introduce changes beyond the approved list.

---

## Step 5 — Verify

Run the relevant test suite for the area touched (see `dev-workflow` skill for commands).
If tests fail: diagnose and fix before handing off.

---

## Step 6 — Hand Off

Tell the user: **"Audit complete — N fixes applied. Ready for `/push`."**
If any approved skill or convention changed during the audit, flag it: "Consider updating the `<skill>` skill to reflect finding #N."

---

## Unbreakable Rules

- **Propose before applying**: no changes without user approval of the findings list.
- **Scope discipline**: only touch what `$1` describes. If you find issues outside scope, note them but do not fix them.
- **Read before writing**: never modify a file you haven't read in this session.
- **Do NOT commit**: that is `/push`'s responsibility.
