---
name: python-code-review
description: Reviews Python code changes for quality, correctness, and adherence to project conventions before commits or PRs. Checks for SOLID violations, DRY/YAGNI issues, missing tests, type safety, security concerns, and style compliance. Use when reviewing diffs, preparing PRs, after implementing features, or when the user asks to review code. Python-specific — covers ruff, pyright, pytest, and modern Python 3.12+ idioms.
---

# Python Code Review

Thorough code review of recent Python changes. Focuses on what matters: correctness, design, and maintainability.

## Step 1: Gather Context

1. Run `git diff` to see unstaged changes (or `git diff --cached` for staged, or `git diff main...HEAD` for full branch diff)
2. Read the project's `CLAUDE.md` and `pyproject.toml` for conventions (ruff rules, pyright config, test config)
3. Read each changed file in full to understand surrounding context

## Step 2: Review (in priority order)

### 2a. Correctness

- Logic errors, off-by-one, wrong comparisons, missing return paths
- Race conditions, resource leaks, unhandled exceptions at boundaries
- Null/None handling — does the code handle missing data at system boundaries?
- API contract violations — does the public interface match its protocol/ABC?

### 2b. Security (OWASP-aware)

- Injection risks (SQL, command, template)
- Secrets or credentials in code or config
- Input validation at system boundaries (user input, external APIs)
- Overly permissive access or missing auth checks

### 2c. Design Principles

**SOLID:**
- **S** — Does each class/module have one reason to change?
- **O** — Can behavior be extended without modifying existing code?
- **L** — Do subtypes honor the contracts of their base types?
- **I** — Are interfaces focused, or do they force unused dependencies?
- **D** — Do high-level modules depend on abstractions, not concretions?

**DRY** — Flag duplicated logic (3+ occurrences). Do NOT flag intentional similarity in tests.

**YAGNI** — Flag speculative abstractions, unused parameters, configurability nobody asked for.

**KISS** — Flag unnecessary complexity. If 50 lines can replace 200, say so.

### 2d. Python Quality

- Type hints on all public interfaces (Python 3.12+ syntax: `str | None`, not `Optional[str]`)
- Functions under 30 lines; if longer, suggest decomposition
- Composition over inheritance
- Context managers for resources, generators for memory efficiency
- No `print()` statements (use logging)
- Google-style docstrings on public APIs
- Protocol pattern over ABCs where appropriate

### 2e. Style & Linting

- Follows project ruff rules (check `pyproject.toml` for `[tool.ruff]` config)
- Line length limit (typically 100 chars)
- Import ordering (isort via ruff `I` rule)
- Consistent naming conventions
- No dead code introduced (unused imports, variables, functions)

### 2f. Test Quality

- New behavior has corresponding tests
- Tests follow Red-Green-TDD: does the test actually fail without the implementation?
- AAA pattern (Arrange-Act-Assert)
- Uses `pytest.param()` with `id` for parametrized tests (never bare tuples)
- Tests cover: happy path, edge cases, error paths
- No testing of framework behavior or trivial getters
- Mocks only at boundaries; prefer dependency injection

## Step 3: Report

Output a structured review using this format:

```
## Review Summary

**Scope**: [files reviewed, lines changed]
**Verdict**: APPROVE | REQUEST_CHANGES | COMMENT

## Critical (must fix)
- [file:line] Issue description → Suggested fix

## Warnings (should fix)
- [file:line] Issue description → Suggested fix

## Suggestions (consider)
- [file:line] Improvement idea

## What's Good
- Brief note on well-done aspects (keep to 1-3 points)
```

## Rules

- Review ONLY changed code. Do not critique pre-existing code unless the change makes it worse.
- Be specific: reference exact file and line, show the problematic code, show the fix.
- Do not nitpick formatting that ruff would auto-fix.
- Do not suggest adding comments/docstrings to code you didn't write.
- If no issues found, say so. Don't invent problems.
- For the checklist, see [checklist.md](checklist.md).
