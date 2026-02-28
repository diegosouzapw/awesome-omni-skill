---
name: github-cli
description: GitHub CLI (gh) mastery for issues, PRs, Projects v2, and automation. Modern development workflow patterns for 2025.
version: 1.0.0
author: YG Starter AI Agent Hub
tags: [github, gh, cli, issues, pr, projects, automation, 2025]
---

# GitHub CLI Skill

## Overview

Master the GitHub CLI (`gh`) for comprehensive project management. This skill covers issue creation, PR workflows, Projects v2 integration, and automation patterns using modern best practices (December 2025).

**When to use:**
- Creating/managing GitHub issues and PRs
- Working with GitHub Projects v2 custom fields
- Automating bulk operations with `gh`
- Following modern branch and PR conventions
- Running GraphQL queries for complex operations

---

## Quick Reference

### Essential Commands (2025)

```bash
# Issue operations
gh issue create --title "feat: Add payment processing" --body "..." --label "enhancement" --milestone "v2.0"
gh issue list --state open --label "backend" --assignee @me --limit 20
gh issue edit 123 --add-label "priority:high" --milestone "v2.1"
gh issue close 123 --comment "Fixed in #456"

# PR operations
gh pr create --title "feat: Add Stripe integration" --body "..." --base main --reviewer @teammate
gh pr checks 456 --watch              # Watch CI status live
gh pr merge 456 --squash --delete-branch
gh pr merge 456 --auto --squash       # Auto-merge when CI passes + approved
gh pr review 456 --approve --body "LGTM!"

# Project operations (Projects v2)
gh project list --owner @me
gh project item-add 1 --owner @me --url https://github.com/org/repo/issues/123
gh project field-list 1 --owner @me

# API operations
gh api repos/:owner/:repo/issues --jq '.[].title'
gh api graphql -f query='query { viewer { login }}'
gh api --method POST repos/:owner/:repo/dispatches -f event_type=deploy
```

### JSON Output + jq Patterns (2025)

```bash
# Get issue numbers matching criteria
gh issue list --json number,labels --jq '[.[] | select(.labels[].name == "bug")] | .[].number'

# PR summary with checks
gh pr list --json number,title,author,statusCheckRollup --jq '.[] | "\(.number): \(.title) - \(.statusCheckRollup)"'

# Count open PRs by author
gh pr list --json author --jq 'group_by(.author.login) | map({author: .[0].author.login, count: length})'

# List stale issues (no activity in 30 days)
gh issue list --json number,title,updatedAt --jq 'map(select(.updatedAt < (now - 2592000 | strftime("%Y-%m-%dT%H:%M:%SZ")))) | .[] | .number'
```

---

## Modern Workflow (2025)

### Branch Naming Convention

```bash
# For GitHub issues
issue/<number>-<brief-description>
# Examples:
issue/372-stripe-integration
issue/385-user-authentication

# For features without issues
feature/<description>
# Example:
feature/dark-mode

# For bug fixes
fix/<issue-number>-<description>
# Example:
fix/401-payment-timeout

# For hotfixes
hotfix/<description>
# Example:
hotfix/critical-security-patch
```

### Complete Feature Workflow

```bash
# 1. Create issue (if not exists)
ISSUE_URL=$(gh issue create \
  --title "feat: Add Stripe payment processing" \
  --body "$(cat <<'EOF'
## Description
Implement Stripe payment processing for subscription billing.

## Acceptance Criteria
- [ ] Stripe SDK integration
- [ ] Payment intent creation endpoint
- [ ] Webhook handling for payment events
- [ ] Database schema for transactions

## Technical Notes
- Use Stripe API v2025-01-01
- Implement idempotency keys
- Add retry logic for webhook failures
EOF
)" \
  --label "enhancement,backend,payment" \
  --milestone "Q1 2026 - Payment System" \
  --json url --jq '.url')

ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')

# 2. Create feature branch
git checkout main && git pull origin main
git checkout -b "issue/${ISSUE_NUM}-stripe-integration"

# 3. Do work, commit with conventional commits
git add . && git commit -m "feat(#${ISSUE_NUM}): Add Stripe payment processing with webhook support"

# 4. Push and create PR
git push -u origin "issue/${ISSUE_NUM}-stripe-integration"

gh pr create \
  --title "feat(#${ISSUE_NUM}): Add Stripe payment processing" \
  --body "$(cat <<'EOF'
## Summary
- Integrated Stripe SDK v14.x
- Added payment intent creation endpoint
- Implemented webhook handling for payment.succeeded
- Added transaction schema and models

## Test Plan
- [x] Unit tests for payment service (95% coverage)
- [x] Integration tests with Stripe test mode
- [x] Webhook event replay testing
- [ ] Manual testing in staging environment

## Security Considerations
- Webhook signature verification enabled
- API keys stored in environment variables
- PCI DSS compliance checklist followed

Closes #${ISSUE_NUM}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)" \
  --base main \
  --label "enhancement,backend,payment" \
  --reviewer @tech-lead
```

