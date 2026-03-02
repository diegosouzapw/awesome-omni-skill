---
name: openclaw-tescmd
slug: openclaw-tescmd
displayName: OpenClaw Tesla (tescmd)
version: 0.9.7
description: Installation and setup guide for Tesla vehicle control and telemetry via the tescmd node.
homepage: https://github.com/oceanswave/openclaw-tescmd
metadata: {"category":"platform","platform":"tesla","node":"tescmd"}
---

# OpenClaw Tesla (tescmd) — Setup Guide

This plugin connects Tesla vehicles to the OpenClaw Gateway via the [tescmd](https://github.com/oceanswave/tescmd) node. Once installed and paired, the plugin automatically registers all tools, commands, slash commands, and telemetry event types.

This document covers **installation and setup only**. Runtime tool usage, workflows, and error handling are provided by the `tescmd` skill (call `tescmd_help` for the full reference).

**What you get:**
- 39 agent-callable tools
- 14 slash commands
- Real-time telemetry streaming
- Supercharger discovery (10,000+ locations via supercharge.info)
- CLI fallback when node is disconnected

**Repositories:**
- Plugin: https://github.com/oceanswave/openclaw-tescmd
- tescmd node (Python CLI): https://github.com/oceanswave/tescmd

---

## Architecture

```
Agent (you)
  ↓  tool calls
OpenClaw Gateway
  ↓  node.invoke.request
openclaw-tescmd Plugin
  ↓  WebSocket dispatch
tescmd Node (Python)
  ├─ Tesla Fleet API (REST)
  ├─ Vehicle Command Protocol (VCSEC — signed commands)
  └─ Fleet Telemetry Stream (WebSocket)
  ↓
Tesla Vehicle
```

The plugin is the **Gateway-side counterpart** to the tescmd node. It defines tool schemas and routes invocations. The tescmd node handles all direct communication with Tesla.

---

## Setup

### Step 1: Check Prerequisites

Before starting, verify the required tools are installed and authenticated.

#### Required: git

```bash
git --version
```

If missing, install it:
- macOS: `xcode-select --install`
- Linux: `sudo apt install git` or `sudo dnf install git`

#### Required: GitHub CLI (gh)

```bash
gh --version
gh auth status
```

If `gh` is not installed:
- macOS: `brew install gh`
- Linux: see https://github.com/cli/cli/blob/trunk/docs/install_linux.md

If not logged in:
```bash
gh auth login
```

**Tell the user:** "Please complete the GitHub CLI login in your terminal. Select your preferences when prompted and finish the browser-based auth flow."

Wait for the user to confirm they have completed the login before continuing.

#### Required: Python 3.11+

```bash
python3 --version
```

Must be 3.11 or higher. If not:
- macOS: `brew install python@3.12`
- Linux: `sudo apt install python3.12` or use pyenv

#### Recommended: Tailscale

Tailscale provides a public HTTPS endpoint for Tesla Fleet Telemetry streaming with zero infrastructure setup.

```bash
tailscale version
tailscale status
```

If not installed:
- macOS: `brew install tailscale` or download from https://tailscale.com/download
- Linux: `curl -fsSL https://tailscale.com/install.sh | sh`

If not logged in:
```bash
sudo tailscale up
```

**Tell the user:** "Please complete the Tailscale login in your browser if prompted."

Wait for the user to confirm before continuing.

---

### Step 2: Install the tescmd OpenClaw Plugin

#### Standard install:

```bash
openclaw plugins install @oceanswave/openclaw-tescmd
```

#### Verify installation:

```bash
openclaw plugins list
```

You should see the plugin listed with version 0.9.0 (or later).

#### Plugin management commands:

| Command | Purpose |
|---------|---------|
| `openclaw plugins list` | List installed plugins |
| `openclaw plugins info openclaw-tescmd` | Plugin details |
| `openclaw plugins doctor` | Check plugin health |
| `openclaw plugins update openclaw-tescmd` | Update to latest |
| `openclaw plugins enable openclaw-tescmd` | Enable the plugin |
| `openclaw plugins disable openclaw-tescmd` | Disable without uninstalling |

---

### Step 3: Install the tescmd CLI

```bash
pip install tescmd
```

Verify:
```bash
tescmd --version
```

---

### Step 4: Run tescmd Setup

The tescmd setup wizard is **interactive** and requires the user to make choices and complete steps in their terminal and browser. You cannot complete this step autonomously.

```bash
tescmd setup
```

**Tell the user:** "I've started the tescmd setup wizard. This is an interactive process that will walk you through:"
1. Creating a Tesla Developer application
2. Generating your EC key pair
3. Hosting your public key (via GitHub Pages or Tailscale Funnel)
4. Registering with the Tesla Fleet API
5. Completing OAuth2 login in your browser
6. Pairing the key with your vehicle (requires physical presence at the vehicle)

"Please follow the prompts in your terminal and let me know when setup is complete."

**Wait for the user to confirm setup is finished before proceeding.**

#### Verify Setup

After the user confirms, check auth status:
```bash
tescmd auth status
```

This should show a valid token. If it shows expired or missing, the user needs to re-run:
```bash
tescmd auth login
```

---

### Step 5: Identify the Vehicle

List vehicles on the account to get the VIN:
```bash
tescmd vehicle list
```

Note the VIN — it is needed for the serve command.

---

### Step 6: Start the tescmd Node and Pair with the Gateway

The tescmd node bridges the Tesla Fleet API to the OpenClaw Gateway. The first connection requires a one-time pairing approval.

#### First-time pairing:

Start the node with just the Gateway URL (no token needed):
```bash
tescmd serve <VIN> --openclaw <gateway_ws_url>
```

The node sends a `node.pair.request` to the Gateway and waits for approval. The pending request expires after **5 minutes**, so approve it promptly.

**In a separate terminal**, approve the pairing:
```bash
openclaw nodes pending                # View waiting pair requests
openclaw nodes approve <requestId>    # Approve the node
```

On approval the Gateway issues an authentication token. The node receives it, saves it to `~/.config/tescmd/bridge.json`, and establishes the authenticated connection. No manual token handling is needed.

**Tell the user:** "Start the tescmd node with `tescmd serve <VIN> --openclaw <gateway_url>`, then in another terminal run `openclaw nodes pending` and `openclaw nodes approve <requestId>` to complete pairing."

**Wait for the user to confirm pairing is complete before continuing.**

#### Subsequent connections (already paired):

Once paired, the node reconnects automatically using the stored token:
```bash
tescmd serve <VIN> --openclaw <gateway_ws_url>
```

You can also pass the token explicitly if needed:
```bash
tescmd serve <VIN> --openclaw <gateway_ws_url> --openclaw-token <gateway_token>
```

#### Node management commands:

| Command | Purpose |
|---------|---------|
| `openclaw nodes pending` | View pending pair requests |
| `openclaw nodes approve <id>` | Approve a node |
| `openclaw nodes reject <id>` | Reject a node |
| `openclaw nodes status` | List paired nodes and their status |

#### Operating modes:

| Mode | Command | Description |
|------|---------|-------------|
| **Full** (default) | `tescmd serve <VIN> --openclaw <url>` | MCP server + telemetry + OpenClaw bridge |
| **Bridge only** | `tescmd serve <VIN> --no-mcp --openclaw <url>` | Telemetry + OpenClaw, no MCP server |
| **With Tailscale** | `tescmd serve <VIN> --tailscale --openclaw <url>` | Exposes MCP via Tailscale Funnel |
| **Dry run** | `tescmd serve <VIN> --dry-run` | Log events as JSONL, no Gateway connection |

#### Key flags reference:

| Flag | Description |
|------|-------------|
| `<VIN>` | Vehicle Identification Number (positional) |
| `--openclaw <ws_url>` | Gateway WebSocket URL (e.g. `ws://host:18789`) |
| `--openclaw-token <token>` | Gateway authentication token (auto-stored after pairing) |
| `--openclaw-config <path>` | Bridge config JSON (default: `~/.config/tescmd/bridge.json`) |
| `--transport <type>` | MCP transport: `streamable-http` (default) or `stdio` |
| `--port <num>` | MCP HTTP port (default: 8080) |
| `--host <addr>` | MCP bind address (default: 127.0.0.1) |
| `--telemetry-port <num>` | Telemetry WebSocket port (default: 4443) |
| `--fields <preset>` | Telemetry fields: `driving`, `charging`, or `all` |
| `--interval <sec>` | Telemetry polling interval in seconds |
| `--no-telemetry` | Disable telemetry streaming |
| `--no-mcp` | Disable MCP server |
| `--no-log` | Disable CSV telemetry logging |
| `--dry-run` | Log events as JSONL without connecting to Gateway |
| `--tailscale` | Expose MCP via Tailscale Funnel |
| `--client-id <id>` | MCP OAuth client ID |
| `--client-secret <secret>` | MCP OAuth client secret |

#### Environment variables (alternative to flags):

These can be set in `~/.config/tescmd/.env`:
```bash
TESLA_CLIENT_ID=your-client-id
TESLA_CLIENT_SECRET=your-client-secret
TESLA_VIN=5YJ3E1EA1NF000000
TESLA_REGION=na                    # na, eu, or cn
OPENCLAW_GATEWAY_URL=ws://gateway.example.com:18789
OPENCLAW_GATEWAY_TOKEN=your-token
TESLA_COMMAND_PROTOCOL=auto        # auto, signed, or unsigned
```

---

### Step 7: Verify the Connection

Once the node is running and paired, confirm it connected to the Gateway:
```bash
openclaw nodes status
```

Or use the agent tool:
- Call `tescmd_node_status` to check connection status

If connected, the plugin's tools are ready. Call `tescmd_help` for the full runtime reference including tool usage, workflows, and error handling.

---

## Troubleshooting Setup

| Problem | Solution |
|---------|----------|
| "no node connected" | Start the node: `tescmd serve <VIN> --openclaw <url>` |
| Pairing request not visible | Check `openclaw nodes pending` — requests expire after 5 minutes. Restart the node to generate a new request. |
| Node connects then disconnects | Check Gateway URL. Run `tescmd auth status` to verify Tesla auth. |
| Auth/token errors | Re-authenticate: `tescmd auth login` |
| Setup wizard issues | Re-run `tescmd setup` or check https://github.com/oceanswave/tescmd |
| Plugin not loading | Run `openclaw plugins doctor`. Check `openclaw plugins list` for the plugin entry. |
| Triggers say "not available" | Restart node with telemetry: remove `--no-telemetry` or add `--fields all` |

---

## Configuration

Minimal — the tescmd node handles all vehicle-specific configuration.

```json
{
  "plugins": {
    "entries": {
      "openclaw-tescmd": {
        "enabled": true,
        "config": {
          "debug": false
        }
      }
    }
  }
}
```

---

## CLI Quick Reference

### tescmd CLI Commands

```bash
tescmd serve <VIN> --openclaw <url>                              # Start node (uses stored token)
tescmd serve <VIN> --openclaw <url> --openclaw-token <token>     # Start node (explicit token)
tescmd setup                        # Interactive setup wizard
tescmd auth status                  # Check auth token status
tescmd auth login                   # Re-authenticate with Tesla
tescmd vehicle list                 # List vehicles on account
tescmd vehicle info                 # Full vehicle data snapshot
tescmd cache status                 # Check cache stats
```
