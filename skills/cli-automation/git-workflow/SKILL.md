---
name: git-workflow
description: Orient repository git workflow conventions. Use when creating branches, writing commits, or opening PRs. Covers branch naming (feat/*, fix/*, etc.), conventional commit messages with co-author footers, multi-commit PR workflow, and gh pr create templates.
---

# Orient Repository Git Workflow

## Overview

The Orient monorepo follows conventional commit practices with specific conventions for branching, commit messages, and pull requests. This guide ensures consistency across the codebase and streamlines the development process.

## Branch Naming

All feature and fix branches follow this pattern:

```
{type}/{feature-name}
```

### Branch Types

| Type        | Purpose                              | Example                  |
| ----------- | ------------------------------------ | ------------------------ |
| `feat/`     | New features                         | `feat/add-dark-mode`     |
| `fix/`      | Bug fixes                            | `fix/auth-token-expiry`  |
| `docs/`     | Documentation updates                | `docs/update-readme`     |
| `refactor/` | Code refactoring (no feature change) | `refactor/extract-utils` |
| `test/`     | Test additions/updates               | `test/add-e2e-coverage`  |
| `perf/`     | Performance improvements             | `perf/optimize-queries`  |
| `ci/`       | CI/CD configuration                  | `ci/update-workflows`    |

### Branch Naming Best Practices

- Use lowercase with hyphens: `feat/user-authentication` ✅ not `feat/UserAuthentication`
- Keep names concise: `feat/dashboard-cards` ✅ not `feat/add-new-dashboard-card-components-with-hover`
- Reference issue when applicable: `feat/add-api-caching-#123`

## Conventional Commits

All commits follow the conventional commit format:

```
{type}({scope}): {subject}

{body}

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

### Commit Types

| Type       | Description                              |
| ---------- | ---------------------------------------- |
| `feat`     | New feature                              |
| `fix`      | Bug fix                                  |
| `docs`     | Documentation                            |
| `style`    | Code style (formatting, semicolons, etc) |
| `refactor` | Code change without feature/bug fix      |
| `perf`     | Performance improvement                  |
| `test`     | Adding or updating tests                 |
| `chore`    | Build process, dependency updates        |
| `ci`       | CI/CD configuration                      |

### Commit Scope

The scope indicates which part of the codebase was modified:

```
feat(dashboard): add dark mode toggle
fix(api): resolve token refresh timeout
test(dashboard): improve component coverage
```

Common scopes in Orient:

- `dashboard` / `dashboard-frontend`
- `api` / `api-gateway`
- `auth`
- `integrations`
- `apps` / `mini-apps`
- `bot-slack` / `bot-whatsapp`
- `database` / `schemas`

### Subject Line

- Imperative mood: "add" not "added" or "adds"
- Don't capitalize first letter: `fix(api): resolve` ✅ not `Fix(api): Resolve`
- No period at end: `feat: add feature` ✅ not `feat: add feature.`
- Limit to 50 characters when possible
- Clear, descriptive: `feat: add integration-active endpoint` ✅ not `feat: update`

### Body (Optional but Recommended)

For complex changes, add a body explaining:

- Why this change is needed
- What problem it solves
- Any relevant implementation details

```
feat(dashboard): add missing integrations display

Display missing integration requirements prominently in the apps list.
Helps new users understand what setup is needed before using an app.

- New MissingIntegrationsBadge component
- Integrations column in apps table
- Hover tooltip with missing requirements
```

### Co-Author Footer

Always include the co-author footer (automated by pre-commit hooks when using Claude Code):

```
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

Or with multiple authors:

```
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
Co-Authored-By: User Name <user.email@company.com>
```

## Multi-Commit PR Workflow

### Creating Commits with Co-Author Footer

Using git directly (manually add footer):

```bash
git commit -m "$(cat <<'EOF'
feat(dashboard): add feature

Detailed explanation of what and why.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

Or using a template file:

```bash
# Create commit with heredoc
git commit -m "feat: add feature body"
```

### Typical Multi-Commit PR

A well-structured PR often contains 2-4 related commits:

```
feat(dashboard): refactor colors to design system
  └─ Updates MiniAppEditor components

feat(dashboard): add missing integrations display
  └─ New component + AppsTab changes

test(dashboard): add comprehensive test coverage
  └─ Component tests + integration tests
```

### Creating Pull Requests

Use `gh pr create` with proper title and body:

```bash
gh pr create --base dev --title "feat: improve miniapp UX" \
  --body "$(cat <<'EOF'
## Summary

Concise 1-3 sentence summary of changes.

## Changes

- Point 1
- Point 2
- Point 3

## Test Coverage

- ✅ All tests passing
- ✅ No regressions
- ✅ New test coverage added

🤖 Generated with Claude Code
EOF
)"
```

### PR Title Format

Follow the same conventional commit format for PR titles:

```
feat(scope): description
fix(scope): description
test(scope): description
```

## Pre-Commit Hooks

The repository uses pre-commit hooks that automatically:

1. Run Prettier for code formatting (~1-2 seconds)
2. Fix formatting issues automatically
3. Skip ESLint/TypeScript checks (run in CI)

**Note:** Commits run through hooks automatically. The `Co-Authored-By` footer is preserved.

## Workflow Example

### Step 1: Create Feature Branch

```bash
git checkout -b feat/improve-miniapp-ux
```

### Step 2: Make Changes and Commit

```bash
# Make changes
git add packages/dashboard-frontend/src/components/AppsTab.tsx

# Commit with footer
git commit -m "$(cat <<'EOF'
feat(dashboard): add missing integrations display

Display missing integration requirements in the apps list.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

### Step 3: Push Branch

```bash
git push -u origin feat/improve-miniapp-ux
```

### Step 4: Create PR

```bash
gh pr create --base dev \
  --title "feat: improve miniapp UX for new users" \
  --body "## Summary
Improve experience for new users by displaying missing integrations.

## Changes
- Add MissingIntegrationsBadge component
- Display integrations column in apps table
- Update Edit with AI styling

🤖 Generated with Claude Code"
```

## Common Issues & Solutions

### Issue: Commit message without co-author

**Solution:** Use heredoc format with `EOF` delimiter:

```bash
git commit -m "$(cat <<'EOF'
feat: add feature

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

### Issue: Need to amend last commit

**Create a new commit instead** (avoid `--amend` unless explicitly requested by user):

```bash
git commit -m "feat: updated feature"
```

### Issue: Commit stuck in detached HEAD state

```bash
# Create branch from current commit
git checkout -b feat/branch-name

# Or checkout dev and try again
git checkout dev
```

### Issue: Pre-commit hook failing

The hook only runs Prettier. If failing:

- Check file formatting issues
- Let the hook auto-fix them
- Stage and commit again

## Best Practices

1. **One logical change per commit** - Each commit should be independently meaningful
2. **Write clear commit messages** - Future developers (including you!) will thank you
3. **Keep PRs focused** - Don't mix unrelated features in one PR
4. **Include tests with features** - Use `test()` commit type
5. **Reference issues** - Include issue numbers in commit messages or PR description
6. **Review before pushing** - Run `git diff` to check your changes
7. **Use branches** - Never commit directly to main/dev
8. **Keep commits small** - Easier to review, understand, and revert if needed

## Commit Consolidation Strategy

### When to Use `--amend` vs Creating New Commits

Understanding when to consolidate commits vs keeping them separate improves both PR clarity and repository history.

#### Prefer Creating New Commits When:

1. **Each commit represents a distinct logical change** (e.g., feature implementation, bug fix, test addition)
   - Example: First commit adds component, second adds tests, third refactors colors
   - Reviewers can understand each change independently
   - Easier to bisect if issues arise
   - Better for understanding code evolution

2. **Before pushing to remote or opening a PR**
   - Keep the iteration history visible
   - Shows problem-solving approach
   - Demonstrates thorough testing at each step
   - Useful for code review comments ("see commit 3 for details")

3. **Multiple attempts at the same fix**
   - Document what was tried and why it changed
   - Example: "First attempt used `jq -s`, second attempt used `jq -cs` for compact output"

#### Use `--amend` When:

1. **Fixing typos or formatting in the last commit** (before pushing)
   - Small corrections that don't warrant a separate commit
   - Example: Missing semicolon, variable name typo

2. **Adding forgotten changes to related work** (before pushing)
   - Ensures logical grouping when pushed
   - Only if you haven't pushed yet (never amend public history!)

3. **The commit hasn't been pushed to remote**
   - Use `git push --force-with-lease` if amending after pushing, but this is discouraged for shared branches

#### Best Practices for PR Review Workflows

**Multiple commits are often better for PRs because they:**

- Show the development process and iteration
- Allow reviewers to follow the reasoning
- Make it easier to discuss specific changes
- Help identify exactly when a bug was introduced (with `git bisect`)

**Example PR workflow (3 commits):**

```
6a7fcf1 fix(ci): fix JSON array formatting in detect-changes workflow
23d6d78 fix(ci): fix JSON array formatting in detect-changes workflow
fe35c9a fix(ci): fix JSON array formatting in detect-changes workflow
```

This shows three separate attempts to fix the same issue - the first attempt used `xargs`, the second added `grep -v`, and the third added `jq -c`. The progression helps reviewers understand:

- Why the first approaches didn't work
- What the final solution was
- How to avoid similar issues in the future

**Consolidate commits with `git rebase -i` only when:**

- Requested by project maintainers
- You're preparing for production release
- The PR is squash-merged anyway (then it doesn't matter)
- You want a completely clean history for a stable branch

### Local Testing of Workflow Changes

**Important:** Test GitHub Actions workflow changes locally before pushing to CI, especially for JSON output formatting:

```bash
# Install act: https://github.com/nektos/act
brew install act

# Test a specific workflow job
act -j detect-changes

# Test with specific event
act pull_request

# Verbose output for debugging
act -j detect-changes -v
```

**Why this matters:** We encountered `##[error]Invalid format '  "app-name"'` errors three times in CI because JSON formatting wasn't caught locally. Using `act` would have caught these issues immediately:

```bash
# This would have revealed the formatting issue:
act -j detect-changes

# Output would show:
# ##[error]Unable to process file command 'output' successfully.
# ##[error]Invalid format '  "simple-todo"'
```

Then fix locally and test again with `act` before pushing.

## GitHub Actions Workflow Debugging

### Multi-Job Dependency Troubleshooting

When using matrix builds with job outputs in GitHub Actions, ensure proper JSON formatting:

#### Output Variable Format Requirements

GitHub Actions workflow output variables must be single-line JSON. Use `jq -c` (compact) flag, not just `jq`:

```bash
# ❌ WRONG - produces pretty-printed JSON with newlines
APPS_JSON=$(echo "$CHANGED_APPS" | tr ' ' '\n' | jq -R . | jq -s .)
echo "apps_list=$APPS_JSON" >> $GITHUB_OUTPUT  # Error: "Invalid format '  "app"'"

# ✓ CORRECT - produces compact single-line JSON
APPS_JSON=$(echo "$CHANGED_APPS" | tr ' ' '\n' | grep -v '^$' | jq -R . | jq -cs .)
echo "apps_list=$APPS_JSON" >> $GITHUB_OUTPUT  # Works: ["app1","app2"]
```

#### Detect-Changes Pattern for Monorepos

For monorepos with selective builds based on changed files:

```bash
# Compare base SHA with current commit
if [ "${{ github.event_name }}" == "pull_request" ]; then
  BASE_SHA=${{ github.event.pull_request.base.sha }}
else
  BASE_SHA=${{ github.event.before }}
fi

# Get changed app directories
CHANGED_APPS=$(git diff --name-only $BASE_SHA ${{ github.sha }} | \
  grep '^apps/' | \
  grep -v '^apps/README.md' | \
  grep -v '^apps/_shared/' | \
  cut -d'/' -f2 | \
  sort -u | \
  tr '\n' ' ')

# Output as JSON array (use jq -cs for compact output)
if [ -z "$CHANGED_APPS" ]; then
  echo "apps=false" >> $GITHUB_OUTPUT
  echo "apps_list=" >> $GITHUB_OUTPUT
else
  echo "apps=true" >> $GITHUB_OUTPUT
  APPS_JSON=$(echo "$CHANGED_APPS" | tr ' ' '\n' | grep -v '^$' | jq -R . | jq -cs .)
  echo "apps_list=$APPS_JSON" >> $GITHUB_OUTPUT
fi
```

#### Conditional Matrix Build

Use the output from detect-changes to conditionally run matrix builds:

```yaml
detect-changes:
  runs-on: ubuntu-latest
  outputs:
    apps: ${{ steps.changes.outputs.apps }}
    apps_list: ${{ steps.changes.outputs.apps_list }}
  steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - name: Detect Changed Apps
      id: changes
      run: |
        # Script above...
        echo "apps=true" >> $GITHUB_OUTPUT
        echo "apps_list=[\"app1\",\"app2\"]" >> $GITHUB_OUTPUT

build-apps:
  needs: detect-changes
  if: needs.detect-changes.outputs.apps == 'true'
  runs-on: ubuntu-latest
  strategy:
    matrix:
      app: ${{ fromJson(needs.detect-changes.outputs.apps_list) }}
    fail-fast: false
  steps:
    - uses: actions/checkout@v4
    # Build steps...
```

### Common Issues & Solutions

**Issue**: `##[error]Invalid format '  "app-name"'` in workflow output

**Cause**: JSON output has whitespace/formatting. GitHub Actions requires compact single-line format.

**Solution**: Use `jq -cs .` instead of `jq -s .` to produce compact output.

**Issue**: Matrix build not running even though files changed

**Cause**: `if: needs.detect-changes.outputs.apps == 'true'` string comparison fails if output has extra whitespace.

**Solution**: Ensure output variable is exactly the string `true` or `false` with no extra formatting.

## References

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Orient Monorepo Structure](/project-architecture)
- [Git Hooks Documentation](git-hooks)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
