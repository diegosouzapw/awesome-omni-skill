---
name: codex
description: Use when the user asks to run Codex CLI (codex exec, codex resume) or references OpenAI Codex for code analysis, refactoring, or automated editing
license: MIT
compatibility: Requires Codex CLI installed
metadata:
  author: ethanolivertroy
  version: "1.0.0"
allowed-tools: Bash(codex:*) AskUserQuestion
---

# Codex Skill Guide

This skill enables the use of Codex CLI for code analysis, refactoring, and automated editing tasks.

## When to Use This Skill

Use this skill when the user:
- Asks to run Codex CLI commands (`codex exec`, `codex resume`)
- References OpenAI Codex for code analysis
- Needs automated code refactoring
- Wants AI-powered code editing

## Running a Task

1. Ask the user (via `AskUserQuestion`) which model to run (`gpt-5.2-codex` or `gpt-5.2`) AND which reasoning effort to use (`xhigh`, `high`, `medium`, or `low`) in a **single prompt with two questions**.
2. Select the sandbox mode required for the task; default to `--sandbox read-only` unless edits or network access are necessary.
3. Assemble the command with the appropriate options:
   - `-m, --model <MODEL>`
   - `--config model_reasoning_effort="<xhigh|high|medium|low>"`
   - `--sandbox <read-only|workspace-write|danger-full-access>`
   - `--full-auto`
   - `-C, --cd <DIR>`
   - `--skip-git-repo-check`
4. Always use --skip-git-repo-check.
5. When continuing a previous session:
   - Use `codex exec --skip-git-repo-check resume --last` via stdin
   - Don't use any configuration flags unless explicitly requested by the user (e.g., if they specify the model or reasoning effort when requesting to resume)
   - Resume syntax: `echo "your prompt here" | codex exec --skip-git-repo-check resume --last 2>/dev/null`
   - All flags must be inserted between `exec` and `resume`
6. **IMPORTANT**: By default, append `2>/dev/null` to all `codex exec` commands to suppress thinking tokens (stderr). Only show stderr if the user explicitly requests to see thinking tokens or if debugging is needed.
7. Run the command, capture stdout/stderr (filtered as appropriate), and summarize the outcome for the user.
8. **After Codex completes**, inform the user: "You can resume this Codex session at any time by saying 'codex resume' or asking me to continue with additional analysis or changes."

### Quick Reference

| Use case | Sandbox mode | Key flags |
| --- | --- | --- |
| Read-only review or analysis | `read-only` | `--sandbox read-only 2>/dev/null` |
| Apply local edits | `workspace-write` | `--sandbox workspace-write --full-auto 2>/dev/null` |
| Permit network or broad access | `danger-full-access` | `--sandbox danger-full-access --full-auto 2>/dev/null` |
| Resume recent session | Inherited from original | `echo "prompt" \\| codex exec --skip-git-repo-check resume --last 2>/dev/null` |
| Run from another directory | Match task needs | `-C <DIR>` plus other flags `2>/dev/null` |

## Following Up

- After every `codex` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to resume with `codex exec resume --last`.
- When resuming, pipe the new prompt via stdin: `echo "new prompt" | codex exec resume --last 2>/dev/null`. The resumed session automatically uses the same model, reasoning effort, and sandbox mode from the original session.
- Restate the chosen model, reasoning effort, and sandbox mode when proposing follow-up actions.

## Error Handling

- Stop and report failures whenever `codex --version` or a `codex exec` command exits non-zero; request direction before retrying.
- Before you use high-impact flags (`--full-auto`, `--sandbox danger-full-access`, `--skip-git-repo-check`) ask the user for permission using AskUserQuestion unless it was already given.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.

## Examples

### Read-only Analysis
```bash
codex exec --skip-git-repo-check \
  -m gpt-5.2-codex \
  --config model_reasoning_effort="high" \
  --sandbox read-only \
  "Analyze the code structure and identify potential improvements" \
  2>/dev/null
```

### Apply Edits
```bash
codex exec --skip-git-repo-check \
  -m gpt-5.2 \
  --config model_reasoning_effort="medium" \
  --sandbox workspace-write \
  --full-auto \
  "Refactor the authentication module to use async/await" \
  2>/dev/null
```

### Resume Session
```bash
echo "Now add error handling to the refactored code" | \
  codex exec --skip-git-repo-check resume --last 2>/dev/null
```

## Notes

- Thinking tokens are suppressed by default using `2>/dev/null`
- Always use `--skip-git-repo-check` to avoid repository validation issues
- The `--full-auto` flag allows Codex to make changes without confirmation
- Resume sessions inherit settings from the original session
