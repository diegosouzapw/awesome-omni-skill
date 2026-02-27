---
name: eama-label-taxonomy
description: Use when managing GitHub issue labels for user requests, setting priorities, or reporting status to users. Covers priority and status label taxonomy. Trigger with `/eama-label-taxonomy`.
agent: eama-main
context: fork
user-invocable: false
compatibility: Requires AI Maestro installed.
version: 1.0.0
---

# EAMA Label Taxonomy

## Overview

This skill provides the label taxonomy relevant to the Assistant Manager Agent (EAMA) role. Each role plugin has its own label-taxonomy skill covering the labels that role manages.

## Prerequisites

1. Access to GitHub CLI (`gh`) command
2. Repository with GitHub issue tracking enabled
3. Understanding of EAMA role responsibilities (user communication and role routing)
4. Familiarity with GitHub issue label syntax

## Instructions

Follow these steps to manage labels as EAMA:

1. **Analyze user request** to determine appropriate priority and type labels
2. **Create issue** with initial labels (`status:backlog`, `priority:*`, `type:*`)
3. **Monitor status changes** from other agents and translate to user-friendly messages
4. **Update priorities** when user expresses urgency or changes priority
5. **Set blocked status** when user requests pause (`status:blocked`)
6. **Report status** by querying issues with relevant labels
7. **Explain labels** to user in clear, non-technical language
8. **Facilitate human review** when Integrator (EIA) escalates significant tasks for user testing/review

**Checklist for label management**:

Copy this checklist and track your progress:

- [ ] Determine priority from user request context
- [ ] Apply appropriate type label (bug/feature/etc)
- [ ] Set initial status to `backlog`
- [ ] Monitor for status changes by other agents
- [ ] Update labels when user changes requirements
- [ ] Facilitate human review when `status:human-review` is set
- [ ] Generate user-friendly status reports

---

## Labels EAMA Manages

### Priority Labels (`priority:*`)

**EAMA has authority to set and change priorities based on user input.**

| Label | Description | When EAMA Sets It |
|-------|-------------|-------------------|
| `priority:critical` | Must fix immediately | User reports production issue |
| `priority:high` | High priority | User emphasizes importance |
| `priority:normal` | Standard priority | Default for new issues |
| `priority:low` | Nice to have | User indicates low urgency |

**EAMA Priority Responsibilities:**
- Set initial priority based on user request
- Escalate priority when user expresses urgency
- De-escalate when user indicates reduced urgency

### Kanban Columns (Canonical 8-Column System)

The full workflow uses these 8 status columns:

| # | Column Code | Display Name | Label | Description |
|---|-------------|-------------|-------|-------------|
| 1 | `backlog` | Backlog | `status:backlog` | Entry point for new tasks |
| 2 | `todo` | Todo | `status:todo` | Ready to start |
| 3 | `in-progress` | In Progress | `status:in-progress` | Active work |
| 4 | `ai-review` | AI Review | `status:ai-review` | Integrator agent reviews ALL tasks |
| 5 | `human-review` | Human Review | `status:human-review` | User reviews BIG tasks only (via EAMA) |
| 6 | `merge-release` | Merge/Release | `status:merge-release` | Ready to merge |
| 7 | `done` | Done | `status:done` | Completed |
| 8 | `blocked` | Blocked | `status:blocked` | Blocked at any stage |

**Task Routing Rules:**
- **Small tasks**: In Progress -> AI Review -> Merge/Release -> Done
- **Big tasks**: In Progress -> AI Review -> Human Review -> Merge/Release -> Done
- **Human Review** is requested via EAMA (Assistant Manager asks user to test/review)
- Not all tasks go through Human Review -- only significant changes requiring human judgment

### Status Labels EAMA Updates

| Label | When EAMA Sets It |
|-------|------------------|
| `status:backlog` | When creating new issue from user request |
| `status:human-review` | When EIA escalates a significant task for user review |
| `status:blocked` | When user requests pause or issue cannot proceed |

---

## Labels EAMA Monitors

### Status Labels (`status:*`)

EAMA reports status to user:
- `status:backlog` - "Your request has been logged"
- `status:todo` - "Your request is queued and ready to start"
- `status:in-progress` - "Work has started on your request"
- `status:ai-review` - "The AI integrator is reviewing the work"
- `status:human-review` - "This needs your review and testing"
- `status:merge-release` - "Code is approved and ready to merge"
- `status:blocked` - "There's a blocker, may need your input"
- `status:done` - "Your request is complete"

### Assignment Labels (`assign:*`)

EAMA explains assignments to user:
- `assign:implementer-*` - "An AI agent is working on this"
- `assign:human` - "This needs human attention"
- `assign:orchestrator` - "The orchestrator is handling this"

---

## EAMA Label Commands

### When User Creates Request

```bash
# Create issue with initial labels
gh issue create \
  --title "$USER_REQUEST_TITLE" \
  --body "$USER_REQUEST_BODY" \
  --label "status:backlog" \
  --label "priority:$PRIORITY" \
  --label "type:$TYPE"
```

### When User Changes Priority

