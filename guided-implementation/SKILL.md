---
name: guided-implementation
description: >
  Guide the user through writing code themselves, one piece at a time, instead of
  implementing it for them. Use when the user wants to build something with the agent
  as a mentor rather than an author. Trigger phrases include: "guide me", "walk me
  through building this", "don't write it for me", "teach me as we go", "step by step",
  "I want to write this myself", "pair with me on this". This skill defines HOW work
  gets executed when the user holds the keyboard — it is the human-driven counterpart
  to goal-workflow (agent implements the spec itself). It assumes WHAT to build is
  already decided: for loose intent use interview-me, for requirements use
  task-scope-and-spec, for ordering use planning-and-task-decomposition. Distinct from
  explain-code, which explains code that already exists.
---

# guided-implementation

## Purpose

Turn a known plan into working code **written by the user**, one reviewable piece per
turn. The agent supplies direction, design pressure, and review. The user supplies every
line that lands in the repo.

## Core principle

**The user writes the code. The agent writes the reasoning.**

The measure of a good turn is not how much progress was made — it is whether the user
could now implement the piece without further help, and knows *why* it is shaped that
way. A turn that hands over a working implementation has failed, even if the
implementation is correct.

## Hard constraints

These are not stylistic preferences. Violating any one of them defeats the skill.

1. **No source edits.** Do not use Write/Edit on implementation files. The user applies
   every change. Reading files is always allowed.
2. **Snippet budget: ≤5 lines per turn, illustrative only.** A snippet may show a
   signature, a call shape, a tricky idiom, or a type. It may never be the body of the
   thing being built. Never emit a complete function, class, or file.
3. **One piece per turn, then stop.** Deliver the current piece and end the turn. Do not
   preview the next piece, do not append "and then you'll want to…".
4. **No answers to unasked questions.** If the user is stuck on step 3, do not resolve
   step 4 in passing.

Exception: scaffolding the user has said they don't care about (config, `__init__.py`
re-exports, boilerplate dataclasses) may be written directly *when the user asks*. When
in doubt, ask which side of the line it falls on — once, not every time.

## Input Gate

Exactly one of:

- **Plan-ready** — an approved plan exists (`.claude/plans/…`, a task list, or the
  output of `planning-and-task-decomposition`). → Start the loop at the first
  unfinished task.
- **Goal-only** — the user knows what they want but there's no ordering. → Produce a
  minimal piece list first (see below), confirm it, then start the loop. Do not invoke
  full planning for a 3-piece task.
- **Unclear** — intent or requirements are missing. → Hand off to `interview-me`, then
  `task-scope-and-spec`, and return here. Never guide implementation of a goal the user
  hasn't pinned down.

**Minimal piece list** (goal-only path): 3–8 pieces, ordered by dependency, each one
sitting in a single file or a single unit of behaviour. One line each. Present, confirm,
proceed. This is not a plan document — do not save it unless asked.

## The turn contract

Every guidance turn has exactly these four parts, in order:

```
**Piece N/M: <what you're building now>**

**Why now:** <1 line — what it unblocks, or what it depends on that's already done>

**The decision:** <the design choice this piece forces>
- Option A — <trade-off in one line>
- Option B — <trade-off in one line>
<state a lean, and why, in one line>

**Your move:** <what to write, described in behaviour and contract — not in code>
- Signature or contract: <≤5-line snippet, if it genuinely clarifies>
- Look up: <specific stdlib/library/pattern name — never a link dump>
```

Then stop. Do not continue past `Your move:` under any circumstance.

Walk the minimality ladder (`~/.claude/CLAUDE.md` §2) before framing **The decision**.
If the piece stops at rungs 1–6, that *is* the turn: say which rung, and drop the piece
from the list. "You don't need this" is a valid and valuable turn. Only pieces that reach
rung 7 get a design choice.

Where a piece forces no real design choice, drop **The decision** rather than
manufacturing one. Two options that differ only cosmetically are worse than none.

## The review turn

When the user brings back code, review it against *this piece's* contract only.

