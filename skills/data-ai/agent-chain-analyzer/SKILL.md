---
name: agent-chain-analyzer
description: Detects and analyzes agent chain anti-patterns where agents invoke other agents sequentially causing massive context load
model: claude-haiku-4-5
---

# Agent Chain Analyzer Skill

<CONTEXT>
You detect and analyze the **Agent Chain anti-pattern** - one of the most severe architectural issues in pre-skills Claude Code projects.

**Agent Chain Pattern**: Agent1 invokes Agent2 via Task tool, which invokes Agent3, forming a sequential chain with massive context load.

**Impact**:
- 180K+ tokens for 4-agent chain (4 × 45K)
- No state persistence across chain
- No user approval workflow
- Fragile (single failure breaks chain)

**Solution**: Convert to Manager-as-Agent + Skills (53% context reduction)

You execute deterministic scripts to detect chains, map dependencies, calculate depth, and estimate context impact.
</CONTEXT>

<CRITICAL_RULES>
1. ALWAYS detect ALL agent chains in project (not just obvious ones)
2. ALWAYS calculate full chain depth and context impact
3. ALWAYS map complete dependency graph
4. ALWAYS return structured JSON output
5. NEVER modify project files (read-only analysis)
6. NEVER miss indirect chains (Agent1 → Agent2 → Agent3 → Agent4)
</CRITICAL_RULES>

<OPERATIONS>

## detect-agent-chains

Detect all agent chain patterns in project.

**Input:**
- `project_path`: Path to Claude Code project root

**Process:**
1. Execute: `scripts/detect-agent-chains.sh "{project_path}"`
2. Parse JSON output
3. Return chain detection results

**Output:**
```json
{
  "status": "success",
  "chains_detected": true,
  "chain_count": 2,
  "chains": [
    {
      "chain_id": "catalog-process-chain",
      "entry_point": "catalog-fetcher",
      "agents": ["catalog-fetcher", "catalog-analyzer", "catalog-validator", "catalog-reporter"],
      "depth": 4,
      "locations": [
        ".claude/agents/catalog-fetcher.md",
        ".claude/agents/catalog-analyzer.md",
        ".claude/agents/catalog-validator.md",
        ".claude/agents/catalog-reporter.md"
      ],
      "invocation_pattern": "Task tool",
      "context_estimate": 180000
    }
  ],
  "total_agents_in_chains": 7,
  "max_chain_depth": 4
}
```

---

## analyze-chain-depth

Analyze chain depth and complexity for each detected chain.

**Input:**
- `project_path`: Path to Claude Code project root
- `chains`: Detected chains from detect-agent-chains operation

**Process:**
1. Execute: `scripts/analyze-chain-depth.sh "{project_path}" "{chains_json}"`
2. Calculate depth metrics
3. Identify complexity factors

**Output:**
```json
{
  "status": "success",
  "depth_analysis": [
    {
      "chain_id": "catalog-process-chain",
      "depth": 4,
      "complexity_score": 0.85,
      "complexity_factors": [
        "Deep chain (4 agents)",
        "No error handling between agents",
        "No state persistence",
        "Sequential only (no parallelism)"
      ],
      "migration_complexity": "high",
      "estimated_refactor_days": 15
    }
  ],
  "average_depth": 3.5,
  "deepest_chain": {
    "chain_id": "catalog-process-chain",
    "depth": 4
  }
}
```

---

## map-chain-dependencies

Map complete dependency graph for agent chains.

**Input:**
- `project_path`: Path to Claude Code project root
- `chains`: Detected chains from detect-agent-chains operation

**Process:**
1. Execute: `scripts/map-chain-dependencies.sh "{project_path}" "{chains_json}"`
2. Build dependency graph
3. Identify data flow between agents

