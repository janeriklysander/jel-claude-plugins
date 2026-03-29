# How to set up 1Password SSH agent for Claude Code sandbox

Use the 1Password SSH agent so that SSH authentication works inside Claude Code's sandbox without exposing your private key files to sandboxed processes. This guide covers macOS and WSL2. By the end, `git fetch`, `git push`, and other SSH-based commands will work inside Claude Code without the sandbox being able to read your keys.

For official reference, see the [1Password SSH agent documentation](https://developer.1password.com/docs/ssh/) and [Claude Code sandboxing documentation](https://code.claude.com/docs/en/sandboxing).

## Why an SSH agent?

Claude Code's sandbox blocks direct reads to `~/.ssh` to prevent credential exfiltration. This also breaks SSH authentication. An SSH agent solves this by holding keys in memory and communicating through a Unix socket — the sandbox only needs access to that socket, not the key files.

## Prerequisites

- 1Password desktop app with SSH keys stored
- **macOS:** no additional dependencies
- **WSL2:** `socat` installed (`sudo apt-get install socat`) and [`npiperelay`](https://github.com/jstarks/npiperelay)

## Step 1: Enable the SSH agent in 1Password

Open **1Password**, go to **Settings** > **Developer**, and enable **"Use the SSH agent"**.

Verify the agent is running:

- **macOS:** run `ssh-add -l` in Terminal
- **WSL2:** run `ssh-add.exe -l` in PowerShell (the agent runs on the Windows side)

Both should list your SSH keys. If the list is empty, import or generate keys in 1Password first — see the [1Password SSH key guide](https://developer.1password.com/docs/ssh/get-started/).

## Step 2: Configure the agent socket

### macOS

1Password sets up the agent socket automatically. Add the following to `~/.ssh/config`:

```text
Host *
    IdentityAgent "~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
```

### WSL2

<details>
<summary>Expand WSL2 instructions</summary>

The 1Password agent runs on Windows and communicates over a named pipe. You need `npiperelay` to bridge it to a Unix socket in WSL2.

#### Install `npiperelay`

1. Download `npiperelay.exe` from the [GitHub releases page](https://github.com/jstarks/npiperelay/releases)
2. Place it in a directory on your Windows filesystem (for example, `C:\Users\<your-username>\AppData\Local\Programs\npiperelay\`)

Replace `<your-username>` with your Windows username throughout this guide.

> `npiperelay.exe` does not need to be on your PATH. The bridge script references it by absolute path.

#### Create the wrapper script

Socat can't pass command-line arguments to the program it launches. Create a wrapper script at `~/.ssh/npiperelay-wrapper.sh` that socat will call:

```bash
#!/bin/bash
exec /mnt/c/Users/<your-username>/AppData/Local/Programs/npiperelay/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent
```

> The `exec` line must be a single line with no line breaks. Update the path to match where you placed `npiperelay.exe`.

Make it executable:

```bash
chmod +x ~/.ssh/npiperelay-wrapper.sh
```

#### Create the bridge script

Create `~/.agent-bridge.sh`:

```bash
#!/bin/bash
export SSH_AUTH_SOCK="$HOME/.ssh/agent.sock"

if ! ss -a 2>/dev/null | grep -q "$SSH_AUTH_SOCK"; then
    rm -f "$SSH_AUTH_SOCK"
    setsid socat UNIX-LISTEN:"$SSH_AUTH_SOCK",fork \
        EXEC:"$HOME/.ssh/npiperelay-wrapper.sh",nofork &
fi
```

Make it executable:

```bash
chmod +x ~/.agent-bridge.sh
```

> The `ss` check detects if the socket is already being listened on. If you see connection errors despite the socket file existing, the `socat` process may have exited — see [Troubleshooting](#troubleshooting).

#### Load the bridge on shell startup

Add to `~/.bashrc` (or `~/.zshrc`):

```bash
# 1Password SSH Agent bridge
source "$HOME/.agent-bridge.sh"
```

#### Configure SSH to use the agent

Create or edit `~/.ssh/config`:

```text
Host *
    IdentityAgent ~/.ssh/agent.sock
```

</details>

## Step 3: Verify SSH

Restart your shell. On WSL2, confirm the agent bridge is running before proceeding:

```bash
echo "$SSH_AUTH_SOCK"    # Should print the path to your agent socket
ls -la ~/.ssh/agent.sock # Should show the socket file
```

If the socket doesn't exist, run `source ~/.agent-bridge.sh` to start the bridge.

Then:

```bash
ssh-add -l
```

You should see your 1Password keys listed. Test a real connection:

```bash
ssh -T git@github.com
```

Expected output:

```text
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

## Step 4: Configure the Claude Code sandbox

The sandbox needs three things to work with the SSH agent. The examples below show only the SSH-relevant settings — `"..."` indicates other config you should keep as-is.

1. **`allowRead`** for the agent socket, `known_hosts`, and SSH config (overrides the `denyRead` on `~/.ssh`)
2. **`allowUnixSockets`** so the sandbox network layer permits the socket connection
3. **`excludedCommands`** for git remote operations (SSH uses port 22, which the sandbox network proxy can't handle — these commands run outside the sandbox)

**macOS:**

```json
{
  "sandbox": {
    "...": "...",
    "excludedCommands": ["git push", "git fetch", "git pull", "git clone"],
    "network": {
      "allowUnixSockets": [
        "~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
      ]
    },
    "filesystem": {
      "...": "...",
      "allowRead": [
        "~/.ssh/known_hosts",
        "~/.ssh/config"
      ]
    }
  }
}
```

<details>
<summary><strong>WSL2</strong></summary>

```json
{
  "sandbox": {
    "...": "...",
    "excludedCommands": ["git push", "git fetch", "git pull", "git clone"],
    "network": {
      "allowUnixSockets": ["~/.ssh/agent.sock"]
    },
    "filesystem": {
      "...": "...",
      "allowRead": [
        "~/.ssh/known_hosts",
        "~/.ssh/config",
        "~/.ssh/agent.sock"
      ]
    }
  }
}
```

Claude Code doesn't source `~/.bashrc`, so `SSH_AUTH_SOCK` isn't set automatically. The environment variable must be inherited from the terminal you launch Claude Code from.

</details>

Restart Claude Code and verify SSH works inside the sandbox:

```bash
ssh-add -l        # Should list your 1Password keys
git fetch --dry-run  # Should succeed without errors
```

For full sandbox configuration (filesystem rules, network isolation, deny lists), see [How to harden Claude Code permissions](claude-code-security-hardening.md).

## Troubleshooting

**`ssh-add -l` shows "could not open a connection to your authentication agent":** Confirm 1Password is running with the SSH agent enabled. On WSL2, check that the `socat` bridge is running: `ss -a | grep agent.sock` should show output. If empty, the bridge isn't running — source `~/.agent-bridge.sh` to restart it.

**`npiperelay.exe` usage output instead of connecting (WSL2 only):** The wrapper script has a line break in the `exec` line. Run `cat ~/.ssh/npiperelay-wrapper.sh` to inspect it, then open it in an editor and ensure the `exec` command is on a single line with no embedded newlines.

**"Host key verification failed" in Claude Code:** The sandbox is blocking access to `~/.ssh/known_hosts`. Add it to `sandbox.filesystem.allowRead`.

**"Could not resolve hostname" for git commands:** Git remote operations need to be in `excludedCommands` since SSH traffic can't go through the sandbox's HTTP proxy.

**Socket already exists errors (WSL2 only):** Remove the stale socket with `rm ~/.ssh/agent.sock` and open a new terminal.
