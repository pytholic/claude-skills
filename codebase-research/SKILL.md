---
name: codebase-research
description: Navigates and maps a codebase to answer where-and-how-organized questions, with ranked, annotated answers instead of raw search output. Use when the user asks where logic lives, where a symbol is defined or used, what imports or depends on a module, whether something is still used, which files handle X, or how the codebase is structured — for example "where is Y defined?", "find all usages of X", "is this used anywhere?", or "give me an overview of the codebase".
---

# Codebase Research

Structured methodology for navigating and understanding codebases of any language or size.

---

## Core Principle

**Search before reading. Open a file only once a search shows it matters.**

Opening files speculatively wastes context and produces noisy answers. Always locate candidates first with search tools, then read only the files confirmed to be relevant.

**Tooling Preference (in order):**
1. `rg` (ripgrep) — fastest, natively respects `.gitignore`
2. `git grep` / `git ls-files` — respects `.gitignore`, available in any git repo
3. `grep -rn` + explicit `--exclude-dir` — fallback only when neither above is available

Never write out long `--exclude-dir` lists when `rg` or `git grep` can replace them.

---

## Triage Gate

Before starting, classify the effort required:

- **Simple lookup** (one symbol, one file, answer obvious from a single search): Run the search, return the annotated result directly.
- **Structural question** (architecture overview, dependency chain, cross-cutting pattern, module responsibility): Use the full phased approach below.

When in doubt, default to the full approach — it takes minutes and prevents incomplete or misleading answers.

---

## Language-Specific Patterns

Identify the project's language/ecosystem and load the relevant reference:

| Language / Stack        | Reference File                               |
|-------------------------|----------------------------------------------|
| Python                  | [references/python.md](references/python.md) |
| JavaScript / TypeScript | [references/js-ts.md](references/js-ts.md)   |

