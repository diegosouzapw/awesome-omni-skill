---
name: approval-workflow
description: Manages Human-in-the-Loop (HITL) approval workflows for sensitive actions. Use when creating approval requests, processing approved items, or implementing safety controls for autonomous actions.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Approval Workflow Skill

This skill implements the Human-in-the-Loop (HITL) approval system that ensures human oversight for sensitive autonomous actions.

## Core Concept

The approval workflow uses folder-based state management:

```
/Pending_Approval/  →  Human reviews  →  /Approved/  →  Execute
                                    →  /Rejected/  →  Archive
```

## Approval Request Format

```markdown
---
type: approval_request
action: [action_type]
created: [ISO timestamp]
expires: [ISO timestamp]
status: pending
priority: [critical|high|medium|low]
requestor: [agent_name]
risk_level: [low|medium|high]
reversible: [true|false]
---

## Action Summary
[Brief description of what will happen]

## Details
[Full action parameters]

## Risk Assessment
- **Reversible**: [Yes|No|Partial]
- **Impact**: [Description]
- **Sensitive Data**: [Yes|No]

## Instructions
- **Approve**: Move to `/Approved/`
- **Reject**: Move to `/Rejected/`
- **Edit**: Modify, then approve
```

## Approval Thresholds

| Action Type | Auto-Approve | Human Required |
|-------------|--------------|----------------|
| Email (known contact) | Reply only | New recipients |
| Payment | < $50 recurring | New payee, > $100 |
| Social post | Scheduled | Immediate, replies |
| File operations | Create/read | Delete |
| WhatsApp | Greetings | Business msgs |

## Workflow States

1. **pending** - Awaiting human decision
2. **approved** - Cleared for execution
3. **rejected** - Denied by human
4. **expired** - Timed out without decision
5. **executed** - Action completed
6. **failed** - Execution error

## Reference

For detailed implementation, see [reference.md](reference.md)

For usage examples, see [examples.md](examples.md)
