---
name: google-email
description: interact w/ Google Gmail inbox
---

# Google Email

## Overview

The `google-email` command is installed globally.

It is a dual-mode email management system for Gmail with offline analysis capabilities.

It provides:

1. **Online Mode**: Pull emails from Gmail to local Markdown storage
2. **Offline Mode**: Query and manage the stored email database without Gmail connectivity

The system is designed for AI agents to integrate email processing into workflows, supporting:
- Batch email ingestion with deduplication
- Email marking (read/unread) with offline metadata
- Pattern analysis on stored emails
- Integration with other tools and scripts

## Core Concepts

### Email IDs
- **Gmail ID**: Alphanumeric identifier from Gmail API
- **SHA1 Hash**: 40-character hex hash of Gmail ID, used as filename
- **Short ID**: First 6 characters of hash (e.g., `6498ce`), used for Git-like partial matching

Example:
```
Gmail ID:   18d676f08ff64932bf93e7ec3c0adb2b
SHA1 Hash:  6498cec18d676f0328ff649bf933e7ec3c0adb2b
Short ID:   6498ce
```

### Storage Format
Each email is stored as a Markdown file in `storage/` with YAML front matter containing metadata, and the HTML body in a code block:

```markdown
---
id: '18d676f08ff64932bf93e7ec3c0adb2b'
threadId: '18d676f08ff64932'
subject: 'Project Status Update'
from:
  name: 'Alice Smith'
  address: 'asmith@company.com'
toRecipients:
  - name: 'Team'
    address: 'team@company.com'
receivedDateTime: '2026-01-05T10:30:00Z'
isRead: false
labelIds:
  - 'INBOX'
  - 'UNREAD'
webLink: 'https://mail.google.com/mail/u/0/#inbox/18d676f08ff64932bf93e7ec3c0adb2b'
body:
  contentType: html
_stored_id: '6498cec18d676f08ff64932bf93e7ec33c0adb2b'
_stored_at: '2026-01-05T18:05:13.476Z'
offline:
  read: true
  readAt: '2026-01-05T18:06:00.000Z'
---

# Project Status Update

```html
<html>
<head>...</head>
<body>
<p>Email content here...</p>
</body>
</html>
```
```

### ID Matching (Git-style)
All CLI commands support partial IDs. The system matches the longest unique prefix:

```bash
# These all refer to the same email:
google-email inbox view 6498cec18d676f08ff64932bf93e7ec33c0adb2b  # Full (40 chars)
google-email inbox view 6498cec18d676f08                          # 16 chars
google-email inbox view 6498ce                                     # 6 chars (short)
google-email inbox view 6498                                       # 4 chars (if unique)
google-email inbox view 6498cec18d676f08ff64932bf93e7ec33c0adb2b.md  # Filename format
```

Error on ambiguity:
```
Error: Ambiguous ID "62". Matches: 62e8e2d5adb20b15..., 62b19cb17ec4628a...
```

## Online Mode: Fetching Emails from Gmail

### Command: `google-email pull`

**Purpose**: Fetch unread emails from Gmail, store locally as Markdown files, mark as read/processed in Gmail.

> **IMPORTANT**: When instructed to fetch more emails, use this command. **Always fetch exactly one email at a time** (`--limit 1`) unless specifically directed to pull more.

**Command**:
```bash
google-email pull --since <date> [--limit N]
```

**Parameters**:
- `--since <date>`: Required. Fetch emails received on/after this date
  - Formats: `YYYY-MM-DD`, `yesterday`, `"7 days ago"`, `"1 day ago"`
- `--limit <n>`: Optional. Stop after processing N emails (default: no limit)

**Behavior**:
1. Fetches all unread emails from inbox since date
2. Paginates through results
3. Skips already-stored files (deduplication via SHA1)
4. Stores each new email as Markdown under `storage/<id>.md`
5. Marks processed emails as read in Gmail
6. Adds "Processed" label and archives emails

**Examples**:

```bash
# RECOMMENDED: Pull exactly 1 new email (safest, most controlled)
google-email pull --since yesterday --limit 1

# Pull from specific date, one at a time
google-email pull --since 2026-01-01 --limit 1

# Pull from last week, one at a time
google-email pull --since "7 days ago" --limit 1

# Pull multiple emails (only when specifically instructed)
google-email pull --since yesterday --limit 5
```

