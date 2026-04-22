---
name: explain-code
description: Explains code with visual diagrams, analogies, and structured walkthroughs. Use when explaining how code works, teaching about a codebase, when the user asks "how does this work?", "what does this do?", "explain this code", or "walk me through this". Distinct from codebase-research which answers WHERE something is — this skill answers HOW and WHY something works.
---

# Explain Code

Structured methodology for explaining code at any level of complexity — from a single function to an entire subsystem.

---

## Core Principle

**TEACH, DON'T DUMP.** The goal is understanding, not information transfer. Match the explanation to the reader's level and the code's complexity. Every explanation should leave the reader able to modify the code confidently.

---

## Cross-Skill Integration

This skill often benefits from a prior navigation step. If the user asks to explain a module, subsystem, or unfamiliar codebase:

1. Check whether a codebase navigation skill exists in the project.
2. If yes, use it first to locate the relevant files, map dependencies, and identify the public API — then explain what you found.
3. If no, do a quick orientation yourself (`git ls-files`, entry points, public API surface) before explaining.

Never explain code you haven't read. If the scope is unclear, read the code first using appropriate tools.

---

## Triage Gate

Classify the explanation scope before starting:

- **Line / expression** — "What does this line do?" → Direct inline answer. No framework needed.
- **Function / method** — "How does this function work?" → Analogy + step-by-step walkthrough.
- **Class / module** — "Explain this class" or "What does this module do?" → Responsibility summary + diagram + key method walkthrough.
- **Subsystem / architecture** — "How does the auth system work?" or "Explain this pipeline" → Layered explanation with diagrams showing component relationships.

---

## Explanation Framework

### Step 1: One-Sentence Summary

Start with a single sentence that captures what the code does in plain language. No jargon. If you can't summarize it in one sentence, you don't understand it well enough yet.

### Step 2: Analogy (for function-level and above)

Compare the code to something concrete and familiar. Good analogies:
- Map to physical processes ("like a factory assembly line where each station transforms the product")
- Map to everyday activities ("like a librarian who checks the card catalog before walking to the shelf")
- Map to well-known systems ("works like a post office — sorts by destination, then routes")

Bad analogies:
- Circular ("it's like a function that processes data" — this explains nothing)
- Overly technical ("it's like a B-tree rebalance" — only useful if the reader already knows B-trees)
- Forced (if no analogy fits naturally, skip it)

### Step 3: Visual Diagram

Select the diagram type based on what aspect of the code matters most:

**Data flow** — When the code transforms data through stages:
```
Input → [Parse] → [Validate] → [Transform] → [Store] → Output
```

**Sequence / interaction** — When multiple components communicate:
```
Client          Server          Database
  │── request ──→│                │
  │              │── query ──────→│
  │              │←── results ────│
  │←── response ─│                │
```

**Class / dependency** — When structure and relationships matter:
```
┌─────────────┐     ┌──────────────┐
│  Controller  │────→│   Service    │
└─────────────┘     └──────┬───────┘
                           │ uses
                    ┌──────▼───────┐
                    │  Repository  │
                    └──────────────┘
```

**State machine** — When the code manages transitions:
```
[Idle] ──start──→ [Running] ──complete──→ [Done]
                      │                      │
                    error                  retry
                      ▼                      │
                  [Failed] ←─────────────────┘
```

**Decision tree** — When the code branches on conditions:
```
Is user authenticated?
├── Yes → Has permission?
│         ├── Yes → Execute action
│         └── No  → 403 Forbidden
└── No  → 401 Unauthorized
```

Pick the diagram type that best reveals the code's core logic. Use ASCII art so it renders everywhere.

### Step 4: Step-by-Step Walkthrough

Walk through the code execution path in order. For each meaningful step:

1. **What** — What happens at this step (in plain language)
2. **Why** — Why is this step necessary (what breaks without it)
3. **Code** — Reference the specific lines/variables involved

Do NOT narrate every line. Focus on the steps that carry the logic — skip boilerplate, imports, and obvious assignments.

For complex code, use a numbered walkthrough that the reader can follow alongside the source:

```
1. Receive the request and extract the user ID from the JWT token (line 12)
2. Query the permissions cache — if miss, fall through to the database (lines 15-18)
3. Check each required permission against the user's set (line 22)
4. Short-circuit on first missing permission and return 403 (line 24)
5. If all pass, delegate to the handler (line 27)
```

### Step 5: Gotchas and Subtleties

Highlight 1-3 things that are non-obvious, easy to get wrong, or commonly misunderstood:

- Hidden assumptions ("this assumes the list is already sorted — if it isn't, the binary search silently returns wrong results")
- Side effects ("calling this function also invalidates the cache, which isn't obvious from the name")
- Performance characteristics ("this is O(n²) because of the nested loop at line 34 — fine for small inputs but will bottleneck on large datasets")
- Naming mismatches ("despite the name `get_user`, this actually creates a user if one doesn't exist")

If nothing is genuinely tricky, skip this step. Don't manufacture gotchas.

---

## Calibration Rules

- **Match the reader's level.** If the reader is a beginner, define terms. If they're an expert, skip basics and focus on design decisions and tradeoffs.
- **Shorter is better.** A crisp 10-line explanation beats a rambling 50-line one. Cut anything that doesn't advance understanding.
- **Answer the question asked.** If they asked "what does line 42 do?", don't explain the entire module.
- **Code references are mandatory.** Every claim about behavior must reference specific lines or functions. Vague explanations like "the system handles errors" are not useful.

---

## Process Violations — Red Flags

Stop and correct if you catch yourself doing any of the following:

- Explaining code you haven't actually read (guessing from names or comments)
- Providing a line-by-line narration instead of focusing on logic flow
- Using an analogy that requires more explanation than the code itself
- Drawing a diagram that doesn't match the actual code structure
- Skipping the one-sentence summary and jumping into details
- Explaining implementation details when the user asked about behavior
- Giving a module-level explanation when the user pointed at a specific function
