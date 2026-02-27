---
name: task-bg
description: Launch and manage background tasks for parallel droid execution using background-manager.py. Factory Droid has no native run_in_background, so this uses manual Python-based background management.
---

<Purpose>
Task-bg launches droids as background tasks for parallel execution. Factory Droid CLI does not have native `run_in_background` support, so this skill uses `background-manager.py` to manually manage background task lifecycle: launch, monitor, collect results, and cleanup.
</Purpose>

<Use_When>
- Need to run a droid task in the background while continuing other work
- User says "background", "task-bg", "run in background"
- Long-running operations (builds, test suites, large refactors)
- Multiple independent tasks that should run concurrently
- Used internally by ultrawork and ralph for parallel droid orchestration
</Use_When>

<Do_Not_Use_When>
- Quick operations that complete in seconds -- run directly
- Tasks that need interactive feedback -- run in foreground
- For full parallel orchestration -- use `ultrawork` which manages task-bg internally
</Do_Not_Use_When>

<What_You_MUST_Do>
1. PARSE - Extract description, prompt, and droid selection
2. LAUNCH - Run `python3 hooks/background-manager.py launch ...`
3. TRACK - Task is recorded in `~/.factory/.omd/background-tasks/`
4. MONITOR - Use `list` for summary or `status <id>` for specific task
5. COLLECT - Use `output <id>` to read stdout/stderr
6. REPORT - Summary of all background task outcomes
</What_You_MUST_Do>

<What_You_MUST_NOT_Do>
1. DO NOT run quick operations in background
2. DO NOT forget to collect results
3. DO NOT skip cleanup of completed tasks
4. DO NOT use for tasks needing interactive feedback
</What_You_MUST_NOT_Do>

<Why_Manual_Python>
Factory Droid CLI does not support `run_in_background: true` like some other platforms. Instead, we use `background-manager.py` which:
- Executes `droid exec --auto <autonomy> -- "<prompt>"` as a background process
- Tracks tasks in `~/.factory/.omd/background-tasks/` as JSON files (task state) and `.out` files (output)
- Provides CLI commands: launch, complete, status, output, list, prune, cleanup, hook-complete
- Integrates with the `SubagentStop` hook via `hook-complete` for automatic completion tracking
</Why_Manual_Python>

<Background_Manager_CLI>

### Launch a task
```bash
python3 hooks/background-manager.py launch "<description>" "<prompt>" <droid> <parent_session_id> [autonomy_level] [model]
```
- `description`: Short task label (3-5 words)
- `prompt`: Full task instructions for the droid
- `droid`: Droid name (e.g., executor-med, explorer)
- `parent_session_id`: Current session ID (use "main" if unknown)
- `autonomy_level`: low | medium (default) | high
- `model`: Optional model override

### Examples
```bash
# Search task (low autonomy - read only)
python3 hooks/background-manager.py launch "Find auth patterns" "Find all authentication implementations in src/" explorer "main" low

# Implementation task (medium autonomy - default)
python3 hooks/background-manager.py launch "Fix API errors" "Fix all validation errors in src/api/" executor-med "main" medium

# Complex task (high autonomy + model override)
python3 hooks/background-manager.py launch "Refactor auth" "Refactor auth module to support OAuth2" executor-high "main" high opus

# Multiple tasks in parallel
python3 hooks/background-manager.py launch "Fix auth" "Fix auth module errors" executor-med "main"
python3 hooks/background-manager.py launch "Fix API" "Fix API route errors" executor-med "main"
python3 hooks/background-manager.py launch "Fix UI" "Fix frontend type errors" executor-med "main"
```

### Monitor tasks
```bash
# List all tasks with status summary (total, running, queued, completed, error)
python3 hooks/background-manager.py list

# Check specific task status (returns full task JSON)
python3 hooks/background-manager.py status <task_id>

# Get task stdout/stderr output from .out file
python3 hooks/background-manager.py output <task_id>
```

### Complete a task manually
```bash
python3 hooks/background-manager.py complete <task_id> "<result>" [error_message]
```

### Cleanup
```bash
# Prune stale/timed-out tasks (marks as ERROR, respects TTL of 30 min)
python3 hooks/background-manager.py prune

# Reset all tasks from memory (does NOT delete .json files)
python3 hooks/background-manager.py cleanup
```

### Hook integration (internal - called by SubagentStop hook)
```bash
# Reads session_id from stdin JSON, finds matching task, marks complete with transcript result
python3 hooks/background-manager.py hook-complete
```
</Background_Manager_CLI>

<Autonomy_Levels>
| Level | Droid exec flag | Use Case |
|-------|----------------|----------|
| `low` | `--auto low` | Read-only analysis, search, documentation |
| `medium` | `--auto medium` | Most development tasks (DEFAULT) |
| `high` | `--auto high` | Complex refactoring, broad file access |
</Autonomy_Levels>

<Available_Droids>
| Droid | Best For |
|-------|----------|
| `basic-searcher` | Simple file/pattern search |
| `explore` / `explorer` | Codebase exploration |
| `librarian` | External documentation research |
| `oracle` | Debugging and root cause analysis |
| `executor-low` | Simple changes, config edits |
| `executor-med` | Standard implementation (DEFAULT) |
| `executor-high` | Complex multi-file changes |
| `hephaestus` | Architecture-level work |
| `code-reviewer` | Code quality review |
| `security-auditor` | Security analysis |
| `test-engineer` | Test creation and coverage |
</Available_Droids>

<Steps>
1. **Parse**: Extract description, prompt, and droid selection from user input
2. **Launch**: Run `python3 hooks/background-manager.py launch ...` via Execute tool
3. **Track**: Task is automatically recorded in `~/.factory/.omd/background-tasks/<task_id>.json`
4. **Monitor**: Use `list` for summary or `status <id>` for specific task
5. **Collect**: Use `output <id>` to read stdout/stderr from `<task_id>.out`
6. **Report**: Summary of all background task outcomes
</Steps>

<Task_Lifecycle>
```
launch() -> RUNNING -> complete() -> COMPLETED
                    -> error      -> ERROR
                    -> prune()    -> ERROR (timeout/stale)
```

Files per task:
- `~/.factory/.omd/background-tasks/<task_id>.json` - Task state (status, progress, metadata)
- `~/.factory/.omd/background-tasks/<task_id>.out` - Stdout/stderr from `droid exec`

Hook: `SubagentStop` event calls `background-manager.py hook-complete` which reads transcript and auto-completes the matching task.
</Task_Lifecycle>

<Output_Format>
## Background Tasks

| Task ID | Droid | Status | Result |
|---------|-------|--------|--------|
| [id] | executor-med | completed | [summary] |
| [id] | test-engineer | running | 50% |

### Completed
- [Task]: [result summary]

### Still Running
- [Task]: [progress]
</Output_Format>
