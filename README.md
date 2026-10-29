# claude-skills

Personal Claude Code skill library — specialized workflows for Python development, debugging, code review, planning, and more.

## Install

First, add the marketplace (one-time setup):

```
/plugin marketplace add pytholic/claude-skills
```

Then install all skills at once:

```
/plugin install all@pytholic-skills
```

Or install a single skill:

```
/plugin install python-dev@pytholic-skills
```

Replace `python-dev` with any skill name from the table below.

### Update
```
# 1. Refresh the marketplace to pick up new versions (one-time per update)
/plugin marketplace update pytholic-skills

# 2. Reinstall to pull the latest version of the skill
/plugin install hac-init@pytholic-skills
```

## Available skills

| Skill | Description |
|---|---|
| `python-code-review` | Review of Python changes for correctness, security, simplicity, and my conventions, with a verdict |
| `codebase-research` | Navigate and map codebases — find symbols, trace dependencies, answer architecture questions |
| `explain-code` | Step-by-step walkthroughs with analogies and diagrams, depth matched to the question |
| `notion-cli` | Notion pages and databases via the `ntn` CLI or the Notion MCP, whichever is cheaper for the job |
| `playwright-cli` | Browser automation and Playwright test workflows |
| `python-dev` | Planning-first Python development with Python 3.13+, ruff, pyright, pytest, pyproject.toml |
| `systematic-debugging` | Four-phase debugging: reproduce → isolate → root cause → fix |
| `task-handover` | Structured HANDOVER.md for cold-start session resumption |
| `write-tests` | Focused pytest coverage for critical paths, edge cases, and error handling |
| `hac-init` | Initialize a Human-Agent Context (HAC) directory with templates and bootstrapping process |
| `python-project-scaffold` | Scaffold a new Python project with production-ready tooling and structure |
| `help` | User-facing cheat sheet of all personal skills — invoke with `/help` |
| `goal-workflow` | Implement a spec fully, check each requirement off, and end with a requirement-by-requirement report |
| `interview-me` | One-question-at-a-time intent extraction for underspecified asks, until ~95% confidence |
| `pr-description` | Generate a minimal, clear PR description from the branch diff and the repo's PR template |
| `task-scope-and-spec` | Scope a loose engineering task into a tight, testable spec list. |
| `planning-and-task-decomposition` | Decompose a scoped goal into an ordered, verifiable implementation plan with checkpoints |
| `guided-implementation` | Guide you through writing the code yourself, one piece per turn — human-driven counterpart to `goal-workflow` |

## Minimality ladder (lives in CLAUDE.md)

The minimality ladder is the single test for "is this code warranted?". Its one copy is
[`shared/minimality-ladder.md`](shared/minimality-ladder.md), adapted from
[ponytail](https://github.com/DietrichGebert/ponytail) (MIT).

The skills don't carry the ladder. They point to it, and `~/.claude/CLAUDE.md` §2 imports
it with `@~/.claude/skills/shared/minimality-ladder.md`, so it loads in every session.
Edit `shared/minimality-ladder.md` only; nothing needs syncing.

Skills that use it: `python-dev` (Phase 2a), `python-code-review` (Simplicity), and
`guided-implementation` (turn contract).

**Installing these skills without my `~/.claude` setup?** Those three skills expect the
ladder in your `CLAUDE.md`. Copy `shared/minimality-ladder.md` into your own `CLAUDE.md`,
or import it with an `@path` line pointing at this file.

## Repo layout

```
<skill-name>/SKILL.md   one directory per skill
shared/                 the minimality ladder, imported by ~/.claude/CLAUDE.md
```

## Usage

Once installed, invoke a skill in Claude Code:

```
/python-dev
/systematic-debugging
/python-code-review
```

Each skill's `SKILL.md` contains the full workflow and instructions Claude follows.
