---
name: aisync
description: Sync AI coding sessions from 14 tools (Claude Code, Codex, Cursor, Aider, Cline, Gemini CLI, Continue, Copilot, Roo Code, Windsurf, Zed AI, Amp, OpenCode, OpenRouter) to Obsidian vault as markdown notes. Use when user wants to backup, export, or sync their AI chat sessions to Obsidian, set up automatic syncing, check sync status, or troubleshoot sync issues. Handles secret redaction automatically. Cross-platform (macOS, Linux, Windows).
---

# AI Sessions Sync v2.2.0

Sync AI coding sessions from **14 different tools** to multiple output formats with analytics, search, and automatic secret redaction.

## Supported Providers (14)

| Provider | Location | Output Folder |
|----------|----------|---------------|
| Claude Code | `~/.claude/projects/**/*.jsonl` | `claude-code-sessions/` |
| Codex CLI | `~/.codex/sessions/**/*.jsonl` | `codex-sessions/` |
| Cursor | `~/.cursor/projects/**/agent-transcripts/*.txt` | `cursor-sessions/` |
| Aider | `~/.aider.chat.history.md` | `aider-sessions/` |
| Cline | VS Code globalStorage | `cline-sessions/` |
| Gemini CLI | `~/.gemini/` | `gemini-cli-sessions/` |
| Continue.dev | `~/.continue/sessions/` | `continue-sessions/` |
| GitHub Copilot | VS Code globalStorage | `copilot-chat-sessions/` |
| Roo Code | VS Code globalStorage | `roo-code-sessions/` |
| Windsurf | Codeium/Windsurf app data | `windsurf-sessions/` |
| Zed AI | `~/.config/zed/conversations/` | `zed-ai-sessions/` |
| Amp (Sourcegraph) | VS Code globalStorage | `amp-sessions/` |
| OpenCode | `~/.local/share/opencode/` | `opencode-sessions/` |
| OpenRouter | `~/Downloads/openrouter*.json` (exports) | `openrouter-sessions/` |

## Output Formats (5)

| Format | Description |
|--------|-------------|
| `obsidian` | Markdown with YAML frontmatter for Obsidian |
| `json` | JSON files (single or per-session) |
| `jsonl` | JSON Lines for streaming/processing |
| `html` | Static website with search |
| `sqlite` | SQLite database with full-text search |

## CLI Commands

```
🤖 AI Sessions Sync v2.0.0

COMMANDS:
  sync       Sync sessions to output format(s)
  search     Search across all sessions  
  stats      Show usage statistics
  report     Generate detailed report
  status     Show detected sessions
  providers  List supported AI tools
  outputs    List output formats
  config     Get/set configuration

QUICK START:
  aisync sync                    # Sync to Obsidian
  aisync sync -f json html       # Sync to JSON + HTML
  aisync search "function"       # Search sessions
  aisync stats                   # View statistics
```

### `aisync sync` - Sync Sessions

```bash
aisync sync [options]

OPTIONS:
    -o, --output DIR       Output directory (default: auto-detect vault)
    -f, --format FORMAT    Output format(s): obsidian, json, jsonl, html, sqlite
    -p, --provider PROV    Only sync specific provider(s)
    --no-analyze           Skip analytics computation
    --json                 Output results as JSON

EXAMPLES:
    aisync sync                          # Sync to Obsidian
    aisync sync -o ~/ai-sessions         # Custom directory
    aisync sync -f obsidian sqlite       # Multiple formats
    aisync sync -p claude-code cursor    # Specific providers
```

### `aisync search` - Search Sessions

```bash
aisync search <query> [options]

OPTIONS:
    -p, --provider PROV    Filter by provider
    -l, --limit N          Max results (default: 20)
    --regex                Use regex pattern
    --json                 Output as JSON

EXAMPLES:
    aisync search "async function"       # Simple search
    aisync search "error" -p cursor      # Filter by provider
    aisync search "def \w+\(" --regex    # Regex search
```

### `aisync stats` - View Statistics

```bash
aisync stats [-f text|json]

OUTPUT INCLUDES:
    - Total sessions and messages
    - Token usage estimates
    - Sessions by provider
    - Top programming languages
    - Activity patterns
```

### `aisync report` - Generate Report

```bash
aisync report [-o FILE]

REPORT INCLUDES:
    - Overview (sessions, messages, tokens)
    - Insights (productivity patterns, streaks)
    - Breakdown by tool and language
```

### Other Commands

```bash
aisync status      # Show vault location and session counts
aisync providers   # List all 12 supported tools
aisync outputs     # List output formats
aisync config      # Get/set configuration
aisync config OBSIDIAN_VAULT "~/vault"  # Set vault path
```

## Installation

Run the installer:

```bash
cd ~/.claude/skills/aisync/scripts
./install.sh
```

This will:
1. Install Python library and CLI
2. Set up automatic syncing (platform-specific)
3. Run initial sync

## Cross-Platform Support

| Platform | Scheduler | Auto-Install |
|----------|-----------|--------------|
| macOS | launchd | ✅ Automatic |
| Linux | systemd/cron | ✅ Automatic |
| Windows | Task Scheduler | 📋 Manual (instructions provided) |

## Configuration

```bash
# Option 1: Environment variable
export OBSIDIAN_VAULT="/path/to/your/vault"

# Option 2: Config file
echo 'OBSIDIAN_VAULT="/path/to/your/vault"' > ~/.aisync.conf

# Option 3: CLI
aisync config OBSIDIAN_VAULT "/path/to/vault"
```

## Features

### Analytics
- Token usage estimation
- Language detection in code blocks
- Activity patterns (peak hours, streaks)
- Usage insights

### Search
- Full-text search across all sessions
- Regex support
- Filter by provider, date
- Find similar sessions

### Secret Redaction
Automatically redacts 20+ patterns:
- API keys (OpenAI, Anthropic, Google, AWS)
- GitHub tokens
- Database URLs
- Private keys
- JWT/Bearer tokens

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No sessions found | Check if AI tools are installed and have sessions |
| Sync not running | Run `aisync status` to check |
| Vault not found | Set `OBSIDIAN_VAULT` env var |
| Search not working | Ensure sessions are synced first |

## Library Files

The skill includes a modular Python library in `lib/`:

```
lib/
├── __init__.py          # Main API (sync_all, etc.)
├── cli.py               # Command-line interface
├── models.py            # Data models (Session, Message)
├── redact.py            # Secret redaction
├── search.py            # Search functionality
├── parsers/             # 12 provider parsers
├── outputs/             # 5 output formats
└── analytics/           # Analytics & insights
```
