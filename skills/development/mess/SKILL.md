---
name: mess
description: Request physical-world tasks from human executors. Use for observations, checks, actions, photos, and purchases that require physical presence.
---

# MESS - Meatspace Execution and Submission System

## Overview

MESS enables AI agents to request physical-world tasks from human executors. Use MESS when you need something done in the real world that you cannot do yourself.

## When to Use MESS

**Use MESS for:**
- Observations: "Is the garage door closed?", "What's the temperature outside?"
- Physical checks: "Is there milk in the fridge?", "Did the package arrive?"
- Actions: "Turn off the living room light", "Start the dishwasher"
- Photos: "Take a picture of the backyard", "Show me what's in the pantry"
- Purchases: "Order more coffee pods", "Buy tickets for Saturday"

**Don't use MESS for:**
- Information you can look up online
- Tasks that don't require physical presence
- Emergencies (call 911 instead)

## Components

| Component | Purpose | Documentation |
|-----------|---------|---------------|
| MCP Server | Claude Desktop integration | [mcp/SKILL.md](mcp/SKILL.md) |
| Exchange Server | Self-hosted REST API | [server/SKILL.md](server/SKILL.md) |
| Web Client | Human executor interface | [client/SKILL.md](client/SKILL.md) |

## Quick Start (MCP Server)

If you have the MESS MCP server configured, you have two tools:

### `mess` - Create or update requests

```yaml
# Create a new request
- v: 1.0.0
- request:
    intent: Check if the garage door is closed
    context:
      - Getting ready for bed
    response_hint:
      - image
```

### `mess_status` - Check request status

```yaml
# Check all pending requests
ref: null

# Check specific request
ref: "2026-02-01-001"
```

## Request Lifecycle

```
pending → claimed → completed
                 → failed
                 → needs_input → claimed → ...
```

1. **pending**: Request created, waiting for executor
2. **claimed**: Executor accepted, working on it
3. **completed**: Task done, response available
4. **failed**: Could not complete
5. **needs_input**: Executor needs clarification

## Best Practices

### Writing Good Requests

**Good:**
```yaml
intent: Check if the front door is locked
context:
  - About to go to sleep
  - Heard a noise earlier
response_hint:
  - text
```

**Bad:**
```yaml
intent: door
```

### Priority Levels

| Priority | Use When |
|----------|----------|
| `background` | Not time-sensitive, can wait hours |
| `normal` | Regular requests, respond when convenient |
| `elevated` | Somewhat urgent, within the hour |
| `urgent` | Time-critical, needs immediate attention |

### Response Hints

- `text` - Written response sufficient
- `image` - Photo would be helpful
- `video` - Video needed (rare)
- `audio` - Voice/sound recording

## Example Workflows

### Morning Check
```yaml
- request:
    intent: What's the weather like outside?
    context:
      - Deciding what to wear
    response_hint:
      - text
      - image
```

### Home Security
```yaml
- request:
    intent: Check all doors and windows are locked
    context:
      - Leaving for vacation tomorrow
    priority: elevated
    response_hint:
      - text
```

### Shopping Request
```yaml
- request:
    intent: Pick up milk and eggs from the store
    context:
      - Running low on breakfast supplies
      - Any brand is fine
    priority: normal
```

## Checking Results

After creating a request, use `mess_status` to check on it:

```yaml
# Response when completed
ref: "2026-02-01-001"
status: completed
intent: Check if the garage door is closed
executor_id: teague-phone
messages:
  - from: teague-phone
    mess:
      - response:
          content:
            - image: "data:image/jpeg;base64,..."
            - "Garage door is closed and locked"
```

## Error Handling

If a request fails or needs input:

```yaml
# Check status
ref: "2026-02-01-001"
status: needs_input
messages:
  - from: executor
    mess:
      - status:
          code: needs_input
          message: "Which garage door - front or back?"
```

Respond with clarification:
```yaml
- status:
    re: "2026-02-01-001"
    code: claimed
- response:
    re: "2026-02-01-001"
    content:
      - "The front garage door (main one)"
```
