---
name: dev-tech-debt-review
description: Detect AI/agentic-specific anti-patterns that traditional linters miss. Analyzes tool/agent boundary violations, prompt debt, context window issues, testing patterns, and more. Returns scored findings with remediation guidance.
---

# dev.tech-debt-review

> Detect AI/agentic-specific tech debt that traditional linters miss.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Usage Modes

| Mode | Command | Scope | When to Use |
|------|---------|-------|-------------|
| **Changed Files** | `/dev.tech-debt-review` | Files changed vs main | Before PR (default) |
| **Full Audit** | `/dev.tech-debt-review --all` | Full src/ directory | Release gate, quarterly review |
| **Category Focus** | `/dev.tech-debt-review --category=boundary` | Specific category | Targeted cleanup |

**Categories**: `boundary`, `prompt`, `context`, `testing`, `error`, `state`, `observability`, `subagent`

---

## Goal

Detect AI/agentic-specific anti-patterns that traditional linters miss:

| Category | What It Finds | Why It Matters |
|----------|---------------|----------------|
| **Tool/Agent Boundary** | Tools making decisions, encoding thresholds | Violates Constitution VII |
| **Prompt Debt** | Hardcoded prompts, no versioning, injection risks | Maintenance nightmare |
| **Context Window** | Unbounded data, no truncation, raw dumps | Performance/reliability |
| **Testing** | Live LLM in unit tests, missing VCR, no evals | CI flakiness, cost |
| **Error Handling** | No retry/backoff, missing timeouts | Reliability |
| **State Management** | Unbounded conversation history | Memory leaks |
| **Observability** | No tracing, missing metrics | Debugging blind spots |
| **Subagent** | Depth > 1, unclear boundaries | Architecture violation |

Traditional linters catch syntax and style. This skill catches **semantic anti-patterns** specific to agentic systems.

---

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output analysis and recommendations.

**SEMANTIC ANALYSIS**: Read and understand code patterns, don't just grep.

**EVIDENCE-BASED**: Every finding must cite file:line and code snippet.

**SCORING**: Aggregate findings into 0-100 score for PR gate decisions.

---

## Constitution Alignment

This skill validates adherence to project principles:

| Principle | What We Check |
|-----------|---------------|
| **I. Local-First** | No data transmission, security of prompts |
| **III. Causal-First** | Recommendations trace to evidence |
| **VII. Intelligent Tooling** | Tools provide data, agent provides judgment |
| **VIII. Compounding Value** | Historical tracking of tech debt |
| **IX. Agent-Aware** | Context optimized for agent cognition |

Reference: [Constitution Mapping](references/constitution-mapping.md)

---

## Execution Steps

### Phase 0: Scope Determination

**You handle this phase directly.**

**Parse user input to determine scope:**

1. **If `--all` flag present**: Full codebase audit
   ```bash
   find src -name "*.ts" -type f | grep -v ".d.ts" | grep -v "node_modules"
   ```

2. **If `--category=X` flag present**: Filter to that category's patterns only

3. **If specific file path provided**: Analyze that file
   ```bash
   ls -la <provided-path>
   ```

4. **Default (no args)**: Changed files only
   ```bash
   git diff --name-only main...HEAD | grep -E '\.tsx?$'
   ```

**Report mode to user:**
- `--all` mode: "Running FULL CODEBASE audit on N files"
- `--category=X`: "Running targeted audit for category: X"
- Default: "Reviewing N files changed vs main"

**Load context:**
- Read `.specify/memory/constitution.md` for principle references
- Check for previous review at `.claude/reviews/tech-debt-*.json` for trend comparison

---

### Phase 1: Parallel Category Analysis (Subagents)

**Invoke 8 category-specific analyzers IN PARALLEL (single message, multiple Task calls).**

Each subagent focuses on one category. Use **Haiku** for single-file analysis, **Sonnet** for cross-file patterns.

