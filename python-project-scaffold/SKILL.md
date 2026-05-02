---
name: python-project-scaffold
description: Scaffold a new Python project with production-ready tooling and structure. Use when the user says "new project", "set up a project", "initialize a Python project", "create a repo", "project template", or asks for boilerplate setup. Generates pyproject.toml, Makefile, pre-commit config, ruff/pyright config, test structure, working memory (.ai/), and directory layout following modern Python conventions with uv, ruff, pyright, and pytest.
---

# Python Project Scaffold

Generate a complete, production-ready Python project structure in one pass.

---

## Core Principle

**EVERY NEW PROJECT STARTS WITH THE SAME SOLID FOUNDATION.**

Do not improvise project setup. Follow this skill to produce a consistent, opinionated scaffold that matches the team's established conventions.

---

## Working Memory (.ai/ directory)

The `.ai/` directory is a shared working memory layer between the human and the agent. It solves the "where were we?" problem — when you return to a project after days or weeks, you read `.ai/status.md` instead of reconstructing state from memory or git history.

Three files, three concepts:

- **`status.md`** — The dashboard. What's active, blocked, and done. First thing read at session start. Includes a one-line project description so the agent has immediate context without opening other files.
- **`decisions.md`** — Lightweight architecture decision records. Why we chose X over Y, and crucially, what we rejected and why. Prevents re-litigating past choices. Append-only — never edit or remove past entries.
- **`tasks/<name>.md`** — One scratchpad per non-trivial task. Contains context, phased plan, notes/findings discovered during work, and open questions/risks.

The `.ai/` directory is always tracked in git — it contains project knowledge that should persist across contributors and sessions. Task files are created on-demand during development; only `status.md`, `decisions.md`, and an empty `tasks/` folder are generated at scaffold time.

For the full protocol on how the agent reads, updates, and maintains these files during sessions, see Section 7 of the global `~/.claude/CLAUDE.md`.

---

## Triage Gate

Before scaffolding, determine the project type:

- **Library / package** — Installable module, published or internal. Uses `[build-system]`, has `src/` or top-level package directory.
- **Application / service** — Runnable entrypoint (CLI, API server, pipeline). Has `main.py` or equivalent.
- **Script / experiment** — Single-file or small-scale. Does NOT need this skill — a simple `.py` file with a `pyproject.toml` for deps is enough.

Ask which type if unclear. The scaffold below covers library and application.

---

## Phase 1: Gather Requirements

Before generating any files, confirm:

1. **Project name** — kebab-case for the repo, snake_case for the Python package
2. **Python version** — Default to 3.13+ unless specified
3. **Project type** — Library or application (see triage gate)
4. **Key dependencies** — Any known runtime dependencies
5. **Extras** — Docs (mkdocs), Docker, CI (GitHub Actions), or none

---

## Phase 2: Generate Structure

### Directory Layout

```
project-name/
├── .ai/                         # Working memory (agent + human shared state)
│   ├── status.md                # Dashboard: active/blocked/done overview
│   ├── decisions.md             # Lightweight ADRs: why we chose X over Y
│   └── tasks/                   # One file per non-trivial task
│       └── .gitkeep
├── .github/
│   ├── workflows/               # CI pipelines
│   ├── pull_request_template.md
│   └── ISSUE_TEMPLATE/
├── docs/                        # mkdocs source (if docs enabled)
├── package_name/
│   ├── __init__.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── interfaces.py        # Protocols and contracts
│   │   ├── models.py            # Domain models
│   │   └── exceptions.py        # Domain exceptions
│   ├── services/                # Business logic
│   │   └── __init__.py
│   ├── infrastructure/          # External adapters
│   │   └── __init__.py
│   └── config.py                # Configuration loading
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   └── __init__.py
│   ├── integration/
│   │   └── __init__.py
│   └── e2e/                     # Optional
│       └── __init__.py
├── .env.sample
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── .yamllint
├── CLAUDE.md
├── CONTRIBUTING.md
├── Makefile
├── README.md
└── pyproject.toml
```

For applications, also generate `main.py` at the root.

---

## Phase 3: File Templates

### .ai/status.md

```markdown
# Project Status

> PROJECT_NAME — PROJECT_DESCRIPTION

## Active

_No active tasks yet._

## Blocked

_Nothing blocked._

## Completed

_No completed tasks yet._
```

### .ai/decisions.md

