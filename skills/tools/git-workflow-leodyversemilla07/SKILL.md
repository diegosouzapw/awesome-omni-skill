---
name: git-workflow
description: Comprehensive Git workflow assistant for commits, branches, merges, and best practices. Use when working with Git operations, creating commits, managing branches, or resolving conflicts.
---

# Git Workflow Skill

## When to Use This Skill

Use this skill when:

- Creating commits with proper messages
- Managing branches and merges
- Resolving merge conflicts
- Reviewing Git history
- Performing Git operations safely
- Following Git best practices

## Commit Message Guidelines

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature for the user
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code formatting (no logic change)
- **refactor**: Code restructuring (no feature/fix)
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Maintenance tasks
- **ci**: CI/CD configuration changes
- **build**: Build system or dependencies

### Rules

1. Subject line: max 50 characters
2. Use imperative mood: "add" not "added" or "adds"
3. No period at end of subject line
4. Body: wrap at 72 characters
5. Explain WHAT and WHY, not HOW
6. Separate subject from body with blank line

### Examples

```
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh to improve user experience.
Users will no longer be logged out during active sessions.

- Add refresh token endpoint
- Implement token rotation logic
- Add background refresh timer

Closes #234
```

```
fix(api): prevent null pointer exception in user profile

Add null check before accessing user.email property.
This was causing crashes when users had incomplete profiles.

Fixes #456
```

## Commit Size Guidelines

### Ideal Commit Sizes

- **Small**: 10-50 lines (bug fixes, small tweaks)
- **Medium**: 50-300 lines (features, refactors)
- **Large**: 300-500 lines (major features - consider splitting)
- **Too Large**: 500+ lines (definitely split into multiple commits)

### Atomic Commit Principle

Each commit should:

1. Build successfully
2. Pass all tests
3. Represent one logical change
4. Be independently revertable

### Staging Strategy

```bash
# Stage specific files only
git add src/component.js tests/component.test.js

# Interactive staging for partial file commits
git add -p filename.js

# Review before committing
git diff --staged

# Commit with message
git commit -m "feat: add user profile component"
```

## Branch Management

### Branch Naming Conventions

```
feature/short-description    # New features
fix/bug-description         # Bug fixes
hotfix/critical-fix         # Production hotfixes
refactor/what-refactored    # Code refactoring
docs/what-documented        # Documentation
test/what-tested            # Test additions
chore/maintenance-task      # Maintenance
```

### Branch Workflow

```bash
# Create and switch to new branch
git checkout -b feature/user-notifications

# Work and commit regularly
git add src/notifications.js
git commit -m "feat: add notification service"

# Keep branch updated with main
git checkout main
git pull origin main
git checkout feature/user-notifications
git merge main

# Push to remote
git push origin feature/user-notifications

# Delete after merge
git branch -d feature/user-notifications
git push origin --delete feature/user-notifications
```

## Merge Strategies

### Fast-Forward Merge (cleanest history)

```bash
git checkout main
git merge feature/simple-change
```

### Merge Commit (preserves branch history)

```bash
git checkout main
git merge --no-ff feature/important-feature
```

### Rebase (linear history)

```bash
git checkout feature/my-branch
git rebase main
# Resolve conflicts if any
git push --force-with-lease origin feature/my-branch
```

### When to Use Each

- **Fast-forward**: Simple, quick changes
- **Merge commit**: Team features, want to preserve context
- **Rebase**: Personal branches, want clean linear history
- **Never rebase**: Public/shared branches

## Conflict Resolution

### Process

```bash
# 1. Start merge or rebase
git merge feature/branch
# CONFLICT appears

# 2. See conflicted files
git status

# 3. Open conflicted file, look for:
incoming branch content

# 4. Resolve by:
# - Keeping one version
# - Combining both versions
# - Writing new solution

# 5. Stage resolved files
git add resolved-file.js

# 6. Complete merge
git commit  # for merge
# or
git rebase --continue  # for rebase
```

### Conflict Prevention

- Pull/merge main frequently
- Make smaller, focused commits
- Communicate with team about file changes
- Use feature flags for large changes

## History Management

### Viewing History

```bash
# Basic log
git log

# Compact view
git log --oneline --graph --all

# See changes in commits
git log -p

# Search commits
git log --grep="search term"
git log --author="name"

# See what changed in a file
git log --follow filename.js
```

### Amending Commits (before push)

```bash
# Fix last commit message
git commit --amend -m "Better message"

# Add forgotten files to last commit
git add forgotten-file.js
git commit --amend --no-edit

# Change multiple commits
git rebase -i HEAD~3
```

### Undoing Changes

```bash
# Discard working directory changes
git checkout -- filename.js

# Unstage files
git reset HEAD filename.js

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a commit (create new commit)
git revert <commit-hash>
```

## Stashing

### When to Stash

- Switch branches but not ready to commit
- Pull latest changes
- Quick context switch
- Save experimental work

### Stash Commands

