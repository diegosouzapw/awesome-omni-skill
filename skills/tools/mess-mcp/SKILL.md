---
name: mess-mcp
description: MCP server tools for creating and managing MESS physical-world task requests. Provides mess, mess_status, mess_capabilities, mess_request, mess_answer, mess_cancel, and mess_fetch tools.
---

# MESS MCP Server

## Overview

The MESS MCP Server provides Claude Desktop with tools to create and manage physical-world task requests. It syncs with GitHub (or local files) to store request threads.

## Tools

### `mess` - Send MESS Protocol Messages

Create new requests or update existing ones using free-form YAML.

**Input:** YAML-formatted MESS message

**Why YAML?** The `mess` tool accepts raw MESS protocol messages, allowing you to:
- Include any valid MESS protocol fields, not just common ones
- Add custom context, metadata, or instructions
- Construct complex multi-part messages
- Use the full expressiveness of the MESS protocol

The examples below show common patterns, but you can include any fields defined in the MESS protocol specification.

#### Creating a Request

```yaml
- v: 1.0.0
- request:
    intent: Check if the garage door is closed
    context:
      - Getting ready for bed
      - Want to make sure house is secure
    priority: normal
    response_hint:
      - image
      - text
```

**Response:**
```yaml
ref: "2026-02-01-001"
status: pending
message: "Request created: 2026-02-01-001"
```

#### Updating a Request

Send a status update:
```yaml
- status:
    re: "2026-02-01-001"
    code: claimed
```

Cancel a request:
```yaml
- cancel:
    re: "2026-02-01-001"
    reason: "No longer needed"
```

### `mess_status` - Check Request Status

Query pending requests or get details on a specific thread.

#### List All Active Requests

```yaml
ref: null
```

**Response:**
```yaml
- ref: "2026-02-01-001"
  status: pending
  intent: Check if the garage door is closed
  requestor: claude-desktop
  executor: null
  priority: normal
  created: "2026-02-01T22:00:00Z"
  hasUpdates: true    # changed since last mess_status call

- ref: "2026-02-01-002"
  status: claimed
  intent: What's in the fridge?
  requestor: claude-desktop
  executor: teague-phone
  priority: normal
  hasUpdates: false   # no changes since last check
```

**Note:** The `hasUpdates` flag indicates whether the thread changed since your last `mess_status` call. New threads always have `hasUpdates: true`. Use `mess_wait` instead of polling if you want to wait for changes efficiently.

#### Get Specific Thread

```yaml
ref: "2026-02-01-001"
```

**Response (v2.1.0 format):**
```yaml
ref: "2026-02-01-001-garage-check"
status: completed
intent: Check if the garage door is closed
requestor: claude-desktop
executor: teague-phone
messages:
  - from: claude-desktop
    received: "2026-02-01T22:00:00Z"
    MESS:
      - v: "1.0.0"
      - request:
          id: garage-check
          intent: Check if the garage door is closed

  - from: teague-phone
    received: "2026-02-01T22:05:00Z"
    re: "2026-02-01-001-garage-check"      # message-level reference
    MESS:
      - status:
          code: completed
      - response:
          content:
            - image:
                resource: "content://2026-02-01-001-garage-check/att-002-image-door.jpg"
                mime: "image/jpeg"
                size: 245891
            - "All clear - garage door is closed and locked"

attachments:
  - name: att-002-image-door.jpg
    resource: "content://2026-02-01-001-garage-check/att-002-image-door.jpg"
```

**Note:** Images are returned as `content://` resource URIs instead of inline base64 to keep responses lightweight. Use `mess_fetch` to fetch attachment content when needed.

## Configuration

### GitHub Mode (Recommended)

```json
{
  "mcpServers": {
    "mess": {
      "command": "node",
      "args": ["/path/to/mcp/index.js"],
      "env": {
        "MESS_GITHUB_REPO": "username/mess-exchange",
        "MESS_GITHUB_TOKEN": "github_pat_xxxxx",
        "MESS_GITHUB_ONLY": "true",
        "MESS_AGENT_ID": "claude-desktop"
      }
    }
  }
}
```

### Local Mode

```json
{
  "mcpServers": {
    "mess": {
      "command": "node",
      "args": ["/path/to/mcp/index.js"],
      "env": {
        "MESS_DIR": "~/.mess",
        "MESS_AGENT_ID": "claude-desktop"
      }
    }
  }
}
```

### Hybrid Mode (Local + GitHub Sync)

