---
name: jules-audit-request
description: Protocol for escalation to Jules when stuck.
trigger: Unsolvable Loop OR Stuck OR Resource Limit OR Help Jules
scope: global
---

# SKILL: Jules Audit Request (The Escape Hatch)

> **Trigger**: "Unsolvable Loop", "Stuck", "Resource Limit", "Help Jules"

## Context
When a local agent (Gentleman, Raphael, Alicia) hits a wall—whether technical (infinite loop, build fail) or architectural (confusion)—it must NOT keep spinning. It must escalate to **Jules** (Google's Agent).

## The Protocol

1.  **Stop**: Cease current execution.
2.  **Compilate**: Gather the "Flight Box" data:
    - Current Task/Goal
    - Recent Errors
    - File Context (Files actively being touched)
    - What has been tried
3.  **Generate**: Produce the **Jules Prompt** below.
4.  **Handover**: User copies this prompt to Jules.

---

### template: JULES_AUDIT_PROMPT

```markdown
@Jules (Google Internal Agent)

# 🚨 PRIORITY AUDIT REQUEST: [Project Name]

## 1. Context & Objective
I am an autonomous agent working on [Project Name].
**Goal**: [One sentence description of what we are trying to do]

## 2. The Blocker
I am stuck in a loop/error state.
**Last Error**:
```text
[Paste error log or description here]
```

## 3. Current State
- **Active Files**: [List key files]
- **Attempted Solutions**:
  1. [Attempt 1]
  2. [Attempt 2]

## 4. Request
Please audit this situation.
1. **Analyze**: Why is the current approach failing?
2. **Strategy**: Propose a sanitized, efficient path forward.
3. **Constraint**: Keep it simple. We are resource-constrained (20GB RAM limit).

> "Jules, bring the light."
```
