---
name: tmux
description: Manage concurrent processes using tmux. Use when running servers, long tasks, or orchestrating multiple agents in separate panes. Essential for sending multi-line text or instructions to other tmux panes safely.
allowed-tools:
  - Bash
---

# Tmux Skill

This skill empowers you to manage multiple concurrent processes (like servers, watchers, or long builds) using `tmux` directly from the `Bash` tool.

Since you are likely already running inside a tmux session, you can spawn new windows or panes to handle these tasks without blocking your main communication channel.

## 1. Verify Environment & Check Status

First, verify you are running inside tmux:

```bash
echo $TMUX
```

If this returns empty, you are not running inside tmux and these commands will not work as expected.

Once verified, check your current windows:

```bash
tmux list-windows
```

## 2. Spawn a Background Process

To run a command (e.g., a dev server) in a way that persists and can be inspected:

1.  **Create a new detached window** with a specific name. This keeps it isolated and easy to reference.

    ```bash
    tmux new-window -n "server-log" -d
    ```

    _(Replace "server-log" with a relevant name for your task)_

2.  **Send the command** to that window.
    ```bash
    tmux send-keys -t "server-log" "npm start" C-m
    ```
    _(`C-m` simulates the Enter key)_

## 3. Sending Text to Panes

**Warning**: `send-keys` interprets control sequences. Multi-line text or text with special characters can trigger tmux modes (like `C-r` for search). Use the right method:

| Content Type | Method |
|--------------|--------|
| Simple shell command | `send-keys "cmd" C-m` |
| Single line, may have special chars | `send-keys -l "text"` then `send-keys C-m` |
| Multi-line text or instructions | `load-buffer` + `paste-buffer` |

**Literal mode** (`-l` flag) prevents interpreting escape sequences:

```bash
tmux send-keys -l -t "target" "text with C-r and other chars"
tmux send-keys -t "target" C-m
```

**Multi-line content** — always use load-buffer:

```bash
cat > /tmp/msg.txt << 'EOF'
Your multi-line content here.
Can include any characters safely.
EOF
tmux load-buffer /tmp/msg.txt && tmux paste-buffer -t "target"
tmux send-keys -t "target" C-m
```

## 4. Interacting with Other Agents

When sending instructions to another Claude instance running in a tmux pane:

```bash
cat > /tmp/instructions.txt << 'EOF'
Fix the authentication bug in src/auth.ts:
1. The token validation is missing null checks
2. Add proper error handling for expired tokens
EOF
tmux load-buffer /tmp/instructions.txt && tmux paste-buffer -t %31
tmux send-keys -t %31 C-m
```

Never use `send-keys` directly for prompts or instructions — the text will likely contain characters that trigger tmux modes.

## 5. Inspect Output (Read Logs)

You can read the output of that pane at any time without switching your context.

**Get the current visible screen:**

```bash
tmux capture-pane -p -t "server-log"
```

**Get the entire history (scrollback):**

```bash
tmux capture-pane -p -S - -t "server-log"
```

_Use this if the output might have scrolled off the screen._

## 6. Interact with the Process

If you need to stop or restart the process:

**Send Ctrl+C (Interrupt):**

```bash
tmux send-keys -t "server-log" C-c
```

**Kill the window (Clean up):**

```bash
tmux kill-window -t "server-log"
```

## 7. Advanced: Chaining Commands

You can chain multiple tmux commands in a single invocation using `';'` (note the quotes to avoid interpretation by the shell). This is faster and cleaner than running multiple `tmux` commands.

Example: Create window and start process in one go:

```bash
tmux new-window -n "server-log" -d ';' send-keys -t "server-log" "npm start" C-m
```

## Quick Reference

| Task | Command |
|------|---------|
| Create window | `tmux new-window -n "ID" -d` |
| Run command | `tmux send-keys -t "ID" "cmd" C-m` |
| Send literal text | `tmux send-keys -l -t "ID" "text"` |
| Send multi-line | `tmux load-buffer file && tmux paste-buffer -t "ID"` |
| Read output | `tmux capture-pane -p -t "ID"` |
| Interrupt | `tmux send-keys -t "ID" C-c` |
| Kill window | `tmux kill-window -t "ID"` |
