---
name: repoint-branch
description: "Extract independent changes from a feature branch into a new PR targeting main."
allowed-tools:
  - Read
  - Bash
  - AskUserQuestion
---

## Context
- Current branch: !`git branch --show-current 2>/dev/null`
- Changed files (vs main): !`git diff --name-only origin/main...HEAD 2>/dev/null | head -50`

# Repoint Branch

Extract independent changes from a compound branch into a new branch targeting main.

## Usage

```bash
# Interactive - shows changes, asks what to extract
/repoint-branch

# Extract specific files/directories to auto-named branch
/repoint-branch .claude/guidelines/

# Extract multiple paths
/repoint-branch .claude/guidelines/ README.md src/config.py

# Glob pattern
/repoint-branch ".claude/**/*.md"

# Specify new branch name with --name
/repoint-branch .claude/guidelines/ --name feature/guidelines-lite

# Full example
/repoint-branch .claude/guidelines/ .claude/commands/ --name feature/claude-config-lite
```

## Reference Files (conditional — read only when needed)

- @~/.claude/skill-references/platform-detection.md - Platform detection for GitHub/GitLab

## Instructions

0. **Detect platform** — follow `@~/.claude/skill-references/platform-detection.md` to determine GitHub vs GitLab. Set `CLI`, `REVIEW_UNIT`, and API command patterns accordingly. All commands below use GitHub (`gh`) syntax; substitute GitLab equivalents if on GitLab.

1. **Parse arguments**:
   - Extract `--name <branch-name>` if provided
   - Remaining args are file paths, directories, or glob patterns
   - If no paths provided, will prompt interactively

2. **Get current context**:
   ```bash
   git branch --show-current  # Current branch
   git fetch origin main
   git diff --name-only origin/main...HEAD  # All changed files
   ```

3. **Determine files to extract** (store as `FILES_TO_EXTRACT`):
   - If paths provided:
     - Expand globs: `git diff --name-only origin/main...HEAD -- <patterns>`
     - Filter to only files that actually changed
   - If no paths:
     - Show all changed files
     - Ask user: "Which files/directories should be extracted to the new branch?"
   - Store the resulting file list as `FILES_TO_EXTRACT` for use in later steps

4. **Validate independence**:
   - Show the files to be extracted
   - Ask: "These files will be extracted. Confirm they don't depend on other changes in this branch? (y/n)"

5. **Determine new branch name**:
   - If `--name` provided, use it
   - Otherwise, suggest based on current branch: `<current-branch>-lite`
   - Ask user to confirm or provide alternative

6. **Create new branch from main**:
   ```bash
   git checkout origin/main
   git checkout -b <new-branch-name>
   ```

7. **Apply the changes**:
   For each file in `FILES_TO_EXTRACT`:
   ```bash
   # Ensure parent directories exist
   mkdir -p $(dirname <filepath>)
   # Get the file content from the original branch
   git show <original-branch>:<filepath> > <filepath>
   ```
   Stage and commit all extracted files:
   ```bash
   git add <FILES_TO_EXTRACT>
   git commit -m "<descriptive message>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
   ```

8. **Push and offer to create PR**:
   ```bash
   git push -u origin <new-branch-name>
   ```
   Ask: "Create a PR to main? (y/n)"
   If yes, run `/pr` skill or:
   ```bash
   gh pr create --base main --title "<title>" --body "..."
   ```

9. **Return to original branch**:
   ```bash
   git checkout <original-branch>
   ```
   Inform user: "Created `<new-branch-name>` with extracted changes. Your original branch is unchanged."

## Example Session

```
$ /repoint-branch .claude/guidelines/ --name feature/guidelines-lite

Current branch: feature/phase2 (based on feature/phase1)

Files to extract:
  .claude/guidelines/git-workflow.md
  .claude/guidelines/python-practices.md

These will be copied to new branch 'feature/guidelines-lite' targeting main.
Confirm these don't depend on feature/phase1 changes? (y/n) y

Creating branch from main...
Applying changes...
Committed: "Add git workflow and python practice guidelines"
Pushed to origin/feature/guidelines-lite

Create PR to main? (y/n) y
PR created: https://github.com/user/repo/pull/42

Returned to feature/phase2.
```

## When to Use This vs `/git:split-pr`

- **`/git:repoint-branch`** — You already know which files to extract into a separate PR. Provide paths and this skill handles branching, copying, committing, and PR creation.
- **`/git:split-pr`** — You have a large PR and need help analyzing *how* to split it. That skill proposes a strategy; use this one to execute each piece.

## Important Notes

- Original branch remains unchanged - changes are copied, not moved
- Only extracts file contents, not commit history
- Verify extracted files don't import/depend on code from parent branches
- If changes depend on parent branch code, this will create broken code on main
