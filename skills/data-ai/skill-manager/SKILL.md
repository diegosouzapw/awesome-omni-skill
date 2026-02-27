---
name: skill-manager
description: Manage, sync, and publish Agent Skills across multiple AI platforms (Claude, Codex, Gemini, Copilot) and marketplace repositories. Use when users want to list skills, sync between platforms, publish to marketplace keys, mirror canonical skills, audit drift, or set up their environment. Triggers on phrases like "list skills", "sync skills", "publish skill", "skill marketplace", "deploy skill", "audit skills", or "skill inventory".
---

# Skill Manager

Manage Agent Skills across multiple AI platforms and tiered marketplace repositories.

## Core Capabilities

1. **Multi-Platform Sync** - Deploy skills across Claude Code, OpenAI Codex, Gemini CLI, and GitHub Copilot
2. **Inventory** - Discover and list all skills across platform locations
3. **Publish** - Push skills to tiered GitHub marketplace repositories (private/team/public)
4. **Marketplace Distribute** - Make marketplace-installed skills available to other AI platforms via symlinks
5. **Audit** - Compare skill versions, detect drift, identify inconsistencies
6. **Validate** - Check skill compliance with the Agent Skills specification

## Source of Truth: How Syncing Works

Skill-manager uses a **single source of truth** model. You designate one platform as your "source" where you create and edit skills, and skill-manager syncs them one-way to other platforms.

**Why one-way sync?**
- Prevents conflicts when the same skill exists on multiple platforms
- Clear ownership: you always know where the authoritative version lives
- Simple mental model: edit in one place, distribute everywhere

**Example:** If Claude Code is your source of truth:
- Create and edit skills in `~/.claude/skills/`
- Run `scripts/sync.py --all` to distribute to Codex, Gemini, Copilot
- Skills created directly in `~/.codex/skills/` will NOT sync back to Claude

**If you accidentally create a skill on the wrong platform:**
1. Copy it manually to your source platform's skills directory
2. Run sync to distribute it properly
3. Optionally delete the original from the non-source platform

**To change your source of truth:** Edit `config.json` and set `"is_source": true` on your preferred platform (and `false` on all others), or re-run `scripts/init.py`.

## Supported Platforms

| Platform | User Skills Path | Detection |
|----------|-----------------|-----------|
| Claude Code | `~/.claude/skills/` | `~/.claude/` exists |
| OpenAI Codex | `~/.codex/skills/` | `~/.codex/` exists |
| Gemini CLI | `~/.gemini/skills/` | `~/.gemini/` exists |
| GitHub Copilot | `~/.copilot/skills/` | `~/.copilot/` exists |

## Quick Start

### First-Time Setup

Run the interactive setup wizard:

```bash
scripts/init.py
```

This will:
1. Detect installed AI platforms on your system
2. Choose which platform is your "source of truth"
3. Configure which platforms to sync skills to
4. Set sync mode (symlink or copy)
5. Optionally configure marketplace repositories

### Sync Skills Between Platforms

After setup, sync a skill from your source platform to all configured targets:

```bash
scripts/sync.py ~/.claude/skills/my-skill
```

Or sync all skills at once:

```bash
scripts/sync.py --all
```

Options:
- `--to claude,codex,gemini,copilot` - Override target platforms
- `--to auto` - Auto-detect installed platforms
- `--to all` - Sync to all known platforms
- `--mode symlink|copy` - Override sync mode
- `--dry-run` - Preview changes without applying
- `--force` - Overwrite existing skills without prompting
- `--all` - Sync all skills from source platform

### List All Skills

```bash
scripts/inventory.py
```

Options:
- `--platform claude|codex|gemini|copilot|all` - Filter by platform
- `--format table|json|yaml` - Output format
- `--verbose` - Show full paths and metadata
- `--source global|project|marketplace` - Include selected source(s), repeatable
- `--exclude-source global|project|marketplace` - Exclude selected source(s), repeatable
- `--global-only|--project-only|--marketplace-only` - Single-source shortcuts
- `--no-marketplace` - Exclude marketplace entries
- `--include-marketplace` - Backward-compatible alias (marketplace is already included by default)

### Inventory Behavior

- Default `scripts/inventory.py` output includes all discovered sources:
  - global skills
  - project skills
  - marketplace skills
- Use source flags only when you want to narrow results.
- Legacy include forms remain supported:
  - `--include-marketplace`
  - `--include marketplace`
  - `--include=marketplace`
