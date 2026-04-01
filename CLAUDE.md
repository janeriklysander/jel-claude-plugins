# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code plugin marketplace (`jel-claude-plugins`) containing three plugins: **docs-system**, **ddd-design**, and **create-hook**. There is no build system, no test suite, and no package manager — the repo is pure markdown and one shell script.

## Linting (the only executable)

```bash
bash plugins/docs-system/skills/documentation/scripts/lint-docs.sh <file-or-directory>
```

Requires Node.js (for `npx markdownlint-cli2`) and optionally Vale. Config files (`.vale.ini`, `.markdownlint-cli2.yaml`) live alongside the script.

## Architecture

```
.claude-plugin/marketplace.json   # Marketplace manifest — registers all plugins
plugins/<name>/
  skills/<skill-name>/SKILL.md    # Skill definition (frontmatter + prompt)
  agents/<agent-name>.md          # Agent definition (frontmatter + prompt)
docs/                             # GitHub Pages site (Jekyll, primer theme)
```

**Skills** use YAML frontmatter: `name`, `description`, `argument-hint`, `allowed-tools`. The body is the skill prompt.

**Agents** use YAML frontmatter: `name`, `description`, `model`, `tools`. The body is the agent system prompt.

Any new or updated plugin must be reflected in three places: `.claude-plugin/marketplace.json`, the plugin's own `README.md`, and the root `README.md`.

### Official Claude Code documentation

- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces.md) — `marketplace.json` schema, plugin source types, naming rules, validation with `claude plugin validate`
- [Skills](https://code.claude.com/docs/en/skills.md) — `SKILL.md` frontmatter reference (all supported fields), string substitutions (`$ARGUMENTS`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_SKILL_DIR}`), discovery rules
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference.md) — `plugin.json` manifest schema, `.claude-plugin/` directory structure, component path rules, environment variables (`${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`)

### docs-system internals

The most complex plugin — has sub-structure worth knowing:

- `skills/documentation/guides/` — writing guidance (Diataxis, tone, code examples, linting)
- `skills/documentation/templates/` — 8 doc-type templates referenced by the skill
- `skills/documentation/scripts/` — `lint-docs.sh` plus Vale/markdownlint config; uses `${CLAUDE_PLUGIN_ROOT}` to resolve paths at runtime
- `agents/tech-writer-reviewer.md` — Sonnet-based review agent with restricted tool access

## GitHub Pages

Served from `docs/` via Jekyll with `jekyll-theme-primer`. Custom `<head>` content for Mermaid rendering lives in `docs/_includes/head-custom.html`.

## Git workflow

- Squash merge only (merge commits and rebase disabled on GitHub)
- PR titles become the squash commit message
- Branches auto-delete after merge
