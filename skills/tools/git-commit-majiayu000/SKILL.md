---
name: git-commit
description: Create conventional commit messages. Use when committing changes or writing commit messages.
---

## Conventional Commits Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code restructuring, no behavior change |
| `perf` | Performance improvement |
| `test` | Adding/updating tests |
| `build` | Build system or dependencies |
| `ci` | CI/CD configuration |
| `chore` | Maintenance tasks |
| `revert` | Reverting previous commit |

## Rules

### Subject Line
- Max 50 characters
- Imperative mood ("add" not "added")
- No period at end
- Lowercase

### Body (optional)
- Wrap at 72 characters
- Explain what and why, not how
- Separate from subject with blank line

### Footer (optional)
- Reference issues: `Closes #123`, `Fixes #456`
- Breaking changes: `BREAKING CHANGE: description`

## Examples

### Simple
```
feat(auth): add OAuth2 login support
```

### With Body
```
fix(api): handle null response from payment gateway

The payment gateway occasionally returns null instead of an error
object when the service is degraded. This caused unhandled exceptions.

Added null check and appropriate error handling.

Fixes #789
```

### Breaking Change
```
refactor(api)!: change user endpoint response format

BREAKING CHANGE: The /api/users endpoint now returns an object
with a `data` wrapper instead of a raw array.

Before: [{ id: 1, name: "John" }]
After: { data: [{ id: 1, name: "John" }], meta: { total: 1 } }
```

## Commands
```bash
# Stage and commit
git add .
git commit -m "feat(scope): description"

# Amend last commit
git commit --amend

# Interactive rebase to fix commits
git rebase -i HEAD~3
```
