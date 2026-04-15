# Follow-up Issue Template

Use this template when creating issues for descoped review findings.

## GitHub Issues format

**Title:** `[code-review] SHORT_DESCRIPTION`

**Body:**

```markdown
## Context

Found during code review of PR #NUMBER (or branch `BRANCH_NAME`).

**Descoped because:** JUSTIFICATION — why this is acceptable to defer.

## Finding

**Severity:** SUGGESTION | NITPICK
**Dimension:** correctness | security | architecture | performance | maintainability | observability | documentation
**File(s):** `path/to/file.ext:LINE`

DESCRIPTION — what the issue is and why it matters.

## Suggested fix

CONCRETE_FIX — code snippet, pseudocode, or actionable prose.

## Acceptance criteria

- [ ] SPECIFIC_CONDITION that indicates the issue is resolved
```

**Labels:** `code-review-follow-up` + dimension label (e.g., `security`, `performance`)

**No auto-assignment.** The team can triage and assign during normal prioritization.

## Adapting for other trackers

The template above targets GitHub Issues. When using a different tracker:

- **Linear:** use the same structure in the issue description; map labels to Linear labels or project tags
- **Jira:** use the description field; map labels to Jira labels or components
- **Other:** preserve the structure as closely as the tracker allows

## Graceful fallback

If the tracker CLI is unavailable (e.g., `gh` not installed, not authenticated), output the issue content in a copyable code block so the user can create it manually.
