---
name: iterm-worktree
description: Manage git worktrees with iTerm2 integration. Use when user asks to create isolated development environments, work on features in parallel, manage worktrees, open worktrees in new tabs/windows, switch between worktrees, or close completed worktrees. Triggers on "create worktree", "new worktree", "feature branch with worktree", "open worktree", "switch worktree", "close worktree", "list worktrees", or mentions wanting to work on multiple features simultaneously.
---

# iTerm2 Worktree Manager

Automate git worktree management with iTerm2 tab/window integration for parallel feature development.

## Quick Reference

| Command | Description |
|---------|-------------|
| `create <branch>` | Create worktree + branch, open in iTerm2 |
| `close <worktree>` | Safely remove worktree after validation |
| `list` | Show all worktrees with iTerm2 tab status |
| `switch <worktree>` | Focus existing worktree tab |
| `open <worktree>` | Open existing worktree in new tab |

## Usage

Run the script at `scripts/worktree.py`:

```bash
python3 scripts/worktree.py <command> [options]
```

### Create Worktree

Create a new worktree with feature branch:

```bash
python3 scripts/worktree.py create feature-auth
python3 scripts/worktree.py create feature-auth --base develop
python3 scripts/worktree.py create feature-auth --from-current  # Branch from current branch
python3 scripts/worktree.py create feature-auth --open-mode new_window
python3 scripts/worktree.py create feature-auth --claude --task "Implement user authentication"
```

Options:
- `--base, -b`: Base branch (default: main/master)
- `--from-current, -f`: Branch from current branch instead of main/master
- `--path, -p`: Custom worktree path (default: sibling directory)
- `--open-mode, -o`: `new_tab`, `new_window`, `new_pane_right`, `new_pane_below`
- `--no-iterm`: Skip iTerm2 automation
- `--claude, -c`: Launch Claude in the new tab
- `--task, -t`: Task description for Claude

### Close Worktree

Safely remove a worktree after validating clean state:

```bash
python3 scripts/worktree.py close feature-auth
python3 scripts/worktree.py close feature-auth --delete-branch
python3 scripts/worktree.py close feature-auth --force
```

Options:
- `--force, -f`: Force removal despite uncommitted/unpushed changes
- `--delete-branch, -d`: Also delete the branch

### List Worktrees

Show all worktrees with iTerm2 tab indicators:

```bash
python3 scripts/worktree.py list
python3 scripts/worktree.py list --json
```

### Switch to Worktree

Focus an existing iTerm2 tab, or open new if not found:

```bash
python3 scripts/worktree.py switch feature-auth
```

### Open Worktree

Open existing worktree in iTerm2:

```bash
python3 scripts/worktree.py open feature-auth
python3 scripts/worktree.py open feature-auth --open-mode new_pane_right
python3 scripts/worktree.py open feature-auth --force  # Open new tab even if exists
```

## Typical Workflow

1. **Start feature**: `create feature-name` - Creates worktree + branch in new tab
2. **Work in isolation**: Each worktree has its own directory and git state
3. **Switch context**: `switch other-feature` - Jump between parallel features
4. **Complete feature**: Push changes, merge PR
5. **Cleanup**: `close feature-name --delete-branch` - Remove worktree and branch

## Safety Features

- Validates no uncommitted changes before closing
- Checks for unpushed commits before closing
- Auto-detects default branch (main/master)
- Prevents duplicate branch/path creation
