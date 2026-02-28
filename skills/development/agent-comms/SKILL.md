---
name: agent-comms
description: "Send messages to other Claude Code sessions via tmux. Hand off complex debugging context, get second opinions, share detailed technical findings across sessions."
---

# Agent Communication

Send messages to and read output from other tmux sessions running Claude Code. Built for coordination between parallel agent sessions.

## Setup

Make the script executable (one-time):
```bash
chmod +x ~/.claude/skills/agent-comms/scripts/agent-msg
```

## Message Format

**Always prefix messages with a header so the receiving session knows it's agent-to-agent:**

```
[AGENT-MSG] From: <your-session-name>
<your actual message here>
```

Get your session name:
```bash
tmux display-message -p '#S'
```

## Commands

```bash
SCRIPT=~/.claude/skills/agent-comms/scripts/agent-msg

# List available sessions
$SCRIPT list

# Send message (types into their terminal + Enter)
$SCRIPT send <session> "<message>"

# Read their terminal output
$SCRIPT read <session> --lines 50

# Get their working directory
$SCRIPT info <session>
```

## Workflow

1. **Get your session name** for identification:
   ```bash
   tmux display-message -p '#S'
   ```

2. **List sessions** to find the target:
   ```bash
   ~/.claude/skills/agent-comms/scripts/agent-msg list
   ```

3. **Read their context** before sending:
   ```bash
   ~/.claude/skills/agent-comms/scripts/agent-msg read worker-1 --lines 50
   ```

4. **Send your message with identification**:
   ```bash
   ~/.claude/skills/agent-comms/scripts/agent-msg send worker-1 "[AGENT-MSG] From: main-session

   Hit a wall on the websocket reconnection logic. Here's where I'm stuck: [detailed context...]"
   ```

5. **Read their response**:
   ```bash
   ~/.claude/skills/agent-comms/scripts/agent-msg read worker-1 --lines 30
   ```

## Example: Complex Debug Handoff

You've been debugging a native module issue for 20 minutes. It's a multi-layered problem involving Node versions, native rebuilds, and cross-project dependencies. You want another agent to take a look with full context:

```bash
# Check what sessions are running
~/.claude/skills/agent-comms/scripts/agent-msg list
# SESSION                  WINDOWS  ATTACHED
# worker-debug             1        no
# main-session             1        yes

# Send the full technical context
~/.claude/skills/agent-comms/scripts/agent-msg send worker-debug "[AGENT-MSG] From: main-session

Need a second opinion on a native module conflict. Here's what I've traced:

THE ISSUE:
- electron-builder fails with 'better-sqlite3 was compiled against a different Node.js version'
- This started after I ran nvm use 22 in another terminal for a different project

WHAT I'VE FOUND:
1. Main project expects Node 20.x for native modules
2. better-sqlite3 got rebuilt against Node 22 when I ran npm install in the other project
3. They share a global .node-gyp cache at ~/.node-gyp
4. The rebuild pulled in headers for 22.x, now electron can't load the module

WHAT I'VE TRIED:
- npm rebuild better-sqlite3 (still uses cached 22.x headers)
- Deleted node_modules, fresh install (same issue)
- electron-rebuild --force (fails with same ABI mismatch)

QUESTION:
Do I need to clear the global .node-gyp cache? Or is there a way to force better-sqlite3 to rebuild with electron's Node headers specifically? Wondering if I'm missing something obvious."

# Wait for them to analyze, then check response
~/.claude/skills/agent-comms/scripts/agent-msg read worker-debug --lines 50
```

## When to Use This

- **Complex debugging**: You've built up 15 minutes of context tracing an issue - share it rather than have another agent start from scratch
- **Cross-project conflicts**: Version mismatches, shared caches, native module issues that span multiple repos
- **Architectural decisions**: "Here's the tradeoff I'm seeing between approach A and B, with all the constraints I've discovered"
- **Rubber duck with context**: Sometimes you just need another agent to see your full reasoning and spot the flaw

## Why the Header Matters

The receiving Claude Code session sees your message as user input. Without the `[AGENT-MSG]` header, it might think a human is typing. The header tells it:
- This is automated agent-to-agent communication
- Which session sent it (for replies)

## Notes

- Session names follow patterns like `terminal-HHMMSS` or custom names you assign
- Always include the `[AGENT-MSG]` prefix
- Be specific about what you need
- The other agent can reply using the same skill