**If the language is not listed** (Go, Rust, Java, C#, etc.), do not stop — infer the standard constructs for that language (e.g., `interface` vs `trait`, `struct` vs `class`, `package` vs `namespace`, `#include` vs `import`) and adapt the generic Phase 2 patterns accordingly.

---

## Phase 1: Orient

Always start here — even for simple lookups, a 30-second orientation prevents searching in the wrong place.

```bash
# 1. Top-level structure and file distribution (git-aware, no noise)
git ls-files | awk -F/ '{print $1}' | sort | uniq -c | sort -rn | head -20

# Fallback if not a git repo:
# find . -maxdepth 2 -type d | grep -vE 'node_modules|\.git|\.venv|dist|build|__pycache__' | sort

# 2. Entrypoints, config, and dependency manifests
git ls-files | grep -iE '^main\.|^app\.|^index\.|^cli\.|Makefile|package\.json|pyproject\.toml|Cargo\.toml|go\.mod|pom\.xml|build\.gradle'
```

Use this map to anchor all subsequent findings before searching.

---

## Phase 2: Classify and Search

Identify the query type below, then apply the matching strategy.
All commands prefer `git grep` / `git ls-files`. Substitute `rg PATTERN` or `grep -rn PATTERN` if not in a git repo.

---

### 2A — File Name Discovery
*"Where are the controllers?" / "Find the user service" / "Which files handle auth?"*

Finding files by name is often faster than searching contents — always try this first for structural questions.

```bash
# Find files matching a name pattern
git ls-files | grep -i 'PATTERN'

# Examples:
git ls-files | grep -iE 'service|controller|route|handler|middleware|repo|repository'
git ls-files | grep -iE 'test|spec'        # All test files
git ls-files | grep -iE '\.config\.'       # Config files
```

---

### 2B — Usage Search
*"Where do we use X?" / "Show all usages of Y" / "Which files call Z?"*

```bash
# Count first — know the scope before enumerating
git grep -l 'PATTERN' | wc -l

# Primary search with line numbers
git grep -n 'PATTERN'

# Narrow to production code (exclude tests/specs)
git grep -n 'PATTERN' | grep -vE '/test|/spec|/mock|_test\.|\.spec\.|\.test\.'
```

**Result ranking** — present findings in this order:
1. Core business/domain logic
2. Configuration and wiring
3. Tests (label them as such — useful for understanding intended behavior)

---

### 2C — Definition Search
*"Where is X defined?" / "Where is class Y?" / "Where is function Z?"*

```bash
# Adapt keywords to the detected language
git grep -n -E '(class|def|function|const|type|interface|struct|trait|fn)\s+SYMBOL\b'

# For exported/public symbols specifically
git grep -n -E 'export.*(class|function|const|type)\s+SYMBOL\b'        # JS/TS
git grep -n -E '^pub (fn|struct|trait|impl)\s+SYMBOL\b'                # Rust
git grep -n -E '^(public|private|protected).*(class|void)\s+SYMBOL\b'  # Java/C#
```

If multiple definitions appear (e.g., abstract base + concrete implementation), surface all of them — this split is architecturally meaningful.

---

### 2D — Dependency Trace
*"What imports X?" / "What depends on Y?" / "What does Z import?"*

```bash
# Who imports this module or symbol? (reverse — who depends on it)
git grep -n -E '(import|from|require|use|#include).*MODULE_OR_SYMBOL'

# What does this file depend on? (forward — what it pulls in)
head -50 path/to/file | grep -E '^import|^from|^require|^use |^#include|^using '
```

**Aliasing — critical step:** Scan results for renamed imports:
- Python: `from module import Foo as Bar` → track `Bar`
- JS/TS: `import { Foo as Bar }` or `import * as Bar` → track `Bar`
- Go: `import alias "package/path"` → track `alias`

If an alias is found, immediately run a secondary Usage Search (2B) for the alias name. Missing aliases is the most common cause of incomplete dependency traces.

---

### 2E — Module Overview
*"What does this module do?" / "Explain the X/ folder"*

```bash
# 1. Files in the module
git ls-files -- 'module_path/*'

# 2. Read the public API declaration first (the entry point declares intent)
cat module_path/__init__.py  2>/dev/null  # Python
cat module_path/index.ts     2>/dev/null  # TypeScript
cat module_path/index.js     2>/dev/null  # JavaScript
cat module_path/mod.rs       2>/dev/null  # Rust
cat module_path/lib.go       2>/dev/null  # Go

# 3. Skim all top-level definitions in the module (no bodies)
git grep -n -E '^(class|def|function|export|pub fn|func|type|interface|struct)\s+' -- 'module_path/*'

# 4. Find the most central file (largest = most logic)
git ls-files -- 'module_path/*' | xargs wc -l 2>/dev/null | sort -rn | head -5
```

Summarize as: **Purpose → Public API → Key classes/functions → How it connects outward**.

---

### 2F — Architecture Map
*"How is the codebase structured?" / "Give me an overview" / "What are the main components?"*

```bash
# 1. All module entry points
git ls-files | grep -E '__init__\.py$|/index\.ts$|/index\.js$|/mod\.rs$'

# 2. Entrypoints (main execution)
git grep -l -E 'if __name__.*__main__|ReactDOM\.render|createRoot|app\.listen|fn main\(\)|func main\(\)'

# 3. Contract layer — abstract bases, interfaces, protocols, traits
git grep -n -E 'class.*\b(ABC|Protocol)\b|^interface |^abstract class |^trait |^type.*interface\b'

# 4. Wiring and dependency injection files
git ls-files | grep -iE 'dependencies|container|config|settings|providers|module|wiring'
```

Present findings as a conceptual layer diagram:
- **Entry layer** — CLI, HTTP handlers, event listeners, main executables
- **Domain / service layer** — business logic, use cases, application services
- **Infrastructure layer** — DB, external APIs, storage, queues, caches
- **Shared / cross-cutting** — config, utils, base classes, types, constants

---

### 2G — Dead Code Check
*"Is X still used?" / "Can I delete Y?"*

```bash
# Step 1: Confirm definition exists
git grep -n -E '(def|class|function|const|type)\s+SYMBOL\b'

# Step 2: All usages excluding the definition line
git grep -n '\bSYMBOL\b' | grep -vE '(def|class|function|const)\s+SYMBOL'

# Step 3: Check if re-exported in the module's public API
git grep -n 'SYMBOL' $(git ls-files | grep -E '__init__\.py$|index\.(ts|js)$')

# Step 4: Test-only callers (a symbol only called from tests is de-facto unused in production)
git grep -n '\bSYMBOL\b' | grep -E '/test|/spec|_test\.|\.spec\.|\.test\.'
```

Return a clear verdict: **Used** / **Unused** / **Uncertain** — with explicit reasoning for each.

---

## Phase 3: Synthesize and Present

**Never output raw search results.** Always annotate and synthesize.

### Output Format

```
## Finding: [one-line direct answer to the question]

### Locations
- `path/to/file:42` — [what is happening here and why it matters]
- `path/to/other:87` — [what is happening here]

### Context
[1–3 sentences on what this tells us about the design or architecture]

### Notes  ← only include if genuinely useful
[Caveats, aliases spotted, related symbols, patterns worth knowing]
```

**Rules:**
- Lead with the answer, not the search process
- File paths must be relative and include line numbers (`path/to/file:42`)
- Every location gets an annotation — no bare paths or line numbers
- If more than ~8 results, group them: "5 usages in `api/routes/`, 2 in `services/`, 1 in tests"
- If nothing found, say so immediately and suggest likely alternatives (typo? aliased import? different module?)

- If a search misses, narrow it with better flags or a different pattern rather than rerunning near-identical wording.
