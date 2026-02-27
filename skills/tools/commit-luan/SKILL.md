---
name: commit
description: "Use when the user wants to commit, save changes, create a conventional commit, amend a commit, make a fixup commit, or squash commits."
user-invocable: true
context: fork
agent: general-purpose
model: haiku
allowed-tools:
  - "Bash(git status)"
  - "Bash(git diff:*)"
  - "Bash(git log:*)"
  - "Bash(git add:*)"
  - "Bash(git commit:*)"
  - "Bash(git notes:*)"
  - "Bash(git branch:*)"
  - "Bash(git rev-parse:*)"
  - Read
  - Glob
  - Grep
  - AskUserQuestion
  - TaskList
  - TaskGet
  - "Bash(ct plan archive:*)"
  - "Bash(ct plan list:*)"
---

# Commit

Create conventional commits explaining WHY changes were made.

## Context

Status: !`git status -sb 2>/dev/null`
Staged diff: !`git diff --cached --stat 2>/dev/null`
Recent commits: !`git log --oneline -5 2>/dev/null`

**Main thread / foreground only.** Workers never commit — they lack full-branch context to write meaningful messages.

## Steps

1. **Analyze**: review context above. If nothing staged, read full `git diff`. If staged, read `git diff --cached` for details.

2. **Message**: conventional commit format — `type(scope): description`, max 72 chars, lowercase, no period, imperative mood. Types: feat|fix|perf|docs|test|style|build|ci|chore|revert. Scope: primary area or omit if global. Multi-line: blank line then body wrapping at 72 chars explaining motivation not mechanics. If task active (TaskList, filter by project + status=in_progress), append ID: `fix(auth): handle token expiry (task-<id>)`

3. **Execute** using HEREDOC for clean formatting:
   ```bash
   git commit -m "$(cat <<'EOF'
   type(scope): description
   EOF
   )"
   ```

## Post-commit

4. **Plan archive:** after successful commit, archive active plans.
   `ct plan list --json` returns array of `{name, path}` — iterate and archive each:
   ```
   ct plan list --json
   # For each entry: ct plan archive <path>
   ```
   Skip silently if no active plans or ct not available.

If gt plugin is loaded → suggest `/gt:submit`. Otherwise → suggest `git push`.

## Hook Failures

Two scenarios require different recovery:

- **Hooks modify files** (formatters, auto-fixers): stage changes and amend — `git add -u && git commit --amend --no-edit`. Safe because the commit already landed; amend just includes the formatter's tweaks.
- **Hooks reject the commit** (lint errors, test failures): show the error, explain the issue, suggest a fix. Do NOT retry automatically — let user decide. After fix, create a NEW commit (don't amend — the original commit never landed, so amend would modify the wrong commit).

## Special Ops

- **Amend** (`--amend`): analyze previous commit + new changes, update message if scope changed. Use `--no-edit` only when new changes don't alter the commit's purpose.
- **Fixup** (`--fixup=<SHA>`): correction targeting a specific earlier commit. User must rebase to squash later.
- **Squash**: combine multiple commits — unify message around primary purpose, not a laundry list.

## Edge Cases

- Nothing staged → ask "Stage all changes?" (all vs tracked vs select)
- Multiple unrelated changes → use `/split-commit` to separate
- Clean tree → "No changes to commit"
