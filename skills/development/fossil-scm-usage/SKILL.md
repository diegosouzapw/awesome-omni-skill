---
name: Fossil SCM Usage
description: This skill should be used when the user asks to "fossil commit", "fossil branch", "fossil merge", "fossil clone", "fossil sync", "fossil ticket", "fossil stash", "fossil timeline", mentions working with a Fossil repository, asks about Fossil vs Git differences, or needs help with Fossil SCM commands and workflows.
version: 1.0.0
---

# Fossil SCM Command Line Reference

Comprehensive guidance for managing projects with Fossil SCM from the command line, covering repository management, commits, branching, multi-user workflows, and the ticketing system.

## Overview

Fossil is a distributed version control system combining source code management with an integrated wiki, ticketing system, and web interface—all in a single executable. Unlike Git, Fossil emphasizes simplicity, autosync by default, and feature-centric branching.

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| Repository | SQLite database file (`.fossil`) containing all project history |
| Working checkout | Directory where files are edited, linked via `fossil open` |
| Artifact | Any versioned item (file, commit, ticket) identified by SHA hash |
| Autosync | Default behavior that automatically pushes/pulls on commit/update |

## Core Differences from Git

Understanding these differences prevents common mistakes:

1. **Autosync is ON by default** - Commits automatically push, updates automatically pull
2. **Branches created during commit** - Use `fossil commit --branch name` not `git checkout -b`
3. **Single repository file** - All history in one `.fossil` SQLite file
4. **Built-in features** - Wiki, tickets, forum included (not separate tools)
5. **No staging area** - All changed files commit together (use `fossil commit file1 file2` for partial)

## Essential Workflows

### Starting a New Project

```bash
fossil init myproject.fossil
mkdir myproject && cd myproject
fossil open ../myproject.fossil
fossil add .
fossil commit -m "Initial commit"
```

### Cloning and Working

```bash
fossil clone https://example.com/repo project.fossil
mkdir work && cd work
fossil open ../project.fossil
```

### Feature Branch Workflow

```bash
# Start feature (creates branch during commit)
fossil commit --branch feature-login -m "Start login feature"

# Work on feature
fossil commit -m "Add login form"
fossil commit -m "Add validation"

# Merge back to trunk
fossil update trunk
fossil merge --integrate feature-login
fossil commit -m "Merged feature-login"
```

### Collaborative Workflow (Autosync)

With autosync enabled (default), collaboration is automatic:

```bash
# Alice commits - automatically pushes
fossil commit -m "Alice's changes"

# Bob updates - automatically pulls Alice's changes
fossil update
```

### Handling Mistakes

```bash
# Move bad commit to "mistake" branch
fossil amend HEAD --branch mistake
fossil amend HEAD --close
fossil update trunk

# Undo last update/merge/revert
fossil undo

# Revert uncommitted changes
fossil revert
```

## Quick Reference

| Task | Command |
|------|---------|
| Create repo | `fossil init repo.fossil` |
| Clone repo | `fossil clone URL repo.fossil` |
| Open repo | `fossil open repo.fossil` |
| Check status | `fossil status` or `fossil changes` |
| Add files | `fossil add file` or `fossil addremove` |
| Commit | `fossil commit -m "msg"` |
| Update | `fossil update` |
| New branch | `fossil commit --branch name` |
| Switch branch | `fossil update branchname` |
| List branches | `fossil branch list` |
| Merge | `fossil merge branchname` |
| Cherry-pick | `fossil merge --cherrypick HASH` |
| View diff | `fossil diff` |
| View history | `fossil timeline` |
| File history | `fossil finfo filename` |
| Annotate/blame | `fossil annotate filename` |
| Stash changes | `fossil stash save -m "msg"` |
| Apply stash | `fossil stash pop` |
| Sync manually | `fossil sync` |
| Push only | `fossil push` |
| Pull only | `fossil pull` |
| Add ticket | `fossil ticket add title "..." status "Open"` |
| Update ticket | `fossil ticket set UUID status "Closed"` |
| Ticket history | `fossil ticket history UUID` |
| List tickets | `fossil ticket show "All Tickets"` |
| Web UI | `fossil ui` |

## Sync and Remote Operations

### Autosync Settings

```bash
# Check current setting
fossil settings autosync

# Enable (default)
fossil settings autosync on

# Disable for manual control
fossil settings autosync off

# Pull only (no auto-push)
fossil settings autosync pullonly
```

### Manual Sync

```bash
fossil sync              # Full sync (push + pull)
fossil push              # Push only
fossil pull              # Pull only
fossil sync --private    # Include private branches
```

### Remote Management

```bash
fossil remote            # Show current remote
fossil remote add URL    # Set default remote
fossil remote list       # List all remotes
```

## Stash Operations

```bash
fossil stash save -m "WIP"     # Save and revert working dir
fossil stash snapshot -m "msg"  # Save but keep working dir
fossil stash list               # List stashes
fossil stash show               # Show most recent as diff
fossil stash pop                # Apply and remove
fossil stash apply              # Apply but keep
fossil stash drop 1             # Delete specific stash
```

## Tags

```bash
# Add during commit
fossil commit --tag v1.0.0 -m "Release"

# Add to existing commit
fossil tag add v1.0.0 HASH

# List tags
fossil tag list

# Remove tag
fossil tag cancel v1.0.0 HASH
```

## Important Settings

| Setting | Description |
|---------|-------------|
| `autosync` | Auto push/pull on commit/update |
| `editor` | Text editor for commit messages |
| `ignore-glob` | Patterns for files to ignore |
| `binary-glob` | Patterns for binary files |
| `case-sensitive` | Case sensitivity for filenames |

```bash
fossil settings                    # List all
fossil settings autosync off       # Set local
fossil settings autosync off --global  # Set global
fossil unset autosync              # Revert to global
```

## Detecting Fossil Repositories

A Fossil working checkout contains either:
- `_FOSSIL_` file (Unix/Linux)
- `.fslckout` file (Windows or when using `--dotfiles`)

Check with: `ls -la _FOSSIL_ .fslckout 2>/dev/null`

## Web Interface

```bash
fossil ui                 # Open local web UI in browser
fossil ui --port 9000     # Specific port
fossil server --port 8080 # External access
```

## Additional Resources

### Reference Files

For comprehensive command documentation with all options and examples:

- **`references/commands.md`** - Complete Fossil command reference including:
  - Repository setup (init, clone, open, close)
  - File operations (add, rm, mv, status)
  - Commit options and amending
  - Branching operations
  - Merging and conflict resolution
  - Ticketing system commands
  - Diff and history commands
  - Undo and revert operations

Consult `references/commands.md` for detailed syntax and options not covered in this quick reference.

## Version Information

Based on Fossil SCM version 2.27. Use `fossil help COMMAND` for authoritative documentation on the installed version.
