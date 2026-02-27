---
name: merge
description: Commit, rebase, and merge the current branch into its base branch (never main).
disable-model-invocation: true
allowed-tools: Read, Bash, Glob, Grep
---

## Safety Rules

**CRITICAL — these rules must NEVER be violated:**

- **NEVER merge into `main` or `master`.** If the resolved base branch is `main` or `master`, stop immediately and inform the user.
- **NEVER force push.** Do not use `--force`, `--force-with-lease`, or `-f` with `git push`.

## Step 1: Verify Branch

```bash
git branch --show-current
```

If the current branch is `main` or `master`, **STOP** and tell the user they must be on a feature branch.

## Step 2: Commit

If there are staged changes, commit them.

1. Run `git log --format="%s" -n 20` to detect the repo's commit convention and match it.
2. If no clear convention exists, default to conventional commits with a scope, e.g. `feat(auth): add login endpoint`.
3. Always include a co-author trailer:
   ```
   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
4. Skip if nothing is staged.

## Step 3: Determine Base Branch

```bash
git config --local --get "branch.$(git branch --show-current).workmux-base" 2>/dev/null
```

If no base branch is configured, **STOP** and tell the user no base branch is set. Do NOT default to main.

**If the base branch is `main` or `master`, STOP** and tell the user: "Refusing to merge into main/master. Set a different base branch or use /pr instead."

## Step 4: Rebase

Rebase onto the local base branch (do NOT fetch from origin first):

```bash
git rebase <base-branch>
```

**IMPORTANT:** Do NOT run `git fetch`. Do NOT rebase onto `origin/<branch>`. Only rebase onto the local branch name.

If conflicts occur:
- BEFORE resolving any conflict, understand what changes were made to each conflicting file in the base branch
- For each conflicting file, run `git log -p -n 3 <base-branch> -- <file>` to see recent changes
- Preserve BOTH the base branch changes AND our branch's changes
- After resolving each conflict, stage the file and continue with `git rebase --continue`
- If a conflict is too complex or unclear, ask for guidance

## Step 5: Merge

Run: `workmux merge --rebase --notification`
