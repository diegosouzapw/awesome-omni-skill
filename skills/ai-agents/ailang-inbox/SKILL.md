---
name: AILANG Inbox
description: Cross-agent communication system for AI workflows. Check messages at session start, send notifications to other agents, and track multi-agent handoffs with correlation IDs.
---

# AILANG Inbox

AILANG's messaging system enables AI agents to communicate asynchronously across sessions and projects.

## Session Start Routine

**At the start of EVERY session, check for messages:**

```bash
# Check for unread messages
ailang messages list --unread

# Or check specific inbox
ailang messages list --inbox user --unread
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `ailang messages list --unread` | Check for new messages |
| `ailang messages list --inbox user` | Check user inbox |
| `ailang messages send user "msg" --from agent` | Send to user |
| `ailang messages ack MSG_ID` | Mark as read |
| `ailang messages ack --all` | Mark all as read |
| `ailang messages read MSG_ID` | View full message |

## Checking Messages

### List Messages

```bash
# All messages
ailang messages list

# Only unread
ailang messages list --unread

# Specific inbox
ailang messages list --inbox user

# Filter by sender
ailang messages list --from sprint-executor

# Limit results
ailang messages list --limit 5

# JSON output (for parsing)
ailang messages list --json
```

### Read Full Message

```bash
# View complete message content
ailang messages read MSG_ID
```

### Acknowledge Messages

```bash
# Mark single message as read
ailang messages ack MSG_ID

# Mark all unread as read
ailang messages ack --all

# Mark all in specific inbox
ailang messages ack --all --inbox user

# Mark as unread again (for retry)
ailang messages unack MSG_ID
```

## Sending Messages

### To User

```bash
# Simple text message
ailang messages send user "Task completed successfully" --from my-agent --title "Status Update"

# With JSON payload
ailang messages send user --json '{"status":"done","result":"All tests passing"}' --from my-agent
```

### To Another Agent

```bash
# Send to specific agent inbox
ailang messages send sprint-executor "Ready for handoff" --from planner

# With correlation ID (for tracking workflows)
ailang messages send sprint-executor --json '{"task":"execute"}' --from planner --correlation workflow_123
```

## Workflow Patterns

### 1. Session Start Check

```bash
# 1. Check for messages
ailang messages list --unread

# 2. If messages exist:
#    - Summarize to user
#    - Ask what action to take

# 3. After handling:
ailang messages ack --all
```

### 2. Agent Handoff

```bash
# Agent A completes work and hands off to Agent B
ailang messages send agent-b --json '{
  "type": "handoff",
  "task": "continue_implementation",
  "artifacts": ["path/to/results/"],
  "context": "Previous work completed"
}' --from agent-a --correlation project_xyz
```

### 3. Completion Notification

```bash
# Notify user that autonomous work is done
ailang messages send user --json '{
  "type": "completion",
  "status": "success",
  "summary": "All 5 milestones completed",
  "artifacts": ["results/v1.0/"]
}' --from sprint-executor --title "Sprint Complete"
```

### 4. Error Reporting

```bash
# Report error to user
ailang messages send user --json '{
  "type": "error",
  "error": "Tests failing at milestone 3",
  "details": "logs/error.log",
  "needs_help": true
}' --from executor --title "Error Encountered"
```

## Correlation IDs

Track related messages across agent handoffs:

```json
{
  "message_id": "msg_20251208_103045_abc123",
  "correlation_id": "workflow_project_x",
  "from": "planner",
  "to": "executor",
  "payload": { ... }
}
```

**Benefits:**
- Track entire workflow chains
- Filter messages by workflow
- Debug multi-agent interactions
- Resume work from where you left off

## Message Types

### Completion
```json
{
  "type": "completion",
  "status": "success",
  "result": "All tests passing",
  "artifacts": ["path/to/output/"]
}
```

### Handoff
```json
{
  "type": "handoff",
  "task": "next_phase",
  "context": "Previous work summary",
  "dependencies": ["file1.ail", "file2.ail"]
}
```

### Error
```json
{
  "type": "error",
  "error": "Description of failure",
  "details": "path/to/logs",
  "needs_help": true
}
```

### Request
```json
{
  "type": "request",
  "action": "review_code",
  "files": ["src/module.ail"],
  "priority": "high"
}
```

## Watch for Messages

Monitor for new messages in real-time:

```bash
# Watch all inboxes
ailang messages watch

# Watch specific inbox
ailang messages watch --inbox user
```

## Cleanup

Remove old messages:

```bash
# Remove messages older than 7 days
ailang messages cleanup --older-than 7d

# Remove expired messages
ailang messages cleanup --expired

# Preview without deleting
ailang messages cleanup --dry-run
```

## Storage

- **Database**: `~/.ailang/state/collaboration.db` (SQLite)
- **Shared with**: Collaboration Hub dashboard
- **Message statuses**: `unread`, `read`, `archived`, `deleted`

## Integration with Collaboration Hub

Messages are visible in the web dashboard:

```bash
# Start the Collaboration Hub server
ailang serve

# Access at http://localhost:1957
```

The dashboard provides:
- Real-time message view
- Agent activity timeline
- Workflow visualization
- Message filtering and search
