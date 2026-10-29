---
name: python-dev
description: Planning-first Python development — understand, design, implement, check — with Python 3.13+, ruff, pyright, pytest, and pyproject.toml. Use when implementing a Python feature or module, refactoring, or making a Python design or architecture decision. For setting up a new project, use python-project-scaffold instead.
---

# Python Development Workflow

Plan before you code. Understand the requirement and pick the design before writing the implementation, at a depth that matches the size of the change.

## Triage

Classify the work first:

- **Small change** (bug fix, single function, config tweak): go straight to Phase 3.
- **Medium feature** (new module, endpoint, or service): do Phases 1 and 2 briefly, then implement.
- **Large feature or new subsystem** (multi-module, cross-cutting, architectural impact): do every phase, top-down: high-level design first, then components, then code.

## Phase 1: Understand

Answer these before designing:

1. **Goal**: what problem are we solving, in one sentence?
2. **Inputs and outputs**: what data flows in and out?
3. **Constraints**: performance, compatibility, external dependencies, existing patterns in the codebase.
4. **Edge cases**: empty inputs, concurrent access, failure modes, missing data.
5. **Done**: what acceptance criteria show it works?

Make routine calls yourself and state the assumption. Ask only when two readings of the requirement lead to materially different designs.

## Phase 2: Design

### 2a. Architecture decision

The minimality ladder comes first (`~/.claude/CLAUDE.md` §2; canonical file `~/.claude/skills/shared/minimality-ladder.md`). Rungs 1–6 mean there is no architecture decision to make: don't build it, reuse it, or use what exists. The rest of this phase applies only once the ladder reaches rung 7.

Then pick the level of abstraction:

- **Simple function**: no state, no side effects, pure transformation
- **Class**: manages internal state or a lifecycle
- **Protocol + Strategy**: several interchangeable behaviors
- **Interface + adapter**: an external boundary (database, API, file system)
- **Facade or service class**: coordinates several components

### 2b. Pattern selection

Use a pattern only for a problem that exists now, not one that might appear later.

| Problem | Pattern | When |
|---------|---------|------|
| Object creation varies by context | Factory Method | 3+ creation paths, or complex creation logic |
| Algorithm varies by context | Strategy (via Protocol) | 2+ interchangeable behaviors |
| External system boundary | Adapter | Wrapping third-party APIs or infrastructure |
| Complex subsystem access | Facade | Simplifying a multi-step internal workflow |
| Swappable implementations, testing | Dependency Injection | The dependency has a real alternative or needs a test double |

Prefer composition over inheritance, unless you are deliberately extending a protocol or entity hierarchy.

### 2c. Module layout

A single module is the right answer until the package earns more structure. When it does:

```
package/
├── __init__.py          # Public API — re-exports only
├── core/
│   ├── protocols.py     # Protocols and abstract contracts
│   ├── entities.py      # Domain objects (dataclasses / Pydantic)
│   └── exceptions.py    # Domain-specific exceptions
├── services/            # Business logic and orchestration
├── infrastructure/      # External adapters (DB, API clients, file I/O)
└── config.py            # Configuration loading
```

Dependencies flow inward: infrastructure → services → core. Core never imports from infrastructure.

## Phase 3: Implement

### Standards

- Type hints on every public interface, in Python 3.13+ syntax (`str | None`, not `Optional[str]`), or the project's `requires-python` floor if lower.
- Follow the project's ruff and pyright config in `pyproject.toml`.
- Explicit over implicit, flat over nested; use early returns to reduce nesting.
- Keep each function to one job. Past about 30 lines, it usually does more than one.
- Context managers for anything with a lifecycle (files, connections, cursors); generators for large data.
- Application code logs through the project logger or `rich.console`. `print()` is fine in scripts, smoke tests, and examples.
- Google-style docstrings on public APIs, stating purpose only.
- Protocols over ABCs when no shared implementation is needed.
- Put helper functions after the function that calls them.

### Error handling

- Define domain-specific exceptions in the package's exceptions module.
- Catch specific exceptions. Don't catch `Exception` to silence an error.
- Validate at system boundaries: user input, external API responses, file reads.

### Environment and dependencies

- Use the project's `.venv` and whatever tooling `pyproject.toml` already configures. For new projects, `uv` is the default.
- Declare dependencies in `pyproject.toml` with version bounds (`>=2.0.0,<3.0.0`), and keep runtime, dev, and docs groups separate.

## Phase 4: Check

The tooling is the acceptance check: `ruff check` and `pyright` pass on the changed files, and new behavior has passing tests. For tests, use the `write-tests` skill. For a deliberate quality review, use `python-code-review` as its own step.

When you report back, state the design choice and its main tradeoff in a few lines.
