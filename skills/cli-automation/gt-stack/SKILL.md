---
name: gt-stack
description: Analyze changes and organize them into a Graphite stack of logically isolated PRs
disable-model-invocation: true
argument-hint: "[scope] (optional — 'staged', 'all', or 'commits:N')"
---

Analyze the current changes and organize them into a Graphite stack of logically isolated branches, each becoming its own PR.

## Steps

### Phase 1: Assess

1. Run `gt ls` to see the current stack state
2. Run `git status` to see staged, unstaged, and untracked files
3. Run `git diff --stat` and `git diff --staged --stat` for an overview
4. Run `git diff` and `git diff --staged` to read the actual diffs
5. If $ARGUMENTS contains `commits:N`, also run `git log --oneline -N` and `git diff HEAD~N..HEAD`
6. Run `git log --oneline -5` to understand the commit style

### Phase 2: Propose a split plan

7. Group changes into logically isolated units. Each unit should:
   - Touch a single concern (one feature, one fix, one refactor)
   - Be independently reviewable
   - Have a clear conventional commit message
8. Present the plan (bottom → top):
   ```
   Proposed stack (bottom → top):
   1. feat(scope): description — files: path/to/file.lua
   2. fix(scope): description — files: path/to/other.zsh
   ```
9. Ask the user to confirm, modify, or reorder before proceeding

### Phase 3: Execute the stack

10. For each entry (bottom to top):
    - Stage the specific files with `git add <file>`
    - Run `gt create <branch-name> -m "type(scope): description"`
    - Verify with `gt ls`
11. Show the final stack with `gt ls`

### Phase 4: Optional submit

12. Ask if the user wants to submit the stack as PRs
13. If yes: `gt submit --stack` (or `gt submit --stack --draft` for drafts)
14. Output the stack summary

## Handling commits:N

When reorganizing existing commits:
- Ensure working tree is clean (stash if needed)
- Use `gt branch split --by-commit` to decompose
- Always show a dry-run plan and get explicit confirmation

## Rules

- NEVER run `gt submit` without explicit user confirmation
- NEVER use `git add .` or `git add -A` — always add specific files
- ALWAYS present the split plan and wait for confirmation before creating branches
- ALWAYS use conventional commits: feat, fix, chore, docs, refactor, test
- ALWAYS end commit messages with: `Co-Authored-By: Claude <noreply@anthropic.com>`
- If the working tree is dirty and there is an existing stack, warn before modifying
- If `gt` is not installed, suggest: `brew install withgraphite/tap/graphite`
- Prefer smaller, focused branches — that's the whole point of stacking
- When unsure how to split, ask the user
