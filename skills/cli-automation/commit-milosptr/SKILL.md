---
name: commit
description: Create a git commit with a well-formatted message. Use after completing changes that should be committed.
argument-hint: [optional message hint]
disable-model-invocation: true
---

# Git Commit

Create a git commit for the current changes.

## Commit Message Format

Use conventional commit style:
```
<type>: <short description>

<optional body with more details>

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring
- `style`: Formatting, missing semicolons, etc.
- `docs`: Documentation changes
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples
```
feat: add warehouse status recalculation endpoint

fix: correct working day calculation for early morning hours

refactor: extract invoice creation logic to service class

test: add feature tests for OrderController
```

## Steps

1. Run `git status` to see all changes
2. Run `git diff` to review staged and unstaged changes
3. Run `git log --oneline -5` to see recent commit style
4. Stage relevant files with `git add <files>` (prefer specific files over `git add .`)
5. Create commit with descriptive message using HEREDOC format:

```bash
git commit -m "$(cat <<'EOF'
<type>: <description>

<optional details>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

6. Run `git status` to verify commit succeeded

## Important Notes

- Do NOT commit `.env` or other sensitive files
- Do NOT use `--amend` unless explicitly requested (creates new commits)
- Do NOT use `--no-verify` unless explicitly requested
- Do NOT push unless explicitly requested
- If pre-commit hooks fail, fix issues and create NEW commit

## User's hint for this commit
$ARGUMENTS
