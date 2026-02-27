---
name: cowork-guide
description: "CRITICAL: Comprehensive guide for CoWork Skills CLI tool. Triggers on: cowork, Skills.toml, skill management, plugin configuration, cowork init, cowork install, cowork config, cowork generate"
---

# CoWork Guide

> Complete reference for the CoWork Skills CLI tool

## Overview

**CoWork Skills** (`cowork` / `co`) is a CLI tool for managing Claude Code skills and plugins:
- Install skills from GitHub repositories
- Generate skills from source code (Rust, TypeScript, Python)
- Manage project-level skill configuration via Skills.toml
- Search GitHub for skill repositories

## Installation

```bash
# Clone and install
git clone https://github.com/anthropics/cowork-skills
cd cowork-skills/cli
cargo install --path .

# Initialize built-in skills
cowork init
```

## Commands Overview

| Command | Description |
|---------|-------------|
| `cowork init` | Install built-in skills to ~/.claude/skills/ |
| `cowork install` | Install skills from GitHub or local path |
| `cowork generate` | Generate skills from a GitHub repository |
| `cowork search` | Search GitHub for skill repositories |
| `cowork plugins` | Manage Claude Code marketplace plugins |
| `cowork config` | Manage project-level skill configuration |
| `cowork list` | List all available skills |
| `cowork status` | Show current configuration |
| `cowork doctor` | Check for configuration issues |
| `cowork test` | Generate trigger tests for skills |

Use `co` as a short alias for `cowork`.

---

## cowork init

Initialize built-in skills to ~/.claude/skills/.

```bash
# Install all built-in skills
cowork init

# List available built-in skills
cowork init --list

# Install specific skills only
cowork init -s memory-filesystem

# Force overwrite existing
cowork init --force
```

**Built-in Skills:**
- `memory-filesystem` - CoALA-based memory system (remember, recall, reflect)

---

## cowork install

Install skills from GitHub or local repositories.

### Basic Usage

```bash
# Install from GitHub (user/repo format)
cowork install ZhangHanDong/rust-skills

# Install from full URL
cowork install https://github.com/user/repo

# Install current project to global
cowork install

# List installed repositories
cowork install --list
```

### Installation Options

```bash
# Install specific skills only
cowork install user/repo -s skill1 -s skill2

# Install to specific agents
cowork install user/repo -a claude-code -a cursor

# Install as plugin (preserves full repo structure)
cowork install user/repo --plugin

# Install to project local (.claude/skills/)
cowork install user/repo --local

# Force reinstall
cowork install user/repo --reinstall

# Update to latest version
cowork install user/repo --update

# Skip confirmation prompts
cowork install user/repo -y
```

### Uninstall

```bash
cowork install --uninstall repo-name
```

### Supported Agents

Install skills to 16+ coding agents:

| Agent | Flag |
|-------|------|
| Claude Code | `--agent claude-code` |
| Cursor | `--agent cursor` |
| Codex | `--agent codex` |
| GitHub Copilot | `--agent github-copilot` |
| Windsurf | `--agent windsurf` |
| Goose | `--agent goose` |
| Kiro | `--agent kiro-cli` |
| Amp | `--agent amp` |
| ... | See `cowork install --help` |

---

## cowork config

Manage project-level skill configuration via Skills.toml.

### Initialize Configuration

```bash
# Initialize with auto-detection of installed plugins/skills
cowork config init

# Skip auto-detection
cowork config init --no-detect

# Force overwrite existing
cowork config init --force
```

Auto-detection scans:
- `~/.claude/` for global plugins
- `~/.claude/skills/` for global skills
- `.claude/` for project plugins
- `.claude/skills/` for project skills

### View Configuration

```bash
# Show current config
cowork config show

# List available skill groups
cowork config groups
```

### Add Dependencies

```bash
# Add from GitHub (global)
cowork config add rust-skills ZhangHanDong/rust-skills

# Add to project local
cowork config add my-skills user/skills --local

# Add specific skills
cowork config add tokio user/tokio-skills -s tokio-runtime -s tokio-sync

# Add as plugin
cowork config add makepad user/makepad-skills --plugin

# Add as disabled
cowork config add old-lib user/old --disabled

# Add with git ref
cowork config add pinned user/repo --ref v1.0.0

# Add development link (symlink for testing)
cowork config add dora-dev /path/to/dora-skills --dev
```

### Install Dependencies

```bash
# Install all dependencies from Skills.toml
cowork config install
```

### Sync Configuration

```bash
# Sync enabled/disabled status with lock file
cowork config sync

# Also update remote repos (git pull)
cowork config sync --update
```

### Enable/Disable

```bash
# Enable skill groups
cowork config enable rust-core makepad

# Enable individual skills
cowork config enable memory-filesystem

# Disable skills or groups
cowork config disable rust-domains
```

### Trigger Configuration

```bash
# Set priority order
cowork config priority dora-router rust-router makepad-router

# Override specific trigger
cowork config override "async" rust-router
cowork config override "widget" makepad-router
```

