---
name: codex-delegator-skill
description: Delegate complex tasks to Codex with structured prompts, tmux sessions, and logging utilities.
license: MIT
---

# Delegate.sh - AI Task Delegation Pattern

A utility for delegating complex tasks to an AI coding agent (`codex`) with structured prompts, logging, and monitoring.

## Quick Start

```bash
# Simple task
./delegate.sh -r "Developer" -g "Create a hello world file" -t "Create hello.md with Hello World"

# Full specification
./delegate.sh \
  -r "Senior Software Architect" \
  -g "Document the authentication flow" \
  -a "ADR document exists with sequence diagrams" \
  -w "Need docs for onboarding" \
  -t "Analyze auth module and create comprehensive documentation" \
  -n "auth-docs"
```

## Command Line Options

| Option | Long Form               | Description                                         |
| ------ | ----------------------- | --------------------------------------------------- |
| `-r`   | `--role`                | Role description of the ideal person for this task  |
| `-c`   | `--common-role`         | Use a predefined role from `common-roles/<name>.md` |
| `-g`   | `--goal`                | The goal/objective of the task                      |
| `-a`   | `--acceptance-criteria` | Success criteria for task completion                |
| `-w`   | `--why`                 | The reasoning behind the task                       |
| `-t`   | `--task-detail`         | Detailed task instructions                          |
| `-n`   | `--name`                | Task name (used in log filenames)                   |
| `-f`   | `--foreground`          | Run in foreground instead of tmux session           |
| `-l`   | `--list-roles`          | List available common roles                         |
| `-s`   | `--status`              | Show detailed status of running sessions            |
|        | `--check`               | Quick check if a specific session is running/done   |
|        | `--check-all`           | Quick status check of all delegate sessions         |
|        | `--continue`            | Send follow-up message to continue a conversation   |
| `-k`   | `--kill`                | Kill a specific running session                     |
|        | `--clean`               | Kill all idle sessions (completed, waiting on read) |
|        | `--clean-all`           | Kill ALL delegate sessions (including running)      |
|        | `--purge [name]`        | Kill session(s) AND delete their log files          |
| `-h`   | `--help`                | Show help message                                   |

## Common Roles

Predefined roles are stored in `common-roles/` as markdown files. Use them with `-c` or `--common-role`:

```bash
# List available roles
./delegate.sh --list-roles

# Use a common role
./delegate.sh -c feature-analyst -g "Extract auth features" -t "..."
./delegate.sh --common-role architect -g "Document API patterns" -t "..."
```

### Available Roles

| Role Name                | Description                                           |
| ------------------------ | ----------------------------------------------------- |
| `feature-analyst`        | BDD/Gherkin expert for user-facing feature extraction |
| `architect`              | ADR writer focusing on patterns, not tech stack       |
| `code-reviewer`          | Identifies bugs, security issues, and improvements    |
| `technical-writer`       | Creates clear documentation with examples             |
| `refactoring-specialist` | Incremental improvements with backward compatibility  |
| `ux-designer`            | UX/UI design documentation and design systems         |

### Adding Custom Roles

Create a markdown file in `common-roles/`:

```markdown
# My Custom Role

Description of the role's expertise, focus areas, and approach.
Keep it to 2-3 sentences that set the right mindset.
```

Then use it: `./delegate.sh -c my-custom-role -g "..." -t "..."`

## The Prompt Structure

The script builds an XML-structured prompt that `codex` receives:

```xml
<role>
{Role description - WHO should do this work}
</role>
<task>
  <goal>{WHAT we want to achieve}</goal>
  <acceptanceCriteria>{HOW we know it's done}</acceptanceCriteria>
  <theWhy>{WHY we're doing this}</theWhy>
  <taskDetail>{Detailed instructions}</taskDetail>
</task>
```

### Why This Structure?

- **Role**: Sets the persona/expertise level. The agent will approach the task from this perspective.
- **Goal**: Clear, concise objective. Keeps the agent focused.
- **Acceptance Criteria**: Measurable success conditions. Prevents ambiguity about completion.
- **The Why**: Context helps the agent make better decisions when facing trade-offs.
- **Task Detail**: Step-by-step instructions, file paths, output formats, examples.

