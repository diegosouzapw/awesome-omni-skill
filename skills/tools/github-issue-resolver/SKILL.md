---
name: github-issue-resolver
description: Strategically resolves GitHub Actions failures, failed pull requests, and Dependabot issues using the gh CLI with intelligent analysis and automated fixes.
---

# GitHub Issue Resolver Skill

## Overview

This skill enables Claude Code to systematically diagnose and resolve GitHub repository maintenance issues, including:

- **GitHub Actions failures**: Analyze logs, identify root causes, implement fixes, and re-trigger workflows
- **Failed pull requests**: Checkout branches, fix check failures, resolve conflicts, and manage reviews
- **Dependabot issues**: Evaluate dependency updates, resolve conflicts, and strategically merge compatible updates
- **Strategic triage**: Prioritize issues by severity and impact, create systematic workflows

### Prerequisites

Before using this skill, ensure:

- ✅ GitHub CLI (`gh`) is installed and authenticated (`gh auth status`)
- ✅ You have write access to the target repository
- ✅ Git is configured with proper user credentials
- ✅ You're in a valid git repository directory
- ✅ Working directory is clean or changes are stashed

### Capabilities

This skill provides:
- Automated log analysis and error pattern recognition
- Intelligent root cause diagnosis for CI/CD failures
- Code-aware fixes for test failures, linting issues, and build errors
- Strategic batching of dependency updates
- Audit trail documentation with PR comments
- Safe rollback procedures

---

## When to Use This Skill

Activate this skill with any of these phrases:

- "Fix the failing GitHub Actions"
- "Resolve failed CI/CD runs"
- "Handle failed pull requests"
- "Manage Dependabot PRs"
- "Triage GitHub issues"
- "Fix failing checks on PR #123"
- "Resolve all Dependabot conflicts"
- "Clean up failed workflow runs"

**Triggering Scenarios:**

- CI/CD pipeline shows red status indicators
- Pull requests blocked by failed checks
- Multiple Dependabot PRs pending with conflicts
- Repository has accumulated technical debt from failed automation
- Need systematic cleanup before a release
- After bulk dependency updates or major refactors

---

## Core Workflows

### 1. GitHub Actions Failure Resolution

**Objective**: Identify, diagnose, and fix failed workflow runs.

#### Step 1: List Failed Runs

```bash
# List recent failed runs
gh run list --status failure --limit 10

# List failed runs for specific workflow
gh run list --workflow "CI" --status failure --limit 5

# Get JSON output for programmatic processing
gh run list --status failure --json databaseId,name,headBranch,conclusion,createdAt --limit 10
```

#### Step 2: Analyze Failure Logs

```bash
# View summary of failed run
gh run view <run-id>

# Get detailed logs for failed jobs only
gh run view <run-id> --log-failed

# Download all logs for offline analysis
gh run view <run-id> --log-failed > failure-logs.txt
```

#### Step 3: Diagnose Root Cause

**Common Failure Patterns:**

| Error Pattern | Root Cause | Action Required |
|---------------|------------|-----------------|
| `npm test failed` | Test failures | Fix failing tests or test setup |
| `golangci-lint` errors | Code quality issues | Fix linting violations |
| `go build` failed | Compilation errors | Fix syntax/type errors |
| `docker build` failed | Dockerfile or dependency issues | Fix Dockerfile, update dependencies |
| `Error: No space left on device` | Runner disk space | Clean up artifacts, optimize build |
| `Error: Process completed with exit code 137` | OOM (Out of Memory) | Reduce memory usage or increase limits |
| `unable to access 'https://...'` | Network/auth issues | Check credentials, retry |

**Diagnostic Process:**

1. Read the full log output
2. Identify the failing step and error message
3. Trace back to root cause (not just symptom)
4. Check if issue is environment-specific or code-specific
5. Determine if fix is needed in:
   - Source code
   - Test files
   - Workflow YAML
   - Dependencies
   - Configuration files

#### Step 4: Implement Fix

**Decision Tree:**

```
Is the failure in source code?
├─ YES → Read affected files, implement fix, run tests locally if possible
└─ NO → Is it in workflow configuration?
    ├─ YES → Edit .github/workflows/*.yml, validate YAML syntax
    └─ NO → Is it a dependency issue?
        ├─ YES → Update package.json/go.mod/requirements.txt
        └─ NO → Document issue, escalate for manual review
```

**Implementation Pattern:**

```bash
# 1. Identify files needing changes from error logs
# 2. Read relevant files
# 3. Make targeted fixes
# 4. Commit changes with descriptive message

git add <affected-files>
git commit -m "fix: resolve <specific-error> in <component>

Fixes GitHub Actions run #<run-id>
Root cause: <brief-explanation>
Changes: <list-of-changes>"

# 5. Push changes
git push origin <branch-name>
```

