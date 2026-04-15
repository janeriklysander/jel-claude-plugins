# Finding Format

Every agent must output findings using this exact structure. Consistent formatting enables the coordinator to deduplicate, normalize, and present findings cleanly.

## Single finding format

```
### [SEVERITY] Short description

**File:** `path/to/file.ext:LINE`
**Dimension:** correctness | security | architecture | performance | maintainability | observability | documentation

**What:** One-sentence description of the issue.

**Why:** Why this matters — what can go wrong, what's the impact.

**Suggestion:**

\`\`\`LANG
// concrete fix or pseudocode
\`\`\`

Or a prose description if a code snippet isn't appropriate.
```

## Field rules

- **SEVERITY** must be one of: `BLOCKER`, `SUGGESTION`, `NITPICK` (uppercase in the heading)
- **File** must include the file path and line number (or line range) where the issue exists
- **Dimension** must match the agent's own dimension (e.g., the correctness agent always writes `correctness`)
- **What** is factual and specific — no vague "this could be better"
- **Why** explains impact to the reader: production risk, maintenance cost, or user impact
- **Suggestion** is a concrete fix — code snippet, pseudocode, or actionable prose. Never omit this field.

## Multiple findings

Output findings as a flat list under a top-level heading:

```
## Correctness Findings

### [BLOCKER] Off-by-one in pagination logic
...

### [SUGGESTION] Missing null check on user lookup
...
```

## No findings

If the agent finds no issues in its dimension, output:

```
## [Dimension] Findings

No issues found.
```

This explicit "no issues" output is important — it confirms the agent ran and reviewed the code, rather than silently skipping it.
