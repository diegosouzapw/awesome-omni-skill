---
name: vimango-cli
description: Manage vimango notes from the command line. Use when you need to create, search, retrieve, or update notes in the vimango notes system backed by PostgreSQL.
---

# vimango-cli

A CLI tool for managing notes in the vimango notes system. Notes are organized by **context** (category/topic) and **folder**, and support full-text search.

## Commands

### List contexts and folders

```bash
vimango-cli contexts
vimango-cli folders
```

Always check available contexts/folders before creating or updating notes so you use valid names.

### Search notes

```bash
vimango-cli search "query" [--limit N]
```

- Query must be at least 3 characters
- Default limit is 5 results
- Returns: rank, title, context, folder, and tid for each match

### Get a note

```bash
vimango-cli get TID
```

- TID is the integer primary key
- Returns the full note content with title, context, folder, and body

### Create a note

```bash
vimango-cli create --title "Title" --note "Body" [--context name] [--folder name] [--star]
```

- `--title` and `--note` are required
- Context and folder default to "none" if not specified
- Use `--star` to favorite the note
- Returns the tid of the created note

### Update a note

```bash
vimango-cli update TID [--title "New Title"] [--context name] [--folder name] [--star/--no-star]
```

- At least one of `--title`, `--context`, `--folder`, or `--star`/`--no-star` must be provided
- Only the specified fields are changed

## Workflow tips

1. **Before creating/updating with a context or folder**, run `vimango-cli contexts` or `vimango-cli folders` to confirm the name exists.
2. **To find a note**, use `vimango-cli search` first, then `vimango-cli get TID` to read the full content.
3. **Exit codes**: 0 on success, 1 on errors (missing context/folder, no results, note not found).
4. **Errors go to stderr**, normal output goes to stdout.