#### Step 5: Re-trigger and Verify

```bash
# Re-run failed jobs
gh run rerun <run-id> --failed

# Watch run progress
gh run watch <run-id>

# Verify success
gh run view <run-id>
```

**Success Criteria:**
- ✅ All jobs complete with green status
- ✅ No new failures introduced
- ✅ Fix is committed with clear message
- ✅ Logs show expected behavior

---

### 2. Failed Pull Request Management

**Objective**: Fix PRs blocked by failed checks or conflicts.

#### Step 1: List and Prioritize PRs

```bash
# List all PRs with checks
gh pr list --json number,title,author,statusCheckRollup

# List PRs with failed checks
gh pr list --search "status:failure"

# List PRs by specific author (e.g., Dependabot)
gh pr list --author "app/dependabot"

# Get detailed PR information
gh pr view <pr-number> --json number,title,statusCheckRollup,mergeable
```

**Prioritization Order:**

1. **Critical**: Security patches, breaking production builds
2. **High**: Feature PRs blocking other work, main branch protection failures
3. **Medium**: Routine updates with failed tests
4. **Low**: Minor dependency updates, documentation changes

#### Step 2: Checkout and Analyze

```bash
# Checkout PR branch locally
gh pr checkout <pr-number>

# View PR status and checks
gh pr checks <pr-number>

# View detailed check information
gh pr view <pr-number>
```

#### Step 3: Fix Check Failures

**Common Scenarios:**

**Scenario A: Failed Tests**
```bash
# 1. Identify failing tests from check output
gh pr checks <pr-number>

# 2. Run tests locally
make test  # or appropriate test command

# 3. Fix failing tests
# [Read test files, identify issues, implement fixes]

# 4. Verify locally
make test

# 5. Commit and push
git add .
git commit -m "fix: resolve test failures in PR #<pr-number>"
git push
```

**Scenario B: Merge Conflicts**
```bash
# 1. Update branch with base
gh pr update-branch <pr-number>

# If conflicts exist:
# 2. Fetch latest
git fetch origin

# 3. Merge base branch
git merge origin/main  # or origin/master

# 4. Resolve conflicts
# [Read conflicted files, resolve markers, test]

# 5. Complete merge
git add .
git commit -m "fix: resolve merge conflicts with main"
git push
```

**Scenario C: Linting/Formatting Issues**
```bash
# 1. Run linter locally
make lint  # or golangci-lint run, npm run lint, etc.

# 2. Auto-fix if possible
make lint-fix  # or appropriate fix command

# 3. Manual fixes for remaining issues
# [Read files, fix violations]

# 4. Verify and commit
make lint
git add .
git commit -m "fix: resolve linting issues in PR #<pr-number>"
git push
```

#### Step 4: Document and Request Review

```bash
# Add comment explaining fixes
gh pr comment <pr-number> --body "Fixed check failures:
- Resolved test failures in pkg/clients/kubernetes_test.go
- Updated linting issues in internal/server/server.go
- Rebased on latest main to resolve conflicts

All checks now passing. Ready for review."

# Request review if needed
gh pr edit <pr-number> --add-reviewer <username>

# Mark as ready for review (if draft)
gh pr ready <pr-number>
```

#### Step 5: Merge When Ready

```bash
# Check merge eligibility
gh pr view <pr-number> --json mergeable,mergeStateStatus

# Merge if all checks pass
gh pr merge <pr-number> --auto --squash  # or --merge, --rebase

# Verify merge
gh pr view <pr-number> --json state,merged
```

---

### 3. Dependabot Issue Handling

**Objective**: Efficiently manage automated dependency updates.

#### Step 1: Identify Dependabot PRs

```bash
# List all Dependabot PRs
gh pr list --author "app/dependabot" --json number,title,headRefName,statusCheckRollup

# Group by dependency type
gh pr list --author "app/dependabot" --json title | jq -r '.[].title' | sort

# Identify PRs with conflicts or failed checks
gh pr list --author "app/dependabot" --search "status:failure"
```

#### Step 2: Categorize Updates

**Semantic Versioning Analysis:**

| Update Type | Example | Risk Level | Strategy |
|-------------|---------|------------|----------|
| **Patch** | 1.2.3 → 1.2.4 | Low | Batch merge |
| **Minor** | 1.2.3 → 1.3.0 | Medium | Test individually |
| **Major** | 1.2.3 → 2.0.0 | High | Manual review required |
| **Security** | Any with security label | Critical | Immediate priority |

**Categorization Process:**

