---
name: commit
description: Create commits with code quality checks and conventional commit messages
---

# /commit - Commit Creation Skill

Create commits with proper code quality checks and conventional commit messages.

## Language Requirement

**IMPORTANT**: All commit messages MUST be written in **English**.

## Step 0: Verify Branch (MANDATORY)

**CRITICAL**: This step MUST be performed before any other step.

### Check Current Branch

```bash
git branch --show-current
```

### Branch Validation Rules

| Current Branch | Action |
|----------------|--------|
| `main` | ❌ **STOP** - Never commit directly to main |
| `develop` | ❌ **STOP** - Never commit directly to develop |
| `feature/*` | ✅ Proceed with commit |
| `bugfix/*` | ✅ Proceed with commit |
| `hotfix/*` | ✅ Proceed with commit |
| `docs/*` | ✅ Proceed with commit |
| `refactor/*` | ✅ Proceed with commit |

### If on `develop` or `main`

1. **STOP immediately** - Do not proceed with commit
2. Inform the user about the branch policy violation
3. Create a feature branch:
   ```bash
   git fetch origin
   git checkout -b feature/<issue-number>-<description> origin/develop
   ```
4. After branch creation, proceed with the commit workflow

### Error Message Template

If on protected branch, display:
```
⚠️ ERROR: Cannot commit directly to '{branch_name}' branch.

This project uses Git Flow. Please create a feature branch first:
  git checkout -b feature/<issue>-<description> origin/develop

See .claude/instructions.md for branching guidelines.
```

## Pre-flight Checks (MANDATORY)

### 1. Format Code

```bash
cargo fmt --all
```

This automatically formats all code. Stage any formatting changes.

### 2. Clippy Check

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

**CRITICAL**: Zero warnings required. Fix all issues before committing.

### 3. Security Check

Verify no secrets in staged files:

- [ ] No API keys
- [ ] No passwords or credentials
- [ ] No tokens or secret strings
- [ ] No `.env` files with real values
- [ ] No `credentials.json` or similar

Files to check: `git diff --cached --name-only`

If secrets are found:

1. Remove secrets from files
2. Add file to `.gitignore` if appropriate
3. WARN the user about the security issue

## Steps

### Step 1: Check Current Status

```bash
git status
```

Identify:

- Staged changes
- Unstaged changes
- Untracked files

### Step 2: Run Pre-flight Checks

Execute format and clippy checks. If clippy fails:

1. Fix all warnings
2. Re-run clippy until clean

### Step 3: Stage Changes

```bash
# Stage specific files
git add <files>

# Or stage all changes
git add .
```

### Step 4: Review Staged Changes

```bash
git diff --cached
```

Verify all intended changes are staged.

### Step 5: Generate Commit Message

Use **Conventional Commits** format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
Co-Authored-By: Claude <noreply@anthropic.com>
```

#### Types

| Type       | Description                                      |
|------------|--------------------------------------------------|
| `feat`     | New feature                                      |
| `fix`      | Bug fix                                          |
| `docs`     | Documentation only changes                       |
| `style`    | Formatting, no code change                       |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvement                          |
| `test`     | Adding or correcting tests                       |
| `chore`    | Maintenance tasks                                |
| `ci`       | CI configuration changes                         |

#### Scope (Optional)

Common scopes for this project:

- `domain` - Domain layer changes
- `application` - Application layer changes
- `adapters` - Adapter layer changes
- `ports` - Port definitions
- `cli` - CLI changes
- `deps` - Dependency updates

#### Examples

```
feat(cli): add --verbose flag for detailed output

fix(domain): correct dependency graph cycle detection

docs: update README with new installation instructions

refactor(adapters): extract common HTTP client logic

test(application): add unit tests for GenerateSbomUseCase
```

### Step 6: Create Commit

Use HEREDOC for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<body if needed>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Step 7: Verify Commit

```bash
git log -1 --format=full
```

Confirm:

- [ ] Commit message is in English
- [ ] Type is correct
- [ ] Co-Authored-By is present
- [ ] No secrets in the commit

## Error Handling

### Clippy Failures

If clippy fails with warnings:

1. Read the warning messages carefully
2. Fix each warning
3. Re-run clippy
4. Once clean, proceed with commit

### Pre-commit Hook Failures

If a pre-commit hook rejects the commit:

1. Read the hook output
2. Fix the issues
3. Stage the fixes
4. Create a NEW commit (do not amend unless specifically needed)

### Secrets Detected

If secrets are found in staged files:

1. **STOP** - Do not commit
2. Remove secrets from the file
3. Consider if the file should be in `.gitignore`
4. WARN the user
5. Ask for confirmation before proceeding

## Example Usage

User: "変更をコミットして"

Claude executes /commit skill:

1. Runs `cargo fmt --all`
2. Runs `cargo clippy --all-targets --all-features -- -D warnings`
3. Checks for secrets in staged files
4. Reviews changes with `git diff --cached`
5. Generates conventional commit message in English
6. Creates commit with Co-Authored-By trailer
7. Confirms commit with `git log -1`
8. Reports success to user