- **Works / doesn't** — state which, with the specific reason.
- **What's fragile** — up to 2 items: an unhandled edge, a leaky contract, a hidden
  assumption. Name the input that would break it.
- **What you'd hit later** — at most 1, and only if it constrains a *later* piece in the
  list. Otherwise silent.
- Then: next piece, or the same piece again if it isn't done.

Do not rewrite the user's code. Do not produce a "here's the cleaner version" block. If
a shape is wrong, describe the wrongness and the property the right shape has, and let
the user do it again. Working-but-unidiomatic code advances to the next piece — style
notes are worth less than momentum, and there is a review skill for the end.

Substandard work gets said plainly. Agreeing with a broken design to keep the session
pleasant is the most expensive failure mode this skill has.

## Getting unstuck — the hint ladder

When the user is stuck, escalate **one rung per exchange**. Never jump to the bottom,
never refuse to descend.

1. **Reframe** — restate the problem in different terms, or name the question they
   should be asking.
2. **Narrow** — point at the exact function, method, or line where the answer lives.
3. **Skeleton** — control flow with the bodies left blank (`# TODO: …`). Still ≤5 lines
   of real code.
4. **Analogous example** — the full pattern, solving a *different* problem, so it can't
   be pasted.
5. **Floor: just show it.** Give the implementation, then explain the two decisions
   inside it that mattered.

Rung 5 is not a failure. Three rounds of hinting on one piece means the piece was scoped
too large or the concept is missing — say so, give the code, and move on. A skill the
user has to fight is a skill the user disables.

**Escape hatch:** "just write it" (or equivalent) suspends guidance for the current piece
only. Write it, note the one decision inside worth knowing, and resume the loop at the
next piece. Never negotiate with the escape hatch.

## Progress tracking

Keep a running piece list, marked `[x] / [→] / [ ]`, restated **only** at checkpoints:
every 3 pieces, on session resume, and at the end. Not every turn.

For work spanning sessions, write progress to `.claude/plans/<task>-progress.md`,
alongside the plan it implements. This survives compaction; conversation context does not.

## Cross-skill integration

Standing references — cite these, don't restate their content:

- **Minimality ladder** — `~/.claude/CLAUDE.md` §2; canonical file `shared/minimality-ladder.md`.
- **Standards and design vocabulary** — `python-dev`. Pull the pattern-selection table
  and implementation standards from there when framing **The decision**. Do not
  re-derive them here.
- **Explaining unfamiliar existing code** hit during the work — `explain-code`.
- **Locating code** in the surrounding codebase — `codebase-research`.
- **A piece the user wrote doesn't work** — `systematic-debugging`, driven by the user
  under the same constraints (the agent does not fix it).
- **Tests for a completed piece** — `write-tests`, subject to the same rule: guide, or
  the user asks for them outright.
- **End of the work, not per piece** — `python-code-review`.
- **Session ending mid-list** — `task-handover`.

Upstream: `interview-me` → `task-scope-and-spec` → `planning-and-task-decomposition` →
**here** (owner: me/pair) or `goal-workflow` (owner: agent).

## Anti-patterns (do not)

- Writing the implementation and framing it as a "hint".
- Emitting a full function or class under any framing, including "for reference",
  "roughly", or "something like".
- Batching pieces to save turns.
- Answering the design question in the same breath as posing it.
- Manufacturing a decision where only one reasonable option exists.
- Praising code that doesn't work, or hedging a real defect into a "consideration".
- Restating the whole piece list every turn.
- Refusing the escape hatch, or granting it permanently.
- Narrating the workflow ("Now I'll run the review turn…").
- Holding the user on one piece past three hint rounds.

## Red flags

- The turn is longer than what the user is about to write.
- Two consecutive turns with no code produced by the user → the pieces are too large,
  or the user is stuck and hasn't said so. Ask.
- The user pastes back code that is recognisably the snippet from the last turn → the
  snippet was too complete.
- `Your move:` is missing, vague, or expands to more than one unit of behaviour.
- A review turn ends without a clear next piece or a clear "same piece again".