```
Task(boundary-analyzer, model=haiku, "Analyze files for Tool/Agent Boundary violations.

PATTERNS TO DETECT:
- JUDGMENT_IN_TOOL: Functions that return judgments ('good', 'bad', 'low', 'high')
- ORCHESTRATION_IN_TOOL: Tools that decide when/how to call other tools
- THRESHOLD_ENCODING: Hardcoded thresholds that encode decisions (if score < 0.3)
- DECISION_RETURN: Functions returning 'suggestedAction', 'shouldDo', etc.

FILES: [list]

For each finding, return:
- pattern: The pattern code (e.g., JUDGMENT_IN_TOOL)
- file: Absolute path
- line: Line number
- snippet: The offending code (max 5 lines)
- evidence: Why this matches the pattern

Return JSON array of findings. Return empty array if no issues.")

Task(prompt-analyzer, model=haiku, "Analyze files for Prompt Debt.

PATTERNS TO DETECT:
- HARDCODED_PROMPT: Prompt strings embedded directly in code (not in prompt files)
- NO_VERSIONING: Prompts without version tracking or timestamps
- INJECTION_RISK: User input concatenated into prompts without sanitization
- PROMPT_SPRAWL: Same prompt logic duplicated in multiple places

FILES: [list]

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(context-analyzer, model=haiku, "Analyze files for Context Window issues.

PATTERNS TO DETECT:
- UNBOUNDED_DATA: Arrays/objects passed to agent without size limits
- NO_SUMMARIZATION: Large data returned without truncation/summarization
- RAW_DUMP: Tool returning raw data structures instead of formatted summaries
- CONTEXT_BLOAT: Unnecessary verbose output in tool responses

FILES: [list]

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(testing-analyzer, model=haiku, "Analyze files for Testing Anti-patterns.

PATTERNS TO DETECT:
- LIVE_LLM_IN_UNIT: Unit tests that make real LLM API calls
- MISSING_VCR: Integration tests without VCR recordings
- MISSING_EVAL: Behavioral scenarios without TruLens evals
- FLAKY_ASSERTION: Tests asserting on non-deterministic LLM output
- NO_MOCK: Tests calling real external services

FILES: [list]
ADR-0011 Reference: Unit=mocked, Integration=VCR, E2E/Evals=live

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(error-analyzer, model=haiku, "Analyze files for Error Handling issues.

PATTERNS TO DETECT:
- NO_RETRY_LOGIC: LLM API calls without retry/backoff
- MISSING_TIMEOUT: API calls without timeout configuration
- SILENT_FAILURE: Errors caught but not logged/propagated
- UNHANDLED_REJECTION: Async operations without error handling
- GENERIC_CATCH: catch(e) without specific error handling

FILES: [list]

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(state-analyzer, model=haiku, "Analyze files for State Management issues.

PATTERNS TO DETECT:
- UNBOUNDED_HISTORY: Conversation history without max length
- MEMORY_LEAK: Growing state structures without cleanup
- NO_CHECKPOINT: Long-running operations without checkpointing
- STALE_STATE: Cached state without invalidation strategy

FILES: [list]

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(observability-analyzer, model=haiku, "Analyze files for Observability gaps.

PATTERNS TO DETECT:
- NO_TRACING: LLM calls without trace/span context
- MISSING_METRICS: No token counting, latency tracking
- NO_DEBUG_LOG: Complex operations without debug logging
- OPAQUE_ERROR: Errors without context for debugging

FILES: [list]

Return JSON array of findings with pattern, file, line, snippet, evidence.")

Task(subagent-analyzer, model=sonnet, "Analyze files for Subagent Architecture violations.

PATTERNS TO DETECT:
- DEPTH_VIOLATION: Subagents spawning sub-subagents (depth > 1)
- UNCLEAR_BOUNDARY: Subagent with multiple unrelated responsibilities
- MISSING_ISOLATION: Subagent sharing mutable state with parent
- RECURSIVE_SPAWN: Agents that can spawn themselves

FILES: [list]
Constitution C8 Reference: Subagent depth limited to 1

Return JSON array of findings with pattern, file, line, snippet, evidence.")
```

**Wait for all subagents to return.**

---

### Phase 2: Constitution Cross-Reference

**You handle this phase directly.**

For each finding from Phase 1, map to Constitution principles:

Reference: [Constitution Mapping](references/constitution-mapping.md)

| Pattern | Principle | Severity Modifier |
|---------|-----------|-------------------|
| JUDGMENT_IN_TOOL | VII | +2 (direct violation) |
| ORCHESTRATION_IN_TOOL | VII | +3 (critical) |
| INJECTION_RISK | I | +5 (security) |
| DEPTH_VIOLATION | C8 | +4 (architecture) |
| LIVE_LLM_IN_UNIT | ADR-0011 | +3 (testing strategy) |

---

### Phase 3: Severity Scoring

**You handle this phase directly.**

**Base deductions per finding type:**

Reference: [Severity Criteria](references/severity-criteria.md)

| Category | Pattern | Base Deduction |
|----------|---------|----------------|
| Boundary | JUDGMENT_IN_TOOL | -8 |
| Boundary | ORCHESTRATION_IN_TOOL | -10 |
| Boundary | THRESHOLD_ENCODING | -6 |
| Prompt | HARDCODED_PROMPT | -3 |
| Prompt | INJECTION_RISK | -15 |
| Prompt | PROMPT_SPRAWL | -4 |
| Context | UNBOUNDED_DATA | -7 |
| Context | RAW_DUMP | -5 |
| Testing | LIVE_LLM_IN_UNIT | -10 |
| Testing | MISSING_VCR | -6 |
| Testing | MISSING_EVAL | -8 |
| Error | NO_RETRY_LOGIC | -6 |
| Error | SILENT_FAILURE | -8 |
| State | UNBOUNDED_HISTORY | -10 |
| State | NO_CHECKPOINT | -5 |
| Observability | NO_TRACING | -5 |
| Observability | MISSING_METRICS | -4 |
| Subagent | DEPTH_VIOLATION | -12 |