## Marketplace System

`skill-manager` works with marketplace keys from `config.json` (for example: `private`, `team`, `public`, `three-dna`, `enterprise-core`). Keys are identifiers only; repository names and plugin names can be arbitrary.

### Publish a Skill

```bash
scripts/publish.py ~/.claude/skills/my-skill --to team
```

This will:
1. Copy the skill to your team marketplace repo
2. Update marketplace.json
3. Update the README.md "Available Skills" table (for new skills)
4. Create a PR for review
5. Sync to other platforms (if configured)

If you don't specify `--to`, you'll be prompted to choose.

**Note on README updates:** When publishing a new skill, the README.md is automatically updated with the skill's entry in the "Available Skills" table. For updates to existing skills, README changes are optional and only made if the skill's description has changed.

### Sync Marketplace Repos

Pull latest from your marketplace repositories:

```bash
scripts/marketplace-sync.py
scripts/marketplace-sync.py --marketplace team
scripts/marketplace-sync.py --status
scripts/marketplace-sync.py --branch master
scripts/marketplace-sync.py --auto-stash
```

### Mirror Canonical Skills to Another Marketplace

Mirror from a canonical source ref (`tag`, `branch`, or `commit`) into a target marketplace with manifest and README updates:

```bash
scripts/marketplace-mirror.py mirror --from public --to three-dna --source-ref v1.2.0
scripts/marketplace-mirror.py mirror --from public --to three-dna --source-ref 8ab12cd --skill skill-manager
scripts/marketplace-mirror.py mirror --from public --to three-dna --source-ref main --dry-run
```

Generate drift reports at any time:

```bash
scripts/marketplace-mirror.py drift --from public --to three-dna
scripts/marketplace-mirror.py drift --from public --to three-dna --source-ref v1.2.0
```

### Distribute Marketplace Skills to Other Platforms

Make skills from your local marketplace clones available to other AI assistants (Codex, Gemini, Copilot) via symlinks. Claude Code is skipped by default since it has native marketplace access.

```bash
scripts/marketplace-distribute.py                       # Distribute all marketplace skills
scripts/marketplace-distribute.py --marketplace team    # Only from team marketplace
scripts/marketplace-distribute.py --skill my-skill      # Only distribute one skill
scripts/marketplace-distribute.py --to codex,gemini     # Only to specific platforms
scripts/marketplace-distribute.py --dry-run             # Preview what would happen
scripts/marketplace-distribute.py --list                # List available marketplace skills
scripts/marketplace-distribute.py --status              # Show distribution status
```

Options:
- `--marketplace NAME` - Only distribute from specific marketplace
- `--skill NAME` - Only distribute a specific skill by name
- `--to PLATFORMS` - Target platforms: codex, gemini, copilot, all
- `--include-claude` - Also create symlinks for Claude (normally skipped)
- `--dry-run` - Preview changes without applying
- `--force` - Overwrite existing skills without prompting
- `--list` - List available marketplace skills
- `--status` - Show current distribution status

**How it works:**
- Scans your local marketplace repo clones under `local_repos_path` from `config.json`
- Creates symlinks in target platform skill directories pointing to marketplace skills
- Skips Claude Code by default (it already has access via the plugin system)

### Audit Skills for Drift

```bash
scripts/audit.py <skill-name>
scripts/audit.py --all
```

### Validate a Skill

```bash
scripts/validate.py <skill-path>
```

### Operator Quick Commands

```bash
# List
scripts/inventory.py

# Publish
scripts/publish.py ~/.claude/skills/my-skill --to team

# Mirror
scripts/marketplace-mirror.py mirror --from public --to team --source-ref main --skill my-skill

# Audit/drift
scripts/marketplace-mirror.py drift --from public --to team
```

## Configuration

Configuration is stored in `config.json`:

```json
{
  "platforms": {
    "claude": {
      "enabled": true,
      "user_path": "~/.claude/skills",
      "is_source": true
    },
    "codex": {
      "enabled": true,
      "user_path": "~/.codex/skills",
      "is_source": false
    },
    "gemini": {
      "enabled": false,
      "user_path": "~/.gemini/skills",
      "is_source": false
    },
    "copilot": {
      "enabled": false,
      "user_path": "~/.copilot/skills",
      "is_source": false
    }
  },
  "sync_mode": "symlink",
  "marketplaces": {
    "private": {
      "repo": "https://github.com/you/skills-private",
      "description": "Personal/experimental skills",
      "visibility": "private"
    },
    "team": {
      "repo": "https://github.com/your-org/skills-team",
      "description": "Shared skills for collaborators",
      "visibility": "private"
    },
    "public": {
      "repo": "https://github.com/your-org/skills-public",
      "description": "Freely available skills",
      "visibility": "public"
    }
  },
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "local_repos_path": "~/GitHub"
}
```

