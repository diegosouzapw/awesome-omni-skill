---
name: commit
description: Complete git workflow - initialize repo if needed, stage and commit everything, push if remote exists, then run /improve and /compact. Use for clean checkpoint commits.
---

# Commit Skill

A complete git workflow skill that ensures a clean repository state after every commit.

## What This Skill Does

1. **Initialize git** (if not already a git repo)
2. **Stage everything** (`git add -A`)
3. **Commit** with an auto-generated or user-provided message
4. **Verify clean state** (`git status --porcelain` must be empty)
5. **Push** (only if a remote exists)
6. **Run /improve** (knowledge capture and tool discovery)
7. **Run /compact** (context compression)

## Usage

```
/commit
/commit -m "your commit message"
```

## Workflow Steps

### Step 1: Initialize Git (if needed)

Check if the current directory is inside a git repository:

```bash
git rev-parse --is-inside-work-tree 2>/dev/null
```

If this fails (exit code non-zero), initialize a new repo:

```bash
git init
```

### Step 2: Stage All Changes

Add all files (tracked, untracked, and deletions):

```bash
git add -A
```

### Step 3: Generate Commit Message

If no message was provided via `-m`, generate one by:

1. Running `git diff --cached --stat` to see what's staged
2. Creating a concise summary of the changes

The commit message should:
- Be concise (under 72 characters for the subject line)
- Describe what changed, not how
- Use imperative mood ("Add feature" not "Added feature")

### Step 4: Commit

```bash
git commit -m "$(cat <<'EOF'
Your commit message here

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**CRITICAL**: Never use `--no-verify`. Pre-commit hooks must run.

### Step 5: Verify Clean State

After commit, verify the repository is completely clean:

```bash
git status --porcelain
```

This must return empty output. If anything remains:
- Investigate why files weren't committed
- Stage and commit any remaining files
- Repeat until `git status --porcelain` returns nothing

### Step 6: Push (if remote exists)

Check if a remote exists:

```bash
git remote -v
```

If output is non-empty AND the current branch has an upstream:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null
```

If upstream exists, push:

```bash
git push
```

If no upstream but remote exists, set upstream and push:

```bash
git push -u origin $(git branch --show-current)
```

If no remote exists, skip pushing silently.

### Step 7: Run /improve

Invoke the improve skill to capture learnings:

```
/improve
```

This runs knowledge capture and tool discovery.

### Step 8: Run /compact

Invoke the compact command to compress context:

```
/compact
```

This summarizes the conversation and reduces token usage.

## Error Handling

### Pre-commit Hook Failures

If the commit fails due to pre-commit hooks:
1. **Fix the issues** reported by the hooks
2. **Re-stage** the fixed files
3. **Try committing again** (new commit, not amend)
4. Never use `--no-verify`

### Nothing to Commit

If `git status` shows nothing to commit:
- Report "Nothing to commit - working tree clean"
- Still run /improve and /compact

### Push Failures

If push fails:
- Report the error to the user
- Do not retry automatically
- Continue with /improve and /compact

## Output Format

```
## Commit Summary

### Repository
[Repo path or "Initialized new repo at: path"]

### Changes Committed
[Brief summary of staged changes]

### Commit
[Commit hash and message]

### Push
[Push result or "No remote configured"]

### Post-Commit
Running /improve...
Running /compact...
```
