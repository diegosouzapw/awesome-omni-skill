---
name: architecture-decisions
description: Make and document architecture decisions using structured frameworks
version: "2.0.0"
sasmp_version: "1.3.0"
bonded_agent: 01-architecture-fundamentals
bond_type: PRIMARY_BOND
last_updated: "2025-01"
---

# Architecture Decisions Skill

## Purpose
Enable structured architecture decision-making through quality attribute analysis, trade-off evaluation, and technology selection using industry-standard frameworks.

---

## Parameters

| Parameter | Type | Required | Validation | Default |
|-----------|------|----------|------------|---------|
| `decision_context` | string | ✅ | min: 30 chars | - |
| `decision_type` | enum | ⚪ | technology\|pattern\|tradeoff | `tradeoff` |
| `quality_priorities` | array | ⚪ | max: 5 items | `["performance", "maintainability"]` |
| `constraints` | object | ⚪ | valid JSON | `{}` |
| `options` | array | ⚪ | min: 2 items | - |

---

## Execution Flow

```
┌──────────────────────────────────────────────────────────┐
│ 1. VALIDATE: Check input parameters                       │
│ 2. CONTEXTUALIZE: Understand problem domain               │
│ 3. IDENTIFY: List quality attributes and constraints      │
│ 4. ANALYZE: Evaluate options against criteria             │
│ 5. SCORE: Create decision matrix                          │
│ 6. RECOMMEND: Provide primary + alternatives              │
│ 7. DOCUMENT: Generate ADR content                         │
└──────────────────────────────────────────────────────────┘
```

---

## Retry Logic

| Error | Retry | Backoff | Max Attempts |
|-------|-------|---------|--------------|
| `VALIDATION_ERROR` | No | - | 1 |
| `CONTEXT_UNCLEAR` | Yes | 1s, 2s, 4s | 3 |
| `INSUFFICIENT_OPTIONS` | Yes | - | 2 |

---

## Logging & Observability

```yaml
log_points:
  - event: skill_invoked
    level: info
    data: [decision_type, quality_priorities]
  - event: analysis_complete
    level: info
    data: [options_count, top_recommendation]
  - event: error_occurred
    level: error
    data: [error_type, context]

metrics:
  - name: decision_time_ms
    type: histogram
  - name: options_evaluated
    type: counter
  - name: confidence_score
    type: gauge
```

---

## Error Handling

| Error Code | Description | Recovery |
|------------|-------------|----------|
| `E001` | Missing decision context | Request clarification |
| `E002` | Conflicting quality attributes | Prioritization dialog |
| `E003` | Insufficient options to compare | Request more alternatives |
| `E004` | Unknown technology domain | Defer to research |

---

## Unit Test Template

```yaml
test_cases:
  - name: "Database selection decision"
    input:
      decision_context: "E-commerce order management system"
      decision_type: "technology"
      quality_priorities: ["reliability", "performance"]
      options: ["PostgreSQL", "MongoDB"]
    expected:
      has_recommendation: true
      has_rationale: true
      confidence_gte: 0.7

  - name: "Missing context error"
    input:
      decision_context: ""
    expected:
      error_code: "E001"

  - name: "Microservices vs Monolith"
    input:
      decision_context: "Startup MVP with 4 developers"
      decision_type: "pattern"
      quality_priorities: ["deployability", "maintainability"]
    expected:
      has_trade_offs: true
      alternatives_count_gte: 1
```

---

## Troubleshooting

### Common Issues

| Symptom | Root Cause | Resolution |
|---------|------------|------------|
| Vague recommendation | Context too broad | Narrow scope, add constraints |
| Analysis paralysis | Too many options | Limit to top 3-4 viable options |
| Low confidence score | Missing information | Request specific metrics/requirements |

### Debug Checklist
```
□ Is problem domain clearly defined?
□ Are quality attributes prioritized?
□ Are all options technically viable?
□ Are constraints explicitly stated?
□ Is success criteria measurable?
```

---

## Examples

### Example: Technology Selection
```yaml
Input:
  decision_context: "Real-time inventory system for retail"
  decision_type: "technology"
  quality_priorities: ["performance", "scalability"]
  options: ["Redis", "PostgreSQL", "MongoDB"]

Output:
  recommendation: "Redis for hot data + PostgreSQL for persistence"
  confidence: 0.85
  trade_offs:
    - "Redis: Fast but requires cache invalidation strategy"
    - "PostgreSQL: ACID compliant but higher latency"
  adr_content: |
    # ADR: Hybrid Redis + PostgreSQL for Inventory
    ## Decision: Use Redis for real-time inventory counts, PostgreSQL for order data
    ## Rationale: Balances performance needs with data durability
```

---

## Integration

| Component | Trigger | Data Flow |
|-----------|---------|-----------|
| Agent 01 | Decision request | Receives context, returns recommendation |
| Agent 02 | ADR creation | Provides decision content for documentation |

---

## Quality Standards

- **Atomic:** Single decision per invocation
- **Traceable:** All decisions link to rationale
- **Reversible:** Document rollback strategy when applicable

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2025-01 | Production-grade: parameters, retry logic, tests |
| 1.0.0 | 2024-12 | Initial release |
