---
name: create-hook
description: Use when the user wants to add a Claude Code hook that runs automatically on tool events (before/after file edits, before commits, etc.). Guides creation of shell scripts and wiring them into settings.json.
---

# Create Hook

Add an automated Claude Code hook that triggers on tool events.

## Steps

### 1. Identify the hook type and trigger

| Hook type     | When it runs                               |
| ------------- | ------------------------------------------ |
| `PreToolUse`  | Before a tool call executes — can block it |
| `PostToolUse` | After a tool call completes                |

Common matchers: `Bash`, `Write`, `Edit`, `Write\|Edit`

For `Bash` hooks that should only fire on specific commands (e.g. `git commit`), the script must inspect the command from stdin and exit 0 early for irrelevant commands.

### 2. Check for `jq`

Hooks receive tool input as JSON on **stdin**. `jq` is the right tool to parse it — check it's installed before writing the script:

```bash
which jq || echo "install jq first"
```

### 3. Write a shell script

Put scripts in `.claude/hooks/` relative to the project root. Never inline logic in `settings.json` — shell scripts are readable and version-controllable.

**Template:**

```bash
#!/usr/bin/env bash
# One-line description of what this hook does.
# Invoked by Claude Code as a <PreToolUse|PostToolUse> hook on <Matcher>.
# Tool input arrives on stdin as JSON.
set -euo pipefail

# Parse the relevant field from the tool input
value=$(jq -r '.tool_input.<field> // ""')

# Early exit for irrelevant cases
if <condition>; then
  exit 0
fi

# Find the repo root portably — never hardcode paths
repo_root=$(git rev-parse --show-toplevel)
cd "$repo_root"

# Do the work
<commands>
```

**Key rules:**

- Use `git rev-parse --show-toplevel` for the repo root — never hardcode absolute paths
- Read from stdin via `jq` — there are no CLI arguments
- Exit 0 to allow, non-zero to block (for `PreToolUse`)
- Make the script executable: `chmod +x .claude/hooks/<name>.sh`

### 4. Wire it into `settings.json`

Reference the script with a **relative path** so it works for all contributors:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/<name>.sh",
            "statusMessage": "Running <description>..."
          }
        ]
      }
    ]
  }
}
```

Use `.claude/settings.json` for shared team hooks. Use `.claude/settings.local.json` for personal hooks (gitignored).

### 5. Test the hook manually

Before relying on it, pipe a representative payload and verify exit codes:

```bash
# Should pass through (non-matching command)
echo '{"tool_input":{"command":"echo hello"}}' | bash .claude/hooks/<name>.sh
echo "exit: $?"

# Should trigger
echo '{"tool_input":{"command":"git commit -m \"test\""}}' | bash .claude/hooks/<name>.sh
echo "exit: $?"
```

## stdin JSON shape by tool

| Tool                        | Useful fields                                      |
| --------------------------- | -------------------------------------------------- |
| `Bash`                      | `.tool_input.command`                              |
| `Write`                     | `.tool_input.file_path`                            |
| `Edit`                      | `.tool_input.file_path`                            |
| `Write\|Edit` (PostToolUse) | `.tool_response.filePath // .tool_input.file_path` |
