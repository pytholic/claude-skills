---
name: systematic-debugging
description: Language-agnostic, four-phase debugging method that finds the root cause before any fix. Use when investigating a bug, a failing test, unexpected behavior, or a runtime or logic error — for example when the user says "debug", "fix this bug", "why is this failing", "not working", or shares a stack trace or error log.
---

# Systematic Debugging

## Core Principle

**No fixes without root-cause investigation first.**

Never apply symptom-focused patches that mask underlying problems. For example, adding `if x is None: return` at the crash site instead of tracing where the missing data originated is a process violation.

## Triage Gate

Not every bug needs the full four-phase framework. Before entering the phases, perform a quick triage:

- **Trivial bug** (typo, wrong variable name, off-by-one that is immediately obvious from the error): Fix it directly, add a brief note explaining the cause, and move on. No hypothesis template needed.
- **Non-trivial bug** (root cause is not immediately obvious, involves data flow across multiple functions/modules, or has failed one fix attempt already): Enter the full four-phase framework below.

When in doubt, use the full framework. It costs minutes and saves hours.

## Cross-Skill Integration

This is a methodology skill. The steps below involve writing tests and implementing code. For each of these activities, check whether a relevant skill exists in the current project and apply it. If no matching skill is found, follow the inline guidance provided.

1. **Writing Reproduction Tests**: Look for any skill related to testing conventions, test authoring, or test structure. Apply its rules when writing the failing test case. If none exists: write a minimal, isolated test that fails on the current bug and passes on the correct behavior. Prefer parameterized tests for boundary conditions. Name tests descriptively so the failure message explains the bug.

2. **Implementing the Fix**: Look for any skill related to the language or framework being used (e.g., a Python development skill, a TypeScript conventions skill, etc.). Apply its style rules, typing expectations, and structural constraints when writing the fix. If none exists: follow the project's established conventions — use explicit types, keep functions focused and short, apply SOLID principles, and match the existing code style.


## The Four-Phase Framework

### Phase 1: Reproduce & Trace

**Goal**: Confirm the bug exists and narrow down where the invalid state originates.

1. **Read every word of the error message.** Extract: the exception type, the failing line, and any chained/cause exceptions.
2. **Reproduce the issue.** Create a minimal failing test case. If you cannot reproduce it, you cannot verify a fix — do not proceed until reproduction is achieved.
3. **Trace data flow backward.** Do NOT stop at the crash site. Work backward through the call chain using these concrete steps:
   - Read the stack trace bottom-to-top, opening each frame's source.
   - Identify the first frame where the data is already invalid.
   - Add targeted logging/print statements at function boundaries around that frame.
   - Inspect input arguments and return values at each boundary until you find where valid data becomes invalid.
4. **Collect evidence.** Record specific observations: exact variable values, log lines, timestamps, or conditions that trigger vs. don't trigger the bug. You will need these in Phase 2.

### Phase 2: Hypothesis (before any fix)

Before writing any fix, state your hypothesis explicitly. This is the step that keeps a fix aimed at the cause rather than the symptom.

Read the [template.md](template.md) file in this directory and fill out every field. The template includes:

- **Symptom**: Where and how the error manifests.
- **Evidence**: Specific observations (variable values, log output, stack frames) that point to the root cause.
- **Root Cause**: The exact logical flaw, broken assumption, or data corruption causing the issue.
- **Proposed Fix**: How to fix the problem at its origin, not at the crash site.

**Do not proceed to Phase 3 until the hypothesis is written and the Evidence field contains at least one concrete observation.**

### Phase 3: Implementation & Verification

1. **Implement a single, targeted fix** that addresses the root cause from your hypothesis: one logical change per attempt. Do not bundle unrelated refactors or "while I'm here" changes, and never catch broad exceptions to silence the error.
2. **Follow project conventions.** Apply the relevant language/framework skill or fall back to the project's existing style.
3. **Run the reproduction test.** It must now pass.
4. **Run the full test suite.** Ensure zero regressions. If new tests fail, determine whether they are pre-existing flaky tests or genuine regressions from your change before proceeding.

### Phase 4: Stop Rules

**Single failure**: If your fix doesn't resolve the reproduction test, return to Phase 2. Re-examine your evidence, invalidate the current hypothesis, and form a new one.

**Three consecutive failures**: STOP ENTIRELY. Three failed hypotheses in a row indicate one of:
- An incorrect mental model of the system
- An architectural/design flaw beyond a single bug fix
- Missing context (undocumented side effects, external dependencies, race conditions)

When you hit this threshold, do not attempt another patch. Instead, fill out the [escalation_template.md](escalation_template.md) and surface it to the user for discussion.

The bug is fixed when the reproduction test and the full suite pass. That test run is the acceptance check; don't add a separate review pass on top of it.