## Logging

All output is logged to `/tmp/delegate-logs/`:

| File                            | Contains                                                    |
| ------------------------------- | ----------------------------------------------------------- |
| `<timestamp>_<name>_stdout.log` | Agent's final communications/summary                        |
| `<timestamp>_<name>_stderr.log` | Verbose thought process, command executions, debugging info |

Logs are **tee'd** - you see output live while it's also saved.

## Running Tasks in Parallel with tmux

Tasks automatically run in tmux sessions by default. This means:
- Jobs persist even if your terminal disconnects
- Easy monitoring with `delegate.sh --status`
- Simple session management built into the script

### Starting Tasks

```bash
# Start a background task (default - uses tmux)
./delegate.sh -c feature-analyst -g "Extract features" -n "my-features"

# Start multiple parallel tasks
./delegate.sh -c feature-analyst -g "Extract features from auth" -n "auth-features"
./delegate.sh -c architect -g "Document API patterns" -n "api-arch"
./delegate.sh -c technical-writer -g "Write README" -n "docs"

# Run in foreground (blocks until complete)
./delegate.sh -f -c feature-analyst -g "Quick task" -n "quick"
```

### Monitoring Tasks

```bash
# Check all running delegate sessions
./delegate.sh --status

# Quick check if tasks are running or done
./delegate.sh --check-all

# Check a specific task
./delegate.sh --check my-features

# Filter by name pattern
./delegate.sh --status features

# Attach to watch a session live (Ctrl+B, D to detach)
tmux attach -t delegate-my-features

# Tail the logs directly
tail -f /tmp/delegate-logs/*my-features*_stderr.log
```

### Continuing a Conversation

If a task completes but you want to give follow-up instructions without starting fresh:

```bash
# Continue an existing session with a new message
./delegate.sh --continue my-features "Now also add edge case scenarios for authentication failures"

# The continuation will:
# - Find the original codex session ID from logs
# - Wait if previous task is still running (exponential backoff)
# - Send your message to continue that conversation
# - Preserve all context from the original task
```

This is useful when:
- The agent finished but missed something
- You want to refine or extend the output
- You have follow-up tasks that build on previous work
- Queuing up the next instruction while current task runs (it will wait automatically)

### Managing Sessions

```bash
# Kill a specific session
./delegate.sh --kill my-features

# Clean up all idle sessions (completed tasks still open)
./delegate.sh --clean

# Kill ALL delegate sessions (including running ones)
./delegate.sh --clean-all

# Delete a session AND its log files completely
./delegate.sh --purge my-features

# Delete ALL sessions and ALL logs (with confirmation prompt)
./delegate.sh --purge
```

### Session States

The `--check-all` command shows three possible states:

| State | Icon | Meaning |
|-------|------|---------|  
| Running | ⏳ | Task is actively processing (codex running) |
| Idle | 💤 | Task complete, tmux session still open (waiting on read) |
| Done | ✅ | Session closed, only logs remain |

### Batch Jobs Pattern

```bash
#!/bin/bash
# run-research-jobs.sh

REPOS=("repo1" "repo2" "repo3")

for repo in "${REPOS[@]}"; do
    ./delegate.sh -c feature-analyst \
        -g "Extract features from $repo" \
        -n "${repo}-features"
    
    ./delegate.sh -c architect \
        -g "Extract architecture from $repo" \
        -n "${repo}-arch"
    
    sleep 1  # Brief pause between starts
done

echo "Started all jobs. Monitor with: ./delegate.sh --status"
```

### Waiting for Jobs to Complete

```bash
#!/bin/bash
# wait-for-jobs.sh

TASKS=("auth-features" "api-arch" "docs")

while true; do
    running=0
    for task in "${TASKS[@]}"; do
        if tmux has-session -t "delegate-$task" 2>/dev/null; then
            ((running++))
            echo "⏳ $task still running..."
        else
            echo "✅ $task completed"
        fi
    done
    
    if [[ $running -eq 0 ]]; then
        echo "🎉 All jobs complete!"
        break
    fi
    
    echo "--- $running jobs running, checking again in 60s ---"
    sleep 60
done
```

