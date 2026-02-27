---
name: julien-workflow-advice-codex
description: Get OpenAI Codex CLI's opinion on code, bugs, or implementation. Use when you want a second AI perspective during coding sessions.
triggers:
  - "ask codex"
  - "codex opinion"
  - "codex review"
  - "what does codex think"
  - "demande à codex"
  - "avis de codex"
  - "codex pense quoi"
  - "get codex feedback"
  - "codex second opinion"
  - "openai codex review"
---

# Ask Codex

Get feedback from OpenAI's Codex CLI on code, bugs, or implementation approaches.

## Prerequisites

## Observability

**First**: At the start of execution, display:
```
🔧 Skill "julien-workflow-advice-codex" activated
```

- Codex CLI installed: `npm install -g @openai/codex`
- Authenticated: `codex login` or sign in with ChatGPT account
- Verify: `codex --version`

## When to Use

- Want a second opinion on code quality
- Need bug analysis
- Code review before commit
- Performance optimization suggestions

## How to Invoke

The user will ask something like:
- "ask codex what it thinks about this code"
- "get codex's opinion on this implementation"
- "fais reviewer ce fichier par codex"

## Execution

### For file review (non-interactive)
```bash
codex exec "Review <file_path> and provide 3 specific improvements for production"
```

### For bug analysis
```bash
codex exec "Analyze <file_path> for potential bugs and edge cases"
```

### For dedicated code review
```bash
codex review
```
Note: `codex review` opens an interactive review session - use `codex exec` for non-interactive feedback.

## Output Format

Present Codex's response clearly to the user. If Codex suggests improvements, summarize them as actionable items.

## Examples

### File review
**User**: "demande à codex de reviewer le benchmark"

```bash
codex exec "Review scripts/benchmark-semantic-router.py for improvements and potential issues"
```

### Bug analysis
**User**: "ask codex to find bugs in this file"

```bash
codex exec "Analyze scripts/semantic-skill-router.py for potential bugs, edge cases, and error handling gaps"
```

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `codex: command not found` | CLI not installed | `npm install -g @openai/codex` |
| `Not authenticated` | Not logged in | Run `codex login` |
| `Model not available` | Account tier issue | Check ChatGPT subscription |
| Response timeout | Large codebase | Use specific file path, not whole repo |

## Notes

- Default model: GPT-5.1 Codex Max
- Supports reasoning levels
- `codex exec` for non-interactive (scripted) use
- `codex review` for interactive review session
- Timeout: allow up to 60s for response

## Skill Chaining

### Input Expected
- File path to review, OR
- Bug analysis request from user

### Output Produced
- **Format**: Markdown feedback from Codex
- **Side effects**: None (read-only)

### Compatible Skills
- Can be used alongside `julien-workflow-advice-gemini` for multiple AI opinions
- Use after writing code, before commit

### Tools Used
- `Bash` (usage: run codex CLI command, timeout 60s)