**Output:**
```json
{
  "status": "success",
  "dependency_graph": {
    "nodes": [
      {"id": "catalog-fetcher", "type": "entry_point"},
      {"id": "catalog-analyzer", "type": "intermediate"},
      {"id": "catalog-validator", "type": "intermediate"},
      {"id": "catalog-reporter", "type": "terminal"}
    ],
    "edges": [
      {
        "from": "catalog-fetcher",
        "to": "catalog-analyzer",
        "data_passed": ["catalog_data", "metadata"],
        "invocation_method": "Task tool"
      },
      {
        "from": "catalog-analyzer",
        "to": "catalog-validator",
        "data_passed": ["analysis_results"],
        "invocation_method": "Task tool"
      },
      {
        "from": "catalog-validator",
        "to": "catalog-reporter",
        "data_passed": ["validation_results"],
        "invocation_method": "Task tool"
      }
    ]
  },
  "data_flow": [
    "catalog_data (fetcher → analyzer)",
    "analysis_results (analyzer → validator)",
    "validation_results (validator → reporter)"
  ],
  "parallelization_opportunities": [
    "analyzer and validator could run in parallel if refactored"
  ]
}
```

---

## estimate-chain-context

Calculate context load impact for agent chains.

**Input:**
- `project_path`: Path to Claude Code project root
- `chains`: Detected chains from detect-agent-chains operation

**Process:**
1. Execute: `scripts/estimate-chain-context.sh "{project_path}" "{chains_json}"`
2. Calculate current context load
3. Estimate post-migration load
4. Calculate reduction percentage

**Output:**
```json
{
  "status": "success",
  "context_estimates": [
    {
      "chain_id": "catalog-process-chain",
      "current_context": {
        "agents": 4,
        "tokens_per_agent": 45000,
        "total_tokens": 180000,
        "overhead_tokens": 40000,
        "grand_total": 220000
      },
      "projected_context": {
        "manager_agent": 45000,
        "skills": 4,
        "tokens_per_skill": 5000,
        "skills_total": 20000,
        "overhead_tokens": 10000,
        "grand_total": 75000
      },
      "reduction": {
        "tokens_saved": 145000,
        "percentage": 0.66,
        "description": "66% context reduction (220K → 75K)"
      }
    }
  ],
  "total_current_context": 220000,
  "total_projected_context": 75000,
  "total_reduction": 0.66
}
```

</OPERATIONS>

<DOCUMENTATION>
Upon completion of analysis, output:

```
✅ COMPLETED: Agent Chain Analyzer
Project: {project_path}
───────────────────────────────────────
Chains Detected: {count}
Max Depth: {depth} agents
Context Impact: {tokens} tokens
Reduction Potential: {percentage}% ({tokens} saved)
───────────────────────────────────────
Results returned to: project-auditor agent
```
</DOCUMENTATION>

<ERROR_HANDLING>

**No chains detected:**
```json
{
  "status": "success",
  "chains_detected": false,
  "chain_count": 0,
  "chains": [],
  "message": "No agent chains found - project uses modern architecture"
}
```

**Script execution failed:**
```json
{
  "status": "error",
  "error": "script_failed",
  "script": "{script_name}",
  "message": "{error_output}",
  "resolution": "Check script permissions and agent file access"
}
```

**Invalid chain data:**
```json
{
  "status": "error",
  "error": "invalid_chain_data",
  "message": "Chain structure malformed",
  "resolution": "Ensure detect-agent-chains returned valid JSON"
}
```

</ERROR_HANDLING>

## Integration

**Invoked By:**
- project-auditor agent (Phase 5: Execute - detailed analysis)

**Depends On:**
- Agent files in `.claude/agents/`
- Grep patterns for Task tool invocations
- Agent invocation patterns (@agent-*, Task tool)

**Outputs To:**
- Anti-patterns list in audit report
- Migration roadmap generator
- Context optimization calculator

## Design Notes

**Why CRITICAL?**

Agent chains are the #1 context load issue in pre-skills projects:
- 4-agent chain = 180K tokens (53% of budget)
- 6-agent chain = 270K tokens (exceeds budget)
- Migration = 58-72% context reduction

**Detection Strategy:**

1. **Direct Detection**: Grep for `Task tool`, `Task(`, `@agent-` in agent files
2. **Indirect Detection**: Follow invocation chains recursively
3. **Validation**: Ensure invoked entity is an agent (not skill)
4. **Mapping**: Build complete dependency graph

**Common Chain Patterns:**

- **Linear Chain**: A → B → C → D (most common)
- **Branching Chain**: A → B → (C, D) (rare, more complex)
- **Recursive Chain**: A → B → A (dangerous, infinite loop risk)

