---
name: hac-init
description: Bootstrap the .hac/ (Human-Agent Context) directory in any project. Use when the user says "set up hac", "add .hac", "set up working memory", "initialize hac", or when beginning multi-session work on a project that lacks a .hac/ directory. Language-agnostic — works for Python, TypeScript, Go, Rust, or any project type. Also trigger when migrating existing plan files into .hac/ or when resetting a stale .hac/ directory.
---

# HAC Init

Bootstrap the `.hac/` (Human-Agent Context) directory for any project.

---

## Core Principle

**HAC is a shared context layer, not a project management tool.**

It exists so that any human or agent can understand the project's current state in under 30 seconds — without reading code, scrolling through chat history, or asking "where were we?" Keep it minimal. Four files, four concepts. Nothing more.

---

## Triage Gate

Before bootstrapping, decide if HAC is needed:

- **Trivial / one-off work** (single-file edit, formatting fix, quick question): HAC adds overhead with no benefit. Skip it.
- **Non-trivial / multi-session work** (multi-step implementation, architectural changes, ongoing project): Bootstrap `.hac/` using the phases below.

When in doubt, ask the user. Don't create `.hac/` silently on the first interaction with a project.

---

## Common Usage Patterns

These are the everyday triggers for touching `.hac/` once it's bootstrapped — not just the initial setup:

- **[Most common] Right before opening a PR** — when a task is done, do the Wrap-Up steps: flip the status to ⚪ Done, move the row from `status.md` to the `README.md` master index, and (only if a task file exists) append a final session log entry. If the work never had a task file, do not create one now.
- **"Record this in HAC decisions"** — the user is flagging a decision they just made. Append it to `decisions.md` (Context / Choice / Why / Rejected) and add a row to the quick reference table and `README.md`.
- **"Set up a HAC task for X"** (e.g. "Set up a HAC task for the analytics subagent DAP integration") — for a bit-complicated or multi-step task, create `.hac/tasks/<task-name>.md` from the task template and add a row to `status.md`'s overview table to track it going forward.

---

## Phase 1: Gather Context

Before generating files, collect:

1. **Project name** — from `CLAUDE.md`, `README.md`, `pyproject.toml`, `package.json`, or the user.
2. **Project description** — one sentence.
3. **Existing plan files** — check for `.claude/*.md`, `TODO.md`, `PLAN.md`, or similar. Offer to migrate them into `.hac/tasks/`.

If `CLAUDE.md` or `README.md` already contains this information, extract it — don't re-ask.

---

## Phase 2: Generate Structure

Create exactly four items:

```
.hac/
├── README.md        # What is .hac + master index
├── status.md        # Dashboard: active, blocked, parked work
├── decisions.md     # Lightweight ADRs (append-only)
└── tasks/           # One scratchpad per non-trivial task (on-demand)
    └── .gitkeep
```

### .hac/README.md

```markdown
# .hac — Human-Agent Context

This directory is the shared context layer between humans and AI agents
working on this project. **HAC** stands for **Human-Agent Context**.

It exists so that when anyone — human or agent — picks up this project
after a break, they can understand the current state in under 30 seconds
without reading code, scrolling through Slack, or asking "where were we?"

## How It Works

| File | Purpose | Who updates it |
|------|---------|----------------|
| `status.md` | Dashboard — what's active, blocked, parked, done | Agent (after each work block) |
| `decisions.md` | Why we chose X over Y | Agent (when decisions are made) |
| `tasks/<name>.md` | Scratchpad per task — plan, findings, log | Agent (during execution) |

## Master Index

Single-glance lookup across everything tracked in this project.

### Tasks

<!-- Rows ordered newest-first by completion date.
     `File` may be `—` for work that was completed without a task file —
     never create one retroactively just to fill this column. -->
| Task | Status | Priority | Owner | File | Updated |
|------|--------|----------|-------|------|---------|
| _No tasks yet_ | | | | | |

### Decisions

| Date | Decision | Choice | Impact |
|------|----------|--------|--------|
| _No decisions yet_ | | | |

## Rules

- This directory is tracked in git. It is project knowledge.
- `decisions.md` is append-only. Never edit or remove past entries.
- Task files are created on-demand, not in advance.
- Don't use `.hac/` for trivial one-off fixes.
```

### .hac/status.md

