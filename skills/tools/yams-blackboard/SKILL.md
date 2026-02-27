---
name: yams-blackboard
description: Blackboard plugin for OpenCode built on YAMS, enabling agents to share findings, tasks, and context via a persistent knowledge graph.
license: GPL-3.0
compatibility: claude-code, opencode
metadata:
  tools: |
    bb_register_agent, bb_list_agents,
    bb_post_finding, bb_get_finding, bb_query_findings, bb_search_findings, bb_acknowledge_finding, bb_resolve_finding,
    bb_create_task, bb_get_ready_tasks, bb_claim_task, bb_update_task, bb_complete_task, bb_fail_task, bb_query_tasks,
    bb_create_context, bb_get_context_summary, bb_set_context, bb_recent_activity, bb_stats, bb_connections
  categories: agent, coordination, blackboard, knowledge-graph
---

# YAMS Blackboard Skill (OpenCode)

## Overview

This skill implements the classic **blackboard pattern** for multi‑agent systems using **YAMS** as the shared memory layer. Agents can:

- Register themselves and publish capabilities.
- Post **findings** (tagged markdown documents) that other agents can query or search.
- Create, claim, and complete **tasks** that coordinate work across agents.
- Group related work into **contexts** and explore relationships via YAMS’ graph commands.

All data is stored in YAMS, so the same powerful search (`yams grep`, `yams search`) and graph (`yams graph`) capabilities are available.

## Quick Reference

```bash
# Start a YAMS session (optional, auto‑starts on OpenCode session creation)
yams session start --name "opencode-session"

# Register an agent
bb_register_agent '{"id":"scanner","name":"Security Scanner","capabilities":["security","code-review"]}'

# Post a finding
bb_post_finding '{"agent_id":"scanner","topic":"security","title":"SQL Injection","severity":"high","confidence":0.94,"content":"..."}'

# Search findings
bb_search_findings '{"query":"SQL injection"}'

# Create and claim a task
bb_create_task '{"title":"Fix injection","type":"fix","created_by":"coordinator","priority":1}'
bb_claim_task '{"task_id":"t-abc123","agent_id":"fixer"}'
```

## Tools

| Tool | Description |
|------|-------------|
| `bb_register_agent` | Register an agent with ID, name, and capabilities |
| `bb_list_agents` | List all registered agents |
| `bb_post_finding` | Store a finding (markdown front‑matter) on the blackboard |
| `bb_get_finding` | Retrieve a finding by its ID |
| `bb_query_findings` | Query findings by topic, agent, severity, etc. |
| `bb_search_findings` | Semantic search across finding contents |
| `bb_acknowledge_finding` | Mark a finding as acknowledged by an agent |
| `bb_resolve_finding` | Mark a finding as resolved with resolution details |
| `bb_create_task` | Create a new coordination task |
| `bb_get_ready_tasks` | List pending tasks with no unmet dependencies |
| `bb_claim_task` | Claim a pending task for an agent |
| `bb_update_task` | Update task status, findings, or artifacts |
| `bb_complete_task` | Mark task as completed and optionally attach results |
| `bb_fail_task` | Mark task as failed with an error message |
| `bb_query_tasks` | Query tasks by type, status, assignee, etc. |
| `bb_create_context` | Create a named context to group work |
| `bb_get_context_summary` | Generate a markdown summary of a context |
| `bb_set_context` | Set the active context for subsequent operations |
| `bb_recent_activity` | Retrieve recent findings and tasks |
| `bb_stats` | Return blackboard statistics (agents, findings, tasks) |
| `bb_connections` | Explore graph connections for an entity path |

## Compaction / Context Recovery

- This skill auto-injects a blackboard summary into `output.context` during OpenCode compaction hooks (`experimental.session.compacting` and `session.compacted`).
- For broader YAMS memory after compaction, use `yams list` (replaces the removed `yams hook`): `yams list --owner opencode --since 24h --limit 20 --format markdown` with optional `--pbi`, `--task`, `--phase`, `--metadata key=value` (repeatable), and `--match-any-metadata`.

Example:

```bash
yams list --owner opencode --since 24h --limit 20 --format markdown --metadata phase=checkpoint
```

### Owner / multi-agent convention

- When registering agents or writing findings/tasks via this plugin, use the shared owner `opencode` so multiple agents can read the same YAMS records. Retrieval: `yams list --owner opencode ...`.

### Canonical agent registration

```bash
bb_register_agent '{"id":"opencode-coordinator","name":"OpenCode Coordinator","capabilities":["coordination","routing","summary"]}'
```

## Example Workflow

```typescript
// Agent A registers and posts a security finding
bb_register_agent({ id: "scanner", name: "Scanner", capabilities: ["security"] })
bb_post_finding({
  agent_id: "scanner",
  topic: "security",
  title: "SQL Injection in auth.ts",
  severity: "high",
  confidence: 0.96,
  content: "Unsanitized user input reaches SQL query...",
})

// Agent B discovers the finding and creates a remediation task
bb_search_findings({ query: "SQL injection" })
bb_create_task({
  title: "Fix SQL injection",
  type: "fix",
  created_by: "coordinator",
  priority: 0,
})

// Agent B claims and works on the task
bb_claim_task({ task_id: "t-123", agent_id: "fixer" })
bb_update_task({ task_id: "t-123", status: "working" })
bb_complete_task({ task_id: "t-123", findings: ["f-xyz"] })

// Resolve the original finding
bb_resolve_finding({ finding_id: "f-abc", agent_id: "fixer", resolution: "Parameterized queries added" })
```

## Development

The TypeScript implementation lives in `blackboard.ts`.  Building the plugin:

```bash
bun install
bun run build
```

Run tests with `bun test`.

---
For more details see the plugin’s `README.md` and the YAMS skill definition at `docs/skills/yams/SKILL.md`.
