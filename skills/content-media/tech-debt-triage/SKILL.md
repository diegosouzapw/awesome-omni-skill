---
name: Tech Debt Triage
description: Score, prioritize, and plan technical debt remediation
category: planning
version: 1.0.0
triggers:
  - sprint-planning
  - quarterly-review
  - debt-assessment
globs: "**/docs/**,**/planning/**"
---

# Tech Debt Triage Skill

Score, prioritize, and plan technical debt remediation.

## Trigger Conditions
- Sprint planning session
- Quarterly tech debt review
- User invokes with "triage tech debt" or "debt assessment"

## Input Contract
- **Required:** Codebase metrics (churn, complexity, defect density)
- **Optional:** Developer pain surveys, postmortem findings, code review history

## Output Contract
- Scored and prioritized debt backlog
- Quick wins identified (low effort, high impact)
- Remediation roadmap with effort estimates
- Contagion analysis (debt spreading to new code)

## Tool Permissions
- **Read:** Code metrics, git history, defect tracker, postmortems
- **Write:** Tech debt reports, backlog items
- **Search:** Code hotspots and churn patterns

## Execution Steps
1. Collect code metrics: churn, complexity, defect density
2. Identify hotspots (high churn + high complexity)
3. Score each debt item: impact (blast radius), effort (team-days), contagion
4. Identify quick wins (effort <1d, impact >medium)
5. Check for recurring patterns from postmortems
6. Create prioritized backlog
7. Estimate remediation effort for top items

## Success Criteria
- All debt items scored on 3 dimensions
- Quick wins identified and flagged
- Top 10 items have effort estimates
- Contagion patterns identified

## Escalation Rules
- Escalate if debt item blocks feature delivery
- Escalate if contagion is spreading to >3 modules
- Escalate if debt remediation requires >1 sprint of dedicated work

## Example Invocations

**Input:** "Triage tech debt for the order processing module"

**Output:** 14 debt items identified. Top 3: 1) OrderProcessor god class (impact: 9, effort: 3d, contagion: HIGH — 6 modules depend on it), 2) Missing retry logic in payment integration (impact: 8, effort: 1d, contagion: MEDIUM), 3) Hardcoded config values (impact: 5, effort: 0.5d, contagion: LOW — quick win). Remediation roadmap: sprint N (quick wins), sprint N+1 (retry logic), sprint N+2-N+3 (OrderProcessor decomposition).
