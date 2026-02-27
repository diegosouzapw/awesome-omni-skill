---
name: slack-chat
description: communicate via slack-chat channels and direct messages with users
---

# Slack Chat

## Overview

The `slack` command is installed globally (located in the `$PATH`).

It is a dual-mode Slack client with offline-first architecture:

1. **Online Mode**: Pull messages from Slack to local Markdown storage, resolve IDs
2. **Offline Mode**: Query and manage stored messages without Slack connectivity

The system is designed for AI agents to integrate Slack processing into workflows, supporting:
- Batch message ingestion with deduplication
- Message marking (read/unread) with offline metadata
- ID resolution caching (users, channels)
- Integration with other tools and scripts

Uses [browser-use](https://github.com/browser-use/browser-use) (via Chrome DevTools Protocol) for Slack API access.

## Core Concepts

### Message IDs
- **Slack ID**: `channel_id:timestamp` or `channel_id:timestamp@thread_ts` for threads
- **SHA1 Hash**: 40-character hex hash of Slack ID, used as filename
- **Short ID**: First 6 characters of hash (e.g., `b89c7a`), used for Git-like partial matching

Example:
```
Slack ID:   C01TECH01:1767815267.099869
SHA1 Hash:  b89c7a14755df6dca57ece8e14f38652e727cbd8
Short ID:   b89c7a

# Thread reply:
Slack ID:   G01GROUP01:1765909149.353759@1765321208.614079
SHA1 Hash:  4cea849d8625c020f8716b8024e256942d6b440b
Short ID:   4cea84
```

### Storage Format
Each message is stored as a Markdown file in `storage/` with YAML front matter containing metadata, and rendered content below:

```markdown
---
channel_id: C01TECH01
timestamp: '1767815267.099869'
thread_ts: null
user_id: WNARLG5HB
type: message
text: Thank you all!
permalink: https://bigco-producta.slack.com/archives/C01TECH01/p1767815267099869
reactions: []
attachments: []
files: []
_stored_id: b89c7a14755df6dca57ece8e14f38652e727cbd8
_stored_at: '2026-01-07T20:06:28.667864+00:00'
offline:
  read: false
---

# Message in C01TECH01

**From:** WNARLG5HB
**Channel:** C01TECH01
**Timestamp:** 1767815267.099869
**Permalink:** [https://...](https://...)

---

Thank you all!
```

### ID Matching (Git-style)
All CLI commands support partial IDs. The system matches the longest unique prefix:

```bash
# These all refer to the same message:
slack-chat inbox view b89c7a14755df6dca57ece8e14f38652e727cbd8  # Full (40 chars)
slack-chat inbox view b89c7a14                                   # 8 chars
slack-chat inbox view b89c7a                                     # 6 chars (short)
slack-chat inbox view b89c                                       # 4 chars (if unique)
```

Error on ambiguity:
```
❌ Ambiguous ID 'b8' matches: b89c7a14..., b8a3f2c1...
```

### ID Resolution Cache
User and channel IDs are cached to `storage/_cache/` to reduce API calls:

```
storage/_cache/
├── users.yml      # User ID → full profile
└── channels.yml   # Channel ID → full info
```

When you run `slack-chat user resolve` or `slack-chat channel resolve`, the result is cached. Subsequent lookups hit the cache first.

## Quick Reference

| Task | Command |
|------|---------|
| **Server** | |
| Start Server | `slack-chat server start` |
| Stop Server | `slack-chat server stop` |
| Server Status | `slack-chat server status` |
| **Pull (Online)** | |
| Pull Messages | `slack-chat pull --since "7 days ago" [--limit N] [--type TYPE]` |
| **Inbox (Offline)** | |
| Summary | `slack-chat inbox summary` |
| List | `slack-chat inbox list [--type TYPE] [--limit N] [--since DATE] [--all]` |
| View | `slack-chat inbox view <id>` |
| Mark Read | `slack-chat inbox read <id>` |
| Mark Thread Read | `slack-chat inbox mark-thread <id>` |
| Mark Channel Read | `slack-chat inbox mark-channel <channel_id>` |
| Mark Unread | `slack-chat inbox unread <id>` |
| **Resolve (Online, Cached)** | |
| Resolve Channel | `slack-chat channel resolve <channel_id>` |
| Resolve User | `slack-chat user resolve <user_id>` |
| **Lookup (Offline)** | |
| List Channels | `slack-chat channel list` |
| Find Channels | `slack-chat channel find <keyword>` |
| List Users | `slack-chat user list` |
| Find Users | `slack-chat user find <keyword>` |
| **Context (Online)** | |
| Message Around | `slack-chat message around <event_id> [-B N] [-A N]` |
| View Thread | `slack-chat client read-message-thread-replies <channel> <thread_ts>` |
| **Write (Online)** | |
| Reply | `slack-chat reply <id \| #channel> "message"` |
| React | `slack-chat react <id> <emoji>` |
| Mute Channel | `slack-chat mute <channel_id>` |
| Post Message | `slack-chat client post-message <channel> "text"` |
| Reply to Thread | `slack-chat client post-thread-reply <channel> <thread_ts> "text"` |
| Add Reaction | `slack-chat client add-reaction <channel> <ts> "emoji"` |

## Online Mode: Fetching Messages from Slack

### Command: `slack-chat pull`

**Purpose**: Fetch unread messages from Slack, store locally as Markdown files.

> **IMPORTANT**: Always fetch with reasonable limits. Use `--limit 10` or `--limit 20` for controlled processing. This ensures manageable batches for review.

**Command**:
```bash
slack-chat pull --since <date> [--limit N] [--type TYPE]
```

**Parameters**:
- `--since <date>`: Required. Fetch messages received on/after this date
  - Formats: `YYYY-MM-DD`, `yesterday`, `"7 days ago"`, `"1 day ago"`
- `--limit <n>`: Optional. Max messages per category (default: 100)
- `--type <type>`: Optional. Filter: `channels`, `dms`, `threads`, `mentions`, `all` (default: `all`)
- `--quiet`: Suppress verbose output

**Behavior**:
1. Fetches unread channels, DMs, subscribed threads, and @mentions
2. Skips already-stored files (deduplication via SHA1)
3. Stores each message as Markdown under `storage/<id>.md`
4. Does NOT mark as read on Slack (that's done via `inbox read`)
5. Prints progress for each stored message

**Examples**:

```bash
# Pull from last 24 hours
slack-chat pull --since "1 day ago" --limit 10

# Pull only threads
slack-chat pull --since yesterday --type threads

# Pull everything from past week (use with caution)
slack-chat pull --since "7 days ago"

# Quiet mode for scripts
slack-chat pull --since yesterday --limit 50 --quiet
```

**Output Example**:
```
📥 Pulling messages since 2026-01-06...
  📢 Fetching channel messages...
    + 19aaf82d... (C02PROJ01)
    + 60e86087... (CL80UCYF5)
  💬 Fetching DM messages...
  🧵 Fetching thread replies...
    + e04ada37... (thread in C0919MFSA4X)
    + 8e206898... (thread in C098Q7ALDNU)
  📣 Fetching mentions...
    + b89c7a14... (mention)

✅ Done: 11 stored, 0 skipped, 23 fetched total
```

**Best Practices**:
- Use `--limit 10` or `--limit 20` for manageable batches
- Use `--since "1 day ago"` for daily processing
- After pulling, use `slack-chat inbox list` to review new messages
- The `permalink` field in stored files opens the message in Slack

## Offline Mode: Analysis & Metadata

All `slack-chat inbox` commands work **offline**, reading/writing files in `storage/`.

### Command: `slack-chat inbox summary`

**Purpose**: Get overall message statistics from local storage.

**Command**:
```bash
slack-chat inbox summary
```

**Output**:
```yaml
summary:
  unread: 11
  read: 0
  total: 11
  by_type:
    channel: 6
    dm: 0
    thread: 5
    mention: 0
```

**Use Cases**:
- Verify message counts before processing
- Track unread vs read ratio
- Check if pull was successful

**Online Mode**: Add `--online` to fetch live counts from Slack API instead.

### Command: `slack-chat inbox list`

**Purpose**: List messages with filtering.

**Command**:
```bash
slack-chat inbox list [--type TYPE] [--limit N] [--since DATE] [--all]
```

**Options**:
- `-t, --type <type>`: Filter by type (`channels`, `dms`, `threads`, `mentions`, `all`)
- `-n, --limit <n>`: Max results (default: 20)
- `--since <date>`: Filter messages after date
- `-a, --all`: Include read messages (default: unread only)
- `--online`: Use live API instead of local storage

**Output Format**:
```
<short_id> / <date> / <channel_id> / <user_id> (type)
  <text preview>
```

**Output Example**:
```
Showing 5 of 11 messages:

b89c7a / 2026-01-07 19:47 / C01TECH01 / WNARLG5HB 
  Thank you all!

4aab08 / 2026-01-07 19:24 / C01TECH01 / WM1HR6E1G 
  ok, puppet ran, nginx restarted

e04ada / 2026-01-07 19:06 / C0919MFSA4X / U025R2RLLM8 (thread)
  Have been back and forth with cloud a bit on camel support

... and 6 more
```

**Use Cases**:
- Scan unread messages
- Filter by type (threads only, DMs only, etc.)
- Find messages from specific period

### Command: `slack-chat inbox view <id>`

**Purpose**: Display full content of a message.

**Command**:
```bash
slack-chat inbox view <id>
```

**Supports**: 
- Partial IDs: `b89c7a`
- Full IDs: `b89c7a14755df6dca57ece8e14f38652e727cbd8`
- Event ID format: `C01TECH01:1767815267.099869`

**Output**: Complete Markdown file with YAML frontmatter and rendered body.

**Use Cases**:
- Read full message content
- Extract data for analysis
- Check message metadata (reactions, attachments, etc.)

**Example**:
```bash
# View by short ID
slack-chat inbox view b89c7a

# View by event ID
slack-chat inbox view C01TECH01:1767815267.099869
```

### How to interpet messsages

When I ask you to view a message, 
the output will have ID numbers in place of channel and user names, for example:

```
events:
- id: D04R3C0G1JT:1766426423.335079
  type: mention
  channel: '#USLACKBOT'
  from: slackbot
  text: <@WNHDFCXTJ> archived the channel <#C08TR8S0QJ0>
  timestamp: '1766426423.335079'
- id: D0849DD71UP:1765928154.858059
  type: reaction
  channel: '#WNHDFCXTJ'
  text: From here not much needed apart from those future meetings y
  timestamp: '1765928154.858059'
  thread_ts: '1765321208.614079'
  url: https://bigco-producta.slack.com/archives/D0849DD71UP/p1765928154858059
- id: G01GROUP01:1765909149.353759@1765321208.614079
  type: reaction (thread reply)
  channel: '#sre-nexus'
  text: <@WM1HR6E1G|teddy> I want you to be the coordinator
  timestamp: '1765909149.353759'
  thread_ts: '1765321208.614079'
  url: https://bigco-producta.slack.com/archives/G01GROUP01/p1765909149353759
```
where:
- `@WNHDFCXTJ` is an example of a user_id
  - indicated by the `@` (may not always appear) as well as the `W` prefix (part of the identifier)
- `#C08TR8S0QJ0` is an example of a channel_id
  - indicated by the `#` (may not always appear) as well as the `C` prefix (part of the identifier)
- `thread_ts` indicates the message is part of a thread (value is the parent message timestamp)
- Event IDs for threaded messages use the format: `CHANNEL_ID:TIMESTAMP@THREAD_TS`

Use the `slack-chat channel resolve <id>` and `slack-chat user resolve <id>` commands to find out what the human-readable version of these are, and only cite these when replying back to me (I don't want to see IDs, unless names are not available.)

### Command: `slack-chat inbox read <id>`

**Purpose**: Mark a message as read.

**Command**:
```bash
slack-chat inbox read <id> [--offline-only]
```

**Behavior**:
- Updates `offline.read: true` in local file
- By default, also marks as read on Slack server
- Use `--offline-only` to skip server-side marking

**Output**:
```yaml
ok: true
marked_read_locally: b89c7a14...
marked_read_on_slack: true
```

**Use Cases**:
- Mark messages as processed
- Track which messages have been reviewed

### Command: `slack-chat inbox mark-thread <id>`

**Purpose**: Mark all messages in a thread as read.

**Command**:
```bash
slack-chat inbox mark-thread <id> [--offline-only]
```

**Behavior**:
- Finds all stored messages with the same thread_ts
- Marks them all as read locally
- By default, also marks thread as read on Slack server
- Use `--offline-only` to skip server-side marking

**Output**:
```yaml
ok: true
thread_ts: '1767773973.649239'
channel_id: C0919MFSA4X
marked_count: 5
marked_ids:
- e04ada37
- 8e206898
- a1b2c3d4
marked_read_on_slack: true
```

**Use Cases**:
- Done reviewing a thread, mark all at once
- Clean up multiple thread replies efficiently

### Command: `slack-chat inbox mark-channel <channel_id>`

**Purpose**: Mark all messages in a channel as read.

**Command**:
```bash
slack-chat inbox mark-channel <channel_id> [--offline-only]
```

**Behavior**:
- Finds all stored messages in the channel
- Marks them all as read locally
- By default, also marks channel as read on Slack (up to latest message)
- Use `--offline-only` to skip server-side marking

**Output**:
```yaml
ok: true
channel_id: C01TECH01
marked_count: 12
marked_ids:
- b89c7a14
- 4aab08f1
- 7d8e9bce
marked_read_on_slack: true
```

**Use Cases**:
- Done with a channel's backlog, mark all at once
- Clear noisy channels efficiently

### Command: `slack-chat inbox unread <id>`

**Purpose**: Mark a message as unread (local only).

**Command**:
```bash
slack-chat inbox unread <id>
```

**Effect**:
- Sets `offline.read: false` in local file
- Does NOT sync to Slack (Slack API doesn't support marking unread)

**Output**:
```yaml
ok: true
marked_unread_locally: b89c7a14...
```

**Use Cases**:
- Recover from accidental marks
- Re-queue messages for processing

## ID Resolution (Online, Cached)

### Command: `slack-chat channel resolve <id>`

**Purpose**: Convert channel ID to human-readable info.

```bash
slack-chat channel resolve C01TECH01
```

**Output**:
```yaml
name: engineering
is_archived: false
is_private: false
created: 1563498384
creator: U0A4DBTGDPZ
topic: Technical discussions
purpose: Engineering team collaboration
members:
- WNARLG5HB
- WMCH36A6Q
```

**Caching**: Result is saved to `storage/_cache/channels.yml`. Subsequent calls return cached data.

### Command: `slack-chat user resolve <id>`

**Purpose**: Convert user ID to human-readable info.

```bash
slack-chat user resolve WNARLG5HB
```

**Output**:
```yaml
name: Alice Johnson
is_bot: false
is_admin: false
email: alice@bigco.com
display_name: alice
title: Software Engineer
tz: America/Los_Angeles
```

**Caching**: Result is saved to `storage/_cache/users.yml`. Subsequent calls return cached data.

**When to use**: Whenever you see a raw ID (like `WNARLG5HB` or `C01TECH01`) and need the human-readable name.

## Lookup Commands (Offline)

These commands search the offline cache without making API calls.

### Command: `slack-chat channel list`

**Purpose**: List all cached channels.

```bash
slack-chat channel list
```

**Output** (tab-separated columns): `id | name | description`
```
C01TECH01	prod-tech-internal-ProductA	Production tech discussion
C098Q7ALDNU	engineering-general	General engineering chat
C6M7U8DFF	ProductA-server-team	ProductA server team coordination
```

**Use Cases**:
- See all channels you've resolved/cached
- Find channel IDs for posting

### Command: `slack-chat channel find <keyword>`

**Purpose**: Find cached channels by name keyword.

```bash
slack-chat channel find prod
slack-chat channel find ProductA
```

**Output** (tab-separated columns): `id | name | description`

Matches channels whose name **contains** the keyword (case-insensitive).
For example, `hal` matches `thal`, `fhaly`, etc.

**Use Cases**:
- Find a channel when you only remember part of the name
- Look up channel ID before posting with `slack-chat reply`

### Command: `slack-chat user list`

**Purpose**: List all cached users.

```bash
slack-chat user list
```

**Output** (tab-separated columns): `id | name | title | project`
```
WNARLG5HB	Alice Johnson	Associate Lead Software Engineer	ProductA IV
WMCH36A6Q	Charlie Brown	Senior Software Engineer	ProductA
U04MYEUTJLU	Dana Evans	Engineering Manager	IT Infrastructure
```

**Use Cases**:
- See all users you've resolved/cached
- Find user IDs for @ mentions

### Command: `slack-chat user find <keyword>`

**Purpose**: Find cached users by keyword (searches all fields).

```bash
slack-chat user find john
slack-chat user find ProductA
slack-chat user find infrastructure
```

**Output** (tab-separated columns): `id | name | title | project`

Searches **all fields** in the user object (case-insensitive), including:
- Name, real_name, display_name
- Title, email
- Team/project custom fields
- Any other string value

**Examples**:
```bash
# Find by name
slack-chat user find shelly

# Find by project/team
slack-chat user find ProductA
slack-chat user find "producta"

# Find by title
slack-chat user find "software engineer"
```

**Use Cases**:
- Find a user when you only remember part of the name
- Find all users on a specific team/project
- Look up user ID for mentions

## Context Commands (Online)

Often, viewing one message in isolation does not provide enough information; To truly understand a given message, often its necessary to look at messages in the surrounding (channel/thread) context (before and after), similar to `grep -B` and `-A` flags.

### Command: `slack-chat message around <event_id>`

**Purpose**: View messages before/after a target message for context.

```bash
slack-chat message around C6M7U8DFF:1766558024.931179 -B 3 -A 3
```

**Options**:
- `-B <n>`: Number of messages before (default: 3)
- `-A <n>`: Number of messages after (default: 3)

**Use Cases**:
- Understand conversation context
- See what led to a message
- Review thread context

### Command: `slack-chat inbox context <event_id>`

**Purpose**: View surrounding context (thread or channel messages).

```bash
slack-chat inbox context C6M7U8DFF:1766558024.931179 --limit 5
```

## Write Commands (Online)

### Command: `slack-chat reply <target> <message>`

**Purpose**: Reply to a channel or thread with smart targeting.

**Command**:
```bash
slack-chat reply <target> "<message>"
```

**Smart Targeting** - accepts multiple ID formats:
- **Channel ID**: `C01TECH01` → posts to channel
- **Channel name**: `#prod-tech-internal-ProductA` → posts to channel (looks up from cache)
- **Storage ID**: `b89c7a` → replies to that message's thread  
- **Event ID**: `C01TECH01:1767815267.099869` → replies to thread
- **Event ID with thread**: `C01TECH01:1767815267.099869@1767773973.649239`

**Channel Name Resolution**:
When using a channel name (with `#` prefix), the system:
1. Looks up the name in the offline cache (`storage/_cache/channels.yml`)
2. If found, uses the cached channel ID
3. If not found, returns an error with suggestion to use `slack-chat channel find`

To cache a channel, use `slack-chat channel resolve <id>` first.

**Output**:
```yaml
ok: true
channel: C01TECH01
message_ts: '1767820123.456789'
thread_ts: '1767815267.099869'
permalink: https://bigco-producta.slack.com/archives/...
```

**Examples**:
```bash
# Post to channel by ID
slack-chat reply C01TECH01 "Hello everyone!"

# Post to channel by name
slack-chat reply "#prod-ProductA" "Hello world!"

# Reply to a thread (by storage ID)
slack-chat reply b89c7a "Thanks for the update!"

# Reply to a thread (by event ID)
slack-chat reply C01TECH01:1767815267.099869 "Got it, will look into this."
```

**Use Cases**:
- Quick replies to messages you just viewed
- Respond to threads without looking up thread_ts manually
- Post updates to channels by name

### Command: `slack-chat react <target> <emoji>`

**Purpose**: Add an emoji reaction to a message.

**Command**:
```bash
slack-chat react <target> <emoji>
```

**Smart Targeting** - accepts:
- **Storage ID**: `b89c7a` → reacts to that message
- **Event ID**: `C01TECH01:1767815267.099869` → reacts to message
- **Event ID with thread**: `C01TECH01:1767815267.099869@1767773973.649239`

**Emoji formats** (with or without colons):
- `thumbsup`, `+1`, `eyes`, `white_check_mark`
- `:thumbsup:`, `:+1:`, `:eyes:`

**Output**:
```yaml
ok: true
channel: C01TECH01
timestamp: '1767815267.099869'
emoji: thumbsup
```

**Examples**:
```bash
# React by storage ID
slack-chat react b89c7a thumbsup

# React by event ID
slack-chat react C01TECH01:1767815267.099869 eyes

# Acknowledge with checkmark
slack-chat react b89c7a white_check_mark
```

**Use Cases**:
- Acknowledge messages quickly
- React to messages you just viewed
- Signal agreement/approval without writing a reply

### Command: `slack-chat mute <channel_id>`

**Purpose**: Mute a channel to stop receiving notifications.

**Command**:
```bash
slack-chat mute <channel_id>
```

**Output**:
```yaml
ok: true
channel: C01TECH01
muted: true
```

**Examples**:
```bash
# Mute a noisy channel
slack-chat mute C01TECH01

# Mute after resolving channel name
slack-chat channel resolve C6M7U8DFF  # Check it's the right channel
slack-chat mute C6M7U8DFF
```

**Use Cases**:
- Silence noisy channels
- Stop notifications from low-priority channels
- Reduce distractions during focus time

## Workflow Scenarios

### Scenario 1: Daily Slack Processing

**Goal**: Pull messages daily, review, mark processed.

```bash
# 1. Start server
slack-chat server start

# 2. Pull recent messages
slack-chat pull --since "1 day ago" --limit 20

# 3. Check statistics
slack-chat inbox summary
# Output: unread: 15, read: 0, total: 15

# 4. List messages for review
slack-chat inbox list --limit 20

# 5. View specific message
slack-chat inbox view b89c7a

# 6. Resolve user ID to name
slack-chat user resolve WNARLG5HB

# 7. Mark as processed
slack-chat inbox read b89c7a

# 8. Verify
slack-chat inbox summary
# Output: unread: 14, read: 1, total: 15
```

### Scenario 2: Thread-focused Processing

**Goal**: Focus on thread replies only.

```bash
# 1. Pull only threads
slack-chat pull --since "3 days ago" --type threads

# 2. List thread messages
slack-chat inbox list --type threads

# 3. View thread reply
slack-chat inbox view e04ada

# 4. Get full thread context (online)
slack-chat message around C0919MFSA4X:1767819961.835769@1767773973.649239 -B 5 -A 5

# 5. Mark as read
slack-chat inbox read e04ada
```

### Scenario 3: Incremental Processing

**Goal**: Process in small batches.

```bash
# Pull small batch
slack-chat pull --since yesterday --limit 5

# Process each
for id in b89c7a 4aab08 7d8e9b; do
  echo "=== Message: $id ==="
  slack-chat inbox view $id
  # ... analyze ...
  slack-chat inbox read $id
done

# Check remaining
slack-chat inbox summary
```

## Agent Workflow: Processing Unread Activity

When the user asks to "check Slack" or "process my unreads":

### Step 1: Pull Fresh Data
```bash
slack-chat pull --since "1 day ago" --limit 20
```

### Step 2: Get Overview
```bash
slack-chat inbox summary
```
If all counts are 0, report "No unread activity" and stop.

### Step 3: List Messages
```bash
slack-chat inbox list --limit 20
```

### Step 4: Present Summary
Parse the output. Present to user:
- Count by type (channels, DMs, threads)
- Preview of important messages

Example:
> You have 11 unread messages: 6 in channels, 5 thread replies.
> Notable: Thread reply from U025R2RLLM8 about "camel support"

### Step 5: Drill Down (on request)
```bash
slack-chat inbox view e04ada              # View message
slack-chat user resolve U025R2RLLM8       # Get user name
slack-chat channel resolve C0919MFSA4X    # Get channel name
```

### Step 6: Mark as Read (on request)
```bash
slack-chat inbox read e04ada
```

## File Organization

```
slack-chat/
├── src/
│   ├── cli.py         # Main CLI
│   ├── server.py      # Browser server (FastAPI)
│   ├── storage.py     # Storage operations
│   └── pull.py        # Pull command
├── storage/           # Local message cache
│   ├── _cache/
│   │   ├── users.yml
│   │   └── channels.yml
│   ├── b89c7a14...md
│   ├── 4aab08f1...md
│   └── ...
└── doc/
    └── SPEC1.md       # Architecture spec
```

## Tips for AI Agents

1. **Pull first**: Always `slack-chat pull` before reading messages to get fresh data
2. **Use partial IDs**: `b89c7a` instead of full `b89c7a14755df6dca...`
3. **Check summary**: `slack-chat inbox summary` to understand data volume
4. **Resolve IDs**: Use `slack-chat user resolve` and `slack-chat channel resolve` to convert IDs to names
5. **Test with limits**: Use `--limit 5` when prototyping workflows
6. **Mark processed**: Always `inbox read` after processing to track state
7. **Use relative dates**: `yesterday`, `"7 days ago"` are clearer than exact dates
8. **Cache is your friend**: ID resolution is cached, so feel free to resolve repeatedly

## Architecture Notes

- **Storage**: All files in `storage/` are Markdown with YAML frontmatter (Git-friendly)
- **No database**: Direct filesystem access
- **Offline-first**: Inbox commands work without server running
- **Deduplication**: SHA1 hashing ensures same message = same storage file
- **ID cache**: User/channel info cached to reduce API calls
- **Server required**: For `pull`, resolve commands, and write operations

## Troubleshooting

- if a command returns `HTTP 500`, 
  ```
  slack-chat server status # check if server is healthy
  slack-chat server stop # stop it
  slack-chat server start # re-start it
  ```
