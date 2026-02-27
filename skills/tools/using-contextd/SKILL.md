---
name: using-contextd
description: Use when starting any session with contextd - introduces core tools for cross-session memory, semantic code search, and error remediation. REQUIRES contextd MCP server.
---

# Using contextd

## Prerequisites: contextd MCP Server

**This skill REQUIRES the contextd MCP server.**

Before using any contextd tools, verify availability:
1. Check for `mcp__contextd__*` tools (use ToolSearch if needed)
2. If tools are NOT available:
   - Inform user: "contextd MCP server not configured"
   - Suggest: "Run `/contextd:init` to configure contextd"
   - Alternative: Use standard Read/Grep/Glob (no cross-session memory)

## Error Handling

| Error | Meaning | Fix |
|-------|---------|-----|
| `Unknown tool: mcp__contextd__*` | MCP not configured | Run `/contextd:init` |
| `Connection refused on port 9090` | Server not running | Run `contextd serve` |
| `Tenant not found` | First use | Will auto-create |

### Input Validation Errors

contextd v1.5+ enforces strict input validation. Common errors:

| Error | Cause | Fix |
|-------|-------|-----|
| `invalid project_path: path contains directory traversal` | Path contains `../` | Use absolute paths or paths within project |
| `invalid tenant_id: must be lowercase alphanumeric with underscores` | Invalid characters in ID | Use format: `my_project`, `org123` (1-64 chars) |
| `invalid project_id: must be lowercase alphanumeric with underscores` | Invalid characters in ID | Same as tenant_id format |
| `invalid include_patterns: contains dangerous characters` | Shell injection chars in glob | Remove `;`, `\|`, `` ` ``, `$` from patterns |
| `invalid patterns: excessive wildcards` | Pattern like `***` | Use standard globs: `*`, `**`, `*.go` |

---

## Pre-Flight Protocol (MANDATORY)

**BEFORE any filesystem operation (Read, Grep, Glob), you MUST:**

1. **`semantic_search(query, project_path: ".")`** - Semantic code search with auto grep fallback
2. **`memory_search(project_id, query)`** - Check past learnings and solutions

Skipping this is a protocol violation.

## Core Tools

| Category | Tools | Purpose |
|----------|-------|---------|
| **Search** | `semantic_search`, `repository_search`, `repository_index` | Code lookup by meaning |
| **Memory** | `memory_search`, `memory_record`, `memory_feedback`, `memory_outcome` | Cross-session learning |
| **Checkpoint** | `checkpoint_save`, `checkpoint_list`, `checkpoint_resume` | Context preservation |
| **Remediation** | `remediation_search`, `remediation_record`, `troubleshoot_diagnose` | Error pattern tracking |
| **Context Folding** | `branch_create`, `branch_return`, `branch_status` | Isolated sub-tasks |
| **Reflection** | `reflect_analyze`, `reflect_report` | Behavior pattern analysis |

## Health Monitoring (HTTP)

contextd exposes HTTP health endpoints for monitoring vectorstore integrity:

| Endpoint | Purpose | Status Codes |
|----------|---------|--------------|
| `GET /health` | Basic health with metadata summary | 200 OK, 503 Degraded |
| `GET /api/v1/health/metadata` | Detailed per-collection status | 200 OK |

**Graceful Degradation (P0)**: If corrupt collections are detected, contextd quarantines them and continues operating with healthy collections. Check health status to detect degraded state.

**Example health check**:
```bash
curl -s http://localhost:9090/health | jq
# {"status":"ok","metadata":{"status":"healthy","healthy_count":22,"corrupt_count":0}}
```

## Search Priority

| Priority | Tool | When |
|----------|------|------|
| **1st** | `semantic_search` | Auto-selects best method (indexed or grep fallback) |
| **2nd** | `memory_search` | Have I solved this before? |
| **3rd** | Read/Grep/Glob | Fallback for exact matches only |

### Path Validation (contextd v1.5+)

All tools accepting `project_path` validate paths before use:
- **No directory traversal**: Paths containing `../` are rejected
- **Affected tools**: `semantic_search`, `repository_index`, `repository_search`, `reflect_report`
- **Use absolute paths** or paths within the current project directory

## The Learning Loop

```
1. SEARCH at task start (MANDATORY)
   semantic_search(query, project_path)
   memory_search(project_id, query)

2. DO the work
   (apply relevant memories)

3. RECORD at completion
   memory_record(project_id, title, content, outcome)

4. FEEDBACK when memories helped
   memory_feedback(memory_id, helpful)
```

## Key Concepts

**Tenant ID**: Derived from git remote (e.g., `github.com/fyrsmithlabs/contextd` -> `fyrsmithlabs`). Verify with: `git remote get-url origin | sed 's|.*github.com[:/]\([^/]*\).*|\1|'`

**Project ID**: Scopes memories. Use repository name (e.g., `contextd`) or `org_repo` format for multi-org.

### ID Format Requirements (contextd v1.5+)

Both `tenant_id` and `project_id` must follow this format:
- **Characters**: Lowercase alphanumeric and underscores only (`a-z`, `0-9`, `_`)
- **Length**: 1-64 characters
- **Valid**: `my_project`, `contextd`, `org123`, `fyrsmithlabs_marketplace`
- **Invalid**: `My-Project` (uppercase, hyphen), `org/repo` (slash), `project..name` (dots)

**Confidence**: Memories have scores (0-1) that adjust via feedback. Higher = ranks first.

## What to Record

**Good memories:**
- Non-obvious solutions
- Patterns that apply broadly
- Design decisions with rationale (the WHY)
- Mistakes and why they failed

**Skip recording:**
- Trivial fixes (typos, syntax)
- Project-specific details (put in CLAUDE.md)

## Recording Design Decisions

When design involves significant discussion, capture the WHY:

```json
{
  "project_id": "contextd",
  "title": "ADR: Registry pattern for DI",
  "content": "DECISION: Use Registry interface.\nWHY: Idiomatic Go, single mock for tests.\nREJECTED: Passing individual services (constructor bloat).",
  "outcome": "success",
  "tags": ["adr", "architecture", "design-decision"]
}
```

## Optional: Conversation Indexing

Index past Claude Code sessions to pre-warm contextd:

```bash
# Via /init command
/init --conversations