```bash
# Save current work
git stash save "WIP: working on feature X"

# List stashes
git stash list

# Apply most recent stash (keep stash)
git stash apply

# Apply and remove stash
git stash pop

# Apply specific stash
git stash apply stash@{2}

# Delete stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

## Remote Operations

### Safe Push/Pull

```bash
# Always pull before push
git pull origin main

# Push with lease (safer than force)
git push --force-with-lease origin feature/branch

# Fetch without merging
git fetch origin

# See remote branches
git branch -r

# Track remote branch
git checkout --track origin/feature/branch
```

## Git Best Practices Checklist

### Before Committing

- [ ] Code builds successfully
- [ ] Tests pass locally
- [ ] Linting passes
- [ ] Only related changes staged
- [ ] Reviewed diff with `git diff --staged`
- [ ] Commit message is clear and descriptive

### Before Pushing

- [ ] Pulled latest changes
- [ ] Resolved any conflicts
- [ ] Tests still pass after merge
- [ ] Commit history is clean

### Before Merging PR

- [ ] All CI checks pass
- [ ] Code reviewed and approved
- [ ] Branch is up to date with target
- [ ] No merge conflicts
- [ ] Tests cover new code

## Common Workflows

### Daily Development Flow

```bash
# Morning: Get latest
git checkout main
git pull origin main

# Start work
git checkout -b feature/new-thing

# Work for 30-60 mins, then commit
git add src/
git commit -m "feat: add initial structure"

# Continue working, commit regularly
# ...commits...

# End of day: Push
git push origin feature/new-thing
```

### Hotfix Flow

```bash
# Create hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# Fix and test
git add fixed-file.js
git commit -m "fix: resolve critical production bug"

# Push and merge ASAP
git push origin hotfix/critical-bug
# Create PR, get quick review, merge
```

### Feature Branch Flow

```bash
# Create feature branch
git checkout -b feature/user-dashboard

# Commit regularly (atomic commits)
# Every 30-90 minutes of work

# Stay updated with main
git fetch origin
git merge origin/main

# When complete, push and create PR
git push origin feature/user-dashboard
```

## Troubleshooting

### "Detached HEAD state"

```bash
# Create branch from current state
git checkout -b recovery-branch

# Or go back to branch
git checkout main
```

### "Your branch has diverged"

```bash
# If you want remote version
git reset --hard origin/main

# If you want local version
git push --force-with-lease origin main

# If you want to merge both
git pull --rebase origin main
```

### "Permission denied (publickey)"

```bash
# Check SSH key
ssh -T git@github.com

# Add SSH key if needed
ssh-keygen -t ed25519 -C "your_email@example.com"
# Add to GitHub settings
```

### Accidentally committed secrets

```bash
# Remove from last commit
git reset HEAD~1
# Edit .gitignore, recommit

# Remove from history (if pushed)
# Use git-filter-repo or BFG Repo-Cleaner
# Then rotate the compromised secrets!
```

## Quick Reference Commands

```bash
# Status & Info
git status                    # Current state
git log --oneline            # Commit history
git diff                     # Unstaged changes
git diff --staged            # Staged changes

# Basic Operations
git add <file>               # Stage changes
git commit -m "message"      # Commit
git push origin <branch>     # Push to remote
git pull origin <branch>     # Pull from remote

# Branching
git branch                   # List branches
git branch <name>            # Create branch
git checkout <branch>        # Switch branch
git checkout -b <branch>     # Create and switch
git branch -d <branch>       # Delete branch

# Merging
git merge <branch>           # Merge branch
git merge --abort            # Cancel merge
git rebase <branch>          # Rebase onto branch
git rebase --abort           # Cancel rebase

# Undoing
git checkout -- <file>       # Discard changes
git reset HEAD <file>        # Unstage file
git reset --soft HEAD~1      # Undo commit, keep changes
git reset --hard HEAD~1      # Undo commit, discard changes
git revert <commit>          # Create reverting commit

# Stashing
git stash                    # Save work
git stash pop                # Restore work
git stash list               # List stashes

# Remote
git remote -v                # Show remotes
git fetch origin             # Get remote changes
git pull origin main         # Fetch and merge
git push origin main         # Push to remote
```

## Integration with CI/CD

### Pre-commit Hooks

Create `.git/hooks/pre-commit`:

```bash
#!/bin/sh
# Run tests before commit
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi
```

### Commit Message Validation

Create `.git/hooks/commit-msg`:

```bash
#!/bin/sh
# Validate commit message format
commit_msg=$(cat "$1")
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
  echo "Invalid commit message format"
  echo "Use: <type>(<scope>): <subject>"
  exit 1
fi
```

## Tips for AI Collaboration

When working with AI agents like Copilot:

1. **Commit frequently**: AI can track changes better
2. **Clear messages**: Help AI understand intent
3. **Branch per feature**: Keep AI context focused
4. **Review AI commits**: Always check `git diff --staged`
5. **Use conventional commits**: AI can generate better messages

## Additional Resources

- Git documentation: https://git-scm.com/doc
- Conventional Commits: https://www.conventionalcommits.org/
- GitHub Flow: https://guides.github.com/introduction/flow/
- Git Flight Rules: https://github.com/k88hudson/git-flight-rules
