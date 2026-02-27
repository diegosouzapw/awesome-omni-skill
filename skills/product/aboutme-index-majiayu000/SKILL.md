---
name: aboutme-index
description: Index-based file discovery using ABOUTME headers. Use INSTEAD of grep or Explore agent when searching for files by purpose or feature. Faster and more accurate than scanning code. Invoke this skill when user asks "which files handle X", "where is Y implemented", or when you need to find files related to a feature or task.
---

# ABOUTME Index

Read `.claude/aboutme-index.json` to find files by purpose instead of grep-searching.

## Usage

1. Read the index:
```bash
cat .claude/aboutme-index.json
```

2. Filter by keyword using jq:
```bash
cat .claude/aboutme-index.json | jq 'to_entries[] | select(.value | test("auth|jwt"; "i"))'
```

3. Read the relevant files.

## Commands

| Command | Description |
|---------|-------------|
| `/aboutme-check` | Find files missing ABOUTME headers |
| `/aboutme-rebuild` | Rebuild the entire index |
| `/aboutme-stale` | Check for stale headers |
| `cat .claude/aboutme-index.json` | Read the index directly |

## Index Format

```json
{
  "app/src/auth.py": "JWT authentication module for AWS Cognito access tokens...",
  "app/src/config.py": "Centralized configuration using Pydantic settings..."
}
```

The index is auto-rebuilt on session start and updated incrementally on file edits.
