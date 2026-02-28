---
name: git-sw
description: Easily switch between multiple Git profiles/configs (TUI + Automated mode)
homepage: https://github.com/thansetan/git-sw
metadata:
  {
    "openclaw":
      {
        "emoji": "🔄",
        "requires": { "bins": ["git-sw"] },
        "install":
          [
            {
              "id": "go",
              "kind": "go",
              "package": "github.com/thansetan/git-sw",
              "bins": ["git-sw"],
              "label": "Install git-sw via go install",
            },
          ],
      },
  }
---

# git-sw 🔄

A CLI tool to switch between multiple Git profiles/configs. Supports both an interactive TUI for humans and a flag-based mode for agents.

## Usage (Agent/Automated)

Always use the `--no-tui` flag when calling this tool from an agent session.

### List Profiles
```bash
git-sw --no-tui list
```

### Create a Profile
```bash
git-sw --no-tui --profile <name> --name "<user-name>" --email "<user-email>" create

# With signing key
git-sw --no-tui --profile <name> --name "<user-name>" --email "<user-email>" --signing-key <key> --key-format <format> create
```

### Switch Profile
```bash
# Locally
git-sw --no-tui --profile <name> use

# Globally
git-sw --no-tui --profile <name> -g use
```

### Delete a Profile
```bash
git-sw --no-tui --profile <name> --yes delete
```

## Options
- `--no-tui`: Required for non-interactive usage.
- `--profile`: The name of the profile.
- `--name`: Git user name.
- `--email`: Git user email.
- `--signing-key`: Signing key (GPG key ID, SSH pub path, or X.509 cert).
- `--key-format`: Signing key format: `openpgp`, `ssh`, or `x509`.
- `--gpg-program`: GPG program path (default: `gpg`).
- `--yes`: Bypasses confirmation prompts.
- `-g`: Global mode.
