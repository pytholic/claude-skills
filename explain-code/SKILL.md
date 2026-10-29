---
name: explain-code
description: Explains how and why code works, with an analogy, a diagram, and a walkthrough matched to the scope of the question. Use when the user asks "how does this work?", "what does this do?", "explain this code", or "walk me through this". For where something lives in a codebase, use codebase-research instead.
---

# Explain Code

Teach, don't dump. The goal is that the reader can modify the code confidently afterward. Match the depth to the reader's level and to the scope of the question.

Read the code before explaining it; never explain from names or comments alone. For a module, subsystem, or unfamiliar codebase, locate the relevant files and public API first (the `codebase-research` skill helps here).

## Scope

- **Line or expression**: answer directly, inline.
- **Function or method**: analogy, then a step-by-step walkthrough.
- **Class or module**: responsibility summary, diagram, then the key methods.
- **Subsystem or architecture**: layered explanation, with a diagram of how the components relate.

Answer the question asked. If they asked about line 42, don't explain the whole module.

## Structure

1. **One-sentence summary** in plain language. If you can't write it, read more code first.
2. **Analogy** (function level and above) that maps to something concrete and familiar. Skip it if nothing fits naturally; a forced or circular analogy is worse than none.
3. **Diagram** in ASCII, chosen for what matters most: data flow, sequence, class/dependency, state machine, or decision tree. It must match the actual code.
4. **Walkthrough** of the execution path. For each meaningful step: what happens, why it's needed, and which lines it refers to (`file:line`). Skip boilerplate, imports, and obvious assignments.
5. **Gotchas** (1–3, only if real): hidden assumptions, side effects, performance traps, names that don't match behavior.

## Calibration

- Beginners get terms defined; experts get design decisions and tradeoffs.
- A crisp 10-line explanation beats a 50-line one.
- Every claim about behavior points to specific lines or functions.
