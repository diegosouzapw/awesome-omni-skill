---
name: cursor
description: Translates task requirements into Cursor CLI commands. Used by cursor-driver agent to execute coding tasks via Cursor.
---

# Cursor CLI Skill Guide

## Baseline Rules

Always apply these for programmatic (headless) execution:
- `-p "<prompt>"` — required for headless mode
- `--output-format text` — recommended for clean output capture

## Command Templates

### New Task (analysis/read-only)
```bash
agent -p "<prompt>" --mode ask --output-format text
```

### New Task (with file edits)
```bash
agent -p "<prompt>" --mode agent --output-format text
```

### New Task (planning only)
```bash
agent -p "<prompt>" --mode plan --output-format text
```

### With model selection
```bash
agent -p "<prompt>" --model gpt-5 --output-format text
```

### Resume Session (latest)
```bash
agent resume -p "<prompt>" --output-format text
```

### Resume Session (specific)
```bash
agent --resume="<chat-id>" -p "<prompt>" --output-format text
```

### List Previous Sessions
```bash
agent ls
```

## Execution Modes

| Task Type | Flag | Notes |
| --- | --- | --- |
| Analysis, review, Q&A | `--mode ask` | Read-only, no file changes |
| Create or edit files | `--mode agent` | Full agent capabilities |
| Planning, architecture | `--mode plan` | Generates plan without execution |

## Model Selection

When the calling agent specifies requirements, translate to flags:

| Requirement | Flag | Notes |
| --- | --- | --- |
| Default / high-quality | `--model gpt-5` | Best for complex reasoning |
| Fast / cheap | `--model gpt-4o` | Quick, straightforward tasks |
| Claude | `--model claude-sonnet` | Anthropic model option |

If not specified, use default model (no flag needed).

## Output Formats

| Format | Flag | Use Case |
| --- | --- | --- |
| Text | `--output-format text` | Programmatic processing, CI/automation |
| Default | (none) | Interactive/human-readable output |

## Cloud Agent Handoff

For complex tasks requiring cloud processing, prefix the prompt with `&`:
```bash
agent "& refactor the auth module and add comprehensive tests"
```

## Interpreting Results

### Success indicators
- Clean text output with expected content
- Exit code 0
- Response addresses the original request

### Failure indicators
- Non-zero exit code
- Error messages in output
- Missing expected deliverables

### Scope creep indicators
- Mentions of "I also..." or "While I was at it..."
- Changes to files not mentioned in the original request
- Response describes work beyond the original request

### Redirection indicators
- Output describes different work than requested
- "Instead of X, I did Y..."
- Solving a different problem than specified

## After Completion

Report to user: "You can resume this Cursor session by saying 'cursor resume'."

### Session Management
- `agent ls` — List all previous conversations
- `agent resume` — Resume most recent session
- `agent --resume="<id>"` — Resume specific session by ID

## Error Handling

- If command exits non-zero: stop and report the error
- If output contains error messages: summarize and report
- If output contains warnings: summarize and ask how to proceed

## Reference

### Useful Patterns

```bash
# Code review (read-only)
agent -p "Review src/auth.py for security issues" --mode ask --output-format text

# Implement feature
agent -p "Add input validation to the login form" --mode agent --output-format text

# Generate plan
agent -p "Plan the migration from REST to GraphQL" --mode plan --output-format text

# Continue previous work
agent resume -p "Now add unit tests for the changes"

# Cloud-powered complex task
agent "& analyze codebase architecture and suggest improvements"
```

### Interactive Mode

For complex multi-step tasks, you may run `agent` without `-p` to enter interactive mode:
```bash
agent
```
Then provide prompts conversationally. Use this when:
- The task requires back-and-forth dialogue
- You need to inspect intermediate results before continuing
- The task scope may evolve based on findings
