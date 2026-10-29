---
name: planning-and-task-decomposition
description: >
  Turn clear requirements into an ordered, verifiable implementation plan. Use when a
  spec or well-understood goal needs decomposition before coding — when the task feels
  too large to start, the order of work isn't obvious, or scope must be communicated.
  Trigger phrases include: "plan this", "break this down", "make an implementation
  plan", "how should we implement this", "create a task list", "what order should we
  build this in". This skill defines HOW to build and in what order — it assumes WHAT
  to build is already pinned down. If requirements are loose, unverified, or missing,
  first run the task-scoping skill (task-scope-and-spec: scope a loose engineering
  task into a tight, testable spec list) or gather inputs per the Input Gate below.
---

# planning-and-task-decomposition

## Purpose

Decompose a known goal into small, ordered, independently verifiable tasks. The output
is a plan document you can follow yourself and hand pieces of to an agent as you go.
Planning is read-only: no code is written while this skill runs.

## Core principle

**Small enough to verify.** Every task must be implementable, testable, and reviewable
in one focused session. If a task can't state its acceptance criteria in ≤3 bullets,
it is two tasks. A plan whose tasks can't fail verification isn't a plan — it's a
wish list.

## Input Gate (triage — run before planning)

Judge the input quality first. Exactly one of:

- **Spec-ready** — requirements are concrete and testable (e.g. output of the
  task-scoping skill, a tight ticket, an approved design). → Plan directly.
- **Loose** — the goal is stated but requirements are vague, mixed with background, or
  depend on unverified codebase claims. → Invoke the task-scoping skill
  (`task-scope-and-spec`: scope a loose engineering task into a tight, testable spec
  list) and plan from its output.
- **Underspecified** — key context is missing and cannot be inferred. → Interview the
  user before anything else:
  - Ask **one question at a time**; stop when confident (~95%), not when exhaustive.
  - Accept answers in any form: direct answers, pasted notes, a Jira/Confluence link,
    a GitHub issue or PR link, a branch or diff name, a file path.
  - When a source link is given, read it before asking the next question.

Never plan on top of guessed requirements. An inference that survives into the plan
must be labeled as an assumption in the plan's Open Questions.

## Workflow (internal — do not narrate)

### 1. Investigate (read-only)
Read the spec and the relevant code. Identify existing patterns and conventions,
affected modules, contracts, and existing tests. Note risks and unknowns. Do not
restructure or "improve" beyond what the spec requires (YAGNI).

