---
name: mercurial-hg
description: Comprehensive Mercurial (Hg) version control system guide covering all operations from basic to advanced. Use when Claude needs to work with Mercurial for (1) Repository initialization and cloning, (2) Basic operations (status, add, commit, pull, push), (3) Branch management and merging, (4) Advanced operations (rebase, histedit, graft), (5) Team collaboration workflows, (6) Patch queue (MQ) operations, (7) Configuration and extensions, or (8) Troubleshooting common issues.
---

# Mercurial Hg

Mercurial is a distributed version control system (DVCS) suitable for scenarios ranging from small personal projects to large-scale enterprise projects.

## Quick Start

### Initialize New Repository

```bash
hg init myproject
cd myproject
```

### Clone Existing Repository

```bash
hg clone https://example.com/repo myproject
```

### Basic Workflow

```bash
# Check current status
hg status

# Add files
hg add file1 file2

# Commit changes
hg commit -m "Describe your changes"

# Pull remote changes
hg pull

# Push local changes
hg push
```

## Common Scenarios

### View History

```bash
# Short history
hg log

# Detailed history
hg log -v

# History for specific file
hg log path/to/file

# Graphic history
hg log -G
```

### Undo Operations

```bash
# Revert uncommitted changes
hg revert file

# Revert all uncommitted changes
hg revert --all

# Backout to specific changeset (keep history)
hg backout <revision>

# Strip to specific changeset (remove history)
hg strip <revision>
```

## When to Read Reference Documentation

| Scenarios | Reference Doc |
|-----------|---------------|
| Detailed basic commands, understanding file status | [basics.md](references/basics.md) |
| Create, switch, and merge branches | [branches.md](references/branches.md) |
| Complex merges, conflict resolution, rebase, histedit | [advanced.md](references/advanced.md) |
| Team collaboration, code review, multi-repo management | [collaboration.md](references/collaboration.md) |
| Patch Queue (MQ) operations | [mq.md](references/mq.md) |
| Configure ~/.hgrc, enable extensions, custom aliases | [config.md](references/config.md) |
| Troubleshoot common issues, recover lost commits | [troubleshooting.md](references/troubleshooting.md) |

## Comparison with Git

| Mercurial | Git |
|-----------|-----|
| hg commit | git commit |
| hg pull + hg update | git pull |
| hg push | git push |
| hg branches | git branch -r |
| hg bookmark | git branch |
| hg merge | git merge |
| hg rebase | git rebase |
| hg share | git worktree |

## Best Practices

1. **Check before commit**: Use `hg diff` and `hg status` to confirm changes
2. **Descriptive commit messages**: Commit messages should clearly explain changes
3. **Pull frequently**: Use `hg pull -u` to regularly get remote updates
4. **Use bookmarks**: For branching workflows, use bookmarks instead of named branches
5. **Handle conflicts**: When merging conflicts, carefully check each conflicted file
6. **Get help**: For any unclear command, use `hg help --verbose <command>` to view official documentation

## Important Concepts

- **Changeset**: Basic unit of commit in Mercurial, identified by hash or revision number
- **Repository**: Complete copy containing project history and current working directory
- **Working Directory**: Files currently being edited
- **Tip**: The latest changeset of the current branch
- **Head**: A changeset with no successors
