---
name: runtime-awareness
description: Full runtime environment inventory — platform, tools, services, paths. Load when checking tooling or capabilities
user-invocable: false
---

# Runtime Awareness

> Full inventory of the runtime environment — platform, tools, services, and cross-boundary capabilities.

## Platform

| Property | Value |
|----------|-------|
| **OS** | Ubuntu 22.04 LTS on WSL2 (kernel 6.6.x) |
| **Host** | Windows, mirrored networking (`networkingMode=mirrored`) |
| **systemd** | Enabled |
| **WSL interop** | Enabled, `appendWindowsPath=true` |
| **Windows user** | `FaroukBakari` → `/mnt/c/Users/FaroukBakari/` (full R/W) |

### Compute Resources

| Resource | Allocation |
|----------|------------|
| CPU | 16 cores |
| RAM | 24 GB |
| Swap | 48 GB |
| GPU | **NVIDIA RTX 4090** (24 GB VRAM), CUDA 12.4, driver 591.59 |
| Storage | C: (886G), D: (1.9T), E: (17T), F: (1.9T), G: (977G), WSL root (1T) |

### Networking

- **Mirrored mode**: WSL and Windows share `localhost` — no port forwarding needed
- Dev servers started in WSL are directly accessible from Windows browsers and vice versa
- Firewall: disabled in WSL config

---

## Tool Access

### Core Tools (Claude Code CLI)

`Read`, `Edit`, `Write`, `Glob`, `Grep`, `Bash`, `WebFetch`, `WebSearch`, `Task` (subagents), `TodoWrite`, `AskUserQuestion`, `EnterPlanMode`.

### Session Introspection Commands

User-invocable slash commands (agent cannot invoke directly — recommend when relevant):

| Command | Shows | When |
|---------|-------|------|
| `/context` | Context usage grid | Before large tasks, context pressure |
| `/cost` | Token stats | After heavy tool use |
| `/compact [focus]` | Manual compaction with optional focus | Context filling up |

**Auto-compaction**: Happens silently at limits — no agent signal. Mitigation: `Compact Instructions` (CLAUDE.md §5) defines preservation priorities. `PreCompact` hook fires with `trigger: "manual"/"auto"`.

### IDE MCP Bridge

| Tool | Capability |
|------|------------|
| `mcp__ide__getDiagnostics` | Read VS Code language diagnostics (type errors, lint warnings) |
| `mcp__ide__executeCode` | Execute Python code in active Jupyter kernel |

### VS Code CLI (`code`)

Available via `.vscode-server` remote CLI. Can:
- Open files/diffs in the editor (`code -g file:line`, `code -d file1 file2`)
- Install/list/uninstall extensions
- Open folders/workspaces

### Windows Executables (via WSL Interop)

| Executable | Use Case |
|------------|----------|
| `powershell.exe` | Windows system commands, registry, scheduled tasks |
| `cmd.exe` | Windows shell commands |
| `explorer.exe` | Open folders/URLs in Windows (`explorer.exe .`, `explorer.exe https://...`) |
| `clip.exe` | Copy stdout to Windows clipboard (`echo text \| clip.exe`) |
| `wslpath` | Path translation: `wslpath -w /home/...` → `\\wsl$\...`, `wslpath -u 'C:\...'` → `/mnt/c/...` |
| Chrome | `"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"` — launch with optional URL |

### NOT Available

- VS Code extension API (`vscode.runCommand`, `askQuestions`, `openSimpleBrowser`)
- MCP servers from `.vscode/mcp.json` (Playwright MCP, filesystem, context7, tavily, etc.)

---

## Windows-Side VS Code Config

Directly editable from WSL — no manual user intervention needed:

| File | WSL Path |
|------|----------|
| Settings | `/mnt/c/Users/FaroukBakari/AppData/Roaming/Code/User/settings.json` |
| Keybindings | `/mnt/c/Users/FaroukBakari/AppData/Roaming/Code/User/keybindings.json` |
| MCP config | `/mnt/c/Users/FaroukBakari/AppData/Roaming/Code/User/mcp.json` |
| Snippets | `/mnt/c/Users/FaroukBakari/AppData/Roaming/Code/User/snippets/` |

---

## Dev Toolchain

| Tool | Version | Notes |
|------|---------|-------|
| Node | 22.20.0 | via nvm |
| Python | 3.11.9 | project venv at `backend/.venv` |
| Poetry | 2.2.1 | |
| Docker | 28.5.2 | Docker Desktop with WSL integration |
| Git | 2.34.1 | SSH key at `~/.ssh/id_rsa.pub` |
| gh CLI | 2.83.2 | |
| Make | 4.3 | |
| Playwright | 1.58.2 | npm; headless Chromium works in WSL |

### VS Code Extensions (notable)

| Extension | Purpose |
|-----------|---------|
| `anthropic.claude-code` | Claude Code extension (installed) |
| `vue.volar` | Vue 3 language support |
| `ms-python.python` + `vscode-pylance` | Python IDE |
| `dbaeumer.vscode-eslint` + `esbenp.prettier-vscode` | Linting/formatting |
| `eamodio.gitlens` | Git integration |
| `ms-toolsai.jupyter` | Jupyter notebooks |
| `ms-vscode.makefile-tools` | Makefile support |

---

## Running Services

| Service | Port | Details |
|---------|------|---------|
| **Ollama** | 11434 | v0.9.0, `llama3.2:1b` installed, GPU-accelerated (RTX 4090) |
| **n8n** | 5678 | Docker container (`n8nio/n8n:latest`) + PostgreSQL (5432) |
| **Docker Desktop** | — | Running, WSL integration active |

### IB TWS / Gateway

- **Installed** at `G:\Jts` (WSL: `/mnt/g/Jts/`)
- Configured: `tradingMode=p` (paper), timezone `Europe/Paris`, `ibkrBranding=pro`
- **Not currently running** — ports 4001/7497 closed
- When running, accessible via mirrored networking on `localhost`

---

## Browser Automation

| Method | Status | Use Case |
|--------|--------|----------|
| WSL headless Chromium | Works | Playwright tests, visual verification, CI |
| Windows Chrome (direct launch) | Works | Open URLs for user, demo |
| Playwright MCP Chrome profile | Installed at `~/.cache/ms-playwright/mcp-chrome-*` | Persistent browser sessions |

---

## Cross-Boundary Capabilities Summary

These are actions that span WSL ↔ Windows and are **confirmed working**:

1. **Launch Windows GUI apps** from WSL (Chrome, Explorer, etc.)
2. **Edit Windows-side files** directly (VS Code config, keybindings, any file on C:-G:)
3. **Share localhost** seamlessly (mirrored networking)
4. **Copy to clipboard** (`| clip.exe`)
5. **Run PowerShell/cmd** for Windows-specific operations
6. **Docker** works transparently across WSL ↔ Windows
7. **GPU compute** available for Ollama, CUDA workloads
