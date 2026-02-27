---
name: github-issues
description: Use when creating, updating, or managing GitHub issues. Ensures consistent format, labeling, and conventions for issue tracking.
---

# GitHub Issues Conventions

## Core Principle
**"Issues describe problems, commits describe solutions."**

Write clear, actionable issues that anyone can pick up and understand. The issue title describes what's wrong or needed; labels and types handle categorization.

## When to Use

**Always use for:**
- Creating new GitHub issues
- Reviewing issue quality before submission
- Planning work breakdown into trackable items
- Writing bug reports or feature requests

**Skip for:**
- Internal notes or comments
- Commit messages (use conventional commits instead)
- Quick discussions that don't need tracking

## Issue Title Guidelines

**Format:** Plain, descriptive sentence case. No prefixes, no types, no emojis.

The title should describe the **problem or need**, not the solution. GitHub's issue types and labels handle categorization.

### Good Titles

| Title | Why it works |
|-------|--------------|
| Login fails with special characters in password | Describes the problem clearly |
| Add team invitation email notifications | States the need directly |
| Expiry dates display in wrong timezone | Specific, observable behavior |
| Dashboard loads slowly with many secrets | Clear performance issue |
| Missing validation on API key name field | Identifies the gap |

### Bad Titles

| Title | Problem | Better Version |
|-------|---------|----------------|
| Fix login bug | Too vague | Login fails with special characters in password |
| feat: Add email notifications | Has prefix (use labels) | Add team invitation email notifications |
| [BUG] Timezone issue | Bracket prefix | Expiry dates display in wrong timezone |
| Performance improvements | No specifics | Dashboard loads slowly with many secrets |
| Update validation | What validation? Where? | Missing validation on API key name field |

## Issue Types

Use GitHub's native issue type feature or labels to categorize:

| Type | When to Use | Example |
|------|-------------|---------|
| **Bug** | Something isn't working as expected | Login fails with valid credentials |
| **Feature** | New functionality that doesn't exist | Add keyboard shortcuts for common actions |
| **Task** | Implementation work item | Set up CI/CD pipeline for auth service |
| **Documentation** | Docs improvements or additions | Document API rate limiting behavior |
| **Enhancement** | Improvement to existing functionality | Improve secret search performance |

## Issue Body Templates

### Bug Report

```markdown
## Description
[Brief description of the bug]

## Steps to Reproduce
1. [First step]
2. [Second step]
3. [Third step]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Environment
- Browser: [e.g., Chrome 120]
- OS: [e.g., macOS 14.2]
- Service: [e.g., auth-service, frontend]

## Additional Context
[Screenshots, error messages, relevant logs]
```

### Feature Request

```markdown
## Problem Statement
[What problem does this solve? Why is it needed?]

## Proposed Solution
[High-level description of the feature]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Alternatives Considered
[Other approaches you thought about]

## Additional Context
[Mockups, examples from other products, related issues]
```

### Task

```markdown
## Context
[Background information and why this task exists]

## Requirements
[What needs to be done]

- [ ] [Requirement 1]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

## Acceptance Criteria
- [ ] [How we know it's done - criterion 1]
- [ ] [How we know it's done - criterion 2]

## Technical Notes
[Implementation hints, constraints, related code]

## Related Issues
- #[related issue number]
```

## Labels Convention

Use consistent labels across the KeyArc project for filtering and organization.

### Priority Labels

| Label | When to Use |
|-------|-------------|
| `priority:critical` | Production is broken, security vulnerability, data loss risk |
| `priority:high` | Major feature blocked, significant user impact |
| `priority:medium` | Important but not urgent, default for most work |
| `priority:low` | Nice to have, minor improvements, can wait |

### Service Labels

| Label | Scope |
|-------|-------|
| `service:auth` | Auth service (signup, login, tokens) |
| `service:gateway` | API gateway (routing, JWT validation) |
| `service:keys` | Key service (secrets, folders, sharing) |
| `service:account` | Account service (profiles, teams, members) |
| `service:frontend` | Angular frontend application |

### Status Labels

