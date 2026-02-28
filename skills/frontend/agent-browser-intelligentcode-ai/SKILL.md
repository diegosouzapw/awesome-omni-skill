---
name: "agent-browser"
description: "Use when you need to reproduce or debug web UI flows (especially auth/OIDC) via the Agent Browser CLI, capture snapshots/screenshots, and extract redirect URLs and on-page errors deterministically. Includes install/setup guidance when the CLI is missing."
category: "process"
scope: "development"
subcategory: "workflow"
tags:
  - development
  - process
  - agent
  - browser
version: "10.2.14"
author: "Karsten Samaschke"
contact-email: "karsten@vanillacore.net"
website: "https://vanillacore.net"
---

# Agent Browser (CLI)

Use this skill when we need to verify UI behavior ourselves (login flows, redirects, error toasts) without guessing.

## Install / Setup (If `agent-browser` Is Missing)

1. Check if the CLI is available:

```bash
command -v agent-browser && agent-browser --help | head
```

2. Install Agent Browser CLI (pick one):

macOS (Homebrew):

```bash
brew install agent-browser
```

Linux:
- First install Node.js + npm (examples):

```bash
# Debian/Ubuntu
sudo apt update && sudo apt install -y nodejs npm

# Fedora
sudo dnf install -y nodejs npm

# Arch
sudo pacman -S --needed nodejs npm
```

If your distro Node.js is too old, use a version manager (nvm/fnm/asdf) and install a current Node LTS.

All platforms (npm, recommended/fallback):

```bash
npm install -g agent-browser
```

Windows note:
- If you don't have Node.js yet, install it first (PowerShell):

```powershell
winget install OpenJS.NodeJS.LTS
```

3. Install the browser runtime the CLI uses (one-time per machine/user):

```bash
agent-browser install
```

Linux-only note:
- If the browser fails to launch due to missing system libraries, try:

```bash
agent-browser install --with-deps
```

4. Sanity check:

```bash
agent-browser open https://example.com/
agent-browser snapshot -i
agent-browser close
```

## Artifacts (Cross-Platform)

Prefer saving evidence inside the repo so it can be attached/shared predictably:

- Default: `.agent/artifacts/agent-browser/<run-id>/`
- Include: screenshots (`.png`) and snapshots (`.txt`)

Example (macOS/Linux):

```bash
run_id="oidc-$(date +%Y%m%d-%H%M%S)"
out_dir=".agent/artifacts/agent-browser/$run_id"
mkdir -p "$out_dir"
echo "$out_dir"
```

Example (Windows PowerShell):

```powershell
$runId = "oidc-" + (Get-Date -Format "yyyyMMdd-HHmmss")
$outDir = ".agent/artifacts/agent-browser/$runId"
New-Item -ItemType Directory -Force -Path $outDir | Out-Null
$outDir
```

## Quick Start (OIDC Debug)

The core loop is: `open` -> `snapshot -i` -> interact (prefer refs like `@e2`) -> re-`snapshot -i` -> capture evidence.

### macOS/Linux (bash)

1. Start a clean session, open the app, and capture baseline evidence:

```bash
run_id="oidc-$(date +%Y%m%d-%H%M%S)"
out_dir=".agent/artifacts/agent-browser/$run_id"
mkdir -p "$out_dir"

agent-browser close || true
agent-browser --session oidc open https://example.com/   # replace
agent-browser --session oidc wait 1000
agent-browser --session oidc screenshot "$out_dir/01-home.png"
agent-browser --session oidc snapshot -i > "$out_dir/01-home.snapshot.txt"
```

2. Prefer clicking by ref from the snapshot (fallback to semantic find if needed):

```bash
# Look in 01-home.snapshot.txt for: button "Sign in" [ref=e2]
agent-browser --session oidc click @e2

# Fallback if refs aren't obvious:
# agent-browser --session oidc find role button click --name "Sign in"

agent-browser --session oidc wait 3000
agent-browser --session oidc get url
agent-browser --session oidc screenshot "$out_dir/02-after-login-click.png"
agent-browser --session oidc snapshot -i > "$out_dir/02-after-login-click.snapshot.txt"
```

3. If the flow fails, capture high-signal diagnostics:

```bash
agent-browser --session oidc errors
agent-browser --session oidc console
agent-browser --session oidc network requests
```

### Windows (PowerShell)

```powershell
$runId = "oidc-" + (Get-Date -Format "yyyyMMdd-HHmmss")
$outDir = ".agent/artifacts/agent-browser/$runId"
New-Item -ItemType Directory -Force -Path $outDir | Out-Null

agent-browser close
agent-browser --session oidc open https://example.com/   # replace
agent-browser --session oidc wait 1000
agent-browser --session oidc screenshot "$outDir/01-home.png"
agent-browser --session oidc snapshot -i | Out-File -Encoding utf8 "$outDir/01-home.snapshot.txt"

# Click by ref from the snapshot, e.g. @e2
agent-browser --session oidc click @e2
agent-browser --session oidc wait 3000
agent-browser --session oidc get url
agent-browser --session oidc screenshot "$outDir/02-after-login-click.png"
agent-browser --session oidc snapshot -i | Out-File -Encoding utf8 "$outDir/02-after-login-click.snapshot.txt"

agent-browser --session oidc errors
agent-browser --session oidc console
agent-browser --session oidc network requests
```

## Interaction Guidance (Stability First)

- Prefer refs from `snapshot -i` (for example `@e2`) for deterministic selection.
- If you can't use refs, prefer `find role ...` over CSS selectors; it is more stable across UI changes.
- Use `--session <name>` consistently so multiple investigations can run without stepping on each other.
- Re-run `snapshot -i` after navigation or DOM changes (refs are tied to the current page state).
- Use `wait <ms>` only as a last resort; prefer waiting on UI cues (for example, `find role heading` or `find text`) when possible.
- Use `agent-browser close` between runs if you need to reset global state.

## Security / Privacy Notes

- Avoid capturing credentials, secrets, or customer data in screenshots/snapshots.
- When you must test auth flows, prefer test accounts and scrub/redact artifacts before sharing outside the immediate team.