```json
{
  "mcpServers": {
    "mess": {
      "command": "node",
      "args": ["/path/to/mcp/index.js"],
      "env": {
        "MESS_DIR": "~/.mess",
        "MESS_GITHUB_REPO": "username/mess-exchange",
        "MESS_GITHUB_TOKEN": "github_pat_xxxxx",
        "MESS_AGENT_ID": "claude-desktop"
      }
    }
  }
}
```

### `mess_capabilities` - Discover Available Capabilities

List physical-world capabilities that executors in this exchange can perform.

**Input:** Optional `tag` filter

#### List All Capabilities

```yaml
# No input needed
```

**Response:**
```yaml
- id: camera
  description: Take and attach photos
  tags: [attachments]
- id: check-door
  description: Check if doors are locked or closed
  tags: [security, physical-access]
- id: hands
  description: Has human hands for physical manipulation
  tags: [physical-access]
```

#### Filter by Tag

```yaml
tag: security
```

**Response:**
```yaml
- id: check-door
  description: Check if doors are locked or closed
  tags: [security, physical-access]
- id: check-stove
  description: Verify stove/oven is turned off
  tags: [security, safety]
```

Use this to understand what kinds of tasks you can request. Capabilities are defined in `capabilities/*.yaml` in the exchange.

### `mess_request` - Create Request (Simple)

A simpler alternative to the raw `mess` tool for creating requests.

**Input:**
```yaml
intent: "Check if the garage door is closed"
context:
  - "Getting ready for bed"
priority: elevated
response_hints:
  - image
```

**Response:**
```yaml
ref: "2026-02-01-001"
status: pending
message: "Request created"
```

### `mess_answer` - Answer Question

Respond to an executor's question when status is `needs_input`.

**Input:**
```yaml
ref: "2026-02-01-001"
answer: "The living room ceiling light, not the lamp"
```

### `mess_cancel` - Cancel Request

Cancel a pending or in-progress request.

**Input:**
```yaml
ref: "2026-02-01-001"
reason: "No longer needed"  # optional
```

### `mess_wait` - Wait for Changes

Wait for changes to threads instead of polling `mess_status` repeatedly.

**Input:**
```yaml
ref: "2026-02-01-001"  # optional: wait for specific thread
timeout: 60             # optional: seconds (default: 60, max: 43200 / 12 hours)
```

**Response:**
```yaml
updated:
  - ref: "2026-02-01-001"
    status: claimed
    hasUpdates: true
waited: 5
timedOut: false
```

**Adaptive polling:** Poll frequency scales with timeout (~30 polls total):
- 60s timeout → polls every 2s
- 5 min → every 10s
- 1 hour → every 2 min
- 12 hours → every 24 min

**When to use:**
- After creating a request, call `mess_wait` to be notified when claimed/completed
- Returns immediately if thread changed since wait started
- Use for efficient long-running waits instead of polling

**Example workflow:**
```yaml
# 1. Create a request
mess_request:
  intent: "Check if the garage door is closed"

# 2. Wait for it to be completed (up to 1 hour)
mess_wait:
  ref: "2026-02-01-001"
  timeout: 3600

# 3. Response returns when status changes or timeout
```

### `mess_fetch` - Fetch Resource Content

**This is how you retrieve images and attachments.** When `mess_status` returns `content://` URIs, use this tool to fetch the actual content.

**Input:**
```yaml
uri: "content://2026-02-01-001/photo.jpg"
```

**Supported URI schemes:**
- `content://{ref}/{filename}` - Attachments (images, files)
- `thread://{ref}` - Full thread data (envelope + messages)
- `thread://{ref}/envelope` - Thread metadata only
- `thread://{ref}/latest` - Most recent message
- `mess://help` - This documentation

**Response for images:**
```yaml
uri: "content://2026-02-01-001/photo.jpg"
mimeType: "image/jpeg"
encoding: "base64"
data: "/9j/4AAQSkZJRg..."
```

**Response for thread data:**
Returns the thread content as YAML.

**Example workflow:**
```yaml
# 1. Check status, see there's an image
mess_status:
  ref: "2026-02-01-001"
# Response includes: resource: "content://2026-02-01-001/photo.jpg"

# 2. Fetch the image
mess_fetch:
  uri: "content://2026-02-01-001/photo.jpg"
# Response includes base64 image data
```

## Request Fields