## Role Templates

Expanded role prompts are in `references/role-templates.md`. Runtime role files live in `common-roles/` and are used by `--common-role`.

## Task Detail Templates

Reusable task briefs are in `references/task-detail-templates.md`.

## Monitoring & Debugging

### Watch a Running Task

```bash
# Follow stderr (verbose output)
tail -f /tmp/delegate-logs/*my-task*_stderr.log

# Follow stdout (final output)
tail -f /tmp/delegate-logs/*my-task*_stdout.log
```

### Check if Tasks are Running

```bash
ps aux | grep "codex exec" | grep -v grep
```

### Search Past Logs

```bash
# Find tasks that created certain files
grep -r "created.*\.feature" /tmp/delegate-logs/

# Find errors
grep -r "error\|failed\|Error" /tmp/delegate-logs/*stderr.log
```

### Kill a Running Task

```bash
# Find the PID
ps aux | grep "codex exec" | grep -v grep

# Kill it
kill <PID>
```

## Best Practices

### 1. Be Specific in Task Details

❌ Bad:
```
-t "Document the code"
```

✅ Good:
```
-t "Create API documentation for /src/api/*.ts files. Include:
- Function signatures with parameter descriptions
- Return value documentation  
- Usage examples for each endpoint
- Error codes and their meanings
Output to /docs/api/"
```

### 2. Set Clear Acceptance Criteria

❌ Bad:
```
-a "Code should be good"
```

✅ Good:
```
-a "All functions have JSDoc comments, README.md exists with setup instructions, no TypeScript errors"
```

### 3. Use Meaningful Task Names

❌ Bad:
```
-n "task1"
```

✅ Good:
```
-n "auth-module-docs"
```

### 4. Provide Context with The Why

```
-w "We're onboarding 5 new developers next month and need documentation 
    to reduce ramp-up time from 2 weeks to 3 days"
```

### 5. Break Large Tasks into Smaller Ones

Instead of one massive task, create focused parallel tasks:
- Features extraction (one per module)
- Architecture extraction (separate from features)
- Documentation (separate from code changes)

## Example: Full Research Pipeline

Here's a complete example of researching multiple repositories:

```bash
#!/bin/bash
REPOS=("repo1" "repo2" "repo3")
OUTPUT_BASE="/path/to/output"

FEATURE_ROLE="Senior Business Analyst with BDD expertise..."
ARCH_ROLE="Senior Software Architect..."

for repo in "${REPOS[@]}"; do
    # Features job
    ./delegate.sh \
        -r "$FEATURE_ROLE" \
        -g "Extract features from $repo" \
        -a "Feature files in $OUTPUT_BASE/$repo/features/" \
        -t "Repo: /repos/$repo. Create .feature files..." \
        -n "${repo}-features" &
    
    # Architecture job  
    ./delegate.sh \
        -r "$ARCH_ROLE" \
        -g "Extract architecture from $repo" \
        -a "ADRs in $OUTPUT_BASE/$repo/architecture/" \
        -t "Repo: /repos/$repo. Create ADR files..." \
        -n "${repo}-architecture" &
    
    sleep 2
done

# Monitor
while pgrep -f "codex exec" > /dev/null; do
    echo "$(date): $(pgrep -fc 'codex exec') tasks running..."
    sleep 300
done

echo "All done! Output in $OUTPUT_BASE"
```

## Troubleshooting

### Task Hangs

```bash
# Check if codex is waiting for input
tail -20 /tmp/delegate-logs/*task-name*_stderr.log

# May need to kill and restart with more specific instructions
```

### No Output Files Created

- Check the task detail has correct output paths
- Verify the agent has write permissions
- Look for errors in stderr log

### Agent Goes Off Track

- Make acceptance criteria more specific
- Add explicit "DO NOT" instructions in task detail
- Break into smaller, more focused tasks

## Files

- `delegate.sh` - The main delegation script
- `common-roles/` - Common role prompts used by `--common-role`
- `references/` - Expanded role and task template references
- `scripts/validate-skills-ref.py` - Skills spec validation script
