---
name: github-workflow
description: Work with GitHub issues, PRs, and releases. Use when managing tasks, creating issues, or preparing PRs.
---

# GitHub Workflow Skill

## When to Use

- Creating/managing GitHub issues
- Preparing PRs with proper format
- Managing milestones and labels

## Issue Management

### Create Issue

```bash
gh issue create --title "[Area] Description" \
  --body "$(cat <<'EOF'
## Task
What needs to be done

## Acceptance Criteria
- [ ] Criteria 1
- [ ] Criteria 2

## Files
`src/core/file.py`
EOF
)" --label "enhancement"
```

### Edit Issue

```bash
# Add labels
gh issue edit 123 --add-label "bug,P0"

# Add to milestone
gh issue edit 123 --milestone "v1.0"

# Close
gh issue close 123
```

### List Issues

```bash
# By label
gh issue list --label "bug"

# By milestone
gh issue list --milestone "v1.0"

# JSON for scripting
gh issue list --json number,title,labels
```

## Pull Requests

### Create PR

```bash
gh pr create \
  --title "feat(core): add feature description" \
  --body "$(cat <<'EOF'
## Summary
- What changed

## Test Plan
- [ ] pytest passes
- [ ] pyright passes
- [ ] ruff passes

## Related Issues
Closes #XX

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Check PR Status

```bash
gh pr status
gh pr view 123
gh pr checks 123
```

### View PR Comments

```bash
gh api repos/:owner/:repo/pulls/123/comments
```

### Merge PR

```bash
gh pr merge 123 --squash --delete-branch
```

## Git Workflow for PRs

### Standard Flow

```bash
# 1. Checkout main and pull
git checkout main && git pull origin main

# 2. Create feature branch
git checkout -b feature/description

# 3. Make changes and commit
git add . && git commit -m "feat(scope): description"

# 4. Push and create PR
git push -u origin feature/description
gh pr create --title "..." --body "..."
```

### Branch Naming

```
feature/{description}
fix/{issue-number}-{description}
docs/{description}
refactor/{description}
```

### Commit Message Format

```
{type}({scope}): {description}

[optional body]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## Pre-PR Validation

```bash
source venv/bin/activate

# All must pass
ruff format src tests
ruff check src tests --fix
pyright src tests
pytest

# Review changes
git diff --stat
```

## Labels

### Create Labels

```bash
# Priority
gh label create "P0" --description "Critical" --color "E11D48"
gh label create "P1" --description "Important" --color "F97316"
gh label create "P2" --description "Enhancement" --color "3B82F6"

# Type
gh label create "bug" --description "Something broken" --color "D73A4A"
gh label create "enhancement" --description "New feature" --color "A2EEEF"
gh label create "docs" --description "Documentation" --color "0075CA"
```

## Quick Reference

```bash
# Issues
gh issue create --title "..." --body "..." --label "..."
gh issue edit 123 --add-label "bug"
gh issue list --label "bug"

# PRs
gh pr create --title "..." --body "..."
gh pr view 123
gh pr merge 123 --squash
gh api repos/:owner/:repo/pulls/123/comments
```
