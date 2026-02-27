---
name: memos_crud
description: Perform CRUD operations on a Memos instance (create, read, update, delete memos).
---

# Instructions

Use this skill when you need to interact with a Memos server to manage notes/memos.
This skill provides a Ruby CLI wrapper around the Memos REST API.
Always prefer using this CLI over raw `curl` requests to ensure consistent error handling and state management.

## prerequisites
- Ensure the `memos_skill` directory is your working directory or you reference the script path correctly.
- The environment variables in `.env` must be configured (checked automatically by the script).

## Commands

Run the CLI using `ruby scripts/memos_cli.rb <command> [options]`.
All commands return JSON.

### 1. List Memos

Fetch a list of memos. default limit is 50.

```bash
ruby scripts/memos_cli.rb list --page-size 10 --state NORMAL
```

**Options:**
- `--page-size N`: Number of items (default 50).
- `--page-token TOKEN`: For pagination.
- `--state STATE`: `NORMAL` or `ARCHIVED`.
- `--filter "content.contains('keyword')"`: Filter by content.

### 2. Create Memo

Create a new memo.

```bash
ruby scripts/memos_cli.rb create --content "My new memo content" --visibility PRIVATE
```

**Options:**
- `--content "STRING"`: (Required) The content of the memo.
- `--visibility VIS`: `PRIVATE` (default), `PROTECTED`, or `PUBLIC`.
- `--pinned BOOL`: `true` or `false`.

### 3. Get Memo

Get a single memo by details.

```bash
ruby scripts/memos_cli.rb get --memo memos/123
```

**Options:**
- `--memo ID`: The memo ID (e.g., `123` or `memos/123`).

### 4. Update Memo

Update an existing memo. Only detailed fields are updated.

```bash
ruby scripts/memos_cli.rb update --memo memos/123 --content "Updated content" --pinned true
```

**Options:**
- `--memo ID`: (Required) Target memo.
- `--content "STRING"`
- `--visibility VIS`
- `--pinned BOOL`
- `--state STATE`: Archive/Unarchive using `ARCHIVED` or `NORMAL`.

### 5. Delete Memo

Delete a memo permanently (or soft delete if the system supports it, but this CLI assumes standard delete).

```bash
ruby scripts/memos_cli.rb delete --memo memos/123
```

**Options:**
- `--memo ID`: (Required) Target memo.
- `--force true`: params to force delete if applicable.

## Output Format

**Success:**
```json
{
  "ok": true,
  "data": { ...memo object... }
}
```

**Error:**
```json
{
  "ok": false,
  "error": "Error message",
  "http_status": 400,
  "payload": { ... }
}
```