### PR Commit Message Format (Conventional Commits)

```bash
# Format: type(#issue): description
feat(#372): Implement Stripe payment integration
fix(#345): Resolve checkout page rendering bug
docs(#336): Update API documentation for v2.0
refactor(#391): Extract payment service into separate module
test(#342): Add 95% coverage for payment webhooks
chore(#376): Upgrade dependencies to December 2025 versions
perf(#401): Optimize database queries for user dashboard
```

### Project Board Integration (Projects v2)

Modern development teams use GitHub Projects v2 with custom fields. After creating an issue:

```bash
# Add issue to project
ITEM_ID=$(gh project item-add 1 --owner your-org \
  --url "https://github.com/your-org/your-repo/issues/${ISSUE_NUM}" \
  --format json | jq -r '.id')

# Set Status to "In Development" (requires GraphQL)
# See templates/project-config-example.json for field IDs
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PROJECT_ID"
    itemId: "'$ITEM_ID'"
    fieldId: "STATUS_FIELD_ID"
    value: {singleSelectOptionId: "IN_PROGRESS_OPTION_ID"}
  }) {
    projectV2Item {
      id
    }
  }
}'
```

> **Note:** Setting custom fields requires GraphQL. See `references/projects-v2.md`.

---

## Labels Reference (Example SaaS App)

### Priority Labels
| Label | Description | Color |
|-------|-------------|-------|
| `priority:critical` | Critical blocker, immediate attention | #d73a4a (Red) |
| `priority:high` | High priority, sprint critical | #ff9800 (Orange) |
| `priority:medium` | Medium priority | #ffeb3b (Yellow) |
| `priority:low` | Low priority, nice to have | #03a9f4 (Blue) |

### Domain Labels
| Label | Description | Color |
|-------|-------------|-------|
| `area:backend` | Backend (Python, FastAPI, PostgreSQL) | #0052cc (Blue) |
| `area:frontend` | Frontend (React, TypeScript, Vite) | #9c27b0 (Purple) |
| `area:database` | Database schema, migrations | #795548 (Brown) |
| `area:infrastructure` | DevOps, CI/CD, deployment | #607d8b (Blue Gray) |
| `area:api` | REST API, GraphQL | #00bcd4 (Cyan) |

### Type Labels
| Label | Description | Color |
|-------|-------------|-------|
| `type:feature` | New feature or enhancement | #a2eeef (Light Blue) |
| `type:bug` | Bug fix | #d73a4a (Red) |
| `type:refactor` | Code improvement, no behavior change | #fbca04 (Yellow) |
| `type:docs` | Documentation updates | #0075ca (Blue) |
| `type:test` | Test coverage improvements | #7057ff (Purple) |
| `type:security` | Security vulnerability or patch | #ee0701 (Dark Red) |

---

## Milestones Reference (Example SaaS App)

Example milestone structure for a SaaS application:

| Milestone | Due Date | Focus |
|-----------|----------|-------|
| v2.0 - Payment System | Q1 2026 | Stripe integration, subscriptions |
| v2.1 - User Management | Q2 2026 | SSO, RBAC, team management |
| v2.2 - Analytics Dashboard | Q2 2026 | Usage metrics, insights |
| v3.0 - Enterprise Features | Q3 2026 | Multi-tenancy, audit logs |
| v3.1 - Mobile App | Q4 2026 | iOS/Android native apps |

---

## Detailed Guides

For specific capabilities, see:

- **Issue Management**: `references/issue-management.md`
  - Bulk operations, templates, parent/sub-issues

- **PR Workflows**: `references/pr-workflows.md`
  - Review workflow, merge strategies, auto-merge

- **Projects v2**: `references/projects-v2.md`
  - Custom fields, GraphQL mutations, this project field IDs

- **GraphQL API**: `references/graphql-api.md`
  - Complex queries, pagination, bulk operations

- **Automation Patterns**: `references/automation-patterns.md`
  - Aliases, error handling, rate limits, scripts

---

## Best Practices

1. **Always use `--json` for scripting** - Parse with `--jq` for reliability
2. **Non-interactive mode for automation** - Use `--title`, `--body` flags
3. **Check rate limits before bulk operations** - `gh api rate_limit`
4. **Use heredocs for multi-line content** - `--body "$(cat <<'EOF'...EOF)"`
5. **Link issues in PRs** - `Closes #123`, `Fixes #456`
6. **Add verification checklists** - Track test plan completion
7. **Never commit to dev/main directly** - Always use feature branches + PRs
