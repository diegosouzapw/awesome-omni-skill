---
name: agency
description: Use Agency CLI to run parallel AI coding tasks in isolated Git worktrees. Invoke when user mentions "agency", "ag", parallel tasks, worktrees, or wants to run multiple coding agents simultaneously.
---

# Agency CLI

Agency is a command-line AI agent orchestrator that runs coding agents in isolated Git worktrees with tmux-managed sessions.

## Essential Commands

### Create and Start a Task

```bash
agency new <slug>                     # Create task, start session, attach immediately
ag new <slug>                         # Short alias
agency new <slug> -e                  # Open editor to write task description
agency new <slug> -f spec.pdf         # Attach a file during creation
agency new <slug> -f a.png -f b.pdf   # Attach multiple files

# Provide description inline (use heredoc for multi-line)
agency new <slug> <<'EOF'
Your task description here.
Can span multiple lines without escaping issues.
EOF
```

### Create a Draft (No Session Yet)

```bash
agency new --draft <slug>      # Create task file only, no worktree or session
agency start <slug>            # Start the draft (creates worktree and session)
```

Note: The "<slug>" should only contain alphanumerical characters and "-". You can keep it relatively short.

### List Tasks

```bash
agency tasks                   # Show all tasks with status, commits, and changes
ag tasks                       # Short alias
```

Output columns: ID, SLUG, STATUS, UNCOMMITTED, COMMITS, BASE, AGENT

**Status codes:**

- **Draft** - Task file exists but not started (no worktree or session created yet)
- **Stopped** - Worktree exists but session is not running (can be restarted with `agency start`)
- **Running** - Agent is actively processing/generating output
- **Idle** - Session exists but agent is waiting for input (task may be complete or paused)
- **Exited** - Session has terminated (agent finished or crashed)

### Merge and Cleanup

```bash
agency merge <slug>            # Merge task branch into base, then cleanup
```

This command:

1. Rebases task branch onto base branch
2. Fast-forward merges into base
3. Deletes the worktree
4. Deletes the task branch
5. Removes the task file

### Stop Without Merging

```bash
agency stop <slug>             # Stop session, keep worktree and branch
```

### Delete a Task

```bash
agency rm --yes <slug>         # Remove task file, worktree, and branch
agency reset <slug>            # Reset worktree and branch, keep task file
```

Use `rm` when you want to completely delete a task without merging changes. Use `reset` if you want to start over but keep the task description.

## Attach Files to Tasks

Files can be attached to a task to provide additional context.

```bash
agency new <slug> -f path/to/file.pdf   # Attach file during creation
agency files add <slug> path/to/file    # Add file to existing task
agency files add <slug> --from-clipboard # Paste image from clipboard
agency files list <slug>                 # List attached files
agency files rm <slug> <file-id>         # Remove a file
agency info                              # Show task context and files (inside session)
```

When files are attached, agents can run `agency info` to get the context. Files are accessible insie task worktrees via `.agency/local/files/`.

**You SHOULD attach files when:**

- The user mentions relevant files associated with the task
- You have created plans, specifications, or design documents that are not committed to the repository
- There are screenshots, diagrams, or reference materials relevant to the task
- The task requires context that exists outside the codebase

**Example:** If you created a markdown plan and want to delegate work to multiple agency tasks, attach that plan to each task so the spawned agents have the full context.

## When to Use Parallel Tasks

**Only run tasks in parallel if they are truly independent.** Tasks that modify the same files or depend on each other's output will cause merge conflicts or broken code.

**Good candidates for parallel tasks:**

- Features in completely separate parts of the codebase
- Independent bug fixes in different modules
- Adding tests for unrelated functionality

**Do NOT run in parallel:**

- Task B depends on Task A's output (run sequentially instead)
- Tasks that modify the same files or modules
- Refactoring that touches shared code

**When in doubt, run tasks sequentially.** Merge conflicts are painful to resolve and waste more time than running tasks one after another.

## Quick Workflow Example

```bash
# Create multiple parallel tasks
agency new --draft feature-auth <<'EOF'
Implement user authentication
EOF
agency new --draft feature-api <<'EOF'
Build REST API endpoints
EOF
agency new --draft fix-tests <<'EOF'
Fix failing unit tests
EOF

# Start all tasks
agency start --no-attach feature-auth
agency start --no-attach feature-api
agency start --no-attach fix-tests

# Or start tasks on creation
agency new --no-attach feature-auth <<'EOF'
Implement user authentication
EOF
agency new --no-attach feature-api <<'EOF'
Build REST API endpoints
EOF
agency new --no-attach fix-tests <<'EOF'
Fix failing unit tests
EOF

# Check status
agency tasks

# When done, merge back
agency merge feature-auth
```

## Sandboxing

**CRITICAL: ALL `agency` commands MUST be run outside the sandbox.**

If you are running in a sandbox you will get errors like `run/agency-tmux.sock (Operation not permitted)`.

IMPORTANT: Despite the error message, this is not because the daemon is not started. It is because the sandbox cannot access the tmux socket file. You MUST run ALL agency commands outside the sandbox:

- agency new ...
- agency start ...
- agency stop ...
- agency tasks ...
- agency merge ...
- agency rm ...
- agency reset ...
- ALL other agency commands

## More Information

Run `agency --help` for the full command reference.
