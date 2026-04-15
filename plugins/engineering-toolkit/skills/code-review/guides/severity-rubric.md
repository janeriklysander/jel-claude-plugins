# Severity Rubric

Every finding MUST be assigned exactly one severity level. Use the litmus tests below to decide.

## Blocker

Must fix before merge. The PR should not be approved with unresolved blockers.

Examples:

- Correctness bug that will manifest in production
- Security vulnerability (injection, auth bypass, secrets in code)
- Data loss or corruption risk
- Breaking API change without a migration path
- Race condition in concurrent code

Security-specific guidance: injection vulnerabilities reachable from user input, missing auth checks on sensitive endpoints, and hardcoded secrets are always Blockers.

**Litmus test:** "Will we need a hotfix or incident response if this ships?"

## Suggestion

Should fix. Meaningfully improves quality and is worth the effort now.

Examples:

- Code smell that will cause maintenance burden
- Missing error handling for likely failure scenarios
- Architectural inconsistency with established patterns
- Performance issue in a known hot path
- Missing observability for a critical code path

**Litmus test:** "Will someone need to come back and fix this within 3 months?"

## Nitpick

Minor. Take it or leave it — the reviewer won't block the PR over this.

Examples:

- Style preference not enforced by a linter
- Marginally better variable or function name
- Minor readability tweak
- Slightly more idiomatic approach

**Litmus test:** "Would a reasonable senior engineer disagree about whether this matters?"

## Severity assignment rules

1. **When in doubt, go lower.** A borderline Blocker/Suggestion is a Suggestion. A borderline Suggestion/Nitpick is a Nitpick.
2. **Context matters.** A missing null check in a payment flow is a Blocker. The same check in a debug-only logging path is a Nitpick.
3. **Compound risk escalates.** Three Suggestions in the same function that together create a fragile mess may warrant escalating one to Blocker.
4. **Never inflate severity to get attention.** If it's not a Blocker, don't call it one.