### Config Fields

| Field | Description |
|-------|-------------|
| `platforms` | Platform configurations (enabled, path, source) |
| `sync_mode` | Default sync mode: `symlink` or `copy` |
| `marketplaces` | GitHub repo URLs keyed by marketplace name |
| `owner` | Your name and email for commits |
| `local_repos_path` | Where marketplace repos are cloned |

## Marketplace Repository Structure

Each marketplace repo follows this structure:

```
<repository-name>/
├── .claude-plugin/
│   └── marketplace.json    # Plugin catalog
├── skills/
│   ├── skill-one/
│   │   ├── SKILL.md
│   │   └── scripts/
│   └── skill-two/
│       └── SKILL.md
└── README.md
```

### marketplace.json Format

```json
{
  "name": "skills-team",
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "metadata": {
    "description": "Shared skills for collaborators",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "skills-team",
      "description": "Shared skills for collaborators",
      "source": "./",
      "strict": false,
      "skills": [
        "./skills/skill-one",
        "./skills/skill-two"
      ]
    }
  ]
}
```

## Workflow: Publishing Skills

### Step 1: Develop Locally

Create and test your skill in your source platform's skills directory:

```
my-skill/
├── SKILL.md           # Required
├── scripts/           # Optional
├── references/        # Optional
└── assets/            # Optional
```

### Step 2: Validate

```bash
scripts/validate.py ~/.claude/skills/my-skill
```

### Step 3: Publish

```bash
scripts/publish.py ~/.claude/skills/my-skill --to team
```

### Step 4: Merge PR

Review and merge the PR created in your marketplace repo.

### Step 5: Users Install

Users add your marketplace and install:

```
/plugin marketplace add your-org/your-marketplace-repo
```

Then browse skills in the `/plugin` UI under the Discover tab.

## Cross-Platform Skill Development

### Writing Portable Skills

- Avoid hardcoding platform-specific paths
- Use environment detection if behavior must differ
- Test on multiple platforms before publishing

### Sync Modes

| Mode | Pros | Cons |
|------|------|------|
| **symlink** | Changes propagate instantly, saves disk space | Requires symlink support |
| **copy** | Works everywhere, independent copies | Must re-sync to propagate changes |

### Skill Specification Compliance

| Field | Required | Max Length | Rules |
|-------|----------|------------|-------|
| `name` | Yes | 64 chars | Lowercase, hyphens, digits only |
| `description` | Yes | 1024 chars | No angle brackets. Include WHAT and WHEN |
| `license` | No | - | License name or file reference |

## Troubleshooting

### Skill Not Appearing After Publish

1. **Merge the PR** - Skills only appear after the PR is merged
2. **Refresh marketplace** - In `/plugin` UI, go to Marketplaces tab and press `u` to update
3. **Verify marketplace.json** - Check that the skill path is in the plugins array

### Sync Not Working

1. Check your config: `cat ~/.claude/skills/skill-manager/config.json`
2. Verify source platform is set: look for `"is_source": true`
3. Verify target platforms are enabled: look for `"enabled": true`
4. Run with `--dry-run` to preview: `scripts/sync.py --all --dry-run`

### Platform Not Detected

1. Ensure the platform is installed and has its config directory
2. Check paths in config.json match your system
3. Re-run `scripts/init.py` to reconfigure

### Publish Fails

1. Check that `gh` CLI is installed and authenticated
2. Verify repository URLs in config.json
3. Run `scripts/marketplace-sync.py --status` to check repo state

### Symlink Issues

```bash
# Check if symlink is valid
ls -la ~/.codex/skills/my-skill

# Check target exists
ls -la $(readlink ~/.codex/skills/my-skill)
```

## IDE Reload Reminder

After syncing or publishing skills, you may need to reload your IDE window for changes to take effect:

- **VS Code**: `Cmd+Shift+P` → "Reload Window"
- This is required because AI assistants load their skill inventory at startup

## See Also

- `skill-creator` skill - For creating new skills from scratch
- [Agent Skills Specification](https://agentskills.io/specification) - Full specification
