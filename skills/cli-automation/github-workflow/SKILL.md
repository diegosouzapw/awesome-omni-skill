---
name: github-workflow
description: GitHub-first workflow - Issues, PRs, milestones, auto-tracking for solo developer productivity
category: workflow
version: 1.0.0
---

# GitHub Workflow Skill

**Complete guide to GitHub integration for autonomous development**

Use this skill when:

- Setting up GitHub workflow for the first time
- Creating issues from test failures or GenAI findings
- Creating pull requests automatically
- Linking work to GitHub issues and milestones
- Automating issue tracking on push/commit

---

## Quick Reference

### Commands Available

```bash
/issue           # Create GitHub issues (test failures, GenAI, manual)
/pr-create       # Create pull request with auto-filled content
```

### Hooks Available

- `auto_track_issues.py` - Auto-create issues on push/commit (configurable)

### Configuration

```bash
# .env file
GITHUB_AUTO_TRACK_ISSUES=true      # Enable auto-tracking
GITHUB_TRACK_ON_PUSH=true          # Track before push
GITHUB_TRACK_THRESHOLD=medium      # Minimum priority (low/medium/high)
GITHUB_DRY_RUN=false               # Preview mode (true = no actual creation)
```

---

## Complete Workflow

### End-to-End: Issue → Feature → PR → Merge

```bash
# 1. Start from GitHub issue
gh issue list
gh issue view 42
gh issue edit 42 --add-assignee @me

# 2. Create feature branch
git checkout -b feature/42-add-user-auth

# 3. Implement feature
"Implement user authentication per issue #42"
# orchestrator runs autonomous pipeline

# 4. Quality check
/full-check

# 5. Commit with issue reference
/commit
# Add to commit message: "Closes #42"

# 6. Create pull request
/pr-create
# Auto-links to issue #42
# Creates draft PR by default

# 7. Review and merge
gh pr view
gh pr ready      # Mark draft as ready
gh pr merge --auto --squash
```

---

## Issue Creation Patterns

### Pattern 1: After Test Failures

```bash
# Run tests
/test

# Tests fail - auto-create issues
/issue
# Menu appears:
# 1. Auto-create from test failures (2 issues) ← Choose this
# 2. Create from GenAI findings
# 3. Create manual issue
# 4. Preview (dry run)
# 5. Cancel

Choice [1-5]: 1

# Result: GitHub issues created automatically
# - Full stack traces included
# - Labeled: automated, test-failure, bug
# - Priority: high
```

**When to use**:

- CI/CD pipeline failures
- Local test failures
- Want automatic bug tracking

### Pattern 2: From GenAI Findings

```bash
# Run GenAI validation
/test-uat-genai        # UX validation
# OR
/test-architecture     # Architecture validation

# Found issues - create tracking issues
/issue
Choice [1-5]: 2

# Result: GitHub issues created from AI analysis
# - UX friction points
# - Architecture drift
# - Performance opportunities
# - Labeled: automated, genai-finding, enhancement
```

**When to use**:

- After GenAI validation
- UX improvements needed
- Architecture cleanup
- Code quality tracking

### Pattern 3: Manual Issue Creation

```bash
/issue
Choice [1-5]: 3

# Interactive prompts:
Title: Memory leak in background sync
Priority [high/medium/low]: high
Type [bug/enhancement/technical-debt/documentation]: bug
Description:
Background sync accumulates memory over time.
After 24 hours, reaches 2GB.
(Press Ctrl+D when done)

Additional labels: performance, memory
Assign to: akaszubski

# Result: Custom GitHub issue created
```

**When to use**:

- Manual bug reports
- Feature requests
- Documentation tasks
- Custom tracking needs

---

## Automatic Issue Tracking

### Enable Auto-Tracking

**Setup** (one-time):

```bash
# Add to .env
cat >> .env << 'EOF'
GITHUB_AUTO_TRACK_ISSUES=true
GITHUB_TRACK_ON_PUSH=true
GITHUB_TRACK_THRESHOLD=medium
GITHUB_DRY_RUN=false
EOF

# Install GitHub CLI
brew install gh

# Authenticate
gh auth login
```

**How it works**:

```bash
# You work normally
/test                  # Tests fail
/commit                # Commit changes
git push               # Push to GitHub

# Hook runs automatically:
# - Detects test failures
# - Creates GitHub issues
# - Labels and prioritizes
# - Links to commits

# No manual /issue needed!
```

**Configuration Options**:

