---
name: memories-cli
description: "CLI reference and workflows for memories.sh — the persistent memory layer for AI agents. Use when: (1) Running memories CLI commands to add, search, edit, or manage memories, (2) Managing lifecycle workflows (session/checkpoint/compaction/consolidation/OpenClaw memory files), (3) Setting up memories.sh in a new project (memories init), (4) Generating AI tool config files (CLAUDE.md, .cursor/rules, etc.), (5) Importing existing rules from AI tools (memories ingest), (6) Managing cloud sync, embeddings, git hooks, or reminders, (7) Troubleshooting with memories doctor, (8) Working with memory templates, links, history, tags, or reminder schedules."
---

# memories-cli

CLI reference for `@memories.sh/cli` — manage memories, generate configs, and sync across tools.

> **The CLI is the primary way to interact with memories.sh.** Use it to store memories, generate native config files, and manage your memory store. For environments where the CLI isn't available (v0, bolt.new, Lovable, or other browser-based agents), use the [MCP server](../memories-mcp/SKILL.md) as a fallback.

## Install & Init

```bash
npm install -g @memories.sh/cli   # or: npx @memories.sh/cli
memories init                      # Initialize in current project
```

`memories init` auto-detects AI tools (Claude Code, Cursor, Windsurf, VS Code) and configures MCP + generates instruction files.

## Command Quick Reference

| Command | Purpose |
|---------|---------|
| `memories add <content>` | Store a memory |
| `memories recall` | Context-aware memories for current project |
| `memories search <query>` | Full-text search |
| `memories list` | List memories with filters |
| `memories edit <id>` | Edit content, type, or tags |
| `memories forget <id>` | Soft-delete a memory |
| `memories generate` | Generate AI tool config files |
| `memories prompt` | Generate a system prompt |
| `memories serve` | Start MCP server |
| `memories session <subcommand>` | Manage explicit sessions (`start`, `checkpoint`, `status`, `end`, `snapshot`) |
| `memories compact run` | Run inactivity compaction worker |
| `memories consolidate run` | Merge duplicates and supersede stale truths |
| `memories openclaw memory <subcommand>` | OpenClaw file-mode workflows (`bootstrap`, `flush`, `snapshot`, `sync`) |
| `memories reminders` | Manage cron reminders (`add`, `list`, `run`, `enable`, `disable`, `delete`) |

## Core Workflows

### 1. New Project Setup

```bash
cd my-project
memories init              # Detect tools, configure MCP, generate files
memories add "Use pnpm" --type rule
memories add "Chose Supabase for auth" --type decision
memories generate          # Update all AI tool configs
```

### 2. Ingest Existing Rules

```bash
memories ingest claude     # Import from CLAUDE.md
memories ingest cursor     # Import from .cursorrules / .cursor/rules/
memories ingest copilot    # Import from copilot-instructions.md
```

### 3. Search & Recall

```bash
memories search "auth"                    # Full-text search
memories search "auth" --semantic         # Vector similarity (requires embeddings)
memories recall                           # Context for current project
memories list --type rule                 # Filter by type
memories list --tags api,auth             # Filter by tags
```

### 4. Generate Configs

```bash
memories generate                         # All detected tools
memories generate claude                  # Only CLAUDE.md
memories generate cursor                  # Only .cursor/rules/memories.mdc
memories diff                             # Preview changes before generating
```

Supported targets: `claude`, `cursor`, `copilot`, `windsurf`, `cline`, `roo`, `gemini`

### 5. Cloud Sync

```bash
memories login                            # Device code auth flow
memories sync                             # Sync local DB to cloud
memories files ingest                     # Upload config files
memories files apply --global --force     # Restore configs on new machine
```

### 6. Embeddings

```bash
memories embed                            # Generate embeddings for all memories
memories embed --dry-run                  # Preview what would be embedded
memories config model <model-name>        # Change embedding model
```

### 7. Maintenance

```bash
memories doctor                           # Diagnose issues
memories stats                            # Memory statistics
memories stale --days 90 --conflicts-only # Find stale conflicting memories
memories review --superseded-only         # Interactive superseded cleanup
memories validate                         # Check memory integrity
```

### 8. Session Lifecycle + Compaction

```bash
# Start session
memories session start --title "checkout timeout triage" --client codex

# Add checkpoints as work progresses
memories session checkpoint <session-id> "Root cause narrowed to auth callback timeout" --kind summary

# Run inactivity compaction worker (batch job)
memories compact run --inactivity-minutes 60 --limit 25

# End session and optionally snapshot
memories session end <session-id> --status closed
memories session snapshot <session-id> --trigger manual
```

### 9. Consolidation + OpenClaw Memory Files

```bash
# Preview consolidation impact first
memories consolidate run --types rule,decision,fact --dry-run

# Apply consolidation
memories consolidate run --types rule,decision,fact

# OpenClaw memory file workflows
memories openclaw memory bootstrap
memories openclaw memory flush <session-id>
memories openclaw memory snapshot <session-id> --trigger reset
memories openclaw memory sync --direction both
```

### 10. Reminders

```bash
memories reminders add "0 9 * * 1-5" "Review open TODOs"
memories reminders list
memories reminders run
```

## Memory Types

Use `--type` flag with `add`:
- **rule** — `memories add "Always use strict mode" --type rule`
- **decision** — `memories add "Chose JWT for auth" --type decision`
- **fact** — `memories add "Rate limit: 100/min" --type fact`
- **note** — (default) `memories add "Refactor auth module"`

## Scopes

- **project** (default) — Scoped to current git repo
- **global** — `memories add "Use TypeScript" --type rule --global`

When using MCP instead of CLI commands (for example from browser tools or agents running outside the repo), use `add_memory` with `project_id` to force project scope.

## Advanced Features

- **Templates**: `memories add --template decision` — structured prompts for common patterns
- **Links**: `memories link <id1> <id2> --type supports` — relate memories
- **History**: `memories history <id>` / `memories revert <id> --to <version>`
- **Tags**: `memories tag <id> add api,auth`
- **Export/Import**: `memories export > backup.yaml` / `memories import backup.yaml`
- **Git Hooks**: `memories hook install` — auto-generate on commit
- **Reminders**: `memories reminders ...` — cron-style prompts persisted in the local DB

## Reference Files

- **Full command reference**: See [references/commands.md](references/commands.md) for all commands with complete options and flags
- **Workflow recipes**: See [references/workflows.md](references/workflows.md) for multi-step recipes and automation patterns
