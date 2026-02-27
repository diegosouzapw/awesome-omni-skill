---
name: trent-tmux
description: Help with tmux, tmuxinator, and sesh session management. Use when the user asks about tmux sessions, tmuxinator templates, session switching, or terminal multiplexing.
---

# Tmux Session Management Skill

## Overview

This system uses tmux with tmuxinator templates and sesh for powerful session management.

## Configuration Locations

- **Tmuxinator templates**: `~/.config/tmuxinator/*.yml`
- **Tmux config**: `~/.tmux.conf`
- **Zsh tmux aliases/functions**: `~/.zshrc` (lines 37-72)
- **Active sessions**: Check with `tmux ls` or `sesh list`

## Tmuxinator Templates

### Current Templates

1. **default.yml** - Flexible project template
   - Takes directory argument: `tmuxinator start default ~/path`
   - Layout: nvim (top 70%), terminal + claude (bottom)
   - Dynamic session naming based on directory

2. **admin.yml** - System monitoring
   - Fixed name: "admin"
   - Layout: btop (left column), 2 terminals (right)
   - Purpose: System monitoring and administration

3. **config.yml** - Dotfiles editing
   - Fixed name: "config"
   - Root: `~/.config`
   - Layout: nvim (top 70%), terminal + claude (bottom)

### Template Structure

**CRITICAL: Pane Indexing**
This tmux configuration uses **1-based indexing** (set in `~/.tmux.conf`):
```tmux
set -g base-index 1        # Windows start at 1
set -g pane-base-index 1   # Panes start at 1
```
- First pane = 1, second = 2, third = 3
- Use `prefix + q` (Ctrl+Space then q) to see pane numbers visually
- When targeting panes in commands: `select-pane -t 1` (not -t 0)

**Tmuxinator YAML Structure:**
```yaml
---
name: session-name  # Fixed name (or use <%= File.basename(...) %> for dynamic)
root: ~/path        # Working directory

on_project_start: tmux resize-pane -t name:0.0 -y 70%  # Optional startup commands

windows:
  - window-name:
      layout: main-horizontal  # or main-vertical, tiled, even-horizontal, etc.
      panes:
        - nvim                 # Command to run in pane 1
        - # empty terminal     # Empty pane (interactive shell) - pane 2
        - claude --resume      # Command for pane 3
```

**Shell Function Structure (for creating windows dynamically):**
```bash
tnw() {
  # Get the session's root directory (uses 1-based indexing internally)
  local session_path=$(tmux display-message -p '#{session_path}')

  tmux new-window -c "$session_path" \; \
    split-window -v -c "$session_path" \; \
    split-window -v -c "$session_path" \; \
    select-layout main-horizontal \; \
    select-pane -t 1 \; \
    send-keys 'nvim' C-m \; \
    select-pane -t 3 \; \
    send-keys 'claude --resume' C-m \; \
    select-pane -t 2
}
```

**Key points for both:**
- Use `#{session_path}` to get session root directory (not current pane path)
- Pane indexing is 1-based: first pane = 1, not 0
- `send-keys 'command' C-m` executes commands in panes (C-m = Enter)
- `select-pane -t N` focuses pane N (1-based)
- `select-layout` applies tmux built-in layouts

### Common Layouts

- `main-horizontal` - Top pane large, others stacked below
- `main-vertical` - Left pane large, others stacked right
- `tiled` - All panes equal size
- `even-horizontal` - Panes side-by-side equal width
- `even-vertical` - Panes top-to-bottom equal height

## Session Management with `ts` (tmux session selector)

The `ts` function (defined in `~/.zshrc`) is the main session switcher:

```bash
ts  # Opens fzf with: "home" + tmuxinator templates + active sessions
```

### How `ts` Works

1. Lists all tmuxinator templates (except default.yml)
2. Adds "home" as an option
3. Shows all active tmux sessions from `sesh list`
4. On selection:
   - If session exists → attach with `sesh connect`
   - If tmuxinator template exists → `tmuxinator start <template>`
   - Otherwise → `tmuxinator start default <name>` (creates new project session)

### Key Insight

Template names come from the **filename** (e.g., `admin.yml` → "admin" in fzf), but the actual tmux session name comes from the `name:` field in the YAML file.

## Tmux Functions & Aliases

From `~/.zshrc`:

### Session Management
```bash
ts          # Tmux session selector (fzf for sessions/templates)
tns <name>  # Tmux new session - create git project & open with tmuxinator
t [name]    # Smart attach/create - attach if exists, create if not
ta <name>   # Attach to session
tn <name>   # Create new session
tls         # List sessions
tk <name>   # Kill session
```

### Window Management
```bash
tnw         # Tmux new window - create window with 3-pane layout
            # Layout: pane 1=nvim, pane 2=terminal (focused), pane 3=claude
            # Bound to: prefix + N
```

### Utilities
```bash
tcopy       # Copy tmux buffer to clipboard
tlines [n]  # Copy last n lines from tmux pane to clipboard
```

## Common Workflows

### Starting Sessions

```bash
ts                                        # Use fzf to select session/template
tns myproject                             # Create ~/git/myproject with git + tmuxinator
tns myproject config                      # Use 'config' template instead of default
tmuxinator start admin                    # Start admin monitoring
tmuxinator start config                   # Edit dotfiles
tmuxinator start default ~/git/myproject  # New project session with default template
```

### Creating Windows

```bash
tnw                # Create new window with 3-pane layout in current session
prefix + N         # Keybinding for tnw (Ctrl+Space then Shift+N)
prefix + c         # Create empty new window (default tmux)
prefix + ,         # Rename current window
```

### Creating New Templates

1. Create `~/.config/tmuxinator/name.yml`
2. Use existing templates as reference
3. Test with `tmuxinator start name`
4. Now available in `sf` automatically

## Auto-Start Behavior

On terminal startup (from `~/.zshrc:272-275`):
- Creates "home" session if it doesn't exist
- Runs in background (detached)
- Provides a default session to attach to

## Best Practices

1. **Fixed names for utilities**: Use `name: admin` for consistent session names
2. **Dynamic names for projects**: Use `<%= File.basename(...) %>` for project-based naming
3. **Exclude default from sf**: The default template is a fallback, not a session target
4. **Keep layouts simple**: 2-3 panes max for usability
5. **Use comments for empty panes**: `# interactive shell` for clarity

## Troubleshooting

- **Wrong session name**: Check `name:` field in YAML vs filename
- **Template not showing in sf**: Ensure `.yml` extension and not named "default"
- **Layout issues**: Try `tmux kill-session -t name` and restart
- **Pane sizes**: Use `on_project_start` with `tmux resize-pane` commands