```bash
# Update priority
gh issue edit $ISSUE_NUMBER --remove-label "priority:normal" --add-label "priority:high"
```

### When User Puts on Hold

```bash
# Mark blocked
gh issue edit $ISSUE_NUMBER --remove-label "status:in-progress" --add-label "status:blocked"
```

### When Facilitating Human Review

```bash
# Present the task to the user for review (escalated from AI Review)
gh issue view $ISSUE_NUMBER --json title,body,labels
# After user approves:
gh issue edit $ISSUE_NUMBER --remove-label "status:human-review" --add-label "status:merge-release"
# After user requests changes:
gh issue edit $ISSUE_NUMBER --remove-label "status:human-review" --add-label "status:in-progress"
```

### When Generating Status Report

```bash
# Get all active issues for user
gh issue list --label "status:in-progress" --json number,title,labels

# Get issues in AI review
gh issue list --label "status:ai-review" --json number,title,labels

# Get issues awaiting human review
gh issue list --label "status:human-review" --json number,title,labels

# Get blocked issues needing attention
gh issue list --label "status:blocked" --json number,title,labels

# Get completed issues
gh issue list --label "status:done" --state closed --json number,title
```

---

## User Communication Patterns

### Explaining Labels to User

When user asks "What's happening with my request?":

```markdown
**Issue #42: Add user authentication**

- **Status**: In Progress (`status:in-progress`)
- **Priority**: High (`priority:high`)
- **Assigned to**: Implementation Agent 1 (`assign:implementer-1`)
- **Type**: New Feature (`type:feature`)

The implementation agent is actively working on this task.
```

### Translating User Requests to Labels

| User Says | Labels to Apply |
|-----------|-----------------|
| "This is urgent!" | `priority:critical` |
| "When you get a chance..." | `priority:low` |
| "Something is broken" | `type:bug`, `priority:high` |
| "Can you add..." | `type:feature` |
| "Put this on hold" | `status:blocked` |
| "Resume this" | Remove `status:blocked`, add `status:todo` |

---

## Quick Reference

### EAMA Label Responsibilities

| Action | Labels Involved |
|--------|-----------------|
| Create issue | `status:backlog`, `priority:*`, `type:*` |
| Change priority | Update `priority:*` |
| Pause work | Add `status:blocked` |
| Resume work | Remove `status:blocked`, add `status:todo` |
| Facilitate human review | Manage `status:human-review` transitions |
| Report to user | Read all labels for status |

### Labels EAMA Never Sets

- `assign:*` - Set by EOA/ECOS
- `review:*` - Managed by EIA
- `effort:*` - Set by EOA during triage
- `component:*` - Set by EOA/EAA

### EAMA's Approval Authority

EAMA can approve:
- Priority changes requested by other agents
- Scope changes that affect user expectations
- Deadline/milestone changes

EAMA must approve:
- Any change to `priority:critical`
- Reassignment to `assign:human`
- Project-level decisions

---

## Output

| Output Type | Format | When Generated |
|-------------|--------|----------------|
| Issue creation | GitHub issue URL with labels | When user creates new request |
| Label update confirmation | "Priority updated to high" | When labels modified |
| Status report | Markdown table of issues and labels | When user requests status |
| Label explanation | Plain text translation of label meaning | When user asks "what's happening?" |

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| Label not found | Typo or label doesn't exist in repo | Create label or fix typo |
| Permission denied | Insufficient GitHub permissions | Request repo access from user |
| Issue not found | Invalid issue number | Verify issue number with user |
| Conflicting labels | Multiple priority labels applied | Remove old label before adding new |
| API rate limit | Too many GitHub API calls | Wait and retry, batch operations |

## Examples

### Example 1: Creating Issue with Labels

**User request**: "The login page is broken, fix it urgently!"

**EAMA actions**:
```bash
gh issue create \
  --title "Login page broken" \
  --body "User reported urgent login page issue" \
  --label "type:bug" \
  --label "priority:critical" \
  --label "status:backlog"
```

**User response**: "Created issue #123 with critical priority. The orchestrator will triage this shortly."

### Example 2: Reporting Status with Labels

**User request**: "What's the status of my authentication work?"

**EAMA actions**:
```bash
gh issue list --label "component:auth" --json number,title,labels
```

**User response**:
```markdown
**Authentication Issues**:

- **#42**: Add OAuth support
  - Status: In Progress (`status:in-progress`)
  - Priority: High (`priority:high`)
  - Assigned to: Implementation Agent 1

- **#45**: Fix password reset
  - Status: Blocked (`status:blocked`)
  - Priority: Normal (`priority:normal`)
  - Blocker: Waiting for email service setup
```

### Example 3: Updating Priority

**User request**: "Actually, make that password reset high priority too."

**EAMA actions**:
```bash
gh issue edit 45 --remove-label "priority:normal" --add-label "priority:high"
```

**User response**: "Priority updated to high for issue #45."

---

## Resources

- **AGENT_OPERATIONS.md** - Core agent operational patterns
- **eama-status-reporting** - User communication patterns
- **eama-user-communication** - Communication style guidelines
- **eama-role-routing** - Role delegation patterns