| Variable                   | Values          | Default | Description                |
| -------------------------- | --------------- | ------- | -------------------------- |
| `GITHUB_AUTO_TRACK_ISSUES` | true/false      | false   | Enable auto-tracking       |
| `GITHUB_TRACK_ON_PUSH`     | true/false      | true    | Track before push          |
| `GITHUB_TRACK_ON_COMMIT`   | true/false      | false   | Track after commit         |
| `GITHUB_TRACK_THRESHOLD`   | low/medium/high | medium  | Minimum priority to track  |
| `GITHUB_DRY_RUN`           | true/false      | false   | Preview only (no creation) |

**Example workflows**:

**Conservative** (manual review):

```bash
GITHUB_AUTO_TRACK_ISSUES=true
GITHUB_TRACK_ON_PUSH=false
GITHUB_DRY_RUN=true

# Result: Shows what would be created, but doesn't create
# You review, then manually run /issue
```

**Aggressive** (full automation):

```bash
GITHUB_AUTO_TRACK_ISSUES=true
GITHUB_TRACK_ON_PUSH=true
GITHUB_DRY_RUN=false

# Result: Issues created automatically on every push
# Fast, automated, but less control
```

**Balanced** (recommended):

```bash
GITHUB_AUTO_TRACK_ISSUES=true
GITHUB_TRACK_ON_PUSH=true
GITHUB_TRACK_THRESHOLD=high
GITHUB_DRY_RUN=false

# Result: Only high-priority issues auto-created
# Manual /issue for medium/low priority
```

---

## Pull Request Patterns

### Pattern 1: Simple Draft PR

```bash
/pr-create

# What happens:
# ✅ Creates draft PR
# ✅ Auto-fills title from commit messages
# ✅ Auto-fills body from commit messages
# ✅ Parses "Closes #42" and links issues
# ✅ Draft mode: Requires manual "Ready for review"

# Result:
# Draft PR created
# URL: https://github.com/user/repo/pull/123
```

**When to use**:

- Default workflow (safe)
- Want manual review before marking ready
- Still working on PR
- Need reviewer feedback before finalizing

### Pattern 2: PR with Reviewer Assignment

```bash
/pr-create --reviewer alice

# Same as Pattern 1, plus:
# ✅ Assigns alice as reviewer
# ✅ Sends notification to alice

# For multiple reviewers:
/pr-create --reviewer alice,bob
```

**When to use**:

- Know who should review
- Team workflow (solo developer: skip this)
- Want immediate review request

### Pattern 3: Ready for Review (Not Draft)

```bash
/pr-create --ready

# What's different:
# ✅ Creates PR as "Ready for review" (not draft)
# ✅ Triggers CI/CD immediately
# ✅ Can be merged immediately if checks pass

# Use with caution - skips draft safety
```

**When to use**:

- Confident PR is ready
- Fast-track workflow
- Small changes
- Solo developer (no team review needed)

---

## Sprint/Milestone Integration

### Link Work to Milestones

**In PROJECT.md**:

```markdown
## CURRENT SPRINT

**Sprint Name**: Sprint 6: Documentation Accuracy
**GitHub Milestone**: https://github.com/user/repo/milestone/6
**Duration**: 2025-10-24 → 2025-10-27
```

**orchestrator reads this**:

```bash
# When you run autonomous pipeline
"Implement feature X"

# orchestrator:
# 1. Reads PROJECT.md current sprint
# 2. Queries GitHub Milestone via gh CLI
# 3. Links work to sprint
# 4. Tracks progress
```

**Manual sprint queries**:

```bash
# List milestones
gh api repos/user/repo/milestones

# View specific milestone
gh api repos/user/repo/milestones/6

# List issues in milestone
gh issue list --milestone "Sprint 6"
```

---

## Issue Lifecycle Management

### Close Issues Automatically

**Via commit message**:

```bash
git commit -m "feat: add user auth

Implements JWT-based authentication with secure token handling.

Closes #42
Fixes #43
Resolves #44"

git push

# Result: Issues #42, #43, #44 auto-closed when PR merges
```

**Keywords that work**:

- `Closes #123`
- `Fixes #123`
- `Resolves #123`
- `Closes: #123`
- `Fixes: #123`

### Close Issues Manually

```bash
gh issue close 42
gh issue close 42 --comment "Fixed in PR #123"
```

### Reopen Issues

```bash
gh issue reopen 42
gh issue reopen 42 --comment "Regression found, reopening"
```

### Clean Up Stale Issues

```bash
# List old open issues
gh issue list --state open --json number,title,updatedAt

# Close stale issues
gh issue close 42 --comment "Closing as stale (no activity in 90 days)"
```

---

## Troubleshooting

### "GitHub CLI not found"

```bash
# Install gh CLI
brew install gh         # Mac
# OR
sudo apt install gh     # Linux
# OR
choco install gh        # Windows

# Verify
gh --version
```

### "Not authenticated"

