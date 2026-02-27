---
name: gitea
description: "Gitea operations via gitea-mcp and tea CLI. Use when user mentions: gitea, tea, or when git remote shows a Gitea instance."
---

# Gitea Skill

This plugin provides two ways to interact with Gitea:

1. **gitea-mcp** (preferred) - MCP server providing 80+ tools for direct API integration
2. **tea CLI** (fallback) - Command-line interface for Gitea operations

## When to Use This Skill

Activate this skill when:
- User mentions "gitea", "tea", or Gitea-related operations
- Working with repositories hosted on Gitea instances (not GitHub/GitLab)
- Managing issues, pull requests, releases, or repos
- Automating Gitea workflows

**Detection**: Run `git remote -v` to check if the remote URL points to a Gitea instance (doesn't contain `github.com` or `gitlab`).

## Tool Selection Priority

### 1. Try gitea-mcp First (Preferred)

The MCP server provides 80+ tools for:
- Repository management (create, fork, list, search)
- Issues and PRs (CRUD, comments, reviews)
- Branches, tags, and releases
- File operations (read/write)
- Actions (workflows, secrets, runs)
- Organizations and teams
- Wiki pages

**Requirements:**
- `GITEA_ACCESS_TOKEN` environment variable set
- `gitea-mcp` binary installed and in PATH

### 2. Fall Back to tea CLI When:

- `GITEA_ACCESS_TOKEN` is not set
- gitea-mcp tools return authentication errors
- MCP server is not available
- User explicitly requests tea CLI

### 3. If Both Fail, Offer to Install

If both gitea-mcp and tea CLI are unavailable or fail:
1. Inform the user about the issue
2. Ask if they want help installing the requirements
3. Refer to the plugin README for installation instructions

## Authentication

### For gitea-mcp (environment variable)

```bash
export GITEA_ACCESS_TOKEN=your_token
export GITEA_HOST=https://your-gitea-instance.com  # optional, default: https://gitea.com
```

### For tea CLI (interactive login)

```bash
# Check existing logins
tea login list

# Add new login (interactive)
tea login add
```

**Generate tokens**: Gitea web UI → Settings → Applications → Generate New Token

## Behavioral Guidelines

1. **Try MCP First**: Attempt gitea-mcp tools before falling back to tea CLI
2. **Before PR Creation**: Check for uncommitted changes with `git status`
3. **Before CI Operations**: Verify pipeline status before triggering new runs
4. **Machine-Readable Output**: When using tea CLI, use `-o json` or `-o yaml` flags for automation
5. **Custom SSH Remotes**: Always use `--repo owner/repo` flag when remote URL is non-standard (see below)

## Handling Custom SSH Remotes

**Problem**: tea CLI auto-detects the repository from git remote URLs, but fails with custom SSH aliases like:
- `ngit@gitea:owner/repo.git`
- `git@custom-host:owner/repo.git`
- Any non-standard SSH URL format

**Error**: `Error: path segment [0] is empty`

**Solution**: Always specify the repo explicitly with `--repo owner/repo`:

```bash
# Check remote URL format first
git remote -v

# If remote uses custom SSH alias (e.g., ngit@gitea:...), always use --repo
tea pr create --repo owner/repo --title "Title" --description "Description"
tea issue list --repo owner/repo
tea pr list --repo owner/repo

# Standard URLs (git@gitea.com:...) work without --repo
tea pr create --title "Title" --description "Description"
```

**Detection**: Run `git remote -v` at the start. If the URL doesn't match standard patterns (`git@gitea.com:` or `https://gitea.com/`), extract `owner/repo` from the path and use `--repo` for all tea commands.

---

## tea CLI Reference (Fallback)

Use these commands when gitea-mcp is unavailable or returns errors.

### Pull Requests

| Operation | Command |
|-----------|---------|
| Create PR | `tea pr create --title "Title" --description "Description"` |
| List PRs | `tea pr list` |
| List open PRs | `tea pr list --state open` |
| View PR | `tea pr view <id>` |
| Checkout PR | `tea pr checkout <id>` |
| Merge PR | `tea pr merge <id>` |
| Close PR | `tea pr close <id>` |
| Reopen PR | `tea pr reopen <id>` |
| Review PR | `tea pr review <id> --approve` |
| Clean merged branches | `tea pr clean` |

### Issues

| Operation | Command |
|-----------|---------|
| Create issue | `tea issue create --title "Title" --body "Description"` |
| List issues | `tea issue list` |
| List open issues | `tea issue list --state open` |
| List closed issues | `tea issue list --state closed` |
| View issue | `tea issue view <id>` |
| Close issue | `tea issue close <id>` |
| Reopen issue | `tea issue reopen <id>` |
| Add comment | `tea issue comment <id> "Comment text"` |
| Edit issue | `tea issue edit <id> --title "New Title"` |

### Repositories

| Operation | Command |
|-----------|---------|
| Clone repo | `tea repo clone <owner/repo>` |
| Fork repo | `tea repo fork <owner/repo>` |
| Create repo | `tea repo create --name <name>` |
| Create from template | `tea repo create-from-template <template-owner/repo> --name <name>` |
| List repos | `tea repos list` |
| Search repos | `tea repos search <query>` |

### Releases

| Operation | Command |
|-----------|---------|
| List releases | `tea release list` |
| Create release | `tea release create --tag <tag> --title "Title"` |
| Delete release | `tea release delete <tag>` |

### Labels

| Operation | Command |
|-----------|---------|
| List labels | `tea label list` |
| Create label | `tea label create --name "bug" --color "#ff0000"` |
| Delete label | `tea label delete <name>` |
| Export labels | `tea label export` |

### Milestones

| Operation | Command |
|-----------|---------|
| List milestones | `tea milestone list` |
| Create milestone | `tea milestone create --title "v1.0"` |
| Close milestone | `tea milestone close <name>` |

### Organizations

| Operation | Command |
|-----------|---------|
| List orgs | `tea org list` |
| View org | `tea org view <name>` |

### Notifications

| Operation | Command |
|-----------|---------|
| List notifications | `tea notifications` |
| Mark as read | `tea notifications --mark-read` |

### Time Tracking

| Operation | Command |
|-----------|---------|
| List time entries | `tea times list` |
| Add time | `tea times add <issue-id> <duration>` |
| Delete time | `tea times delete <id>` |

### Authentication

| Operation | Command |
|-----------|---------|
| List logins | `tea login list` |
| Add login | `tea login add` |
| Set default | `tea login default <name>` |
| Delete login | `tea login delete <name>` |

## Output Formats

Tea supports multiple output formats for automation:

```bash
# JSON output
tea issue list -o json
tea pr list -o json

# YAML output
tea issue list -o yaml

# Simple/minimal output
tea issue list -o simple
```

## Common Workflows

### Create a Pull Request

```bash
# Check for uncommitted changes first
git status

# Check remote URL format
git remote -v

# If using custom SSH remote (e.g., ngit@gitea:...), use --repo explicitly
tea pr create --repo owner/repo --title "Add new feature" --description "This PR adds..."

# For standard remotes, --repo is optional
tea pr create --title "Add new feature" --description "This PR adds..."

# Or interactively
tea pr create
```

### Work on an Issue

```bash
# View the issue
tea issue view 42

# Checkout related PR if exists
tea pr checkout 42

# Add a comment
tea issue comment 42 "Working on this now"

# Close when done
tea issue close 42
```

### Quick Repository Setup

```bash
# Clone a repository
tea repo clone owner/repo

# Or fork first then clone
tea repo fork owner/repo
tea repo clone your-username/repo
```

### CI/Actions Secrets

```bash
# List secrets
tea repo secrets list

# Add a secret
tea repo secrets add <name> <value>

# Delete a secret
tea repo secrets delete <name>
```

## Global Flags

| Flag | Description |
|------|-------------|
| `-l, --login` | Use a specific login |
| `-r, --repo` | Override repository |
| `-o, --output` | Output format (simple, table, csv, tsv, yaml, json) |
| `--remote` | Use specific git remote |

## Installation

```bash
# macOS
brew install tea

# Arch Linux
pacman -S tea

# Alpine
apk add tea

# From binary
# Download from https://dl.gitea.com/tea/

# Docker
docker pull gitea/tea
```

## Resources

- Official Repository: https://gitea.com/gitea/tea
- Documentation: https://docs.gitea.com/usage/command-line
- Downloads: https://dl.gitea.com/tea/
