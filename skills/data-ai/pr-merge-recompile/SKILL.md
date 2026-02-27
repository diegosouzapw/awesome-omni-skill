---
name: pr-merge-recompile
description: "Checkout a PR, merge origin/main, regenerate agentic workflows, and push. Use when: (1) A PR needs to be rebased and workflows recompiled, (2) User provides a PR URL and wants to update it with latest main, (3) User says 'recompile PR' or 'update PR workflows'."
---

# PR Merge and Recompile Workflows

Checks out a PR branch, merges origin/main, regenerates all agentic workflows, and pushes the changes.

## Instructions

When the user invokes this skill with a PR URL (e.g., `https://github.com/githubnext/gh-aw/pull/12062`), perform these steps:

### 1. Checkout the PR

```bash
gh pr checkout <PR_NUMBER> --repo <OWNER/REPO>
```

Extract the PR number and repo from the URL provided.

### 2. Fetch and merge origin/main

```bash
git fetch origin main
git merge origin/main
```

If there are merge conflicts, stop and inform the user to resolve them manually.

### 3. Regenerate agentic workflows

```bash
make recompile
```

This regenerates all `.lock.yml` files from the workflow markdown sources.

### 4. Commit and push

If there are changes after recompile:

```bash
git add -A
git commit -m "Merge main and regenerate agentic workflows

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

### 5. Report completion

Provide the user with:
- Confirmation that the PR was updated
- Link to the PR
- Summary of any workflow changes

## Usage Examples

```
/pr-merge-recompile https://github.com/githubnext/gh-aw/pull/12062
```

## Notes

- Requires `gh` CLI to be authenticated
- Requires the repository to have `make recompile` target
- Will abort if merge conflicts occur