```bash
# For each Dependabot PR, extract version change from title
# Example: "Bump golang from 1.23-alpine to 1.25-alpine"
# - Package: golang
# - From: 1.23
# - To: 1.25
# - Type: Minor version bump

# Check for security labels
gh pr view <pr-number> --json labels

# Check for breaking changes in release notes
gh pr view <pr-number> --json body
```

#### Step 3: Strategic Batching

**Batch Merge Decision Tree:**

```
Is this a security update?
├─ YES → Merge immediately after checks pass
└─ NO → Is this a patch version?
    ├─ YES → Safe to batch with other patches
    └─ NO → Is this a minor version?
        ├─ YES → Can batch if same dependency ecosystem and checks pass
        └─ NO → Major version → Review breaking changes, merge individually
```

**Batching Strategy:**

```bash
# Batch 1: All patch updates for same package manager
# Example: npm patches
gh pr list --author "app/dependabot" --search "Bump @types" --json number

# Batch 2: Security updates (highest priority)
gh pr list --author "app/dependabot" --label "security" --json number

# Batch 3: Minor updates with passing checks
gh pr list --author "app/dependabot" --search "Bump" --json number,statusCheckRollup
```

#### Step 4: Resolve Dependabot Conflicts

**Conflict Resolution Pattern:**

```bash
# 1. Checkout Dependabot PR
gh pr checkout <pr-number>

# 2. Rebase on latest main
git fetch origin main
git rebase origin/main

# 3. If conflicts in lock files (package-lock.json, go.sum, etc.)
# Accept Dependabot's changes and regenerate

# For npm:
git checkout --theirs package-lock.json
npm install

# For Go:
git checkout --theirs go.sum
go mod tidy

# For Python:
git checkout --theirs poetry.lock
poetry lock --no-update

# 4. Verify build still works
make build && make test

# 5. Complete rebase
git add .
git rebase --continue
git push --force-with-lease

# 6. Ask Dependabot to rebase (alternative)
gh pr comment <pr-number> --body "@dependabot rebase"
```

#### Step 5: Use Dependabot Commands

**Dependabot Command Reference:**

```bash
# Rebase PR to resolve conflicts
gh pr comment <pr-number> --body "@dependabot rebase"

# Recreate PR
gh pr comment <pr-number> --body "@dependabot recreate"

# Merge PR (if auto-merge enabled)
gh pr comment <pr-number> --body "@dependabot merge"

# Squash and merge
gh pr comment <pr-number> --body "@dependabot squash and merge"

# Ignore this dependency
gh pr comment <pr-number> --body "@dependabot ignore this dependency"

# Ignore this major version
gh pr comment <pr-number> --body "@dependabot ignore this major version"
```

#### Step 6: Validate and Merge

**Validation Checklist:**

- [ ] All checks passing
- [ ] No merge conflicts
- [ ] Lock files regenerated correctly
- [ ] Build succeeds locally
- [ ] Tests pass
- [ ] No unexpected dependency additions
- [ ] Security vulnerabilities resolved (if applicable)

**Merge Execution:**

```bash
# Enable auto-merge for Dependabot PRs with passing checks
gh pr merge <pr-number> --auto --squash

# Or merge immediately
gh pr merge <pr-number> --squash --delete-branch

# Batch merge multiple PRs
for pr in <pr-numbers>; do
  gh pr merge $pr --auto --squash
done
```

---

### 4. Priority Triage System

**Objective**: Systematically process multiple issues in optimal order.

#### Triage Workflow

**Phase 1: Discovery**

```bash
# Get comprehensive view of repository health
gh run list --status failure --limit 20
gh pr list --search "status:failure"
gh issue list --label "bug" --state open

# Export to structured format
gh run list --status failure --json databaseId,name,conclusion,createdAt > failed-runs.json
gh pr list --json number,title,statusCheckRollup,labels > prs.json
```

**Phase 2: Categorization**

**Priority Matrix:**

| Priority | Criteria | Examples | Action Timeframe |
|----------|----------|----------|------------------|
| **P0** | Production broken, security vulnerabilities | Main branch build failing, CVE patches | Immediate |
| **P1** | Blocking work, failing releases | Feature branch CI broken, release workflow failed | Today |
| **P2** | Non-blocking failures | Dependabot minor updates with conflicts | This week |
| **P3** | Minor issues, documentation | Linting in draft PRs, doc build warnings | When time permits |

**Phase 3: Execution Plan**

```bash
# Create ordered task list based on priority
# Execute in sequence:

# 1. P0: Fix main branch
# 2. P1: Unblock feature work
# 3. P2: Clean up Dependabot backlog
# 4. P3: Housekeeping

# Track progress with comments
gh issue create --title "GitHub Maintenance - $(date +%Y-%m-%d)" \
  --body "## Triage Summary

**P0 Issues:**
- [ ] Fix main branch CI (#<run-id>)
- [ ] Merge security patch (#<pr-number>)

**P1 Issues:**
- [ ] Fix feature PR #<number>
- [ ] Resolve workflow timeout

**P2 Issues:**
- [ ] Merge 5 Dependabot PRs
- [ ] Update outdated workflows

**P3 Issues:**
- [ ] Clean up stale branches
- [ ] Update documentation"
```