```markdown
# Architectural Decisions

Append new decisions at the top. Each entry records context, the choice made,
the reasoning, and what was explicitly rejected.

<!-- Template:
## YYYY-MM-DD: Decision Title
- **Context:** What situation or constraint prompted this decision?
- **Choice:** What did we decide?
- **Why:** Why this option over others?
- **Rejected:** What alternatives were considered and why not?
-->
```

### .ai/tasks/ (template reference)

Individual task files are created on-demand, not at scaffold time. The standard
template for a task file (`.ai/tasks/<task-name>.md`):

```markdown
# Task: [Task Name]

**Status:** Not Started | In Progress | Blocked | Done (YYYY-MM-DD)
**Created:** YYYY-MM-DD
**Updated:** YYYY-MM-DD

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
```

### CLAUDE.md

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What This Is

PROJECT_NAME — PROJECT_DESCRIPTION

## Commands

```bash
# Setup
make setup                          # Create .venv and install all deps

# Testing
make test                           # Run all tests
make test-unit                      # Unit tests only
make test-cov                       # Tests with coverage report
uv run pytest tests/unit/test_x.py  # Single file
uv run pytest -k "test_name"        # By name pattern

# Code quality
make lint                           # Check with ruff
make format                         # Auto-format with ruff
```

## Architecture

_Describe the high-level architecture here once it stabilizes._

## Conventions

- Python 3.13+, type hints everywhere (pyright strict)
- Google-style docstrings
- ruff for linting + formatting (line-length 100)
- uv for package management
- Tests: pytest with unit/integration separation
```

### pyproject.toml

```toml
[project]
name = "PROJECT_NAME"
dynamic = ["version"]
description = "PROJECT_DESCRIPTION"
authors = [
    {name = "AUTHOR", email = "EMAIL"},
]
requires-python = ">=3.13.0"
readme = "README.md"
license = {text = "MIT"}
dependencies = []

[dependency-groups]
dev = [
    "pytest",
    "pre-commit",
    "pytest-cov",
    "pytest-mock",
    "ruff",
]
docs = [
    "mkdocs",
    "mkdocs-material",
    "mkdocstrings[python]",
]

[build-system]
requires = ["hatchling", "hatch-vcs"]
build-backend = "hatchling.build"

[tool.hatch.build]
packages = ["PACKAGE_NAME"]

[tool.hatch.version]
source = "vcs"

[tool.hatch.build.hooks.vcs]
version-file = "PACKAGE_NAME/_version.py"

[tool.uv]
package = true

[tool.ruff]
line-length = 100
target-version = "py313"
exclude = ["__pycache__"]

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "Q",    # flake8-quotes
    "A",    # prevent clobbering python builtins
    "ISC",  # implicit string concatenation
    "UP",   # pyupgrade
    "RUF",  # ruff-specific rules
    "D",    # pydocstyle
    "I",    # isort
    "ANN",  # flake8-annotations
    "T20",  # flake8-print (no print statements)
]
ignore = [
    "E501",
    "A001",
    "D100", "D101", "D102", "D103", "D104", "D105",
    "D205", "D209", "D210", "D400", "D401", "D404",
]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["ANN401", "ANN201", "ANN202", "ANN001"]
"main.py" = ["T20"]

[tool.ruff.format]
docstring-code-format = true
docstring-code-line-length = 100

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.pytest.ini_options]
testpaths = ["tests/"]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "e2e: marks end-to-end tests (deselect with '-m \"not e2e\"')",
]
addopts = ["--durations=0", "--strict-markers", "--doctest-modules", "--ignore=tests/e2e"]
filterwarnings = ["ignore::DeprecationWarning", "ignore::UserWarning"]

[tool.coverage.run]
omit = [
    "PACKAGE_NAME/config.py",
    "PACKAGE_NAME/logging.py",
]

[tool.pyright]
include = ["PACKAGE_NAME"]
pythonVersion = "3.13"
typeCheckingMode = "strict"
reportMissingTypeArgument = "warning"
reportUnnecessaryCast = "warning"
reportUnusedIgnoreComment = "warning"

[[tool.pyright.executionEnvironments]]
root = "tests"
typeCheckingMode = "basic"
```

### Makefile

```makefile
SHELL := bash

.PHONY: setup install install-dev test test-cov test-unit test-integration lint format clean help

setup:
	@echo "Setting up environment..."
	uv sync --group dev --group docs
	@echo "Done. Activate with: source .venv/bin/activate"

install:
	uv sync

install-dev:
	uv sync --group dev

test:
	uv run pytest -x --tb=no -rs

test-cov:
	uv run pytest -x --cov=PACKAGE_NAME --cov-report=term-missing

