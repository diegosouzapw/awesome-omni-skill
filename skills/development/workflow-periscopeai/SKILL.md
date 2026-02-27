---
name: workflow
description: Start, monitor, and manage workflow executions in Periscope
delegates-to: workflow-operator
---

# /workflow - Execute and Manage Workflows

Use this skill to start, monitor, and manage workflow executions in the Periscope platform.

## What You Can Do

1. **Start workflows** - Execute a workflow with input data
2. **Monitor execution** - Check status and progress
3. **Send signals** - Trigger workflow events
4. **Schedule workflows** - Plan future executions
5. **Use triggers** - Start workflows via signals, messages, or webhooks

## How to Use

Describe what you want to do with workflows:

- "Start the approval workflow for order 12345"
- "Check the status of my running workflows"
- "Cancel workflow abc-123"
- "Send an approval signal to workflow xyz"
- "Schedule the report workflow for tomorrow at 9am"

## Common Operations

### Start a Workflow
```
Start workflow: approval_workflow
Input: {"request_id": "REQ-001", "amount": 5000}
```

### Check Status
```
Get status for workflow: approval_workflow-uuid
```

### Send Signal
```
Send signal "approve" to workflow: approval_workflow-uuid
Data: {"approved_by": "manager@company.com"}
```

### Schedule for Later
```
Schedule workflow: daily_report
Run at: 2024-01-15 09:00 UTC
```

## Workflow Types

Common workflow types available:

| Workflow | Task Queue | Purpose |
|----------|------------|---------|
| approval_workflow | periscope-queue | Multi-step approvals |
| data_processing_workflow | periscope-queue | ETL pipelines |
| business_process_workflow | periscope-queue | Business automation |
| agent_coordination_workflow | periscope-ai-queue | AI agent coordination |

## Workflow Statuses

| Status | Description |
|--------|-------------|
| running | Currently executing |
| completed | Finished successfully |
| failed | Failed with error |
| cancelled | Manually cancelled |
| timed_out | Exceeded timeout |

## Reference Documentation

- [Temporal Concepts](../../../docs/temporal-concepts.md) - Workflow fundamentals
- [Queues and Workers](../../../docs/queues-and-workers.md) - Task queue configuration

## Delegated Agent

This skill delegates to the **workflow-operator** agent which has access to:
- `periscope-workflows-dev` MCP server (18 tools)
- `periscope-tasks-dev` MCP server (~11 tools)
