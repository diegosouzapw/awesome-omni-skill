---
name: outline-load
description: Outline the load profile for a system or component — map traffic patterns, peak usage, data growth, and resource consumption over time.
user_invocable: true
argument-hint: <component or system path>
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
---

# Outline Load Profile

Map the load characteristics of a component or system to understand its scaling behavior.

**Arguments:** $ARGUMENTS

## Workflow

### Step 1: Identify the Component

Read the target component from `$ARGUMENTS`. Explore its:
- Entry points (API endpoints, CLI commands, event handlers, scheduled jobs)
- Data stores it reads from and writes to
- External service calls

### Step 2: Map Request Flow

For each entry point, trace the request path:

1. **Ingress**: How does work arrive? (HTTP, queue, cron, user action)
2. **Processing**: What computation happens? (CPU-bound, I/O-bound, memory-bound)
3. **Egress**: Where do results go? (response, database write, file output, notification)

### Step 3: Estimate Load Dimensions

For each flow, estimate:

| Dimension | Current | 10x | 100x |
|-----------|---------|-----|------|
| Requests/sec | | | |
| Data per request | | | |
| Total storage | | | |
| Concurrent connections | | | |
| Memory per request | | | |

Fill in what can be determined from code. Mark unknowns as `?`.

### Step 4: Identify Bottlenecks

Look for:
- **Single-threaded paths**: Synchronous processing that blocks
- **Unbounded collections**: Arrays/lists that grow with input size
- **N+1 queries**: Database/API calls inside loops
- **Global state**: Shared mutable state across requests
- **Missing pagination**: Queries without LIMIT/OFFSET
- **Large file reads**: Reading entire files into memory

### Step 5: Output the Load Profile

```markdown
## Load Profile: [Component]

**Entry Points**: [count] ([list])
**Processing Type**: [CPU-bound | I/O-bound | Memory-bound | Mixed]

**Current Load Estimate**:
[table from Step 3]

**Bottlenecks Identified**:
1. [file:line] — [description]
2. [file:line] — [description]

**Scaling Risks**:
- [risk 1]
- [risk 2]

**Recommendation**: [scale-ready | needs-work | redesign-required]
```

This feeds into `/scale-review:evaluate` for detailed analysis.
