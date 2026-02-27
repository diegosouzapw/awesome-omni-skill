---
name: ai-agent-config
description: Manage AI coding skills across platforms (Claude Code, Antigravity, Cursor, Windsurf) using ai-agent-config CLI. Use when the user wants to sync skills to/from GitHub, install to multiple platforms, add custom skill sources, or configure skill management settings.
---

# AI Agent Config Management

Complete guide for the `ai-agent-config` CLI tool - universal skill management across AI coding platforms.

## Core Commands

### GitHub Sync

```bash
# Initialize with repository
ai-agent init --repo https://github.com/username/my-skills.git

# Push local skills to GitHub
ai-agent push --message "Added new skills"

# Pull skills from GitHub
ai-agent pull

# Bi-directional sync
ai-agent sync --message "Sync latest"
```

### Installation

```bash
# Install to all detected platforms
ai-agent install

# Force reinstall
ai-agent install --force

# Install specific skill
ai-agent install --skill backend-patterns
```

### Source Management

```bash
# Add custom source
ai-agent source add https://github.com/company/skills.git \
  --name company-skills \
  --branch main

# List sources
ai-agent source list

# Enable/disable
ai-agent source enable company-skills
ai-agent source disable company-skills
```

### Configuration

```bash
# Get/set config values
ai-agent config get repository.url
ai-agent config set repository.autoSync true

# Edit in $EDITOR
ai-agent config edit

# Validate config
ai-agent config validate
```

### Platform Info

```bash
# Show detected platforms
ai-agent platforms

# List installed skills
ai-agent list
```

## Common Workflows

### First-Time Setup

**User**: "Set up ai-agent-config with my GitHub"

**Assistant**:
```bash
# Initialize with your repository
ai-agent init --repo https://github.com/yourname/my-ai-skills.git

# This will:
# 1. Create ~/.ai-agent/config.json
# 2. Clone your repo to ~/.ai-agent/sync-repo
# 3. Enable auto-sync (pull before push)

# Push current skills
ai-agent push --message "Initial setup"
```

### Daily Workflow

**User**: "Update all my skills"

**Assistant**:
```bash
# Pull latest from GitHub (auto-installs)
ai-agent pull
```

### Adding Custom Skills

**User**: "Add skills from my company repo"

**Assistant**:
```bash
# Add custom source
ai-agent source add https://github.com/company/ai-skills.git \
  --name company-skills \
  --branch main

# Update from new source (auto-installs)
ai-agent update --source company-skills
```

### Team Sharing

**User**: "Share my skills with team"

**Assistant**:
```bash
# Push to GitHub
ai-agent push --message "Added debugging skills"

# Team members pull (auto-installs)
ai-agent pull
```

## Architecture

### File Locations
- **Config**: `~/.ai-agent/config.json`
- **Sync Repo**: `~/.ai-agent/sync-repo/` (local clone)
- **Cache**: `~/.ai-agent-config-cache/` (external sources)

### Platform Paths
- Claude Code: `~/.claude/skills/`
- Antigravity: `~/.gemini/antigravity/skills/`
- Cursor: `~/.cursor/skills/`
- Windsurf: `~/.windsurf/skills/`
- Codex: `~/.codex/skills/`

## Configuration

Key config settings:

```json
{
  "repository": {
    "url": "https://github.com/username/skills.git",
    "branch": "main",
    "autoSync": true
  },
  "sources": {
    "official": [...],
    "custom": [...]
  }
}
```

## Examples

### Setup for Team

```bash
# Create shared repository
ai-agent init --repo https://github.com/team/shared-skills.git

# Add company skills
ai-agent source add https://github.com/company/official-skills.git \
  --name company

# Push setup
ai-agent push --message "Team setup complete"
```

### Migration

```bash
# Initialize ai-agent-config
ai-agent init --repo https://github.com/username/skills.git

# Install to all platforms
ai-agent install --force

# Push existing skills
ai-agent push --message "Migrated to ai-agent-config"
```

## Troubleshooting

### Skills Not Installing

```bash
# Force reinstall
ai-agent install --force

# Check platforms detected
ai-agent platforms
```

### GitHub Sync Conflicts

Auto-sync handles most conflicts automatically. If issues persist:

```bash
cd ~/.ai-agent/sync-repo
git status
```

### Config Issues

```bash
# Validate
ai-agent config validate

# Reset if corrupted
ai-agent config reset
```

## Tips

- **Auto-Sync**: Keep enabled to prevent conflicts
- **Regular Pulls**: Pull frequently for latest skills
- **Descriptive Messages**: Use `--message` for clear commit messages
- **Force Reinstall**: Use `--force` when skills aren't updating