```markdown
# Status — PROJECT_NAME

> PROJECT_DESCRIPTION

## Overview

Active, blocked, and parked work. Completed tasks live only in the
master index (`README.md`). This table shows what needs attention now.

<!-- Rows ordered by priority (P0 → P1 → P2); within same priority: Active before Blocked before Parked -->
| Task | Status | Priority | Owner | Updated |
|------|--------|----------|-------|---------|
| _No active tasks_ | | | | |

### Status Labels

- 🟢 **Active** — Currently being worked on
- 🔴 **Blocked** — Waiting on an external dependency, decision, or review
- 🔵 **Parked** — Noted for future; not currently scheduled
- ⚪ **Done** — Author (agent or human) judges the work complete (e.g. before opening a PR) → move row to README.md master index

### Transitions

- 🟢 → ⚪ when the agent or user judges the work complete
- 🟢 → 🔴 when an external dependency, unresolved question, or pending review blocks progress
- 🔴 → 🟢 when the blocker is resolved
- 🔵 → 🟢 when the idea is promoted to active work (create a task file)
- Any → ⚪ moves the row from this table to the README.md master index

Done is a local judgment, not an external approval. `.hac/` does not mirror
PR/review state — the PR is its own review surface. If a task needs a human
to sign off before it can be considered complete, keep it 🔴 Blocked
("blocked on review of X") rather than inventing a separate review state.

### Priority Levels

- **P0** — Must be done now, blocks other work
- **P1** — Important, do next
- **P2** — Nice to have, do when bandwidth allows

## Parked Ideas

Captured during sessions for later consideration. Promote to a task
when the need becomes concrete.

<!-- Rows ordered newest-first by Date -->
| Idea | Origin | Date | Notes |
|------|--------|------|-------|
| _Nothing parked_ | | | |

**When to promote:** When a parked idea is referenced by an active task,
when the user explicitly schedules it, or when a new requirement makes
it necessary. On promotion: create a task file, add a row to the
Overview table above, and remove the row from this section.

## Blocked Items

_Nothing blocked._

## Notes

_Anything that doesn't fit in a task file but is useful to know right now._
```

### .hac/decisions.md

```markdown
# Decisions — PROJECT_NAME

## Quick Reference

| Date | Decision | Choice | Impact |
|------|----------|--------|--------|
| _No decisions yet_ | | | |

---

<!-- Append new decisions below this line, newest first.

Format:
## YYYY-MM-DD: Decision Title
- **Context:** What situation or constraint prompted this decision?
- **Choice:** What did we decide?
- **Why:** Why this option over others?
- **Rejected:** What alternatives were considered and why not?
-->
```

### .hac/tasks/ (template reference — not generated at scaffold time)

Individual task files are created on-demand during development. The standard
template for a task file (`.hac/tasks/<task-name>.md`):

```markdown
# Task: [Task Name]

| Field | Value |
|-------|-------|
| **Status** | 🟢 Active |
| **Priority** | P1 |
| **Owner** | @name |
| **Created** | YYYY-MM-DD |
| **Updated** | YYYY-MM-DD |
| **Scope** | ~N sessions |

## Context

What are we trying to achieve and why?

## Plan

- [ ] Step 1
- [ ] Step 2
- [ ] Step 3

## Notes / Findings

Discoveries made during execution. Facts, not opinions.

## Open Questions / Risks

Unknowns that need resolution before or during implementation.

## Session Log

- **YYYY-MM-DD:** What happened this session. What's next.
```

---

## Phase 3: Post-Bootstrap

After generating files:

1. Replace `PROJECT_NAME` and `PROJECT_DESCRIPTION` placeholders in `status.md` and `decisions.md`.
2. If prior plan files exist (`.claude/*.md`, `TODO.md`, etc.), offer to migrate them into `.hac/tasks/`.
3. Ensure `.hac/` is **not** in `.gitignore` — it must be tracked in git.
4. **Update the project's `CLAUDE.md` with the HAC protocol** — this is mandatory, not optional.

   Without this step, future Claude sessions will not know `.hac/` exists and will not read or update it automatically. Every session would require manual prompting.

   First, check whether the HAC section is already present:
   - Search the project `CLAUDE.md` (and the global `~/.claude/CLAUDE.md`) for the string `".hac/"` or `"HAC"`.
   - If the section **already exists** in either file: skip this step — do not add a duplicate.
   - If the section is **absent**: append the HAC section from `hac-init/README.md` to the project's `CLAUDE.md`. Create the file if it does not exist.

5. Confirm to the user what was created. If the HAC section was added to `CLAUDE.md`, say so. If it was already present (globally or in the project), say that too so the user knows it was checked.

---

## Rules

- Don't create `.hac/` for a trivial one-off task.
- Create task files on demand, never at scaffold time.
- Never create a task file retroactively for completed work. Task files are forward-looking working memory; a done task with no file needs no file, and a wrap-up request is never a reason to create one.
- Keep `.hac/` tracked in git, never in `.gitignore`.
- Don't duplicate the full task list: `README.md` is the master index of all tasks including done; `status.md` is the dashboard of active, blocked, and parked work only.
- `decisions.md` is append-only. Never edit or remove past entries.
- Don't mirror PR or review state, and don't add a review status. Done is a local judgment; use 🔴 Blocked for "awaiting review".
- Keep to the four-item structure unless the user explicitly asks for more.
