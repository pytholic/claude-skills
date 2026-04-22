---
name: python-project-scaffold
description: Scaffold a new Python project with production-ready tooling and structure. Use when the user says "new project", "set up a project", "initialize a Python project", "create a repo", "project template", or asks for boilerplate setup. Generates pyproject.toml, Makefile, pre-commit config, ruff/pyright config, test structure, and directory layout following modern Python conventions with uv, ruff, pyright, and pytest.
---

# Python Project Scaffold

Generate a complete, production-ready Python project structure in one pass.

---

## Core Principle

**EVERY NEW PROJECT STARTS WITH THE SAME SOLID FOUNDATION.**

Do not improvise project setup. Follow this skill to produce a consistent, opinionated scaffold that matches the team's established conventions.

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
2. **Python version** — Default to 3.12+ unless specified
3. **Project type** — Library or application (see triage gate)
4. **Key dependencies** — Any known runtime dependencies
5. **Extras** — Docs (mkdocs), Docker, CI (GitHub Actions), or none

---

## Phase 2: Generate Structure

### Directory Layout

```
project-name/
├── .github/
│   ├── workflows/           # CI pipelines
│   ├── pull_request_template.md
│   └── ISSUE_TEMPLATE/
├── docs/                    # mkdocs source (if docs enabled)
├── package_name/
│   ├── __init__.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── interfaces.py    # Protocols and contracts
│   │   ├── models.py        # Domain models
│   │   └── exceptions.py    # Domain exceptions
│   ├── services/            # Business logic
│   │   └── __init__.py
│   ├── infrastructure/      # External adapters
│   │   └── __init__.py
│   └── config.py            # Configuration loading
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   └── __init__.py
│   ├── integration/
│   │   └── __init__.py
│   └── e2e/                 # Optional
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

### pyproject.toml

```toml
[project]
name = "PROJECT_NAME"
dynamic = ["version"]
description = "PROJECT_DESCRIPTION"
authors = [
    {name = "AUTHOR", email = "EMAIL"},
]
requires-python = ">=3.12.0"
readme = "README.md"
license = {text = "MIT"}
dependencies = []

[dependency-groups]
dev = [
    "pytest>=9.0.0,<10.0.0",
    "pre-commit>=4.0.0,<5.0.0",
    "pytest-cov>=5.0.0,<6.0.0",
    "pytest-mock>=3.14.0,<4.0.0",
    "ruff>=0.11.0,<1.0.0",
]
docs = [
    "mkdocs>=1.6.0,<2.0.0",
    "mkdocs-material>=9.5.0,<10.0.0",
    "mkdocstrings[python]>=1.0.0,<2.0.0",
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
target-version = "py312"
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
pythonVersion = "3.12"
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
3.12
```

### .env.sample

```bash
# Copy to .env and fill in values. DO NOT commit .env to version control.
```

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
