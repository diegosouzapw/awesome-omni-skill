---
name: kitten
description: Control and interact with Kitty terminal windows - list windows, read output, send commands, and launch new terminals
user_invocable: true
---

# Kitten Terminal Remote Control

Control your Kitty terminal windows directly from the agent. See what's running in every terminal, send commands, and manage windows.

## Quick Start: Get Oriented First

**Always start with `summary` to see all your terminals at a glance:**

```bash
python3 {{skill_dir}}/scripts/kitten_control.py summary
```

This gives you a complete markdown overview of ALL open terminals showing:

```
════════════════════════════════════════════════════════════════════════════════
##  WINDOW 172  │  🟡 ACTIVE  │  ✅ READY  │  🐚 shell
════════════════════════════════════════════════════════════════════════════════

| Property | Value |
|----------|-------|
| **Window ID** | `172` |
| **Status** | ✅ READY - Can receive commands |
| **Process** | 🐚 shell |
| **CWD** | `/Users/me/workspace/myproject` |
| **Git Branch** | `main` |
| **Session Age** | 2d ago |

**Quick commands:**
- Send text: `send-text -w 172 -e "your command"`
- Send Ctrl+C: `send-key -w 172 ctrl+c`

```last 20 lines of terminal output...```
```

### What Summary Shows You

| Annotation | Meaning |
|------------|---------|
| **✅ READY** | Shell is at prompt, can receive commands |
| **⏳ BUSY** | Running a process, may need Ctrl+C first |
| **🐚 shell** | Idle shell |
| **🤖 agent** | AI agent (codex, claude, etc.) |
| **🔗 ssh** | SSH connection (shows target host) |
| **🔨 build** | Build process (make, npm, cargo) |
| **🌐 server** | Running server |
| **📝 editor** | vim, nvim, etc. |
| **Git Branch** | Current branch in that directory |
| **SSH Target** | Remote host for SSH sessions |

### Summary Options

```bash
# Default: 20 lines from each window
python3 {{skill_dir}}/scripts/kitten_control.py summary

# More context: 50 lines per window
python3 {{skill_dir}}/scripts/kitten_control.py summary -n 50

# Just last command output (cleaner)
python3 {{skill_dir}}/scripts/kitten_control.py summary -e last_cmd_output

# Full scrollback (large output)
python3 {{skill_dir}}/scripts/kitten_control.py summary -e all -n 100
```

---

## Interacting With Windows

Once you've identified a window from the summary, use its **Window ID** to interact:

### Send Commands

```bash
# Send a command and press Enter
python3 {{skill_dir}}/scripts/kitten_control.py send-text -w 172 -e "git status"

# Send text without Enter
python3 {{skill_dir}}/scripts/kitten_control.py send-text -w 172 "partial text"

# Send Ctrl+C to interrupt
python3 {{skill_dir}}/scripts/kitten_control.py send-key -w 172 ctrl+c

# Send other keys: escape, enter, tab, up, down, ctrl+d, etc.
python3 {{skill_dir}}/scripts/kitten_control.py send-key -w 172 escape
```

### Read Output

```bash
# Get current screen content
python3 {{skill_dir}}/scripts/kitten_control.py get-text -w 172

# Get last command's output (cleanest)
python3 {{skill_dir}}/scripts/kitten_control.py get-text -w 172 -e last_cmd_output

# Get full scrollback
python3 {{skill_dir}}/scripts/kitten_control.py get-text -w 172 -e all
```

### Window Management

```bash
# Focus a window
python3 {{skill_dir}}/scripts/kitten_control.py focus -w 172

# Close a window
python3 {{skill_dir}}/scripts/kitten_control.py close -w 172

# Launch new window in specific directory
python3 {{skill_dir}}/scripts/kitten_control.py launch --cwd /path/to/project

# Launch new tab with title
python3 {{skill_dir}}/scripts/kitten_control.py launch --type tab --title "Build"
```

---

## Common Workflows

### Check on a build/process
```bash
# 1. Get summary to find the window
python3 {{skill_dir}}/scripts/kitten_control.py summary -n 10

# 2. Found window 236 running a build, get its output
python3 {{skill_dir}}/scripts/kitten_control.py get-text -w 236 -e last_cmd_output
```

### Run a command in an existing shell
```bash
# 1. Find a READY shell from summary
python3 {{skill_dir}}/scripts/kitten_control.py summary -n 5

# 2. Window 172 is ✅ READY, send command
python3 {{skill_dir}}/scripts/kitten_control.py send-text -w 172 -e "make test"

# 3. Check the result
python3 {{skill_dir}}/scripts/kitten_control.py get-text -w 172 -e last_cmd_output
```

### Stop a runaway process
```bash
# Send Ctrl+C
python3 {{skill_dir}}/scripts/kitten_control.py send-key -w 236 ctrl+c

# Or send SIGINT signal
python3 {{skill_dir}}/scripts/kitten_control.py signal -w 236 SIGINT
```

---

## Requirements

- Kitty terminal with `allow_remote_control yes` in kitty.conf
- The `kitten` command in PATH (comes with Kitty)