| Field | Required | Description |
|-------|----------|-------------|
| `intent` | Yes | What you need done (be specific) |
| `id` | No | Your local identifier for tracking (exchange assigns canonical `ref`) |
| `context` | No | List of relevant context strings |
| `priority` | No | `background`, `normal`, `elevated`, `urgent` |
| `response_hint` | No | Expected response types: `text`, `image`, `video`, `audio` |

### Free-Form Extensions

The MESS protocol is extensible. Beyond the standard fields above, you can include:

```yaml
- v: 1.0.0
- request:
    intent: Check the garden irrigation system
    context:
      - Haven't watered in 3 days
    priority: normal
    # Standard fields above, custom fields below:
    location: backyard
    equipment_needed:
      - hose access
      - manual valve knowledge
    safety_notes:
      - Watch for wasps near the shed
    deadline: before 6pm today
```

Custom fields are preserved in the thread and visible to executors. Use them for domain-specific context that doesn't fit the standard fields.

## Status Codes

| Status | Meaning |
|--------|---------|
| `pending` | Waiting for executor to claim |
| `claimed` | Executor is working on it |
| `in_progress` | Executor actively working |
| `needs_input` | Executor needs clarification |
| `completed` | Task finished successfully |
| `failed` | Could not complete |
| `declined` | Executor declined the request |
| `cancelled` | Request was cancelled |

## Common Patterns

### Create and Wait

```python
# 1. Create request
mess:
  - v: 1.0.0
  - request:
      intent: Is anyone home?

# 2. Periodically check status
mess_status:
  ref: "2026-02-01-001"

# 3. When completed, read the response
```

### Urgent Request

```yaml
- v: 1.0.0
- request:
    intent: Did I leave the stove on?
    context:
      - Just left the house
      - Can't remember if I turned it off
    priority: urgent
    response_hint:
      - image
      - text
```

### Follow-up After needs_input

```yaml
# Original request got needs_input status
# Executor asked: "Which light?"

# Send clarification (v2.1.0: answer block, message-level re: handled automatically)
- answer:
    id: light-clarification
    value: "The living room ceiling light, not the lamp"
```

## Resources

The MCP server uses `content://` and `thread://` URIs for attachments and thread data.

### Fetching Attachments

When `mess_status` returns a thread, images and files are referenced as resource URIs:

```yaml
image:
  resource: "content://2026-02-01-001/att-002-image-door.jpg"
  mime: "image/jpeg"
  size: 245891
```

**To fetch the actual image, use `mess_fetch`:**

```yaml
mess_fetch:
  uri: "content://2026-02-01-001/att-002-image-door.jpg"
```

This keeps status responses lightweight and avoids blowing up context with large base64 payloads.

### Resource URI Format

```
content://{thread-ref}/{attachment-filename}
```

Examples:
- `content://2026-02-01-001-garage-check/att-002-image-door.jpg`
- `content://2026-02-01-003-fridge/att-005-image-contents.jpg`

### Thread Resources

Read thread data with attachments automatically rewritten to `content://` URIs:

```
thread://{ref}          # Full thread (envelope + messages)
thread://{ref}/envelope # Just metadata (status, executor, history)
thread://{ref}/latest   # Most recent message only
```

**Example - Read full thread:**
```yaml
# Request: thread://2026-02-01-001

# Response:
envelope:
  ref: "2026-02-01-001"
  status: completed
  intent: Check the garage door
messages:
  - from: claude-desktop
    MESS:
      - request:
          intent: Check the garage door
  - from: teague-phone
    MESS:
      - response:
          content:
            - image:
                resource: "content://2026-02-01-001/att-001-door.jpg"
            - "Door is closed"
```

**Example - Read just envelope:**
```yaml
# Request: thread://2026-02-01-001/envelope

# Response:
ref: "2026-02-01-001"
status: completed
executor: teague-phone
updated: "2026-02-01T22:05:00Z"
```

## Error Handling

**Thread not found:**
```yaml
error: "Thread 2026-02-01-999 not found"
```

**Missing required field:**
```yaml
error: "Missing re: field"
```

**GitHub API error:**
```yaml
error: "GitHub API error: 401"
```

## File Locations

### macOS
- Config: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Local MESS dir: `~/.mess/`

### Linux
- Config: `~/.config/claude/claude_desktop_config.json`
- Local MESS dir: `~/.mess/`

### Windows
- Config: `%APPDATA%\Claude\claude_desktop_config.json`
- Local MESS dir: `%USERPROFILE%\.mess\`