**Scoring formula:**
```
score = max(0, 100 - sum(deductions))
```

**Score interpretation:**

| Score | Grade | Action |
|-------|-------|--------|
| 90-100 | A | Ship it |
| 80-89 | B | Minor cleanup before merge |
| 70-79 | C | Plan remediation sprint |
| 60-69 | D | Urgent fixes needed |
| < 60 | F | Block release |

---

### Phase 4: Generate Report

**You handle this phase directly.**

**Output format:**

```markdown
## Tech Debt Review Report

**Score**: XX/100 (Grade: X)
**Trend**: [+/-N vs last review] or [First review - no trend data]
**Files Analyzed**: N
**Scope**: [changed files | full codebase | category: X]

---

### Executive Summary

| Category | Findings | Impact | Constitution |
|----------|----------|--------|--------------|
| Boundary | N | -XX | VII |
| Testing | N | -XX | ADR-0011 |
| ... | ... | ... | ... |
| **Total** | **N** | **-XX** | |

---

### P0 - Block Release

These issues MUST be fixed before merge:

#### [PATTERN_CODE] in file.ts:123

**Code:**
```typescript
// The offending code snippet
```

**Problem**: [Why this is an anti-pattern]

**Constitution**: Violates Principle [X]

**Remediation**: [Specific fix guidance]

---

### P1 - This Sprint

[Summary findings - less detail than P0]

---

### P2 - Backlog

| Pattern | File | Line | Category |
|---------|------|------|----------|
| ... | ... | ... | ... |

---

### Historical Trend

| Date | Score | Grade | Change |
|------|-------|-------|--------|
| [today] | XX | X | - |
| [prev] | XX | X | +/-N |

---

### Anti-Pattern Reference

See: [AI Anti-Patterns Guide](references/ai-anti-patterns.md)
```

---

### Phase 5: Save Historical Data

**You handle this phase directly.**

Save review results to `.claude/reviews/tech-debt-YYYY-MM-DD.json`:

```json
{
  "version": "1.0.0",
  "timestamp": "2026-01-26T14:30:00Z",
  "scope": "changed_files|all|category:X",
  "filesAnalyzed": 145,
  "score": 78,
  "grade": "C",
  "categories": {
    "boundary": { "findings": 3, "impact": -23 },
    "testing": { "findings": 5, "impact": -30 },
    ...
  },
  "findings": [
    {
      "pattern": "JUDGMENT_IN_TOOL",
      "category": "boundary",
      "file": "src/tools/analyzer.ts",
      "line": 42,
      "snippet": "...",
      "severity": 8
    }
  ],
  "trend": {
    "previousScore": 72,
    "previousDate": "2026-01-19",
    "change": 6
  }
}
```

---

## Integration with Other Skills

| Skill | Relationship |
|-------|--------------|
| `/dev.test-review` | Complementary - test-review checks test quality, tech-debt-review checks testing anti-patterns |
| `/dev.verify-wiring` | Sequential - verify-wiring first, then tech-debt-review |
| `/arch-review` | Parallel - different focus areas |
| `/dev.pr` | Gate - score threshold can block PR |

**Suggested workflow:**
```
/dev.verify-wiring → /dev.tech-debt-review → /dev.test-review → /dev.pr
```

---

## Red Flags (Auto-Fail)

These patterns automatically set grade to F:

| Pattern | Why |
|---------|-----|
| INJECTION_RISK | Security vulnerability |
| DEPTH_VIOLATION | Architecture violation |
| Multiple ORCHESTRATION_IN_TOOL | Fundamental design issue |

---

## References

- [AI Anti-Patterns Guide](references/ai-anti-patterns.md)
- [Constitution Mapping](references/constitution-mapping.md)
- [Severity Criteria](references/severity-criteria.md)
- [ADR-0011: Testing Strategy](../../docs/architecture/adr/0011-testing-strategy-for-agentic-components.md)
- [ADR-0019: Tool/Agent Boundary](../../docs/architecture/adr/0019-tool-agent-boundary-temporal.md)
- [Constitution](../../.specify/memory/constitution.md)

---

## Handoff

After running this skill:

- **Grade A/B**: Proceed to `/dev.pr`
- **Grade C**: Review findings, plan fixes, proceed if time-boxed
- **Grade D/F**: Fix critical issues, re-run before PR
