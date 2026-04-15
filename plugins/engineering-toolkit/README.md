# engineering-toolkit

A combined code review and documentation plugin for Claude Code. It dispatches specialist agents to review your changes across 7 dimensions, normalizes findings by severity, creates follow-up issues for anything descoped, and guides you through writing any type of technical document.

## Prerequisites

Install the plugin from a Claude Code session:

```text
/plugin install engineering-toolkit@jel-claude-plugins
```

Optionally, install the [GitHub CLI](https://cli.github.com/) (`gh`) for PR diffing and follow-up issue creation. The plugin works without it but falls back to manual workflows.

## Skills

### `/code-review`

The coordinator skill. It gathers the diff, triages which specialists to dispatch, runs them, normalizes findings, and presents a structured report.

```text
/code-review 42
/code-review https://github.com/owner/repo/pull/42
/code-review feature/my-branch
/code-review
```

### `/docs`

Writes or improves a document. Tell it what you want to document and it asks the right questions, proposes a structure, and writes it.

```text
/docs
/docs ADR for switching from REST to GraphQL
```

## Agents

**7 specialist agents:**

| Agent | Model | Focus |
|-------|-------|-------|
| `correctness` | Opus | Logic bugs, edge cases, off-by-one, null handling, race conditions |
| `security` | Opus | OWASP Top 10, injection, auth bypass, secrets, input validation |
| `documentation` | Opus | Doc quality, missing doc updates, linting, Diataxis adherence |
| `architecture` | Sonnet | Coupling, SOLID, API design, separation of concerns, pattern consistency |
| `performance` | Sonnet | Algorithmic complexity, N+1 queries, caching, resource leaks |
| `maintainability` | Sonnet | Clean code, naming, readability, dead code, duplication |
| `observability` | Sonnet | Logging, metrics, tracing, error reporting, alertability |

## Review flow

1. **Gather** — obtains the diff and reads full file contents for context
2. **Triage** — assesses which agents are needed and presents a table for confirmation
3. **Dispatch** — runs selected agents in parallel with the diff, file contents, and review guides
4. **Normalize** — deduplicates findings, validates severity, groups by severity then file
5. **Present** — outputs a structured report with Blockers, Suggestions, and Nitpicks
6. **Follow-up** — creates tracked issues for any descoped findings

## Severity levels

- **Blocker** — must fix before merge. "Will we need a hotfix?"
- **Suggestion** — should fix. "Will someone fix this within 3 months?"
- **Nitpick** — take or leave. "Would a senior engineer disagree?"

See [severity-rubric.md](skills/code-review/guides/severity-rubric.md) for the full rubric.

## Documentation templates

Eight templates are included:

| Template | Diataxis type |
| --- | --- |
| README | Orientation + quickstart |
| Quickstart | Tutorial |
| How-to guide | How-to |
| API reference | Reference |
| ADR | — |
| RFC / Design doc | — |
| Migration guide | — |
| Runbook | — |

ADRs, RFCs, migration guides, and runbooks fall outside the Diataxis taxonomy but follow their own established conventions.

## Optional dependencies

If these aren't installed, the skill and documentation agent skip linting and note it in their output.

- **[markdownlint](https://github.com/DavidAnson/markdownlint-cli2)** — structural markdown quality (runs via `npx`, auto-downloaded on first use; requires Node.js)
- **[Vale](https://vale.sh/)** — prose quality: passive voice, weasel words, tone

Install Vale:

```bash
brew install vale        # macOS
snap install vale --edge # Linux
```

Or download directly from [vale.sh](https://vale.sh/docs/vale-cli/installation/).

## Follow-up issues

Descoped findings get tracked as issues (GitHub Issues by default). The plugin checks the project's `CLAUDE.md` for a preferred issue tracker. If `gh` isn't available, it outputs copyable issue content instead.