| Label | Meaning |
|-------|---------|
| `status:ready` | Issue is fully specified, ready to work on |
| `status:blocked` | Waiting on external dependency or decision |
| `status:needs-info` | Requires more information before proceeding |
| `status:in-progress` | Someone is actively working on this |

### Scope Labels

| Label | Domain |
|-------|--------|
| `scope:security` | Security-related work (encryption, auth, audit) |
| `scope:api` | API design, endpoints, contracts |
| `scope:ui` | User interface, UX, visual design |
| `scope:database` | Database schema, migrations, queries |

## Organizing Work

GitHub provides organizational layers above individual issues. For KeyArc, use milestones and sub-issues to keep work structured without overhead.

### Milestones

Use milestones to group issues toward a release or phase goal.

**KeyArc Milestones:**

| Milestone | Purpose | Example Issues |
|-----------|---------|----------------|
| Phase 1: Auth & Core | Authentication, database, gateway setup | Auth service scaffold, JWT implementation |
| Phase 2: Key Management | Secret storage and organization | Key service endpoints, folder management |
| Phase 3: Team Sharing | Multi-user and team features | Team creation, key sharing, RBAC |
| v1.0 Launch | Production readiness | Performance tuning, security audit, docs |

**When to use milestones:**
- Release planning (what's in v1.0?)
- Phase tracking (what's left for Phase 1?)
- Sprint goals (if using time-boxed iterations)

**Creating a milestone:**
```bash
gh api repos/OWNER/REPO/milestones -f title="Phase 1: Auth & Core" -f description="Authentication service, database setup, and API gateway" -f due_on="2026-03-01T00:00:00Z"
```

**Assigning issues to milestones:**
```bash
gh issue edit 123 --milestone "Phase 1: Auth & Core"
```

### Sub-issues

Use sub-issues to break down larger features or service implementations into trackable pieces.

**Pattern:** Create a parent issue for a major feature or service, then add sub-issues for individual tasks.

**Example: Auth Service Implementation**

```
Parent: Implement Auth Service (#10)
├── Sub: Create auth service project scaffold (#11)
├── Sub: Implement signup endpoint (#12)
├── Sub: Implement login endpoint (#13)
├── Sub: Add JWT token generation (#14)
├── Sub: Add password reset flow (#15)
└── Sub: Write auth service integration tests (#16)
```

**Parent issue template:**

```markdown
## Overview
[High-level description of the feature or service]

## Sub-issues
This work is broken down into the following tasks:
- [ ] #11 - Create auth service project scaffold
- [ ] #12 - Implement signup endpoint
- [ ] #13 - Implement login endpoint
- [ ] #14 - Add JWT token generation
- [ ] #15 - Add password reset flow
- [ ] #16 - Write auth service integration tests

## Acceptance Criteria
- [ ] All sub-issues completed
- [ ] Service deployed to staging
- [ ] Integration tests passing

## Technical Notes
[Architecture decisions, constraints, links to docs]
```

**When to use sub-issues:**
- Service implementation (5+ related tasks)
- Complex features spanning multiple components
- Work that needs to be parallelized across contributors

**When NOT to use sub-issues:**
- Simple features (just use a single issue)
- Unrelated tasks (use labels/milestones instead)
- Bugs (usually standalone)

### GitHub Project Board

KeyArc uses a GitHub Project board to track work status. **Every issue must be added to the project board.**

**Project Reference:**
- Project ID: `PVT_kwDODzuJ-c4BNYGm`
- Status Field ID: `PVTSSF_lADODzuJ-c4BNYGmzg8Y71s`
- Phase Field ID: `PVTSSF_lADODzuJ-c4BNYGmzg8Y8FI`
- Track Field ID: `PVTSSF_lADODzuJ-c4BNYGmzg8Y8FM`
- Project URL: https://github.com/orgs/KeyArc/projects/1

**Field Options:**

| Field | Option | ID |
|-------|--------|-----|
| Status | Ready | `e5dd8d9b` |
| Status | Blocked | `a149a0a4` |
| Status | In Progress | `b3644764` |
| Status | In Review | `9a932cb4` |
| Status | Done | `f3192413` |
| Phase | Phase 1: Foundation | `acaef4ee` |
| Phase | Phase 2: Authentication | `205962fc` |
| Track | Dev | `e916e7bd` |
| Track | DevOps | `8612f72a` |
| Track | QA | `a28706f5` |

### Complete Issue Creation Workflow

When creating a new issue, follow these steps:

**Step 1: Create the issue with milestone and labels**
```bash
gh issue create --repo KeyArc/keyarc \
  --title "Issue title here" \
  --body "Issue body..." \
  --milestone "Phase 1: Foundation" \
  --label "priority:high,track:dev,scope:infra"
```

**Step 2: Add to project board**
```bash
gh project item-add 1 --owner KeyArc --url https://github.com/KeyArc/keyarc/issues/ISSUE_NUMBER
```

**Step 3: Set project fields (status, phase, track)**
```bash
# Get the item ID first
ITEM_ID=$(gh project item-list 1 --owner KeyArc --format json | jq -r '.items[] | select(.content.number == ISSUE_NUMBER) | .id')

# Set status to Ready
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODzuJ-c4BNYGm"
    itemId: "'"$ITEM_ID"'"
    fieldId: "PVTSSF_lADODzuJ-c4BNYGmzg8Y71s"
    value: {singleSelectOptionId: "e5dd8d9b"}
  }) { projectV2Item { id } }
}'

# Set phase (use option ID from Field Options table)
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODzuJ-c4BNYGm"
    itemId: "'"$ITEM_ID"'"
    fieldId: "PVTSSF_lADODzuJ-c4BNYGmzg8Y8FI"
    value: {singleSelectOptionId: "acaef4ee"}
  }) { projectV2Item { id } }
}'

# Set track (use option ID from Field Options table)
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODzuJ-c4BNYGm"
    itemId: "'"$ITEM_ID"'"
    fieldId: "PVTSSF_lADODzuJ-c4BNYGmzg8Y8FM"
    value: {singleSelectOptionId: "e916e7bd"}
  }) { projectV2Item { id } }
}'
```

### Quick Reference Commands

**Add issue to project:**
```bash
gh project item-add 1 --owner KeyArc --url https://github.com/KeyArc/keyarc/issues/NUMBER
```

**Update status to In Progress:**
```bash
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODzuJ-c4BNYGm"
    itemId: "ITEM_ID"
    fieldId: "PVTSSF_lADODzuJ-c4BNYGmzg8Y71s"
    value: {singleSelectOptionId: "b3644764"}
  }) { projectV2Item { id } }
}'
```

**List project items with status:**
```bash
gh project item-list 1 --owner KeyArc --format json | jq '.items[] | {number: .content.number, title: .content.title, status: .status}'
```

### GitHub API Tips

Use `gh api` - handles auth automatically:
```bash
gh api repos/OWNER/REPO/issues -q '.[] | {number, title}'  # GET + jq
gh api repos/OWNER/REPO/issues/10/comments -X POST -f body="text"  # -f for strings
gh api repos/OWNER/REPO/issues/10/dependencies/blocked_by -X POST -F issue_id=123  # -F for integers
```

For project boards, use `gh api graphql -f query='mutation {...}'`.

Auto-unblock workflow: `.github/workflows/auto-unblock-issues.yml` moves issues to "Ready" when blockers close.

### Organizational Hierarchy

```
Milestone (Phase 1: Auth & Core)
├── Parent Issue: Implement Auth Service (#10)
│   ├── Sub-issue: Signup endpoint (#12)
│   ├── Sub-issue: Login endpoint (#13)
│   └── Sub-issue: JWT tokens (#14)
├── Parent Issue: Set up API Gateway (#20)
│   ├── Sub-issue: JWT validation (#21)
│   └── Sub-issue: Route configuration (#22)
└── Standalone Issue: Configure PostgreSQL (#5)
```

This gives you:
- **Milestone view:** "What's left for Phase 1?"
- **Parent issue view:** "What's left for Auth Service?"
- **Individual issues:** Assignable, trackable units of work

## Good vs Bad Examples

### Example 1: Bug Report

**Bad:**
```
Title: Login broken
Body: The login doesn't work sometimes. Please fix.
```

**Good:**
```
Title: Login fails when password contains ampersand character

## Description
Users cannot log in when their password contains an & character.

## Steps to Reproduce
1. Create account with password "Test&123"
2. Log out
3. Attempt to log in with same credentials

## Expected Behavior
Login succeeds with valid credentials

## Actual Behavior
Error: "Invalid credentials" even though password is correct

## Environment
- Browser: Chrome 120, Firefox 121
- OS: macOS 14.2

## Additional Context
Suspect URL encoding issue with special characters in authHash.
```

### Example 2: Feature Request

**Bad:**
```
Title: Add notifications
Body: We need notifications for the app.
```

**Good:**
```
Title: Add email notifications for expiring API keys

## Problem Statement
Users have no way to know when their API keys are about to expire
unless they manually check the dashboard. This leads to service
outages when keys expire unexpectedly.

## Proposed Solution
Send email notifications at configurable intervals before key expiry:
- 30 days before (default)
- 7 days before
- 1 day before

## Acceptance Criteria
- [ ] User can configure notification preferences
- [ ] Emails sent at specified intervals before expiry
- [ ] Email contains key name, expiry date, and link to dashboard
- [ ] Respects user's notification opt-out preference

## Alternatives Considered
- In-app notifications only: Not helpful if user doesn't check app
- Slack integration: Too complex for v1, consider as future enhancement
```

### Example 3: Task

**Bad:**
```
Title: Set up database
Body: Need to set up the database.
```

**Good:**
```
Title: Configure PostgreSQL database with Alembic migrations

## Context
The auth service needs a database to store user credentials and
encrypted vault keys. Using PostgreSQL for consistency with
architecture decisions.

## Requirements
- [ ] Create PostgreSQL instance on Fly.io
- [ ] Configure SQLAlchemy async engine
- [ ] Set up Alembic for migrations
- [ ] Create initial user table migration
- [ ] Add database health check endpoint

## Acceptance Criteria
- [ ] Service connects to database on startup
- [ ] Migrations run successfully
- [ ] Health check returns database status
- [ ] Connection pooling configured appropriately

## Technical Notes
- Use asyncpg driver for async support
- Follow existing SQLAlchemy patterns from src/shared/models/
- Database URL from FLY_PG_PROXY_CONN_STRING

## Related Issues
- #12 (Create auth service scaffold)
```

## Red Flags

Signs of poor issue quality that need revision:

**Title problems:**
- Includes type prefix (feat:, bug:, etc.)
- Uses brackets like [BUG] or [FEATURE]
- Too vague ("Fix issue", "Update code")
- Describes solution instead of problem ("Use async/await")
- Contains emojis

**Body problems:**
- No reproduction steps for bugs
- No acceptance criteria
- Missing context for why this matters
- Wall of text without structure
- No environment details for bugs

**Scope problems:**
- Combines multiple unrelated changes
- Too large to complete in one PR
- Dependencies not identified

## Key Principles

1. **Describe problems, not solutions** - The title should say what's wrong or needed, not how to fix it

2. **One issue, one concern** - Each issue should track a single piece of work

3. **Include reproduction steps** - For bugs, always include steps to reproduce

4. **Define "done"** - Acceptance criteria tell you when the issue is complete

5. **Use labels consistently** - Labels enable filtering and help with planning

6. **Link related issues** - Connect issues that depend on or relate to each other

7. **Keep it actionable** - Someone should be able to pick up the issue and start working

## Integration with Workflow

Issues connect to other parts of the development workflow:

- **Issue** describes the problem/need
- **Branch** named after issue (`feature/123-add-notifications`)
- **Commits** describe solutions (conventional commits: `feat:`, `fix:`)
- **PR** links back to issue (`Closes #123`)
- **Issue closed** when PR merges

This creates full traceability from problem to solution.