test-unit:
	uv run pytest -v --tb=no -ra tests/unit/

test-integration:
	uv run pytest -v --tb=no -ra tests/integration/

lint:
	uv run ruff check .

format:
	uv run ruff format .

clean:
	@find . -type f -name ".DS_Store" -delete
	@find . -type d -name "__pycache__" -exec rm -rf {} +
	@find . -type d -name ".pytest_cache" -exec rm -rf {} +
	@find . -type d -name ".ruff_cache" -exec rm -rf {} +
	@find . -type d -name ".mypy_cache" -exec rm -rf {} +
	@find . -type f -name "*.pyc" -delete
	@echo "Clean complete."

help:
	@echo "Commands: setup install install-dev test test-cov test-unit test-integration lint format clean"
```

### .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: debug-statements
      - id: check-docstring-first
      - id: check-yaml
        args: ["--unsafe"]
      - id: check-toml
      - id: check-executables-have-shebangs
      - id: check-case-conflict
      - id: check-added-large-files
        args: ["--maxkb=10000"]
      - id: no-commit-to-branch
        args: ["--branch", "main", "--branch", "develop"]

  - repo: https://github.com/econchick/interrogate
    rev: 1.5.0
    hooks:
      - id: interrogate
        args: [--config=pyproject.toml]
        pass_filenames: false
        additional_dependencies: [setuptools]

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.13
    hooks:
      - id: ruff
        args: [--fix, --config, pyproject.toml]
      - id: ruff-format
        args: [--config, pyproject.toml]

  - repo: https://github.com/executablebooks/mdformat
    rev: 0.7.21
    hooks:
      - id: mdformat
        args: ["--number"]
        exclude: "^docs/"
        additional_dependencies:
          - mdformat-gfm
          - mdformat-tables
          - mdformat_frontmatter

  - repo: https://github.com/codespell-project/codespell
    rev: v2.1.0
    hooks:
      - id: codespell
        args: ["--skip=*.lock,docs/site/assets/*"]

  - repo: https://github.com/adrienverge/yamllint
    rev: v1.29.0
    hooks:
      - id: yamllint
        args: ["-c=.yamllint"]
```

### .yamllint

```yaml
yaml-files:
  - "*.yaml"
  - "*.yml"
  - ".yamllint"
extends: default
rules:
  line-length:
    max: 120
  document-start: disable
  truthy: disable
```

### .python-version

```
3.13
```

### .env.sample

```bash
# Copy to .env and fill in values. DO NOT commit .env to version control.
```

### .gitignore (append these lines for .ai/)

Ensure `.ai/` is tracked in git (it should be). Task files contain project
knowledge that should persist across contributors. Do NOT gitignore `.ai/`.

---

## Phase 4: Post-Scaffold Steps

After generating files, run:

```bash
# 1. Initialize git
git init && git add . && git commit -m "chore: initial project scaffold"

# 2. Set up environment
make setup

# 3. Install pre-commit hooks
uv run pre-commit install

# 4. Verify everything works
make lint
make test
```

Report any failures and fix before declaring scaffold complete.

---

## Customization Notes

- **Replace all placeholders**: `PROJECT_NAME`, `PACKAGE_NAME`, `PROJECT_DESCRIPTION`, `AUTHOR`, `EMAIL`
- **Adapt ruff rules**: Add or remove lint rules based on project needs. The default set is opinionated but covers the most common issues.
- **Docs are optional**: Only generate `docs/` and `mkdocs.yml` if the user requests documentation.
- **CI is optional**: Only generate `.github/workflows/` if the user requests CI setup.
- **e2e tests are optional**: Only create `tests/e2e/` if the project has external dependencies worth testing end-to-end.
- **.ai/ is always created**: The working memory directory is part of every non-trivial project. Task files are created on-demand during development, not at scaffold time.

---

## Process Violations — Red Flags

Stop and correct if you catch yourself doing any of the following:

- Generating files without confirming project name and type
- Skipping pyright or ruff config in `pyproject.toml`
- Using `requirements.txt` instead of `pyproject.toml` with dependency groups
- Creating a `setup.py` or `setup.cfg` instead of `pyproject.toml`
- Omitting `.pre-commit-config.yaml`
- Using `pip` or `poetry` instead of `uv`
- Putting test configuration in a separate `pytest.ini` instead of `pyproject.toml`
- Generating a flat `tests/` directory without `unit/` and `integration/` separation
- Skipping the `.ai/` working memory directory
