---
name: tmux
description: Remote-control tmux sessions for interactive CLIs by sending keystrokes and scraping pane output.
emoji: 🧵
requires:
  bins: ["tmux"]
os: ["darwin", "linux"]
---

# tmux Skill

Use tmux only when you need an interactive TTY. Prefer the shell tool for non-interactive tasks.

## When to use

- Running interactive CLIs (python REPL, node REPL, etc.)
- Long-running processes that need monitoring
- Parallel task execution with multiple sessions
- Any command that requires a persistent terminal

## Quickstart (isolated socket)

```bash
SOCKET_DIR="${TMPDIR:-/tmp}/chatdock-tmux"
mkdir -p "$SOCKET_DIR"
SOCKET="$SOCKET_DIR/chatdock.sock"
SESSION=chatdock-main

# Create a new session
tmux -S "$SOCKET" new -d -s "$SESSION" -n shell

# Send commands
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- 'echo Hello World' Enter

# Capture output
tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
```

After starting a session, always provide monitor commands to the user:

```
To monitor:
  tmux -S "$SOCKET" attach -t "$SESSION"
  tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
```

## Socket convention

- Default socket: `$TMPDIR/chatdock-tmux/chatdock.sock`
- Use a consistent socket path for all ChatDock sessions

## Targeting panes

Target format: `session:window.pane` (defaults to `:0.0`)

```bash
# List all sessions
tmux -S "$SOCKET" list-sessions

# List all panes
tmux -S "$SOCKET" list-panes -a
```

## Sending input safely

```bash
# Literal text (preferred)
tmux -S "$SOCKET" send-keys -t target -l -- "$cmd"

# With Enter key
tmux -S "$SOCKET" send-keys -t target -- "ls -la" Enter

# Control keys
tmux -S "$SOCKET" send-keys -t target C-c   # Ctrl+C
tmux -S "$SOCKET" send-keys -t target C-d   # Ctrl+D
```

## Watching output

```bash
# Capture recent history (last 200 lines)
tmux -S "$SOCKET" capture-pane -p -J -t target -S -200

# Watch for a specific pattern (poll every 0.5s)
while ! tmux -S "$SOCKET" capture-pane -p -t target | grep -q "pattern"; do
  sleep 0.5
done
echo "Pattern found!"
```

## Running Python REPLs

For Python REPLs, set `PYTHON_BASIC_REPL=1` to avoid readline issues:

```bash
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- 'PYTHON_BASIC_REPL=1 python3 -q' Enter
```

## Parallel execution (multiple agents)

```bash
SOCKET="${TMPDIR:-/tmp}/multi-agent.sock"

# Create multiple sessions
for i in 1 2 3; do
  tmux -S "$SOCKET" new-session -d -s "task-$i"
done

# Launch tasks
tmux -S "$SOCKET" send-keys -t task-1 "cd /project1 && npm test" Enter
tmux -S "$SOCKET" send-keys -t task-2 "cd /project2 && npm test" Enter
tmux -S "$SOCKET" send-keys -t task-3 "cd /project3 && npm test" Enter

# Poll for completion
for sess in task-1 task-2 task-3; do
  if tmux -S "$SOCKET" capture-pane -p -t "$sess" -S -3 | grep -q "\\$"; then
    echo "$sess: DONE"
  else
    echo "$sess: Running..."
  fi
done
```

## Cleanup

```bash
# Kill a specific session
tmux -S "$SOCKET" kill-session -t "$SESSION"

# Kill all sessions on socket
tmux -S "$SOCKET" kill-server

# Remove socket file
rm -f "$SOCKET"
```

## Windows/WSL Note

This skill requires macOS or Linux. On Windows, use WSL and install tmux inside WSL.
