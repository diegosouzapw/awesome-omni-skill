---
name: "GitHub Actions"
description: "Detect GitHub repositories, check GitHub Actions status, find workflow runs by commit/branch/PR, download and analyze CI logs, show workflow status and timing. Use when user asks about CI failures, workflow logs, Actions status, pipeline issues, or needs to troubleshoot failed builds."
model: haiku
allowed-tools:
  - Bash(gh:*)
  - Bash(git:*)
  - Bash(rg:*)
  - Read
  - Grep
---

# GitHub Actions Troubleshooting Skill

This skill helps analyze and troubleshoot GitHub Actions workflows in the current repository.

## When to Use This Skill

- User asks about CI/CD failures or build errors
- User mentions "GitHub Actions", "workflow", "pipeline", or "CI logs"
- User wants to see status of recent workflow runs
- User needs to troubleshoot a failed commit or pull request
- User asks about a specific workflow run

## Prerequisites

Check for `gh` CLI availability:
```bash
which gh
```

If `gh` is not available, inform the user that the GitHub CLI (`gh`) is required and provide installation instructions for their platform.

### Authentication and Access

Before proceeding, verify that `gh` is authenticated **with the correct account** that has access to the repository:

```bash
# Check current authentication status
gh auth status

# Verify which account is active
gh api user --jq '.login'
```

**IMPORTANT**: If the repository is in an organization (e.g., `organization/repo`), ensure the authenticated account has access to that organization.

#### Interactive Account Switching

The `check_gh_cli()` function automatically validates repo access and, when running in an interactive terminal:
- Detects if current account lacks access to the repository
- Lists all available authenticated accounts
- Prompts you to select and switch to the correct account
- Verifies the selected account has access
- Automatically proceeds if access is granted

**Example interactive session:**
```
Current account 'personal-user' cannot access 'company/private-repo'

Available accounts:
 1. personal-user
 2. work-user

Select an account to switch to (1-2, or 'n' to skip): 2
Switching to account: work-user
Successfully switched to work-user
✓ Account work-user has access to company/private-repo
```

In non-interactive environments (scripts, CI/CD), the function will display an error message with manual instructions instead of prompting.

Common authentication issues:
- **Wrong account**: Authenticated with personal account but repo is in organization
- **Multiple accounts**: Need to switch to the right one using `gh auth switch`
- **Missing permissions**: Account lacks access to private repo or organization

## Step 1: Detect GitHub Repository

Check if the current directory is a GitHub repository:

```bash
git remote get-url origin 2>/dev/null | grep -q github.com
```

If this fails or returns non-GitHub URL, inform user this is not a GitHub repository.

Extract owner and repo name:
```bash
git remote get-url origin | sed -E 's#.*github\.com[:/]([^/]+)/([^.]+)(\.git)?#\1/\2#'
```

## Step 2: Check if GitHub Actions is Enabled

Two methods to verify Actions is configured:

**Method 1: Check for workflow files**
```bash
ls -la .github/workflows/
```

**Method 2: Query GitHub API**
```bash
gh api repos/:owner/:repo/actions/workflows --jq '.total_count'
```

If no workflows exist, inform user that GitHub Actions is not configured for this repository.

## Step 3: Finding Workflow Runs

### By Commit SHA
When user mentions a specific commit or references HEAD:
```bash
# Get commit SHA if needed
COMMIT_SHA=$(git rev-parse HEAD)

# Find runs for that commit
gh run list --commit $COMMIT_SHA --json databaseId,status,conclusion,workflowName,headBranch,createdAt --limit 10
```

### By Branch
```bash
gh run list --branch <branch-name> --json databaseId,status,conclusion,workflowName,createdAt --limit 10
```

### Recent Failures
```bash
gh run list --status failure --json databaseId,status,conclusion,workflowName,headBranch,createdAt --limit 5
```

### All Recent Runs
```bash
gh run list --limit 20 --json databaseId,status,conclusion,workflowName,headBranch,createdAt
```

