---
name: clawdhub-reskill-usage
description: Teaches AI agents how to use reskill — a Git-based package manager for AI agent skills. Covers CLI commands, install formats, configuration, publishing, and common workflows.
version: 0.1.0
author: reskill
tags:
  - cli
  - package-manager
  - skills
  - reskill
---

# reskill Usage Guide

reskill is a Git-based package manager for AI agent skills. It provides declarative configuration (`skills.json` + `skills.lock`), flexible versioning, and multi-agent support for installing, managing, and sharing skills across projects and teams.

## Security & Trust

This skill instructs the agent to run CLI commands via [reskill](https://github.com/nicepkg/reskill), an open-source, Git-based package manager for AI agent skills.

**Why a specific registry?**
reskill supports multiple registries. This skill defaults to the Rush community registry (`https://rush.zhenguanyu.com/`) — the primary public registry for the reskill ecosystem, similar to how npm defaults to `https://registry.npmjs.org`. Users can override it at any time via `--registry`, the `RESKILL_REGISTRY` environment variable, or `defaults.publishRegistry` in `skills.json`.

**CLI execution approach:**
To avoid downloading remote code on every invocation, we recommend installing reskill globally first:

```bash
npm install -g reskill
```

If a global installation is not available, `npx reskill@latest` can be used as a fallback. The agent should check for a global install (`which reskill`) before falling back to npx.

**Requirements:** Node.js >= 18.0.0

**Registry resolution** (highest to lowest priority):

1. `--registry <url>` CLI option
2. `RESKILL_REGISTRY` environment variable
3. `defaults.publishRegistry` in `skills.json`
4. Default: `https://rush.zhenguanyu.com/`

## When to Use This Skill

Use this skill when the user:

- Wants to install, update, or manage AI agent skills
- Mentions `skills.json`, `skills.lock`, or reskill-related concepts
- Wants to publish a skill to a registry
- Asks about supported install formats (GitHub, GitLab, HTTP, OSS, registry, etc.)
- Encounters reskill-related errors or needs troubleshooting
- Wants to set up a project for skill management
- Asks about multi-agent skill installation (Cursor, Claude Code, Codex, etc.)

## Quick Start

```bash
# Initialize a new project
reskill init

# Install a skill
reskill install github:anthropics/skills/skills/frontend-design@latest

# List installed skills
reskill list
```

## Commands

| Command               | Alias                | Description                               |
| --------------------- | -------------------- | ----------------------------------------- |
| `init`                | -                    | Initialize `skills.json`                  |
| `find <query>`        | -                    | Search for skills in the registry         |
| `install [skills...]` | `i`                  | Install one or more skills                |
| `list`                | `ls`                 | List installed skills                     |
| `info <skill>`        | -                    | Show skill details                        |
| `update [skill]`      | `up`                 | Update skills                             |
| `outdated`            | -                    | Check for outdated skills                 |
| `uninstall <skill>`   | `un`, `rm`, `remove` | Remove a skill                            |
| `publish [path]`      | `pub`                | Publish a skill to the registry ¹         |
| `login`               | -                    | Authenticate with the registry ¹          |
| `logout`              | -                    | Remove stored authentication ¹            |
| `whoami`              | -                    | Display current logged in user ¹          |
| `doctor`              | -                    | Diagnose environment and check for issues |
| `completion install`  | -                    | Install shell tab completion              |

> ¹ Registry commands (`publish`, `login`, `logout`, `whoami`) use the configured registry (default: `https://rush.zhenguanyu.com`).

Run `reskill <command> --help` for complete options and detailed usage.

### Common Options

| Option                    | Commands                             | Description                                                   |
| ------------------------- | ------------------------------------ | ------------------------------------------------------------- |
| `--no-save`               | `install`                            | Install without saving to `skills.json` (for personal skills) |
| `-g, --global`            | `install`, `uninstall`, `list`       | Install/manage skills globally (user directory)               |
| `-a, --agent <agents...>` | `install`                            | Specify target agents (e.g., `cursor`, `claude-code`)         |
| `--mode <mode>`           | `install`                            | Installation mode: `symlink` (default) or `copy`              |
| `--all`                   | `install`                            | Install to all agents                                         |
| `-y, --yes`               | `install`, `uninstall`, `publish`    | Skip confirmation prompts                                     |
| `-f, --force`             | `install`                            | Force reinstall even if already installed                     |
| `-s, --skill <names...>`  | `install`                            | Select specific skill(s) by name from a multi-skill repo      |
| `--list`                  | `install`                            | List available skills in the repository without installing    |
| `-r, --registry <url>`    | `install`                            | Registry URL override for registry-based installs             |
| `-j, --json`              | `list`, `info`, `outdated`, `doctor` | Output as JSON                                                |

## Source Formats

reskill supports installing skills from multiple sources:

```bash
# GitHub shorthand
reskill install github:user/skill@v1.0.0

# GitLab shorthand
reskill install gitlab:group/skill@latest

# Full Git URL (HTTPS)
reskill install https://github.com/user/skill.git

# Full Git URL (SSH)
reskill install git@github.com:user/skill.git

# GitHub/GitLab web URL (with branch and subpath)
reskill install https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines

# Custom registry (self-hosted GitLab, etc.)
reskill install gitlab.company.com:team/skill@v1.0.0

# HTTP/OSS archives
reskill install https://example.com/skills/my-skill-v1.0.0.tar.gz
reskill install oss://bucket/path/skill.tar.gz
reskill install s3://bucket/path/skill.zip

# Registry-based
reskill install @scope/skill-name@1.0.0 --registry https://rush.zhenguanyu.com
reskill install skill-name --registry https://rush.zhenguanyu.com

# Install multiple skills at once
reskill install github:user/skill1 github:user/skill2@v1.0.0
```

### Monorepo Support

For repositories containing multiple skills, specify the path to the skill directory:

```bash
# Shorthand format with subpath
reskill install github:org/monorepo/skills/planning@v1.0.0
reskill install gitlab:company/skills/frontend/components@latest

# URL format with subpath
reskill install https://github.com/org/monorepo.git/skills/planning@v1.0.0
reskill install git@gitlab.company.com:team/skills.git/backend/apis@v2.0.0

# GitHub web URL automatically extracts subpath
reskill install https://github.com/org/monorepo/tree/main/skills/planning
```

**Requirements**: The specified directory must contain a valid `SKILL.md` file following the [Agent Skills Specification](https://agentskills.io).

### HTTP/OSS URL Support

Skills can be installed directly from HTTP/HTTPS URLs pointing to archive files:

| Format       | Example                                                    | Description              |
| ------------ | ---------------------------------------------------------- | ------------------------ |
| HTTPS URL    | `https://example.com/skill.tar.gz`                         | Direct download URL      |
| Aliyun OSS   | `https://bucket.oss-cn-hangzhou.aliyuncs.com/skill.tar.gz` | Aliyun OSS URL           |
| AWS S3       | `https://bucket.s3.amazonaws.com/skill.tar.gz`             | AWS S3 URL               |
| OSS Protocol | `oss://bucket/path/skill.tar.gz`                           | Shorthand for Aliyun OSS |
| S3 Protocol  | `s3://bucket/path/skill.tar.gz`                            | Shorthand for AWS S3     |

**Supported archive formats:** `.tar.gz`, `.tgz`, `.zip`, `.tar`

### Version Formats

| Format | Example           | Description                        |
| ------ | ----------------- | ---------------------------------- |
| Exact  | `@v1.0.0`         | Lock to specific tag               |
| Latest | `@latest`         | Get the latest tag                 |
| Range  | `@^2.0.0`         | Semver compatible (>=2.0.0 <3.0.0) |
| Branch | `@branch:develop` | Specific branch                    |
| Commit | `@commit:abc1234` | Specific commit hash               |
| (none) | -                 | Default branch (main)              |

## Configuration

### skills.json

The project configuration file, created by `reskill init`:

```json
{
  "skills": {
    "planning": "github:user/planning-skill@v1.0.0",
    "internal-tool": "internal:team/tool@latest"
  },
  "registries": {
    "internal": "https://gitlab.company.com"
  },
  "defaults": {
    "installDir": ".skills",
    "targetAgents": ["cursor", "claude-code"],
    "installMode": "symlink"
  }
}
```

- `skills` — Installed skill references (name → source ref)
- `registries` — Custom Git registry aliases
- `defaults.installDir` — Where skills are stored (default: `.skills`)
- `defaults.targetAgents` — Default agents to install to
- `defaults.installMode` — `symlink` (default, recommended) or `copy`

### Environment Variables

| Variable            | Description                                     | Default                        |
| ------------------- | ----------------------------------------------- | ------------------------------ |
| `RESKILL_CACHE_DIR` | Global cache directory                          | `~/.reskill-cache`             |
| `RESKILL_TOKEN`     | Auth token (takes precedence over ~/.reskillrc) | -                              |
| `RESKILL_REGISTRY`  | Default registry URL                            | `https://rush.zhenguanyu.com`  |
| `DEBUG`             | Enable debug logging                            | -                              |
| `NO_COLOR`          | Disable colored output                          | -                              |

## Multi-Agent Support

Skills are installed to `.skills/` by default and can be integrated with any agent:

| Agent          | Path                                  |
| -------------- | ------------------------------------- |
| Cursor         | `.cursor/rules/` or `.cursor/skills/` |
| Claude Code    | `.claude/skills/`                     |
| Codex          | `.codex/skills/`                      |
| Windsurf       | `.windsurf/skills/`                   |
| GitHub Copilot | `.github/skills/`                     |
| OpenCode       | `.opencode/skills/`                   |

Use `--agent` to target specific agents, or `--all` to install to all detected agents:

```bash
# Install to specific agents
reskill install github:user/skill -a cursor claude-code

# Install to all detected agents
reskill install github:user/skill --all
```

## Publishing

### Authentication

```bash
# Login with a token (obtain from the registry web UI)
reskill login --registry https://rush.zhenguanyu.com --token <token>

# Check current login status
reskill whoami

# Logout
reskill logout
```

Tokens are stored in `~/.reskillrc`. You can also use the `RESKILL_TOKEN` environment variable (takes precedence, useful for CI/CD).

### Publishing a Skill

```bash
# Validate without publishing (recommended first step)
reskill publish --dry-run --registry https://rush.zhenguanyu.com

# Publish the skill
reskill publish --registry https://rush.zhenguanyu.com

# Publish from a specific directory
reskill publish ./path/to/skill --registry https://rush.zhenguanyu.com

# Skip confirmation
reskill publish -y --registry https://rush.zhenguanyu.com
```

The skill directory must contain a valid `SKILL.md`. A `skill.json` with `name`, `version`, and `description` is also required for publishing.

## Common Workflows

### First-Time Project Setup

```bash
# 1. Initialize the project
reskill init -y

# 2. Install skills your project needs
reskill install github:user/skill1@v1.0.0 github:user/skill2@latest -y

# 3. Verify installation
reskill list

# 4. Commit skills.json and skills.lock to version control
# (These files ensure team members get the same skill versions)
```

### Team Collaboration

When a teammate clones the project, they run:

```bash
# Reinstall all skills from skills.json (like npm install)
reskill install
```

This reads `skills.json` + `skills.lock` and installs the exact same versions.

### Checking and Updating Skills

```bash
# Check which skills have newer versions
reskill outdated

# Update all skills
reskill update

# Update a specific skill
reskill update skill-name
```

### Global vs Project-Level Installation

| Scope   | Flag | Directory               | Use Case                                   |
| ------- | ---- | ----------------------- | ------------------------------------------ |
| Project | -    | `.skills/` (in project) | Team-shared skills, committed to git       |
| Global  | `-g` | `~/.agents/skills/`     | Personal skills, available in all projects |

```bash
# Project-level (default)
reskill install github:user/skill

# Global (personal, all projects)
reskill install github:user/skill -g

# Personal project-level (not saved to skills.json)
reskill install github:user/skill --no-save
```

### Diagnosing Issues

```bash
# Run environment diagnostics
reskill doctor

# JSON output for programmatic use
reskill doctor --json
```

The `doctor` command checks: reskill version, Node.js version, Git availability, cache directory, `skills.json` validity, `skills.lock` sync, installed skills integrity, and network connectivity.

## Troubleshooting

| Error Message                        | Cause                                 | Solution                                                 |
| ------------------------------------ | ------------------------------------- | -------------------------------------------------------- |
| `skills.json not found`              | Project not initialized               | Run `reskill init`                                       |
| `Unknown scope @xyz`                 | No registry configured for this scope | Check `registries` in `skills.json` or use full Git URL  |
| `Skill not found`                    | Skill name doesn't exist in registry  | Verify skill name; check `reskill find <query>`          |
| `Version not found`                  | Requested version doesn't exist       | Run `reskill info <skill>` to see available versions     |
| `Permission denied`                  | Auth issue when publishing            | Run `reskill login`; check token scope                   |
| `Token is invalid or expired`        | Stale authentication                  | Re-authenticate with `reskill login --token <new-token>` |
| `Network error`                      | Cannot reach Git host or registry     | Check network; run `reskill doctor` for diagnostics      |
| `Conflict: directory already exists` | Skill already installed               | Use `--force` to reinstall                               |

### Private Repositories

reskill uses your existing git credentials (SSH keys or credential helper). For CI/CD environments:

```bash
# GitLab CI
git config --global url."https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.company.com/".insteadOf "https://gitlab.company.com/"
```
