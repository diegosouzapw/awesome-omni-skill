---
name: codex-implement
description: Delegate complex implementation tasks to Codex. Use when user says codex implement, delegate implementation, or has a complex coding task.
---

# Codex Implement Skill

Delegate complex implementation tasks to the Codex agent for high-quality code.

## When to Use

- Complex multi-file implementations
- Algorithms requiring careful design
- User explicitly wants Codex to implement

## Steps

Before any file operations, resolve the `.agent-collab` directory so commands work outside the project root:

```bash
AGENT_COLLAB_DIR="${AGENT_COLLAB_DIR:-}"
if [ -n "$AGENT_COLLAB_DIR" ]; then
  if [ -d "$AGENT_COLLAB_DIR/.agent-collab" ]; then
    AGENT_COLLAB_DIR="$AGENT_COLLAB_DIR/.agent-collab"
  elif [ ! -d "$AGENT_COLLAB_DIR" ]; then
    AGENT_COLLAB_DIR=""
  fi
fi

if [ -z "$AGENT_COLLAB_DIR" ]; then
  AGENT_COLLAB_DIR="$(pwd)"
  while [ "$AGENT_COLLAB_DIR" != "/" ] && [ ! -d "$AGENT_COLLAB_DIR/.agent-collab" ]; do
    AGENT_COLLAB_DIR="$(dirname "$AGENT_COLLAB_DIR")"
  done
  AGENT_COLLAB_DIR="$AGENT_COLLAB_DIR/.agent-collab"
fi
```

If `$AGENT_COLLAB_DIR` does not exist, stop and ask for the project root.

### 1. Gather Requirements

From user, collect:
- What needs implementing
- Target file paths
- Constraints and patterns to follow
- Dependencies and interfaces

### 2. Create Implementation Spec

Write to `$AGENT_COLLAB_DIR/requests/task.md`:

```markdown
# Task Request for Codex

## Task Type: IMPLEMENT

## Timestamp
[Current timestamp]

## Implementation Request
[Detailed description]

## Target Files
- Primary: [main file path]
- Secondary: [supporting files]

## Requirements
1. [Requirement 1]
2. [Requirement 2]
...

## Interfaces & Contracts
[Interfaces the code must satisfy]

## Existing Code Context
[Relevant existing code to integrate with]

## Patterns to Follow
[Reference existing patterns]

## Constraints
- [List constraints]
```

### 3. Update Status

Write `pending` to `$AGENT_COLLAB_DIR/status`

### 4. Trigger Codex

```bash
tmux send-keys -t 1 '$read-task' && sleep 0.5 && tmux send-keys -t 1 Enter Enter
```

### 5. Notify User

Tell user briefly that implementation was delegated to Codex.

### 6. Wait for Codex (Background Polling)

Start a background polling loop to wait for Codex to complete. Run this EXACT bash command (with `$AGENT_COLLAB_DIR/status`) using the Bash tool with `run_in_background: true`:

```bash
while [ "$(cat "$AGENT_COLLAB_DIR/status")" != "done" ]; do sleep 5; done; echo "CODEX_COMPLETE"
```

Note: Use 5 second intervals since implementations take longer.

CRITICAL: Use the resolved `$AGENT_COLLAB_DIR/status` path so polling works outside the project root. Use background execution so you can continue helping the user while waiting.

### 7. Auto-Read Response

When poll completes, automatically:
1. Read `$AGENT_COLLAB_DIR/responses/response.md`
2. Present the implementation to user
3. Ask if user wants to integrate the code
4. Reset `$AGENT_COLLAB_DIR/status` to `idle`
