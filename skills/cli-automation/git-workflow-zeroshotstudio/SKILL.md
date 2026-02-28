---
name: git-workflow
description: Git operations with branch/PR standards enforcement
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - mcp__github__create_pull_request
  - mcp__github__create_issue
  - mcp__github__create_branch
---

# /git-workflow - Git Operations with Standards

Orchestrate git operations with safety checks, branch naming enforcement, and PR creation.

## Usage

```bash
/git-workflow branch feature/my-feature    # Create a properly named branch
/git-workflow commit                       # Commit with conventional format
/git-workflow pr                           # Create PR with standards
/git-workflow help                         # Show this help
```

## Arguments

$ARGUMENTS - Subcommand and options

## Subcommands

### `branch <name>` - Create Branch

Creates a new branch with naming validation.

**Valid prefixes**: `feature/`, `bugfix/`, `hotfix/`, `chore/`, `docs/`, `refactor/`

**Steps:**
1. Validate branch name follows convention
2. Check for uncommitted changes
3. Create and switch to new branch
4. Push with upstream tracking (optional)

**Example:**
```bash
/git-workflow branch feature/123-voice-pipeline
```

### `commit` - Commit with Standards

Creates a commit following conventional commit format.

**Pre-commit checks (automatic):**
1. Run `/verify-clean` - linters, tests, sensitive files
2. Run `/docs-audit` - documentation accuracy
3. Stage changes if needed
4. Prompt for commit message in format: `{type}({scope}): {description}`

**Commit types**: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`

**Example output:**
```
feat(vps): Add WordPress deployment automation

- Configure nginx reverse proxy
- Set up automated backups
- Add health check endpoint

Closes #123
```

### `pr` - Create Pull Request

Creates a PR with proper title format and linked issues.

**Steps:**
1. Verify branch is pushed to remote
2. Extract issue number from branch name (if present)
3. Generate PR title from commits: `{type}: {description}`
4. Create PR with summary and test plan
5. Link to issue if found

**PR body format:**
```markdown
## Summary
- Brief description of changes

## Test Plan
- [ ] Tests pass locally
- [ ] Documentation updated
- [ ] No linting errors

Closes #123
```

## Branch Naming Enforcement

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New functionality | `feature/voice-pipeline` |
| `bugfix/` | Bug fixes | `bugfix/auth-401-error` |
| `hotfix/` | Urgent production fixes | `hotfix/db-connection-leak` |
| `chore/` | Maintenance, dependencies | `chore/upgrade-node-22` |
| `docs/` | Documentation only | `docs/api-examples` |
| `refactor/` | Code restructuring | `refactor/auth-module` |

## Workflow Integration

```
1. /git-workflow branch feature/123-desc   # Start feature
2. [Development work]
3. /verify-clean                           # Check quality
4. /docs-audit                             # Check docs
5. /git-workflow commit                    # Commit changes
6. /git-workflow pr                        # Create PR
```

## Output

Provide clear feedback at each step:
- Branch creation confirmation
- Pre-commit check results
- Commit hash and message
- PR URL when created
- Any validation warnings or errors
