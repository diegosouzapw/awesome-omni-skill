---
name: Git Commit Helper
description: Creates well-formatted conventional commits with intelligent change analysis. Use when creating commits, committing changes, staging files, or when the user mentions "commit", "git commit", or wants to save their work to version control. Analyzes diffs to suggest splitting commits when multiple concerns are detected.
---

# Git Commit Helper

Helps create atomic, well-formatted commits following conventional commit standards. Automatically analyzes changes to determine if they should be split into multiple commits.

## Core Responsibilities

1. **Parallel git operations** - Run `git status` and `git diff` concurrently for speed
2. **Smart staging detection** - Check if files are already staged, otherwise auto-stage
3. **Intelligent diff analysis** - Determine if changes should be split into multiple commits
4. **Conventional commit messages** - Generate properly formatted commit messages
5. **Commit attribution** - Add Claude Code attribution to commit messages

## When to Use This Skill

**Use this skill when**:
- User explicitly requests commit creation
- Creating commits with proper conventional format
- Analyzing changes to determine if they should be split
- Need to generate meaningful commit messages automatically
- Working with staging area and need guidance

**Use direct git commands when**:
- User wants to run specific git commands directly
- Debugging git issues
- Complex git operations beyond basic commits

## Workflow

### 1. Gather Information (Parallel)

```bash
# Run concurrently for speed
git status & git diff --staged & git diff & git diff --stat
```

### 2. Analyze and Decide

**Single commit if**:
- All changes relate to one purpose
- Changes are tightly coupled
- Splitting would create incomplete features
- Total diff is reasonably sized

**Split commits if**:
- Different functionality (features vs bug fixes vs refactoring)
- Different subsystems (auth vs API vs UI)
- Different file types (code vs docs vs config)
- Different purposes (implementation vs tests vs dependencies)

### 3. Stage Strategically

```bash
# Stage all (excluding secrets)
git add .

# Stage specific files
git add path/to/file1.js path/to/file2.js

# Stage by pattern
git add src/auth/*

# Stage interactively
git add -p file.js
```

### 4. Create Commit

Always use HEREDOC format:

```bash
git commit -m "$(cat <<'EOF'
<type>[optional scope]: <description>

[optional body explaining why, not what]

[optional footer: Fixes #123]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 5. Verify

```bash
git status    # Verify commit created
git log -1    # Review last commit
```

## Conventional Commit Types

- `feat` - A new feature
- `fix` - A bug fix
- `docs` - Documentation only changes
- `style` - Code style changes (formatting, semicolons, etc.)
- `refactor` - Code change that neither fixes a bug nor adds a feature
- `perf` - Performance improvement
- `test` - Adding or correcting tests
- `build` - Build system or external dependencies
- `ci` - CI configuration files and scripts
- `chore` - Other changes that don't modify src or test files
- `revert` - Reverts a previous commit

**With scope**: `feat(auth): add password reset`
**Breaking change**: `feat!: remove deprecated API` (note the `!`)

## Commit Message Rules

**Description line**:
- Use imperative mood: "add" not "added" or "adds"
- Don't capitalize first letter
- No period at the end
- Keep under 72 characters
- Be specific: ✓ `fix: resolve race condition in data sync` ✗ `fix: bug fix`

**Body** (optional):
- Wrap at 72 characters
- Explain "why" not "what"
- Include motivation and context

**Footer** (optional):
- Reference issues: `Fixes #123`, `Closes #456`, `See also: #789`
- Breaking changes: `BREAKING CHANGE: removed /v1/auth endpoint`

## Security: Never Commit Secrets

**Always exclude**:
- `.env*`, `credentials.json`, `secrets.json`
- `*.pem`, `*.key`, `*.p12`, `*.pfx`
- API tokens, passwords, private keys

**If secrets detected**: Warn user, suggest `.gitignore` or environment variables, only proceed if user explicitly confirms

## Pre-commit Hook Handling

**Standard flow**:
1. Attempt commit
2. If hooks fail, read error message
3. Fix issues hooks identified
4. Retry commit

**If hooks modify files**:
```bash
# Check if safe to amend (should show Claude as author, not pushed)
git log -1 --format='%an %ae'
git status

# If safe, amend
git add .
git commit --amend --no-edit

# Otherwise, create new commit
git commit -m "$(cat <<'EOF'
chore: apply pre-commit hook fixes

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Bypass hooks**: Only when user explicitly requests `--no-verify`

## Common Patterns

### Simple Commit
```bash
git status & git diff
git add .
git commit -m "$(cat <<'EOF'
feat: add user dashboard with activity widgets

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
git status
```

### Commit Staged Files Only
```bash
git diff --staged
git commit -m "$(cat <<'EOF'
fix: resolve email validation for plus addressing

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Split Commits
```bash
# First commit
git add src/auth/*
git commit -m "$(cat <<'EOF'
feat: implement OAuth2 authentication flow

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Second commit
git add tests/auth/*
git commit -m "$(cat <<'EOF'
test: add OAuth2 flow integration tests

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### With Detailed Body
```bash
git commit -m "$(cat <<'EOF'
refactor: extract API client configuration to separate module

Moved API client setup logic from individual service files to
centralized config module to reduce duplication and simplify updates.

Fixes #123

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## Quick Troubleshooting

- **Hooks rejected commit**: Read error, fix issues, retry
- **Wrong message**: `git commit --amend -m "new message"` (if not pushed)
- **Forgot files**: `git add file && git commit --amend --no-edit` (if not pushed)
- **Need to split**: `git reset HEAD~1`, then stage and commit separately (if not pushed)
- **Committed secrets**: `git reset HEAD~1`, restage without secrets (if not pushed; if pushed, use git-filter-repo)

## Best Practices

1. Each commit = one logical change (atomic commits)
2. Test before committing
3. Review diffs before staging
4. Write clear, descriptive messages
5. Commit frequently (small commits > large commits)
6. Don't mix unrelated changes
7. Use conventional format for parseable history
8. Reference issue tracker when relevant

## Performance Tips

- Run git commands in parallel when independent
- Use `git diff --stat` for quick overview before full diff
- Use HEREDOC directly (no temp files)
- Cache staging status to avoid repeated checks

## Integration

- **Git hooks**: Compatible with pre-commit, commit-msg, prepare-commit-msg
- **Changelog generators**: conventional-changelog, semantic-release, standard-version
- **Issue trackers**: Reference issues in footers (`Fixes #123`)

## Quick Reference

```bash
# Analyze (parallel)
git status & git diff --staged & git diff & wait

# Stage
git add .                           # All files
git add path/to/file1 path/to/file2 # Specific files
git add src/**/*.ts                 # Pattern

# Commit
git commit -m "$(cat <<'EOF'
<type>: <description>

[optional body]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Verify
git status && git log -1

# Fix (if not pushed)
git commit --amend --no-edit  # Amend
git commit --amend -m "msg"   # Change message
git reset HEAD~1              # Undo commit
```

## Resources

- Conventional Commits: https://www.conventionalcommits.org/
- Git Documentation: https://git-scm.com/doc
- Git Best Practices: https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project
