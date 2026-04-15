# Review Principles

## Core philosophy

**Leave the codebase better than you found it.**

Every finding must include a concrete fix suggestion. "This is bad" is not a review comment — "This is bad, here's how to fix it" is.

## What a good review looks like

1. **Finds real problems** — bugs that will bite someone, not theoretical concerns
2. **Teaches** — explains WHY something is an issue, not just WHAT to change
3. **Respects the author** — assumes competence, asks before assuming intent
4. **Stays in scope** — reviews the diff, not the entire codebase history
5. **Prioritizes** — clearly separates must-fix from nice-to-have

## Descoping rules

Sometimes a finding is valid but out of scope for the current PR. Descoping is acceptable when:

- The issue predates this PR and the PR doesn't make it worse
- Fixing it would require changes significantly beyond the PR's scope
- The issue is a Suggestion or Nitpick in an area the PR only touches incidentally

Descoping is NOT acceptable when:

- The finding is a Blocker — blockers must be fixed before merge, period
- The PR introduces the issue — you broke it, you fix it
- The fix is trivial (< 5 minutes of work)

**Every descoped item MUST have a tracked follow-up issue.** No silent descoping. The issue must include enough context for someone else to pick it up without re-doing the review.

## Anti-patterns to avoid

- **Bike-shedding** — spending review time on formatting when there are logic bugs
- **Severity inflation** — calling everything a Blocker to force changes
- **Rubber stamping** — approving without meaningful review
- **Scope creep** — demanding refactors unrelated to the PR's purpose
- **Premature optimization flags** — only flag performance issues that are real bottlenecks or algorithmic mistakes, not hypothetical slowness
- **Duplicate findings** — if two agents find the same issue, keep the more specific one