**Phase 4: Execution**

Follow workflows 1-3 above for each issue, documenting progress:

```bash
# After each fix, update tracking issue
gh issue comment <tracking-issue-number> --body "✅ Completed: Fixed main branch CI
- Root cause: Test timeout in integration suite
- Fix: Increased timeout from 5m to 10m
- Verification: Run #<new-run-id> passed"
```

**Phase 5: Summary Report**

```bash
# After session, create summary
gh issue comment <tracking-issue-number> --body "## Session Summary

**Resolved:**
- ✅ 3 failed workflow runs fixed
- ✅ 5 Dependabot PRs merged
- ✅ 2 feature PRs unblocked

**Pending:**
- ⏳ 1 major version upgrade needs manual review
- ⏳ 2 PRs waiting for external review

**Metrics:**
- Time saved: ~2 hours of manual work
- Issues closed: 8
- Success rate: 90%"
```

---

## Command Reference

### Essential gh CLI Commands

#### Repository Information

```bash
# View repository details
gh repo view

# Clone repository
gh repo clone <owner>/<repo>

# Fork repository
gh repo fork <owner>/<repo>
```

#### Workflow Runs

```bash
# List runs
gh run list [--workflow <name>] [--status <status>] [--limit <n>]

# View run details
gh run view <run-id> [--log] [--log-failed]

# Rerun workflows
gh run rerun <run-id> [--failed]  # Rerun only failed jobs
gh run rerun <run-id>             # Rerun all jobs

# Watch run progress
gh run watch <run-id>

# Download run artifacts
gh run download <run-id>

# Cancel run
gh run cancel <run-id>
```

#### Pull Requests

```bash
# List PRs
gh pr list [--author <user>] [--label <label>] [--search <query>]

# View PR details
gh pr view <pr-number> [--json <fields>]

# Create PR
gh pr create --title "<title>" --body "<description>"

# Checkout PR
gh pr checkout <pr-number>

# Check PR status
gh pr checks <pr-number>

# Update PR branch
gh pr update-branch <pr-number>

# Comment on PR
gh pr comment <pr-number> --body "<message>"

# Merge PR
gh pr merge <pr-number> [--squash|--merge|--rebase] [--auto] [--delete-branch]

# Close PR
gh pr close <pr-number>

# Reopen PR
gh pr reopen <pr-number>

# Mark PR ready for review
gh pr ready <pr-number>
```

#### Issues

```bash
# List issues
gh issue list [--label <label>] [--state <state>] [--assignee <user>]

# View issue
gh issue view <issue-number>

# Create issue
gh issue create --title "<title>" --body "<body>"

# Comment on issue
gh issue comment <issue-number> --body "<message>"

# Close issue
gh issue close <issue-number>

# Edit issue
gh issue edit <issue-number> [--title "<title>"] [--body "<body>"]
```

#### Workflows

```bash
# List workflows
gh workflow list

# View workflow
gh workflow view <workflow-name>

# Run workflow
gh workflow run <workflow-name>

# Enable/disable workflow
gh workflow enable <workflow-name>
gh workflow disable <workflow-name>
```

### JSON Output and Parsing

**Useful JSON Fields:**

```bash
# Workflow runs
gh run list --json databaseId,name,headBranch,conclusion,createdAt,updatedAt

# Pull requests
gh pr list --json number,title,state,statusCheckRollup,mergeable,labels,author

# Issues
gh issue list --json number,title,state,labels,assignees,createdAt

# Parse with jq
gh pr list --json number,title,statusCheckRollup | \
  jq '.[] | select(.statusCheckRollup[].conclusion == "failure")'
```

---

## Decision Trees

### 1. Failure Diagnosis Flow

