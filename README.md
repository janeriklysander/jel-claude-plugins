# Agentic Plugins

Curated plugins for [Claude Code](https://claude.com/claude-code) — a documentation system, a DDD design facilitator, and a hook creation guide.

## Plugins

| Plugin | Description |
| --- | --- |
| **docs-system** | Documentation skill with Diataxis framework, 8 templates, linting, and a tech-writer-reviewer agent |
| **ddd-design** | Structured Domain-Driven Design discovery sessions with strategic and tactical modeling |
| **create-hook** | Step-by-step guide for creating Claude Code hooks with shell scripts |

## Installation

Install a plugin from this marketplace:

```bash
claude plugin add janeriklysander/agentic-plugins --name docs-system
```

Replace `docs-system` with the plugin you want. To install all plugins from the marketplace:

```bash
claude plugin add janeriklysander/agentic-plugins
```

## Reference guides

These standalone guides live in `docs/` and are not plugins — they're reference documentation for Claude Code users:

- [1Password SSH agent setup](docs/1password-ssh-agent-setup.md) — configure 1Password's SSH agent to work inside Claude Code's sandbox (macOS and WSL2)
- [Claude Code security hardening](docs/claude-code-security-hardening.md) — three-layer permission and sandbox configuration to reduce prompt fatigue and protect against prompt injection

## Optional dependencies

The **docs-system** plugin includes a lint script that uses [Vale](https://vale.sh/) and [markdownlint](https://github.com/DavidAnson/markdownlint-cli2). These are optional — if not installed, the documentation skill and tech-writer-reviewer agent will skip linting and note it in their output.

## Contributing

Feedback and bug reports are welcome via [GitHub Issues](https://github.com/janeriklysander/agentic-plugins/issues). There is no roadmap — this is a personal collection shared with the community.

## License

[MIT](LICENSE)
