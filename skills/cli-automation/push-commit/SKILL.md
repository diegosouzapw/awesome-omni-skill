---
name: push-commit
description: Stage all changes, commit with a descriptive message, and push to the remote. Trigger with "push commit", "commit and push", "push changes", "/push-commit".
---

# Push & Commit

Stage all local changes, generate a commit message, and push to the current branch's remote.

## Workflow

### Step 1: Check State

Run these in parallel:

- `git status` — see what's changed (never use `-uall`)
- `git diff --stat` — summary of staged and unstaged changes
- `git log --oneline -5` — recent commit messages for style reference
- `git branch --show-current` — confirm current branch

If there are no changes (no untracked files, no modifications), stop and tell the user there's nothing to commit.

### Step 2: Stage Changes

Run `git add -A` to stage everything.

**Exception:** If you see files that likely contain secrets (`.env`, `credentials.json`, API keys, tokens), do NOT stage them. Warn the user and list the suspicious files. Stage everything else by name.

### Step 3: Generate Commit Message

Analyze the staged diff and write a commit message that:

- Summarizes the nature of the changes (new feature, update, fix, refactor, docs, etc.)
- Focuses on the **why**, not the **what**
- Follows the style of recent commits in the repo
- Is 1-2 sentences max
- Ends with `Co-Authored-By: Claude <noreply@anthropic.com>`

Use a HEREDOC to pass the message:

```bash
git commit -m "$(cat <<'EOF'
Your commit message here.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Step 4: Push

Run `git push origin <current-branch>`.

If the branch has no upstream, use `git push -u origin <current-branch>`.

### Step 5: Confirm

Run `git status` to verify the working tree is clean, and tell the user the commit hash and what was pushed.

## Edge Cases

- **No changes:** Tell the user there's nothing to commit. Don't create an empty commit.
- **Secrets detected:** Warn the user and skip those files. Stage and commit everything else.
- **Push fails (no upstream):** Retry with `-u` flag to set upstream tracking.
- **Push fails (rejected):** Tell the user — do NOT force push. Suggest `git pull --rebase` first.
- **Pre-commit hook fails:** Fix the issue, re-stage, and create a NEW commit (never amend).