**Output Example**:
```
Fetching unread emails since: 2026-01-05
Processing limit: 1
Found 14 unread emails.
✓ Stored: (71c95a+NOTICE: Password Expiration)
  → Marking as read...
  → Adding Processed label and archiving...
  ✓ Updated in Gmail

Reached processing limit of 1. Stopping.

Summary:
  Available:  14
  Processed:  1
  Written:    1
  Skipped:    0
```

## Offline Mode: Analysis & Metadata

### Local Query & Management

All `google-email` commands work purely offline, reading/writing Markdown files in `storage/`.

### Command: `google-email inbox summary`

**Purpose**: Get overall email statistics

**Command**:
```bash
google-email inbox summary
```

**Output**:
```
Folder Summary:
===============
Inbox:
  Unread: 46
  Read:   1
  Total:  47

Overall:
  Unread: 46
  Read:   1
  Total:  47
```

### Command: `google-email inbox list [OPTIONS]`

**Purpose**: List emails with filtering and display

**Command**:
```bash
google-email inbox list [--limit N] [--since DATE] [--all]
```

**Options**:
- `-l, --limit <n>`: Max results (default: 10)
- `--since <date>`: Filter emails after date
- `-a, --all`: Include read emails (default: unread only)

**Output Example**:
```
Showing 3 of 46 emails:

f86bca / Today 10:00 / Science Operations <ScienceOperations@company.com>
Science record review request

cda64a / Today 9:46 / Alice Johnson <alice@company.com>
Re: [team] Create new feature (PR #537)

8fb5bf / Today 9:01 / U.S. Payroll <payroll@company.com>
Payroll for 2026

... and 43 more
```

### Command: `google-email inbox view <id>`

**Purpose**: Display full YAML content of an email

**Command**:
```bash
google-email inbox view <id>
```

**Supports**: Partial IDs, full IDs, or filename format

### Command: `google-email inbox read <id>`

**Purpose**: Mark an email as read (offline only)

**Command**:
```bash
google-email inbox read <id>
```

**Effect**:
- Adds/updates `offline.read: true` to YAML front matter
- Adds `offline.readAt` timestamp
- Does NOT sync back to Gmail (offline only)

### Command: `google-email inbox unread <id>`

**Purpose**: Mark an email as unread (offline only, reverses read state)

**Command**:
```bash
google-email inbox unread <id>
```

## Integration Patterns for AI Agents

### Pattern 1: Fetch & Analyze

```bash
# Typical agent workflow
1. google-email pull --since yesterday       # Get new emails
2. google-email inbox summary                # Get stats
3. google-email inbox list --limit 50        # Scan subjects
4. google-email inbox view <id>              # Get full content
5. <agent analyzes>
6. google-email inbox read <id>              # Mark processed
```

### Pattern 2: Incremental Processing

```bash
# Process in batches to avoid overwhelming
1. google-email pull --since yesterday --limit 20    # Get batch
2. for each email:
   a. google-email inbox view <id> | extract content
   b. Send to AI for analysis
   c. Store result in database
   d. google-email inbox read <id>  # Mark done
3. Repeat if more emails available
```

## Data Access Examples

### Extract Sender Name
```bash
google-email inbox view 6498ce | yq '.from.name'
# Output: "Alice Smith"
```

### Extract All Recipients
```bash
google-email inbox view 6498ce | yq '.toRecipients[].address'
```

### Extract Email Subject
```bash
google-email inbox view 6498ce | yq '.subject'
```

### Extract Received Date
```bash
google-email inbox view 6498ce | yq '.receivedDateTime'
```

### Check Read Status (Offline)
```bash
google-email inbox view 6498ce | yq '.offline.read'
```

## Tips for AI Agents

1. **Always check summary first**: `google-email inbox summary`
2. **Use partial IDs**: Shorter `6498ce` instead of full hash
3. **Pipe to tools**: `google-email inbox view <id> | yq '.field'`
4. **Test with limit**: Use `--limit 5` or `--limit 1` when prototyping
5. **Mark processed**: Always `read` after processing to track state
6. **Use relative dates**: `yesterday`, `"7 days ago"` are clearer

## Architecture Notes

- **Storage**: All files in `storage/` are Markdown with YAML front matter (Git-friendly)
- **No database**: Direct filesystem access via Bun file API
- **Stateless**: Each command is independent
- **Deduplication**: SHA1 hashing ensures same Gmail ID = same storage file
- **Offline metadata**: `offline.*` fields are never overwritten by pull
- **Single direction**: `pull` syncs Gmail → storage; `read/unread` mark locally only
