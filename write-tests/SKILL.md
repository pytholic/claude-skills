---
name: write-tests
description: Writes focused pytest tests for critical paths, edge cases, and error handling, without redundant tests. Use when the user asks for tests, including brief asks like "write tests", "add tests for this", or "write important tests", or as the test step of an implement-test-review request.
argument-hint: "[target file or code] [critical|full|integration]"
arguments: [target, focus]
---

# Pytest Test Writing — Focused and Valuable

Write a small set of high-value tests. Skip trivial code, framework behavior, and implementation details.

## Inputs

- **target** ($target): a file path such as `src/module.py`, or code to test. If it's empty, use the files implemented or changed earlier in this session. If several files changed and it's unclear which need coverage, ask.
- **focus** ($focus): `critical` (default, core logic only), `full` (adds edge cases), or `integration`.

## What to test, in priority order

1. Critical business logic: core algorithms, calculations, transformations
2. Public API contracts: input/output behavior, state changes
3. Error handling: exceptions, validation, recovery
4. Edge cases: boundaries, empty or `None`, type coercion
5. Integration points: external dependencies, I/O

## What not to test

- Trivial getters, setters, property access
- Framework or library code
- Private internals and internal state, except where a private helper carries real logic (name those tests `test__<name>`)
- Constants and configuration without logic
- Generated code (dataclasses, ORM models without custom logic)

## Standards

- Name tests `test_<what>_<condition>_<expected>`.
- AAA structure (Arrange, Act, Assert), one logical assertion group per test.
- Python 3.13+ type hints, or the project's `requires-python` floor if lower.
- Parametrize similar cases with `pytest.param(..., id="...")`, never bare tuples:

  ```python
  @pytest.mark.parametrize(
      ("value", "expected"),
      [
          pytest.param(5, 25, id="positive"),
          pytest.param(0, 0, id="zero"),
          pytest.param(-3, 9, id="negative"),
      ],
  )
  def test_square_returns_expected(value: int, expected: int) -> None:
      assert square(value) == expected
  ```

- Mock with pytest-mock's `mocker` fixture (`mocker.patch(...)`, `mocker.patch.object(...)`), never `unittest.mock` directly. Mock only at boundaries; prefer dependency injection.
- Use built-in fixtures where they fit: `tmp_path`, `monkeypatch`, `caplog`, `capsys`.
- Shared setup goes in `conftest.py` fixtures (function scope by default). Never import functions or values from `conftest.py`; use fixtures.
- When the same configured mock appears in 3+ tests, make it a `conftest.py` fixture. Override it locally only where a test needs different behavior.
- When the same literal (a threshold, an ID, an expected code) appears 2+ times, give it a named constant (`SCREAMING_SNAKE_CASE`) or a named local variable. This applies inside `pytest.param` entries too. It prevents silent drift when one occurrence changes and makes the value's meaning explicit.

## Coverage

Coverage is a smell test, not a target. Aim for full coverage of critical logic and error paths, and skip trivial code. Use `# pragma: no cover` sparingly, for unreachable branches. If `pyproject.toml` sets `--cov-fail-under`, meet it; don't invent a threshold.

```bash
pytest --cov=src --cov-report=term-missing
```

## Layout

```
tests/
├── conftest.py          # Shared fixtures only
├── unit/                # Mirror the src/ structure
└── integration/
```

## Deliverable

A focused test file with parametrized cases, error-path coverage for public APIs, and `conftest.py` fixtures only where setup is reused.