# What it extracts:
# - Error -> fix patterns (remediations)
# - Learnings (memories)
# - User corrections (policies)
```

Conversations are scrubbed for secrets before processing.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using Read/Grep before contextd | `semantic_search` FIRST |
| Not searching at task start | Always `memory_search` first |
| Forgetting to record learnings | `memory_record` at task completion |
| Re-solving fixed errors | `remediation_search` when errors occur |
| Context bloat from sub-tasks | Use `branch_create` for isolation |

---

## Memory Lifecycle

### Temporal Decay & Expiration

Memories have a confidence score (0-1) that decays over time without reinforcement:

| Age | Decay Factor | Result |
|-----|--------------|--------|
| < 7 days | 1.0 | Full confidence |
| 7-30 days | 0.9 | Slight decay |
| 30-90 days | 0.7 | Moderate decay |
| > 90 days | 0.5 | Significant decay (but never deleted) |

**Boost confidence via:**
- `memory_feedback(memory_id, helpful=true)` - Resets decay timer
- Memory reuse in solutions - Auto-boosted when applied

**Expiration policies:**
- `ttl_days: 365` - Auto-archive after 1 year without activity
- `never_expire: true` - For ADRs and critical decisions

### Memory Types

| Type | Purpose | Default TTL |
|------|---------|-------------|
| `learning` | General knowledge gained | 180 days |
| `remediation` | Error -> fix mappings | 365 days |
| `decision` | ADR/architecture choices | Never |
| `failure` | What NOT to do | 365 days |
| `pattern` | Reusable code patterns | 180 days |
| `policy` | STRICT constraints | Never |

Tag memories with type: `tags: ["type:learning", "category:testing"]`

---

## Query Expansion & Fuzzy Matching

### Automatic Query Expansion

`semantic_search` and `memory_search` automatically expand queries:

| Original Query | Expanded To |
|----------------|-------------|
| "auth error" | "auth error", "authentication failure", "login issue", "401", "403" |
| "test fails" | "test fails", "test failure", "assertion error", "spec broken" |
| "slow query" | "slow query", "performance", "N+1", "timeout", "latency" |

Disable expansion: `expand_query: false`

### Fuzzy Matching

Handles typos and variations:

```json
{
  "query": "authetication",
  "fuzzy_threshold": 0.8,  // 0-1, higher = stricter
  "fuzzy_max_edits": 2     // Levenshtein distance
}
```

Results include match quality:
- `exact` - Literal match
- `semantic` - Meaning match
- `fuzzy` - Typo-tolerant match

---

## Hierarchical Namespaces

Structure project IDs for multi-team organizations:

```
org/team/project/module

Examples:
  fyrsmithlabs/platform/contextd/api
  fyrsmithlabs/platform/contextd/vectorstore
  fyrsmithlabs/marketplace/fs-dev
```

**Search scopes:**
- `fyrsmithlabs/*` - All org memories
- `fyrsmithlabs/platform/*` - All platform team
- `fyrsmithlabs/platform/contextd` - Specific project

---

## Audit Fields

All memory operations record:

| Field | Description | Auto-set |
|-------|-------------|----------|
| `created_by` | Agent/session that created | Yes |
| `created_at` | ISO timestamp | Yes |
| `updated_at` | Last modification | Yes |
| `usage_count` | Times retrieved in searches | Yes |
| `last_used_at` | Last retrieval timestamp | Yes |

Query audit data:
```json
{
  "query": "authentication",
  "include_audit": true
}
```

---

## Claude Code 2.1 Patterns

### Background Task Execution

Use `run_in_background: true` for long-running searches:

```
Task(
  subagent_type: "general-purpose",
  prompt: "Search memories for authentication patterns across all projects",
  run_in_background: true
)

// Continue other work...

// Later, collect results:
TaskOutput(task_id, block: true)
```

### Task Dependencies with addBlockedBy

Chain dependent memory operations:

```
task1 = Task(prompt: "Index repository")
task2 = Task(prompt: "Search indexed code", addBlockedBy: [task1.id])
task3 = Task(prompt: "Record findings", addBlockedBy: [task2.id])
```

### PreToolUse Hook Example

Auto-search memories before any Read operation:

```json
{
  "hook_type": "PreToolUse",
  "tool_name": "Read",
  "prompt": "Before reading {{tool_input.file_path}}, search memories for relevant context about this file or module."
}
```

### PostToolUse Hook Example

Auto-record learnings after successful operations:

```json
{
  "hook_type": "PostToolUse",
  "tool_name": "Edit",
  "condition": "tool_output.success == true",
  "prompt": "If this edit fixed a bug or implemented a pattern worth remembering, call memory_record."
}
```

---

## Event-Driven State Sharing

Skills can share state via memory events:

```json
{
  "event": "memory_created",
  "filter": {"tags": ["type:decision"]},
  "notify": ["consensus-review", "self-reflection"]
}
```

Subscribe to events:
- `memory_created` - New memory recorded
- `memory_feedback` - Feedback given
- `remediation_recorded` - New fix documented
- `checkpoint_saved` - State preserved