## Step 4: Viewing Workflow Run Details

### Summary View
```bash
gh run view <run-id> --verbose
```

This shows:
- Workflow name and status
- Triggered by and event
- All jobs with their status
- Job steps when using --verbose

### Check Status Only
```bash
gh run view <run-id> --json status,conclusion,workflowName,headBranch --jq '.'
```

## Step 5: Analyzing Logs

### Failed Steps Only (Recommended First)
```bash
gh run view <run-id> --log-failed
```

This shows only the logs for steps that failed, making it easier to identify issues.

### Full Logs
```bash
gh run view <run-id> --log
```

### Specific Job Logs
```bash
# First, list jobs to get job ID
gh run view <run-id> --json jobs --jq '.jobs[] | {id: .databaseId, name: .name, status: .status, conclusion: .conclusion}'

# Then view specific job
gh run view <run-id> --job <job-id> --log
```

## Step 6: Common Troubleshooting Patterns

### Pattern: Recent push failed
```bash
# Get the last commit SHA
COMMIT_SHA=$(git rev-parse HEAD)

# Find runs for that commit
RUNS=$(gh run list --commit $COMMIT_SHA --json databaseId,status,conclusion,workflowName)

# If any failed, get the run ID and show failed logs
RUN_ID=$(echo "$RUNS" | jq -r 'first(.[] | select(.conclusion == "failure")) | .databaseId')

if [ -n "$RUN_ID" ]; then
  echo "Found failed run: $RUN_ID"
  gh run view $RUN_ID --log-failed
fi
```

### Pattern: Check CI status before merging
```bash
# Get current branch
BRANCH=$(git branch --show-current)

# Show recent runs on this branch
gh run list --branch $BRANCH --limit 5 --json databaseId,status,conclusion,workflowName,createdAt
```

### Pattern: Compare with successful runs
```bash
# Find last successful run of a workflow
gh run list --workflow <workflow-name> --status success --limit 1 --json databaseId,headSha

# Find failed runs
gh run list --workflow <workflow-name> --status failure --limit 5 --json databaseId,headSha,createdAt
```

## Helper Script

Use the helper script in `scripts/gh_actions_helper.sh` for common operations:

```bash
source "$(dirname "$0")/scripts/gh_actions_helper.sh"

# Check if in GitHub repo with Actions
check_github_actions_repo

# Get latest run for current commit
get_latest_run_for_commit "$(git rev-parse HEAD)"

# Analyze common failure patterns in logs
analyze_failure_logs "$RUN_ID"
```

## Error Handling

### `gh` not authenticated
```bash
gh auth status
```

If not authenticated:
```bash
gh auth login
```

### Wrong account or insufficient access

If you see errors like `HTTP 404: Not Found` when accessing organization repositories:

```bash
# Check which account is currently active
gh auth status
gh api user --jq '.login'

# List all authenticated accounts
gh auth status --show-token=false

# Switch to a different account
gh auth switch

# Or login with the correct account
gh auth login
```

The `check_gh_cli()` helper function will detect this automatically and provide specific guidance about which account you're using and what's needed.

### Rate limiting
GitHub API has rate limits. Check status:
```bash
gh api rate_limit
```

### Private repositories
Ensure `gh` has appropriate permissions:
```bash
gh auth refresh -s read:org,repo
```

## Output to User

Always provide:
1. **Context**: What workflow/job failed
2. **Status**: Current state (failed, in_progress, etc.)
3. **Key errors**: Extract relevant error messages from logs
4. **Actionable next steps**: What to fix or investigate

Example response format:
```
Workflow "CI" failed on commit abc123 (5 minutes ago)

Failed job: "test"
Failed step: "Run tests"

Error found:
  ERROR: Test suite failed
  FAILED tests/test_api.py::test_endpoint - AssertionError

Next steps:
- Run `pytest tests/test_api.py::test_endpoint` locally
- Check recent changes to test_api.py or endpoint logic
```
