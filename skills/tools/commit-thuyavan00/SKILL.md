---
name: commit
description: Create a git commit with a well-crafted message following project conventions
disable-model-invocation: true
allowed-tools: Bash(git *)
---

# Git Commit Skill

Create a git commit following best practices for this repository.

## Process

1. **Analyze the current state** - Run these in parallel:
   - `git status` - See all untracked and modified files
   - `git diff` - See both staged and unstaged changes
   - `git log --oneline -10` - Review recent commit message style

2. **Draft commit message**:
   - Summarize the nature of changes (new feature, enhancement, bug fix, refactor, docs, etc.)
   - Be concise (1-2 sentences) focusing on "why" rather than "what"
   - Use imperative mood (e.g., "add", "fix", "update", not "added", "fixed")
   - Follow the style of recent commits in this repo
   - Match scope to actual changes:
     - "add" = wholly new feature/file
     - "update" = enhancement to existing feature
     - "fix" = bug fix
     - "refactor" = code restructuring
     - "feat" = new feature (if using conventional commits)
     - "chore" = maintenance tasks

3. **Stage and commit** - Run these sequentially:
   - Stage relevant files by name (prefer specific files over `git add -A`)
   - Create commit with message ending in:
     ```
     Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
     ```
   - Run `git status` after commit to verify success

4. **Format commit messages** using HEREDOC:
   ```bash
   git commit -m "$(cat <<'EOF'
   Your commit message here.

   Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
   EOF
   )"
   ```

## Safety Rules

- **NEVER** update git config
- **NEVER** run destructive commands without user confirmation:
  - `git push --force`
  - `git reset --hard`
  - `git checkout .`
  - `git clean -f`
  - `git branch -D`
- **NEVER** skip hooks (`--no-verify`, `--no-gpg-sign`) unless explicitly requested
- **NEVER** amend commits unless explicitly requested
- **CRITICAL**: If pre-commit hook fails, fix the issue and create a NEW commit (don't use `--amend`)
- Prefer staging specific files by name rather than `git add -A` or `git add .`
- Don't commit if there are no changes
- Don't commit sensitive files (.env, credentials, etc.)

## Arguments

- No arguments: Commit all current changes
- With file paths: Commit only specified files

## Example Usage

```
/commit
/commit src/components/Button.tsx
```
