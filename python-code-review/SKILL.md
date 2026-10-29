---
name: python-code-review
description: Reviews Python changes for correctness, security, simplicity, and my coding conventions, and gives a verdict. Use when the user asks for a review of a diff, branch, PR, or files, including quick or casual asks ("quick review", "check this over", "does this look OK"), or as the review step of an implement-test-review request. Targets Python 3.13+ unless the project's pyproject.toml pins a lower requires-python.
context: fork
agent: general-purpose
allowed-tools: Read, Grep, Glob, Bash
---

# Python Code Review

Review recent Python changes against the way I write code: get the job done, use modern best practices, keep it simple. Correctness comes first, then simplicity, then conventions.

## Step 1: Gather context

1. Find the change: `git diff` (unstaged), `git diff --cached` (staged), or `git diff <base>...HEAD` for a branch. If the user named files or a PR, review those instead.
2. Read the project's `CLAUDE.md` and `pyproject.toml`: ruff and pyright config, test config, and the `requires-python` floor.
3. Read each changed file in full, plus the callers of any changed public interface.
4. Run the tools on the changed files and use their real output:
   - `ruff check <files>` and `ruff format --check <files>`
   - `pyright <files>`, or the project's configured type checker. Ignore stub warnings for external dependencies.
   - The tests that cover the change, when they run quickly.

   If a tool isn't installed or configured, say so and inspect manually.

## Step 2: Review, in priority order

### Correctness

- Logic errors, off-by-one, wrong comparisons, missing return paths
- `if`/`elif`/`match` fall-through branches that are correct only by accident
- Errors at boundaries: unhandled exceptions, swallowed exceptions (`except Exception: pass`, bare `except:`, errors reduced to a log line), resources not closed on the error path
- `None` and missing data where data enters the system
- A public interface that doesn't match its Protocol/ABC or its callers
- Async: unawaited coroutines, blocking calls inside the event loop, state shared across tasks
- Mutable default arguments and mutable class attributes shared across instances

### Security

- Injection (SQL, shell, template)
- Secrets or credentials in code or config
- Missing validation of user input or external API responses
- Overly broad access or missing auth checks

### Simplicity

Judge every added abstraction, parameter, and config option against the minimality ladder in `~/.claude/CLAUDE.md` §2. If it isn't in your context, read `~/.claude/skills/shared/minimality-ladder.md`. Name the rung that should have caught it. The rung name is the finding, and it keeps review and implementation speaking one language. Never flag the never-traded-away items as over-engineering.

Also flag:
- Code more complex than it needs to be: redundant branches, needless intermediate variables, reimplemented stdlib, deep nesting. Give the simpler form in one line.
- A design pattern with no concrete problem it solves.
- Inheritance where composition works, unless the change deliberately extends a protocol or entity hierarchy.
- Logic duplicated at 3+ sites. Don't flag intentional similarity in tests.

### Modern Python

- Type hints on public interfaces, in 3.13+ syntax (`str | None`, `list[int]`) or the project's `requires-python` floor
- Context managers for anything with a lifecycle; generators for large data
- Protocols over ABCs when no shared implementation is needed
- A function that does more than one thing. Past about 30 lines, decomposition is usually due; report it as a suggestion, not a defect.
- Application code logs through the project logger or `rich.console`. `print()` is fine in scripts, smoke tests, and examples.

### My conventions

Check the changed code against the Coding Conventions section of `~/.claude/CLAUDE.md`: object-creation prefixes, file roles and names, helper order, docstring and comment rules, and test naming. If that section isn't in your context, read it.

### Tests

- New or changed behavior has tests. Doc-only changes don't need them unless fixtures or examples change.
- The tests would fail without the change, and they cover the happy path, edge cases (empty, `None`, zero, negative, single item), and error paths.
- `pytest.param(..., id=...)` for parametrize, never bare tuples. AAA structure. `mocker` from pytest-mock, mocks only at boundaries.
- Shared setup lives in `conftest.py` fixtures and is never imported. Private-method tests are named `test__<name>`.

### Comments and docstrings (changed lines only)

- Google-style docstrings on public APIs that state purpose only. One line when the signature is self-explanatory. Class docstrings go on the class, not `__init__`.
- Flag comments and docstrings that are stale, redundant, or misleading, or that explain rationale, cross-reference other functions or pipeline stages, or name concrete classes in a module docstring. Suggest the shorter version.
- Don't ask for comments on code that already reads clearly.

## Step 3: Report

```
## Review Summary

**Scope**: [files reviewed, lines changed]
**Verdict**: APPROVE | REQUEST_CHANGES | COMMENT

## Critical (must fix)
- [file:line] Issue → Suggested fix

## Warnings (should fix)
- [file:line] Issue → Suggested fix

## Suggestions (consider)
- [file:line] Improvement

## What's Good
- 1–3 short points
```

## Rules

- Review every changed file, and only the changed code. Mention pre-existing issues only when the change makes them worse.
- Report everything that could cause incorrect behavior, a test failure, a security issue, or a misleading result, and let the tiers do the filtering. Leave out formatting that ruff fixes and pure preference.
- Be specific: exact file and line, the problematic code, and the fix.
- If nothing is wrong, say so.