```
Identify failed run/PR
    ↓
Retrieve error logs
    ↓
Parse error message
    ↓
┌─────────────────────────────────────────────┐
│ Error Type?                                 │
├─────────────────────────────────────────────┤
│ Test Failure                                │
│   → Read test file                          │
│   → Identify assertion/expectation mismatch │
│   → Fix code or test                        │
│   → Run locally, commit, push               │
├─────────────────────────────────────────────┤
│ Build/Compile Error                         │
│   → Read source file at error line          │
│   → Fix syntax/type/import error            │
│   → Verify compilation, commit, push        │
├─────────────────────────────────────────────┤
│ Linting/Formatting                          │
│   → Run linter locally                      │
│   → Auto-fix if possible                    │
│   → Manual fix remaining issues             │
│   → Verify, commit, push                    │
├─────────────────────────────────────────────┤
│ Dependency/Installation                     │
│   → Check lock file consistency             │
│   → Update dependencies                     │
│   → Regenerate lock file                    │
│   → Test build, commit, push                │
├─────────────────────────────────────────────┤
│ Infrastructure/Timeout                      │
│   → Check workflow YAML                     │
│   → Adjust timeout/resource limits          │
│   → Optimize build if needed                │
│   → Commit workflow changes                 │
├─────────────────────────────────────────────┤
│ Flaky/Intermittent                          │
│   → Rerun without changes                   │
│   → If persists, investigate race condition │
│   → Add retry logic or fix timing issue     │
└─────────────────────────────────────────────┘
    ↓
Re-trigger workflow
    ↓
Verify success
```

### 2. Dependabot Merge Strategy

```
Dependabot PR detected
    ↓
Extract version change from title
    ↓
┌─────────────────────────────────────┐
│ Security label present?             │
│ YES → Priority: CRITICAL            │
│       Action: Merge immediately     │
│       after checks pass             │
└─────────────────────────────────────┘
    ↓ NO
┌─────────────────────────────────────┐
│ Version change type?                │
├─────────────────────────────────────┤
│ PATCH (1.2.3 → 1.2.4)              │
│   → Risk: LOW                       │
│   → Batch with other patches        │
│   → Merge group together            │
├─────────────────────────────────────┤
│ MINOR (1.2.3 → 1.3.0)              │
│   → Risk: MEDIUM                    │
│   → Check release notes             │
│   → Test individually               │
│   → Merge if checks pass            │
├─────────────────────────────────────┤
│ MAJOR (1.2.3 → 2.0.0)              │
│   → Risk: HIGH                      │
│   → Read CHANGELOG/migration guide  │
│   → Check for breaking changes      │
│   → Flag for manual review          │
│   → May require code updates        │
└─────────────────────────────────────┘
    ↓
Check PR status
    ↓
┌─────────────────────────────────────┐
│ Conflicts?                          │
│ YES → Rebase on main                │
│       Regenerate lock files         │
│       Or: @dependabot rebase        │
└─────────────────────────────────────┘
    ↓ NO
┌─────────────────────────────────────┐
│ Checks passing?                     │
│ YES → Proceed to merge              │
│ NO  → Investigate failure           │
│       Fix if possible               │
│       Or: Close and flag            │
└─────────────────────────────────────┘
    ↓
Execute merge
    ↓
Verify merged successfully
    ↓
Delete branch
```

### 3. Escalation Criteria

**When to Stop and Request Manual Intervention:**

```
Issue encountered
    ↓
Attempt automated fix
    ↓
┌─────────────────────────────────────────────┐
│ Escalate to human if:                       │
├─────────────────────────────────────────────┤
│ ❌ Fix requires architectural decision      │
│ ❌ Multiple failed attempts (>3)            │
│ ❌ Breaking changes in major version update │
│ ❌ Security implications unclear            │
│ ❌ Requires external service access         │
│ ❌ Merge would overwrite others' work       │
│ ❌ Repository protection rules block action │
│ ❌ Insufficient permissions                 │
│ ❌ Ambiguous requirements                   │
│ ❌ Would require force-push to shared branch│
└─────────────────────────────────────────────┘
    ↓
Document findings
    ↓
Create issue with:
  - Problem description
  - Attempted solutions
  - Logs/error messages
  - Recommended next steps
    ↓
Notify team
```

---

## Safety Guidelines

### Actions Requiring Confirmation

**ALWAYS ask before:**