### 2. Map dependencies (high-level → low-level)
Sketch what depends on what (schema → models → endpoints → clients → UI, or the
project's equivalent). Implementation order follows the graph bottom-up. High-risk or
unknown-heavy tasks go early — fail fast.

### 3. Slice vertically
Prefer one complete, testable feature path per task over building a whole layer at a
time. Each slice leaves the system working. Horizontal slicing ("all the models, then
all the endpoints") is allowed only when a shared contract genuinely must exist first
— in that case, define the contract as its own small task, then slice vertically.

### 4. Write tasks
Each task uses the Task contract below. Fold setup, config, and docs into the task
whose deliverable needs them; split only where a reviewer could reject one task while
approving its neighbor.

### 5. Order and checkpoint
Arrange tasks so dependencies are satisfied and the system builds/tests green after
every task. Insert a checkpoint after every 2–3 tasks and at phase boundaries.

### 6. Map coverage (before presenting)
Every spec requirement maps to a task — list any that don't. This is a coverage check
against the spec, not a re-read of the plan.

## Task contract

```
## Task N: <short title — if it contains "and", it's two tasks>

**Description:** One paragraph — what this task accomplishes and why now.

**Acceptance criteria:** (≤3, each testable)
- [ ] ...

**Verification:**
- [ ] `pytest tests/... -v` (or project equivalent) — expected result stated
- [ ] Typecheck/lint passes (`pyright`, `ruff check`, or project equivalent)
- [ ] Manual check: <what to observe, if applicable>

**Interfaces:**
- Consumes: <exact names/signatures this task uses from earlier tasks, or "None">
- Produces: <exact names/signatures later tasks rely on, or "None">

**Dependencies:** Task numbers, or "None"
**Files:** exact paths (create/modify/test)
**Size:** XS | S | M   (L/XL = break it down)
**Owner:** <me | agent | pair>   (who drives this task — default: me)
```

Sizing: XS = single function/config, S = one component or endpoint (1–2 files),
M = one feature slice (3–5 files). Anything larger than M, or spanning two independent
subsystems, gets split before it enters the plan.

## Output contract

Save to `.claude/plans/YYYY-MM-DD-<task-name>.md` in the project (user preference
overrides), then present a short summary in chat — goal, phase names, task count,
checkpoints, not the full plan re-pasted. The document:

```
# Implementation Plan: <name>

**Goal:** <one sentence>
**Approach:** <2–3 sentences; key architecture decisions + rationale>

## Phase 1: <name>
Task 1 … Task N (full task contracts)
### Checkpoint: <what must be green before continuing>

## Phase 2: …

## Parallelizable
<independent slices safe to run concurrently; contracts to define first — omit if none>

## Risks
<risk → impact → mitigation — omit if none>

## Open Questions / Assumptions
<anything needing a human decision; assumptions carried from the Input Gate>
```

## Handoff (you drive — agent assists)

The default assumption is that **you implement the plan and pair with the agent**, not
that the agent runs the whole thing. After the plan is approved, don't auto-execute —
offer to help per task, in whatever mode you ask for:

- **Implement (a task, or the whole plan if small)** — the agent runs it through the
  Implementation loop below; scale (one task's criteria vs. the full task list) is your
  call.
- **Guide me while I implement it** — you write the code; the agent gives one piece per
  turn with the design decision it forces, then reviews what you wrote. Use
  `guided-implementation` (owner: me/pair). This is the default for tasks marked
  `Owner: me`.
- **Review my code** — the agent reviews a part you wrote. For anything substantial,
  use the dedicated review skill (`python-code-review`: SOLID / correctness / security /
  style / tests, with a verdict) rather than reviewing ad hoc.
- **Write tests** — the agent adds tests for a task per the project's test conventions.

### Implementation loop (agent-driven tasks)
Whenever the agent implements a task (or the whole plan) itself, run it through
`goal-workflow` (implement a spec fully, check each requirement off, report pass/fail) —
pass the task's acceptance criteria (or the full task list, for whole-feature runs) as its
spec input. This replaces ad hoc self-checking: requirement coverage comes back as a report
you can skim. No separate self-review step is needed.

Quality review is a deliberate second step, not automatic. Once `goal-workflow` reports
the acceptance criteria satisfied, run `python-code-review` (correctness, security,
simplicity, and my conventions, with a verdict) once.

### Fresh session
If work will continue in a new session, produce a handover first (the session-handover
skill, `task-handover`: structured HANDOVER.md for cold-start resumption) — written
plans and handovers survive context boundaries.

## Anti-patterns (do not)

- No code during planning — the deliverable is the plan document.
- No auto-executing the whole plan; you drive unless you say otherwise.
- No placeholder tasks ("implement the feature", "add tests") without concrete
  acceptance criteria.
- No XL tasks, no task without a verification step, no plan without checkpoints.
- No speculative scope: if the spec doesn't require it, it isn't in the plan.
- No restating the user's background or narrating the workflow steps.
- No architecture essay — decisions get 1–2 lines of rationale, not sections.
- No skipping `goal-workflow` for agent-driven implementation, and no second review pass
  once `python-code-review` has run.

## Clarifying questions

Outside the Input Gate, ask at most one question, and only if the answer would change
task boundaries or ordering. Otherwise plan with a labeled assumption.
