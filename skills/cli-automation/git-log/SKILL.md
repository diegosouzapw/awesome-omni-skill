---
name: git-log
description: Show commit history for a file or the entire repository. Returns commit information including hash, author, date, and message.
allowed-tools: Bash
metadata:
  category: git
  version: 1.0.0
---

# Git Log Skill

This skill retrieves commit history for a file or the entire repository, providing detailed information about each commit.

## Usage

The git-log skill retrieves commit history with optional filtering by path, author, and date range.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| path | string | No | - | Filter history to specific file |
| limit | int | No | 10 | Maximum number of commits to return |
| since | datetime | No | - | Only show commits after this date |
| until | datetime | No | - | Only show commits before this date |
| author | string | No | - | Filter by author name or email |

### Example Usage

Get recent repository history:
```
Show the last 10 commits
```

Get history for a specific file:
```
Show commit history for /src/main.go
```

Filter by author:
```
Show commits by author "Alice"
```

Filter by date range:
```
Show commits from the last week
```

### Result Format

Each commit includes:
- Full commit hash (40 characters)
- Short hash (7 characters)
- Author name and email
- Author timestamp
- Committer name and email (if different)
- Commit timestamp
- Subject (first line of message)
- Full commit message
- Parent commit hashes
- Files changed in the commit

### Commit Information Fields

| Field | Description |
|-------|-------------|
| hash | Full 40-character commit hash |
| short_hash | Abbreviated 7-character hash |
| author | Commit author name |
| author_email | Author's email address |
| author_time | When the commit was authored |
| committer | Person who applied the commit |
| committer_email | Committer's email |
| commit_time | When commit was applied |
| subject | First line of commit message |
| message | Full commit message |
| parent_hashes | Parent commit references |
| files_changed | List of files modified |

### Best Practices

1. Start with a reasonable limit (10-20) for performance
2. Use path filter to focus on specific file history
3. Combine author and date filters for targeted searches
4. Check for merge commits (multiple parents) when tracing changes
5. Use short_hash for display, full hash for references
