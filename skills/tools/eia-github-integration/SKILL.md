---
name: eia-github-integration
description: "Use when integrating GitHub Projects. Trigger with GitHub sync, label setup, or PR workflow requests."
license: Apache-2.0
compatibility: Requires GitHub CLI version 2.14 or higher, GitHub account with write permissions to target repositories, and basic Git knowledge. Requires AI Maestro installed.
metadata:
  author: Emasoft
  version: 1.0.0
agent: api-coordinator
context: fork
workflow-instruction: "support"
procedure: "support-skill"
user-invocable: false
---

# GitHub Integration Dispatcher

## Overview

This skill is the **entry point** for all GitHub integration tasks in agent orchestration workflows. It routes you to specialized skills based on your task type. Use this skill to determine which specialized GitHub skill to invoke.

## Prerequisites

Before using any GitHub integration skill, ensure:
1. GitHub CLI version 2.14 or higher is installed (`gh --version`)
2. GitHub CLI is authenticated (`gh auth status`)
3. You have write permissions to the target repository
4. Basic familiarity with Git commands

**First-time setup:**
```bash
# Install GitHub CLI
brew install gh   # macOS
# or see https://cli.github.com/manual/installation for other platforms

# Authenticate
gh auth login

# Verify authentication
gh auth status
```

**For detailed setup instructions**, see [references/prerequisites-and-setup.md](references/prerequisites-and-setup.md).

## Decision Tree: Which Skill to Use?

Use this decision tree to route to the appropriate specialized skill:

### I need to work with Pull Requests

**→ Use `eia-github-pr-workflow`**

This skill covers:
- Creating pull requests linked to issues
- PR status monitoring and CI/CD integration
- Merge strategies (squash, merge commit, rebase)
- Auto-merge configuration
- PR workflow automation

### I need to sync with GitHub Projects V2

**→ Use `eia-github-projects-sync`**

This skill covers:
- Bidirectional synchronization between agent tasks and GitHub Projects V2
- Creating and configuring project boards
- Status column management (Backlog, Todo, In Progress, AI Review, Human Review, Merge/Release, Done, Blocked)
- Custom field configuration (Priority, Due Date, Effort)
- Automation rules (auto-add, auto-archive, status transitions)
- Conflict resolution and sync health monitoring

### I need to orchestrate Kanban board operations

**→ Use `eia-kanban-orchestration`**

This skill covers:
- Managing the 9-label classification system (feature, bug, refactor, test, docs, performance, security, dependencies, workflow)
- Issue lifecycle management across Kanban columns
- Label-based filtering and reporting
- Kanban-specific automation patterns

### I need to use Git worktrees for parallel work

**→ Use `eia-git-worktree-operations`**

This skill covers:
- Creating and managing Git worktrees for parallel feature development
- Worktree-based PR workflows
- Cleanup and maintenance of worktrees

### I need to perform GitHub API operations

**→ See [references/api-operations.md](references/api-operations.md)**

This reference covers:
- Direct GitHub API calls (REST and GraphQL)
- Authentication methods (token, app, OAuth)
- Rate limiting and pagination
- Webhook configuration
- Advanced query patterns

### I need to manage multiple GitHub identities

**→ See [references/multi-user-workflow.md](references/multi-user-workflow.md)**

This reference covers:
- SSH key setup for multiple accounts
- SSH host aliases configuration
- GitHub CLI multi-account authentication
- Identity switching and repository configuration
- Using the `gh_multiuser.py` script for automated identity management

## Batch Operations (Unique to This Skill)

When you need to perform operations that span multiple GitHub areas (e.g., bulk label changes across issues AND PRs, or cross-project synchronization):

### Batch Label Operations

**Reference:** [references/batch-operations.md](references/batch-operations.md)

Use when:
- Updating labels on multiple issues simultaneously
- Bulk closing stale issues
- Filtering by multiple criteria (label + status + assignee + date)
- Previewing changes before executing (dry-run mode)
- Creating audit trails for batch operations

**Quick example:**
```bash
# Bulk add label to all open issues with "feature" label
gh issue list --label "feature" --state open --json number --jq '.[].number' | \
  xargs -I {} gh issue edit {} --add-label "priority:high"
```

### Automation Scripts

**Reference:** [references/automation-scripts.md](references/automation-scripts.md)

Use when:
- Syncing GitHub Projects V2 with agent tasks (`sync-projects-v2.py`)
- Bulk assigning labels at scale (`bulk-label-assignment.py`)
- Monitoring PR status and CI/CD failures (`monitor-pull-requests.py`)
- Importing issues from CSV/JSON (`bulk-create-issues.py`)
- Generating project status reports (`generate-project-report.py`)

## Instructions

1. Verify that GitHub CLI version 2.14 or higher is installed by running `gh --version` in your terminal.
2. Confirm authentication status with `gh auth status`. If not authenticated, run `gh auth login` and follow the prompts.
3. Identify your task type by consulting the **Decision Tree** section above (Pull Requests, Projects V2, Kanban, Worktrees, API Operations, or Multi-User).
4. Navigate to the specialized skill indicated by the decision tree (e.g., `eia-github-pr-workflow` for PR tasks).
5. If your task spans multiple areas (e.g., bulk label changes across issues and PRs), use the **Batch Operations** section in this skill directly.
6. For batch operations, always run a dry-run first by previewing the affected items with `gh issue list` or `gh pr list` before executing changes.
7. After completing your GitHub operation, verify the result by checking the repository state (e.g., `gh issue view <number>`, `gh pr status`).
8. If errors occur, consult the **Error Handling** section below or the detailed [references/troubleshooting.md](references/troubleshooting.md).

### Checklist

Copy this checklist and track your progress:

- [ ] Verify GitHub CLI version 2.14+ is installed: `gh --version`
- [ ] Confirm authentication: `gh auth status`
- [ ] Identify task type from the Decision Tree (PR, Projects V2, Kanban, Worktrees, API, Multi-User)
- [ ] Navigate to the specialized skill or use Batch Operations if task spans multiple areas
- [ ] For batch operations: run dry-run preview first (`gh issue list` or `gh pr list`)
- [ ] Execute the GitHub operation using the appropriate skill or batch command
- [ ] Verify the result by checking repository state (`gh issue view`, `gh pr status`)
- [ ] If errors occurred, consult Error Handling section or [references/troubleshooting.md](references/troubleshooting.md)

## Output

This skill produces the following outputs depending on the operation performed:

- **Routing decision**: The name of the specialized skill to invoke (e.g., `eia-github-pr-workflow`, `eia-github-projects-sync`).
- **Batch operation results**: A summary of affected items (issue numbers, PR numbers) and the changes applied (labels added/removed, statuses changed).
- **Automation script output**: JSON or plain-text reports from the Python automation scripts (e.g., `generate-project-report.py` produces a Markdown status report).
- **Verification output**: Confirmation messages from `gh` CLI commands showing the current state of issues, PRs, or project boards after modifications.
- **Dry-run previews**: Lists of items that would be affected by a batch operation, displayed before execution for review.

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `gh: command not found` | GitHub CLI is not installed or not in PATH | Install with `brew install gh` (macOS) or see [references/prerequisites-and-setup.md](references/prerequisites-and-setup.md) |
| `HTTP 401 - Bad credentials` | Authentication token expired or revoked | Re-authenticate with `gh auth login` and verify with `gh auth status` |
| `HTTP 403 - Resource not accessible` | Insufficient permissions on the target repository | Request write access from the repository owner, or check that your token has the required scopes (`repo`, `project`) |
| `HTTP 422 - Validation Failed` | Invalid field values (e.g., non-existent label name, malformed project field) | Verify the label exists with `gh label list` or check project field names with `gh project field-list` |
| `API rate limit exceeded` | Too many API calls in a short period | Wait for the rate limit reset (check `gh api rate-limit`), or use GraphQL to batch multiple queries into one request |
| `Could not resolve to a Project` | Wrong project number or the project is in a different organization | Verify the project number with `gh project list --owner <org>` and ensure you are targeting the correct owner |
| `xargs: gh: terminated by signal 13` | Pipe broken during batch operation, often due to API errors mid-stream | Re-run the batch operation with smaller batch sizes or add error handling with `xargs -I {} sh -c 'gh issue edit {} --add-label "label" || true'` |

## Examples

**Example 1: Route to the correct skill for a PR task**

You receive a request: "Create a PR for the feature branch and link it to issue #42."
This is a Pull Request task. According to the decision tree, invoke `eia-github-pr-workflow`:
```bash
# Switch to eia-github-pr-workflow skill, then run:
gh pr create --base main --head feature-branch --title "Implement feature X" \
  --body "Closes #42" --assignee "@me"
```

**Example 2: Bulk add a label to all open bug issues**

You need to add the `priority:critical` label to all issues labeled `bug` that are currently open:
```bash
# Step 1: Preview affected issues (dry-run)
gh issue list --label "bug" --state open --json number,title --jq '.[] | "\(.number): \(.title)"'

# Step 2: Apply the label change
gh issue list --label "bug" --state open --json number --jq '.[].number' | \
  xargs -I {} gh issue edit {} --add-label "priority:critical"

# Step 3: Verify a sample issue
gh issue view 15 --json labels --jq '.labels[].name'
```

**Example 3: Check which skill to use for a Projects V2 sync request**

You receive a request: "Sync the agent task board with the GitHub Project for repository X."
This is a GitHub Projects V2 synchronization task. According to the decision tree, invoke `eia-github-projects-sync`:
```bash
# Verify the project exists first
gh project list --owner Emasoft --format json

# Then follow the eia-github-projects-sync skill instructions for bidirectional sync
uv run python scripts/sync-projects-v2.py --repo Emasoft/repo-x --project 3 --direction bidirectional
```

## Troubleshooting

If you encounter issues with any GitHub integration task, see [references/troubleshooting.md](references/troubleshooting.md) for:
- Authentication failures and re-authentication
- Projects V2 synchronization issues
- Pull request linking problems
- Label system issues
- GitHub CLI installation and PATH issues
- API rate limiting
- Webhook delivery failures

## Resources

**Core References:**
- [references/prerequisites-and-setup.md](references/prerequisites-and-setup.md) - GitHub CLI installation and authentication
- [references/multi-user-workflow.md](references/multi-user-workflow.md) - Managing multiple GitHub identities
- [references/api-operations.md](references/api-operations.md) - Direct API operations
- [references/batch-operations.md](references/batch-operations.md) - Bulk operations and filtering
- [references/automation-scripts.md](references/automation-scripts.md) - Python automation scripts
- [references/troubleshooting.md](references/troubleshooting.md) - Common issues and solutions

**Specialized Skills:**
- `eia-github-pr-workflow` - Pull request operations
- `eia-github-projects-sync` - Projects V2 bidirectional synchronization
- `eia-kanban-orchestration` - Kanban board and 9-label system management
- `eia-git-worktree-operations` - Git worktree management

---

**Skill Version:** 2.0.0
**Last Updated:** 2026-02-05
**Maintainer:** Skill Development Team
**Changelog:**
- 2.0.0: Refactored as thin dispatcher to specialized skills, removed duplicated content
- 1.2.0: Added cross-platform `gh_multiuser.py` script with configuration-driven identity management
- 1.1.0: Added Multi-User Workflow reference for owner/developer identity separation
