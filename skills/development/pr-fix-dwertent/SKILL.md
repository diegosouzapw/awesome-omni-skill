---
name: pr-fix
description: Address PR review comments by applying fixes locally. Fetches review comments via GitHub API, evaluates each one, applies fixes where appropriate, and generates a report. Use when the user asks to fix PR comments, address PR feedback, or handle PR review suggestions.
---

# PR Fix

## Prerequisites

- `gh` CLI authenticated and available
- The repo must be cloned locally
- Working tree should be clean (no uncommitted changes) before starting

## Workflow

### Step 1: Identify the PR

If the user provides a PR number, use it. Otherwise, detect the current branch and find the open PR:

```bash
gh pr list --head "$(git branch --show-current)" --json number,title,url --jq '.[0]'
```

### Step 2: Checkout the PR branch

Ensure you are on the correct branch. If not already on it, check it out:

```bash
gh pr checkout <NUMBER>
```

### Step 3: Fetch all review comments

Fetch both inline review comments and general PR conversation comments:

```bash
# Inline code review comments (attached to specific lines)
gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate

# General PR-level comments (conversation thread)
gh api repos/{owner}/{repo}/issues/{number}/comments --paginate

# PR review summaries (review bodies submitted with approve/request changes)
gh api repos/{owner}/{repo}/pulls/{number}/reviews --paginate
```

Filter out comments prefixed with `[cursor-auto-review]` — those are machine-generated and should be ignored.

### Step 4: Evaluate each comment

For every review comment, decide how to handle it:

1. **Fix it** — the issue is clear and the fix is straightforward. Apply the fix locally. Choose the best solution, not necessarily what the reviewer literally suggested. If you see a better approach, use it.
2. **Skip it** — the suggestion doesn't make sense, is wrong, or would make the code worse. Note why in the report.
3. **Flag it** — the issue is unclear, the right fix is ambiguous, or the comment is too vague to act on. Do NOT attempt a fix. Flag it for the user's attention.

Guidelines:
- Not all suggestions need to be implemented. Reviewers can be wrong or vague.
- When a reviewer raises a valid point but proposes a bad solution, fix the underlying issue your way.
- When the issue itself is unclear, flag it — don't guess.
- Do NOT add code comments that don't need to be there. Every code change should look like a developer wrote it, not a bot.

### Step 5: Apply fixes locally

Make changes directly to the local files. Do NOT:
- `git commit`
- `git push`
- Create new branches
- Run any git operations that modify history

The user will review and commit when ready.

### Step 6: Generate the report

Write a report to `pr-fix-report.md` in the repo root. Format:

```markdown
# PR Fix Report — PR #<NUMBER>

## Needs Attention

These comments require your review — either the issue was unclear, I chose a different approach than suggested, or I decided not to fix.

| # | File | Comment | Action | Why |
|---|------|---------|--------|-----|
| 1 | `path/to/file.go:42` | Reviewer said X | **Flagged** — unclear what the intended behavior should be | |
| 2 | `path/to/file.go:88` | Reviewer suggested Y | **Fixed differently** — used Z instead because ... | |
| 3 | `path/to/other.go:15` | Reviewer asked for W | **Skipped** — suggestion would break ... | |

## Fixed

| # | File | Comment | What I did |
|---|------|---------|------------|
| 4 | `path/to/file.go:20` | Missing nil check | Added nil check before dereference |
| 5 | `path/to/util.go:55` | Unused error return | Now returns and handles the error |

## Skipped (no action needed)

| # | File | Comment | Why |
|---|------|---------|-----|
| 6 | `path/to/file.go:10` | Style preference | Not a functional issue |
```

Rules for the report:
- **"Needs Attention" section comes first** — these are the items the user must look at
- One line per comment. Keep descriptions to one sentence.
- Include the file path and line number from the review comment
- Include a brief quote or paraphrase of the reviewer's comment
- For "Fixed differently" items, briefly explain what you did instead and why