```bash
# Authenticate
gh auth login

# Follow prompts:
# 1. Choose: GitHub.com
# 2. Choose: HTTPS
# 3. Choose: Login with web browser
# 4. Authorize in browser

# Verify
gh auth status
```

### "Permission denied creating issue"

**Check**:

```bash
# Verify repo access
gh repo view

# Check token scopes
gh auth status
```

**Fix**:

```bash
# Re-authenticate with full scopes
gh auth login --scopes repo,write:discussion,workflow
```

### "Issue creation failed"

**Check**:

```bash
# Test manually
gh issue create --title "Test" --body "Test issue"

# If works: Plugin hook issue
# If fails: GitHub CLI issue
```

**Common causes**:

- Rate limiting (wait 60 seconds)
- Network issues (check connection)
- Repository archived (can't create issues)

### "No test failures found"

**Cause**: `/issue` option 1 requires test failures

**Fix**:

```bash
# Run tests first
/test

# If all pass: No issues to create
# If failures: Now /issue will find them
```

---

## Best Practices

### ✅ DO

1. **Start from issues**
   - Every feature should have a GitHub issue
   - Reference issue in branch name: `feature/42-add-auth`
   - Reference issue in commit: `Closes #42`

2. **Use draft PRs by default**
   - `/pr-create` (draft mode)
   - Review yourself before marking ready
   - Safe guard against premature merges

3. **Enable auto-tracking for high priority**
   - `GITHUB_TRACK_THRESHOLD=high`
   - Auto-create for critical bugs
   - Manual /issue for enhancements

4. **Link to PROJECT.md goals**
   - In issue description: "Supports Goal #1: Solo developer productivity"
   - Maintains strategic alignment
   - Makes prioritization clear

5. **Use conventional commits**
   - `feat:`, `fix:`, `docs:`, `refactor:`
   - Clean commit history
   - Auto-generates better PR descriptions

### ❌ DON'T

1. **Skip GitHub issues**
   - All work should be traceable
   - Even quick fixes: Create issue first

2. **Create ready PRs blindly**
   - Avoid `/pr-create --ready` unless confident
   - Draft mode is safer default

3. **Auto-track everything**
   - Don't set `GITHUB_TRACK_THRESHOLD=low`
   - Creates noise, too many issues
   - Medium or high only

4. **Forget to clean up**
   - Close stale issues regularly
   - Archive completed milestones
   - Keep issue list clean

5. **Break the workflow**
   - Don't commit directly to main
   - Don't skip PRs "because solo developer"
   - Workflow builds good habits

---

## Integration with Other Commands

### With Testing

```bash
# Run tests
/test

# If failures: Auto-create issues
/issue
```

### With Commits

```bash
# Format + test + security
/commit

# Standard commit (full checks)
/commit --check

# Push to GitHub (future)
/commit --push
```

### With Alignment

```bash
# Check PROJECT.md alignment
/align-project

# Fix issues found
/align-project

# Create issue for remaining work
/issue
```

---

## Advanced: Custom Issue Templates

**Create** `.github/ISSUE_TEMPLATE/bug_report.md`:

```markdown
---
name: Bug Report
about: Report a bug
labels: bug
---

## Description

Clear description of the bug

## Steps to Reproduce

1. Step 1
2. Step 2
3. See error

## Expected Behavior

What should happen

## Actual Behavior

What actually happens

## Environment

- OS:
- Version:

## Additional Context

Any other context
```

**Use**:

```bash
gh issue create --template bug_report.md
```

---

## Quick Command Reference

```bash
# ISSUES
/issue                        # Interactive issue creation
gh issue list                 # List all issues
gh issue view 42              # View specific issue
gh issue create               # Manual issue creation
gh issue close 42             # Close issue

# PULL REQUESTS
/pr-create                    # Draft PR (default)
/pr-create --ready            # Ready PR
/pr-create --reviewer alice   # PR with reviewer
gh pr list                    # List PRs
gh pr view                    # View current PR
gh pr merge                   # Merge PR

# MILESTONES
gh api repos/user/repo/milestones    # List milestones
gh issue list --milestone "Sprint 6" # Issues in milestone

# WORKFLOW
git checkout -b feature/42-description   # Create branch
/commit                                  # Commit with checks
git push                                 # Push changes
/pr-create                               # Create PR
```

---

## See Also

- `docs/GITHUB-WORKFLOW.md` - Complete workflow guide
- `docs/GITHUB_AUTH_SETUP.md` - Authentication setup
- `commands/issue.md` - `/issue` command reference
- `commands/pr-create.md` - `/pr-create` command reference
- `hooks/auto_track_issues.py` - Auto-tracking hook code

---

**Use this skill to master GitHub-first autonomous development workflow**