- Force-pushing to shared branches (main, develop, release/*)
- Deleting branches that aren't merged
- Merging PRs without passing checks
- Closing PRs without explanation
- Making changes to workflow files that affect required checks
- Merging major version dependency updates
- Rebasing PRs with many commits
- Executing bulk operations (>10 PRs/issues)

**NEVER do without explicit permission:**

- Force-push to main/master branch
- Disable required status checks
- Bypass branch protection rules
- Delete the repository
- Revoke access tokens
- Modify GitHub Actions secrets
- Change repository settings

### Rollback Procedures

**If a fix causes new failures:**

```bash
# 1. Identify problematic commit
git log --oneline -n 5

# 2. Revert the commit
git revert <commit-sha>
git push

# 3. Or reset if not pushed
git reset --hard HEAD~1
git push --force-with-lease  # Only on feature branches!

# 4. Comment on PR/issue
gh pr comment <pr-number> --body "Rolled back changes from commit <sha> due to <reason>.
New failures: <description>
Investigating alternative approach."

# 5. Trigger fresh workflow run
gh run rerun <run-id>
```

**If merge causes problems:**

```bash
# 1. Identify merge commit
git log --merges -n 5

# 2. Revert merge (if pushed to main)
git revert -m 1 <merge-commit-sha>
git push

# 3. Document rollback
gh issue create --title "Rollback: <PR title>" \
  --body "Reverted PR #<number> due to:
  - <issue-1>
  - <issue-2>

Original PR will be updated and re-merged after fixes."
```

### What NOT to Automate

**Manual review required for:**

- Major version bumps with breaking changes
- Security vulnerability fixes that change APIs
- Changes to authentication/authorization logic
- Database migrations
- Infrastructure as Code changes (Terraform, CloudFormation)
- Changes to CI/CD pipelines that affect deployment
- Dependency updates in production-critical services
- Modifications to .github/workflows files that affect required checks
- PRs from external contributors in public repos

**Criteria for manual review:**

- Business logic changes
- Performance-critical code
- Customer-facing features
- Regulatory compliance requirements
- Multi-service coordination needed
- Requires domain expertise

---

## Examples

### Example 1: Fix Failing Test in CI

**Scenario**: Go test suite fails on main branch

```bash
# 1. Identify failure
$ gh run list --status failure --limit 5
✗  CI  feat: add new API endpoint  main  1234567  about 5 minutes ago

# 2. View failure logs
$ gh run view 1234567 --log-failed
Test:  TestAPIEndpoint/POST_request
Error: Expected status 200, got 500
FAIL   github.com/example/repo/internal/api

# 3. Checkout main branch
$ git checkout main
$ git pull

# 4. Read the failing test
$ cat internal/api/endpoint_test.go
# [Analyze test expectations]

# 5. Read the implementation
$ cat internal/api/endpoint.go
# [Identify bug: missing error handling]

# 6. Fix the issue
$ cat > internal/api/endpoint.go << 'EOF'
func HandlePost(w http.ResponseWriter, r *http.Request) {
    var data RequestData
    if err := json.NewDecoder(r.Body).Decode(&data); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return  // FIX: Was missing return, causing 500 error
    }
    // ... rest of handler
}
EOF

# 7. Verify locally
$ go test ./internal/api/...
PASS

# 8. Commit and push
$ git add internal/api/endpoint.go
$ git commit -m "fix: add missing return after error in POST handler

Fixes GitHub Actions run #1234567
Root cause: Missing return statement after error response
caused subsequent code to execute and panic.

Changes:
- Added return after BadRequest error in HandlePost
- Verified with: go test ./internal/api/..."

$ git push origin main

# 9. Verify CI passes
$ gh run watch
✓  CI  fix: add missing return after error in POST handler  main  1234568  passed
```

### Example 2: Resolve Dependabot Conflicts

**Scenario**: Multiple Dependabot PRs have merge conflicts

```bash
# 1. List Dependabot PRs
$ gh pr list --author "app/dependabot"
#42  Bump golang from 1.23-alpine to 1.25-alpine
#41  Bump github.com/stretchr/testify from 1.8.0 to 1.9.0
#40  Bump golang.org/x/oauth2 from 0.15.0 to 0.16.0

# 2. Check for conflicts
$ gh pr view 42 --json mergeable
{"mergeable": "CONFLICTING"}

# 3. Checkout PR
$ gh pr checkout 42
remote: Enumerating objects: 5, done.
Switched to branch 'dependabot/docker/golang-1.25-alpine'

# 4. Rebase on main
$ git fetch origin main
$ git rebase origin/main
Auto-merging go.mod
CONFLICT (content): Merge conflict in go.mod

# 5. Resolve conflict in go.mod
$ cat go.mod
<<<<<<< HEAD
go 1.23
=======
go 1.25
>>>>>>> Bump golang from 1.23-alpine to 1.25-alpine

# Accept Dependabot's version
$ cat > go.mod << 'EOF'
module github.com/example/repo

go 1.25

require (
    // ... dependencies
)
EOF

# 6. Regenerate go.sum
$ go mod tidy

# 7. Verify build
$ make build
$ make test

# 8. Complete rebase
$ git add go.mod go.sum
$ git rebase --continue
$ git push --force-with-lease

# 9. Verify checks pass
$ gh pr checks 42
✓  CI - Build and Test
✓  CodeQL
✓  Dependency Review

# 10. Enable auto-merge
$ gh pr merge 42 --auto --squash
Pull request #42 will be automatically merged when all requirements are met

# 11. Repeat for other Dependabot PRs
```

### Example 3: Batch Merge Compatible Updates

**Scenario**: Merge multiple safe Dependabot patch updates

```bash
# 1. Identify patch updates
$ gh pr list --author "app/dependabot" --json number,title | \
  jq -r '.[] | select(.title | contains("Bump @types")) | "\(.number): \(.title)"'

45: Bump @types/node from 20.10.0 to 20.10.5
44: Bump @types/react from 18.2.0 to 18.2.4
43: Bump @types/jest from 29.5.0 to 29.5.1

# 2. Verify all checks passing
$ for pr in 45 44 43; do
    echo "PR #$pr:"
    gh pr checks $pr --json state,conclusion | \
      jq -r '.[] | "\(.state): \(.conclusion)"'
done

PR #45: COMPLETED: SUCCESS
PR #44: COMPLETED: SUCCESS
PR #43: COMPLETED: SUCCESS

# 3. Check for conflicts
$ for pr in 45 44 43; do
    mergeable=$(gh pr view $pr --json mergeable -q .mergeable)
    echo "PR #$pr: $mergeable"
done

PR #45: MERGEABLE
PR #44: MERGEABLE
PR #43: MERGEABLE

# 4. Auto-merge all
$ for pr in 45 44 43; do
    gh pr merge $pr --auto --squash --delete-branch
    echo "✓ PR #$pr queued for auto-merge"
done

✓ PR #45 queued for auto-merge
✓ PR #44 queued for auto-merge
✓ PR #43 queued for auto-merge

# 5. Monitor merge progress
$ watch -n 5 'gh pr list --author "app/dependabot" --json number,state,merged'

# 6. Verify all merged
$ gh pr list --author "app/dependabot" --state merged --limit 5
✓  #45  Bump @types/node from 20.10.0 to 20.10.5       (merged 2m ago)
✓  #44  Bump @types/react from 18.2.0 to 18.2.4        (merged 3m ago)
✓  #43  Bump @types/jest from 29.5.0 to 29.5.1         (merged 4m ago)
```

### Example 4: Comprehensive Repository Triage

**Scenario**: Clean up accumulated technical debt

```bash
# 1. Create triage issue
$ gh issue create --title "Repository Maintenance - $(date +%Y-%m-%d)" --body "
## GitHub Maintenance Triage

**Created**: $(date)
**Status**: In Progress

### Failed Workflows
$(gh run list --status failure --limit 10 --json name,conclusion,createdAt | \
  jq -r '.[] | "- [ ] \(.name) - \(.conclusion) (\(.createdAt))"')

### Failed PRs
$(gh pr list --search "status:failure" --json number,title | \
  jq -r '.[] | "- [ ] #\(.number): \(.title)"')

### Dependabot Backlog
$(gh pr list --author "app/dependabot" --json number,title | \
  jq -r '.[] | "- [ ] #\(.number): \(.title)"')
"

Created issue #123

# 2. Prioritize issues
# P0: Main branch CI failure (#1234567)
# P1: Feature PR blocked (#87)
# P2: 12 Dependabot PRs

# 3. Fix P0: Main branch CI
$ gh run view 1234567 --log-failed > failure.log
$ cat failure.log | grep -A 5 "FAILED"
# [Implement fix as in Example 1]

$ gh issue comment 123 --body "✅ **P0 Fixed**: Main branch CI
- Root cause: Test timeout
- Fix: Increased timeout, optimized test setup
- Verification: Run #1234580 passed"

# 4. Fix P1: Feature PR
$ gh pr checkout 87
$ gh pr checks 87
# [Implement fixes as in Example 2]

$ gh issue comment 123 --body "✅ **P1 Fixed**: Feature PR #87 unblocked
- Fixed linting issues
- Resolved merge conflicts
- All checks passing, ready for review"

# 5. Handle P2: Dependabot backlog
# [Batch process as in Example 3]

$ gh issue comment 123 --body "✅ **P2 Progress**: Dependabot cleanup
- Merged 8 patch updates (auto-merge)
- Merged 3 minor updates after testing
- Flagged 1 major update for manual review (#92)"

# 6. Summary
$ gh issue comment 123 --body "## Triage Complete

**Summary**:
- ✅ 1 critical CI failure resolved
- ✅ 1 feature PR unblocked
- ✅ 11/12 Dependabot PRs merged
- ⏳ 1 major version upgrade awaiting review (#92)

**Time saved**: ~3 hours of manual work
**Next steps**: Review major version upgrade #92"

$ gh issue close 123
```

---

## Error Handling

### Common Failure Modes

#### 1. Authentication Failures

**Error:**
```
gh: authentication failed
```

**Recovery:**
```bash
# Check authentication status
$ gh auth status

# Re-authenticate
$ gh auth login

# Refresh token
$ gh auth refresh
```

#### 2. Permission Denied

**Error:**
```
Resource not accessible by personal access token
```

**Recovery:**
```bash
# Check required scopes
$ gh auth status

# Token needs these scopes:
# - repo (full control)
# - workflow (update GitHub Actions)
# - write:packages (if using container registry)

# Re-authenticate with correct scopes
$ gh auth refresh -h github.com -s repo,workflow
```

#### 3. Workflow Not Found

**Error:**
```
could not find workflow run: 404 Not Found
```

**Recovery:**
```bash
# List recent runs to find correct ID
$ gh run list --limit 20

# Verify run ID exists
$ gh api repos/{owner}/{repo}/actions/runs/{run-id}

# If run too old, it may be archived - use workflow name
$ gh workflow view "CI" --limit 50
```

#### 4. Merge Conflicts

**Error:**
```
merge conflict in <file>
```

**Recovery:**
```bash
# Abort merge
$ git merge --abort

# Or resolve conflicts:
$ git status
# Edit conflicted files, remove markers
$ git add <resolved-files>
$ git commit

# Or use --theirs for lock files
$ git checkout --theirs package-lock.json
$ npm install
$ git add package-lock.json
$ git commit
```

#### 5. Rate Limiting

**Error:**
```
API rate limit exceeded
```

**Recovery:**
```bash
# Check rate limit status
$ gh api rate_limit

# Wait for rate limit reset
$ gh api rate_limit | jq .resources.core.reset

# Or use authenticated requests (higher limit)
$ gh auth login  # Increases limit from 60/hr to 5000/hr
```

### Recovery Strategies

#### Strategy 1: Retry with Backoff

```bash
# For transient failures
retry_count=0
max_retries=3

while [ $retry_count -lt $max_retries ]; do
    if gh run rerun <run-id>; then
        echo "Success!"
        break
    else
        retry_count=$((retry_count + 1))
        wait_time=$((2 ** retry_count))
        echo "Retry $retry_count/$max_retries after ${wait_time}s..."
        sleep $wait_time
    fi
done
```

#### Strategy 2: Fallback to API

```bash
# If gh CLI fails, use direct API
gh api repos/{owner}/{repo}/actions/runs/{run-id}/rerun \
  -X POST \
  --silent
```

#### Strategy 3: Graceful Degradation

```bash
# If automated fix fails, document for manual intervention
if ! make test; then
    gh pr comment <pr-number> --body "⚠️ Automated fix attempted but tests still failing.

**Error:**
\`\`\`
$(make test 2>&1 | tail -20)
\`\`\`

Manual review required. See logs for details."

    exit 1
fi
```

### When to Abort

**Abort execution if:**

- ❌ Exceeded maximum retry attempts (default: 3)
- ❌ Detected circular dependency in fixes
- ❌ Working directory becomes dirty unexpectedly
- ❌ GitHub API returns 403 Forbidden (not just rate limit)
- ❌ Would cause data loss (force-push to protected branch)
- ❌ Cannot verify current state (git status fails)

**Abort procedure:**

```bash
# 1. Stop all operations
# 2. Document state
echo "ABORTED: $(date)" > abort-state.log
git status >> abort-state.log
gh run list --limit 5 >> abort-state.log

# 3. Revert any partial changes
git reset --hard HEAD
git clean -fd

# 4. Create issue with diagnostic info
gh issue create --title "Automated maintenance aborted" \
  --body "$(cat abort-state.log)" \
  --label "automation-failure"

# 5. Notify user
echo "❌ Automation aborted. See issue for details."
```

---

## Best Practices

### Before You Start

1. **Verify setup**:
   ```bash
   gh auth status
   git status
   make test  # Ensure baseline works
   ```

2. **Create tracking issue** for audit trail

3. **Understand repository conventions**:
   - Branch naming (feat/, fix/, chore/)
   - Commit message format
   - PR merge strategy (squash vs merge)

### During Execution

1. **Small, focused commits**: One fix per commit
2. **Descriptive messages**: Include run/PR numbers
3. **Test before push**: Run tests locally when possible
4. **Comment proactively**: Explain automated actions
5. **Monitor progress**: Use `gh run watch` and `gh pr checks`

### After Completion

1. **Verify all changes merged**
2. **Close tracking issues**
3. **Clean up branches**: Delete merged branches
4. **Document patterns**: Note recurring issues for prevention

---

## Skill Activation

To use this skill, simply say:

- "Fix failing GitHub Actions"
- "Clean up Dependabot PRs"
- "Resolve PR #123 check failures"
- "Triage repository issues"

Claude Code will then:
1. Assess the current state using `gh` commands
2. Prioritize issues by severity
3. Systematically fix issues following this skill's workflows
4. Document all actions taken
5. Verify resolutions
6. Provide summary report

**Remember**: This skill automates mechanical tasks but escalates complex decisions to you. You stay in control while saving hours of manual work.
