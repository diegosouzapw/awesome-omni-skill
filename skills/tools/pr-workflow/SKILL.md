---
name: pr-workflow
description: Create branches, commits, and pull requests following project conventions. Use when the user wants to commit changes, create a PR, push code, or mentions git workflow.
allowed-tools: Bash, Read, Glob
---

# Pull Request Workflow

Complete workflow for creating branches, commits, and PRs in ZChain.

## Branch Creation

```bash
# Ensure on latest main
git checkout main
git pull origin main

# Create feature branch
git checkout -b <type>/<description>
```

### Branch Types

| Prefix | Use Case |
|--------|----------|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `chore/` | Dependencies, config, maintenance |
| `refactor/` | Code restructuring |

## Making Changes

### Before Committing

```bash
# Build
dotnet build src/ZChain.sln

# Run tests
dotnet test src/ZChain.sln

# Check what changed
git status
git diff
```

### Commit Message Format

```
Short summary in imperative mood (50 chars max)

Optional body explaining the "why" (wrap at 72 chars).

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Good Commit Messages

```bash
# Feature
git commit -m "Add multi-threaded mining support

Implement parallel nonce search using Task.WhenAny.
First thread to find valid hash wins and cancels others.

Co-Authored-By: Claude <noreply@anthropic.com>"

# Fix
git commit -m "Fix block state validation in verification

Co-Authored-By: Claude <noreply@anthropic.com>"

# Chore
git commit -m "Update NuGet packages to latest versions

Co-Authored-By: Claude <noreply@anthropic.com>"
```

## Creating Pull Request

### Push Branch

```bash
git push -u origin <branch-name>
```

### Create PR with GitHub CLI

```bash
gh pr create --title "Short description" --body "$(cat <<'EOF'
## Summary
- Change 1
- Change 2

## Test plan
- [x] Unit tests pass
- [x] Integration tests pass
- [ ] Manual testing (if applicable)

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### PR Title Guidelines

- Use imperative mood: "Add feature" not "Added feature"
- Be specific: "Add multi-threaded mining" not "Update miner"
- Match primary commit message

## After PR Created

### Address Review Comments

CodeRabbit will automatically review. Address any:
- Security concerns (Critical/High)
- Regression risks
- Code quality suggestions

### Merge Strategy

Use squash merge to keep main history clean:
```bash
gh pr merge <number> --squash
```

## Cleanup

After merge:
```bash
git checkout main
git pull
git branch -d <branch-name>  # Delete local branch
```

## Quick Reference

```bash
# Full workflow
git checkout main && git pull
git checkout -b feature/my-feature

# ... make changes ...

dotnet build src/ZChain.sln
dotnet test src/ZChain.sln
git add .
git commit -m "Add my feature

Co-Authored-By: Claude <noreply@anthropic.com>"
git push -u origin feature/my-feature
gh pr create --title "Add my feature" --body "## Summary
- Added feature

## Test plan
- [x] Tests pass"
```

## Repository Config

User configuration for this repo:
```bash
git config user.name "z3d"
git config user.email "925699+z3d@users.noreply.github.com"
```
