---
name: help
description: Use when the user asks "what can I ask you to do?", "what skills do you have?", "show me a usage guide", or types /help. Presents a user-facing cheat sheet of all personal skills grouped by category.
---

# Skills Usage Guide

Present this guide to the user exactly as formatted below.

---

## Development

| What you want | What to say |
|---|---|
| Architect or implement a Python feature | `"implement X"` / `"architect Y"` / `"refactor this module"` |
| Scaffold a new Python project from scratch | `"new Python project"` / `"set up a project"` / `"create a repo"` |
| Write pytest tests for a file or module | `"write tests for X"` / `"add test coverage to Y"` |

## Code Review & Quality

| What you want | What to say |
|---|---|
| Review Python code changes before a commit or PR | `"review my code"` / `"review the diff"` / `"review before PR"` |

## Debugging

| What you want | What to say |
|---|---|
| Diagnose a bug or failing test systematically | `"debug this"` / `"fix this bug"` / `"why is X failing"` |

## Understanding Code

| What you want | What to say |
|---|---|
| Find where something lives in the codebase | `"where is X?"` / `"find all uses of Y"` / `"how is this structured?"` |
| Understand how code works with a walkthrough | `"explain this code"` / `"how does X work?"` / `"walk me through Y"` |

## Browser & UI

| What you want | What to say |
|---|---|
| Automate browser interactions or test a UI | `"automate this page"` / `"test this UI"` / `"click X on the page"` |

## Session Management

### HAC (Human-Agent Context)

| What you want | What to say |
|---|---|
| Set up `.hac/` working memory on a project | `"set up hac"` / `"add .hac"` / `"initialize working memory"` |
| Create a task in HAC for the current work | `"create a hac task for X"` / `"track this in hac"` |
| Update HAC with progress from this session | `"update hac"` / `"log this session in hac"` |
| Record an architectural decision | `"add a decision to hac"` / `"record this decision in hac: [choice and why]"` |
| Park an idea for later | `"park this idea in hac"` / `"add to parked ideas: X"` |
| Mark a task as done | `"mark X as done in hac"` / `"close the hac task for X"` |
| Mark a task as blocked | `"block the hac task for X"` / `"X is blocked on Y, update hac"` |

### Handover

| What you want | What to say |
|---|---|
| Hand off work to a new session | `"wrap up"` / `"hand off"` / `"let's continue in a new chat"` |
