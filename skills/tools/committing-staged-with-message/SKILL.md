---
name: committing-staged-with-message
description: Generate commit message for staged changes, pause for approval, then commit. Stage files first with `git add`, then run this skill.
compatibility: Designed for Claude Code
metadata:
  model: haiku
  argument-hint: (no arguments needed)
  disable-model-invocation: true
  allowed-tools: Bash, Read, Glob, Grep
---

# Commit staged with Generated Message

## Step 1: Analyze Staged Changes

Run these commands using the Bash tool to gather context:

- `git diff --staged --name-only` - List staged files
- `git diff --staged --stat` - Diff stats summary
- `git log --oneline -5` - Recent commit style
- `git diff --staged` - Review detailed staged changes. **Size guard**: if `--stat` shows >10 files or >500 lines changed, skip the full diff and rely on `--stat` + `--name-only` to generate the message.

## Step 2: Generate Commit Message

Use the Read tool to check `.gitmessage` for commit message format and syntax.

**The commit message body MUST include (concisely — no padding, no redundancy):**

1. **What changed**: bullet points per file or logical group
2. **Symbols added/removed** (when applicable): functions, classes, tests
3. **Diff stats**: lines added/removed (from `--stat` summary line) — MUST be the last line of the body
   - Format: `+ symbol_name`, `- symbol_name`
   - Omit for config/docs/formatting-only changes

Keep the message laser-focused. Do not repeat the subject line in the body.

## Step 3: Pause for Approval

**Please review the commit message.**

- **Approve**: "yes", "y", "commit", "go ahead"
- **Edit**: Provide your preferred message
- **Cancel**: "no", "cancel", "stop"

## Step 4: Commit

Once approved:

- `git commit --gpg-sign -m "[message]"` - Commit staged changes with approved message (GPG signature mandatory)
- `git status` - Verify success
