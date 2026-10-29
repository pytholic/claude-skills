---
name: goal-workflow
description: Implements a spec fully, checks each requirement off against the implementation, and ends with a requirement-by-requirement report. Use when the user runs /goal-workflow, asks to "implement the spec" or "verify against the spec", or wants to go from a requirement list to a verified implementation in one pass.
---

You are running the goal-workflow skill. Your job: implement a spec fully, check each requirement off, report results.

## Step 1 — Get the spec

Check `args` (the text after `/goal-workflow`):
- **File path** (starts with `/`, `./`, `~/`, or ends with `.md`/`.txt`/`.yaml` etc.) → read the file
- **Inline text** → use it directly
- **Nothing** → ask: "Paste your spec or give me a file path."

## Step 2 — Set the goal

Parse the spec into a numbered list of discrete, verifiable requirements. State out loud:

> **Goal:** Implement all N requirements below. Done when every item is verified complete.
> 1. …
> 2. …

If the spec is ambiguous on any point, ask before starting — not mid-implementation.

## Step 3 — Implement

Work through each requirement in order. Check it off as you go. Keep changes focused; don't add anything not in the spec.

## Step 4 — Check the spec off

Walk the numbered requirements and confirm each one is satisfied, citing the file, function,
or test that satisfies it. Run the project's tests, type checker, and linter — those are the
acceptance check. Do this yourself in the main context; don't spawn an agent per requirement.

**Exception — large specs.** For a spec with roughly a dozen or more requirements that are
genuinely independent of one another, fanning out is worth the cost. Only then, use the
`Workflow` tool below. Keep the agent count to one per requirement, never more.

```js
export const meta = {
  name: 'spec-verify',
  description: 'Verify each spec requirement against the implementation',
  phases: [{ title: 'Verify' }],
}

const REQS = args  // array of requirement strings passed via args

const VERDICT = {
  type: 'object',
  properties: {
    satisfied: { type: 'boolean' },
    evidence: { type: 'string' },
  },
  required: ['satisfied', 'evidence'],
}

const results = await parallel(REQS.map((req, i) => () =>
  agent(
    `Requirement: "${req}"\n\nCheck whether this requirement is fully satisfied in the current code/output. Be concrete — cite file names, function names, or output values as evidence.`,
    { label: `verify:${i}`, phase: 'Verify', schema: VERDICT }
  ).then(v => ({ req, ...v }))
))

return results.filter(Boolean)
```

Pass `args` as the array of requirement strings.

## Step 5 — Report

Print a markdown table:

```
## Implementation Report

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | …           | ✅     | …        |
| 2 | …           | ❌     | …        |

**Result: X / Y requirements satisfied.**
```

If anything failed, list what's missing and ask: "Fix the gaps, or done for now?"