### Generate Output

```bash
# Generate SKILLS.md from config
cowork config apply

# Generate to custom path
cowork config apply -o ./docs/SKILLS.md

# Generate dynamic router based on installed plugins
cowork config router

# Generate router with hooks for auto-triggering
cowork config router --hooks
```

---

## Skills.toml

Project-level configuration file stored at `.cowork/Skills.toml`.

### Basic Structure

```toml
[project]
name = "my-project"
description = "Project description"

[skills.global]
enabled = ["memory-filesystem"]
disabled = []

[skills.install]
rust-skills = "ZhangHanDong/rust-skills"

[skills.groups]
enabled = ["rust-core"]
disabled = []

[triggers]
priority = ["dora-router", "rust-router"]

[triggers.overrides]
"async" = "rust-router"
```

### Dependency Forms

**Simple form:**
```toml
[skills.install]
rust-skills = "ZhangHanDong/rust-skills"
```

**Detailed form:**
```toml
[skills.install]
tokio = { repo = "user/tokio-skills", skills = ["tokio-runtime", "tokio-sync"] }
local = { path = "../my-local-skills" }
pinned = { repo = "user/repo", ref = "v1.0.0" }
makepad = { repo = "user/makepad-skills", plugin = true }
my-project = { repo = "user/skills", local = true }
old-lib = { repo = "user/old", enabled = false }
```

### Dev Links

Development symlinks for testing local skills:

```toml
[skills.dev]
my-skill = "/path/to/my-skill-project"
dora-dev = { path = "/path/to/dora-skills" }
dora-plugin = { path = "/path/to/dora-skills", plugin = true }
```

### Skill Groups

| Group | Skills | Description |
|-------|--------|-------------|
| `rust-core` | 8 | Basic Rust (ownership, concurrency, error handling) |
| `rust-patterns` | 7 | Design patterns (domain modeling, performance) |
| `rust-domains` | 7 | Domain-specific (web, CLI, fintech, embedded) |
| `makepad` | 11 | Makepad UI framework |
| `dora` | 8 | Dora-rs robotics framework |
| `dora-hubs` | 9 | Dora hub integrations |

---

## cowork generate

Generate skills from source code of any GitHub repository.

```bash
# Generate from GitHub repo
cowork generate user/repo

# Specify language(s)
cowork generate tokio-rs/tokio --lang rust
cowork generate vercel/next.js --lang typescript

# Only generate llms.txt
cowork generate user/repo --llms-only -o ./output

# Generate from existing llms.txt
cowork generate --from-llms ./llms.txt

# Specify git ref
cowork generate user/repo --ref v1.0.0
```

### Supported Languages

| Language | Parser | Extracts |
|----------|--------|----------|
| Rust | `syn` | pub fn, struct, enum, trait, impl |
| TypeScript | `tree-sitter` | export function, class, interface, type |
| Python | `tree-sitter` | def, class (excluding `_` private items) |

### Output

1. **llms.txt** - API documentation following llms.txt specification
2. **SKILL.md** - Generated skill files with triggers and references

---

## cowork search

Search GitHub for skill repositories.

```bash
# Search by keyword
cowork search tokio

# Search by GitHub topic
cowork search agent-skill --topic

# Limit results
cowork search rust-skills --limit 5

# Show detailed results
cowork search rust-skills --verbose
```

---

## cowork plugins

Manage Claude Code marketplace plugins.

```bash
# List marketplace plugins
cowork plugins list

# Show plugin status
cowork plugins status

# Uninstall a plugin
cowork plugins uninstall rust-skills

# Enable/disable plugins
cowork plugins enable rust-skills
cowork plugins disable rust-skills

# List marketplaces
cowork plugins marketplaces

# Remove a marketplace
cowork plugins remove-marketplace rust-skills
```

---

## Storage Locations

| Location | Purpose |
|----------|---------|
| `~/.cowork/repos/` | Cloned GitHub repositories |
| `~/.claude/skills/` | Global skills directory |
| `~/.claude/<plugin>/` | Global plugins |
| `.claude/skills/` | Project-local skills |
| `.claude/<plugin>/` | Project-local plugins |
| `.cowork/Skills.toml` | Project configuration |
| `.cowork/Skills.lock` | Installed packages lock |

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_TOKEN` | Required for generate/search commands |

---

## Troubleshooting

### Check Configuration

```bash
cowork doctor
cowork status
```

### Common Issues

**Skills not loading:**
1. Check if skill exists: `cowork list`
2. Verify enabled status: `cowork config show`
3. Ensure triggers match: `cowork test triggers`

**Installation fails:**
1. Check GitHub token: `echo $GITHUB_TOKEN`
2. Verify repo exists: `gh repo view user/repo`
3. Try with verbose: `cowork install user/repo --reinstall`

**Plugin conflicts:**
1. Set trigger priority: `cowork config priority router1 router2`
2. Override specific triggers: `cowork config override "keyword" skill-name`
3. Check for conflicts: `cowork test --check-conflicts`
