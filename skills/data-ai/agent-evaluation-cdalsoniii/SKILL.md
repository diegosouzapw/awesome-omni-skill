---
name: Agent Evaluation
description: Evaluate agent performance using a structured scoring rubric
category: ai-agents
version: 1.0.0
triggers:
  - agent-config-change
  - evaluation-cadence
  - performance-review
globs: "**/agents/**,**/skills/**"
---

# Agent Evaluation Skill

Evaluate agent performance using a structured scoring rubric.

## Trigger Conditions
- Agent configuration change
- Evaluation cadence (monthly)
- User invokes with "evaluate agents" or "agent scorecard"

## Input Contract
- **Required:** Agent(s) to evaluate
- **Required:** Evaluation criteria or rubric
- **Optional:** Baseline scores from prior evaluation

## Output Contract
- Evaluation scorecard (500-point rubric)
- Per-dimension scores and findings
- Improvement recommendations
- Comparison against baseline

## Tool Permissions
- **Read:** Agent configs, agent output logs, telemetry
- **Write:** Evaluation reports
- **Search:** Agent invocation history

## Execution Steps
1. Load evaluation rubric (architecture 100, security 100, ops 100, testing 100, docs 100)
2. For each agent, review recent outputs and effectiveness
3. Score each dimension with evidence
4. Compare against baseline scores
5. Identify improvement opportunities
6. Generate scorecard and recommendations

## Success Criteria
- All dimensions scored with evidence
- Comparison against prior evaluation
- Top 3 improvement recommendations per agent
- Overall portfolio health assessment

## Escalation Rules
- Escalate if any agent scores below 50% on any dimension
- Escalate if agent evaluation reveals conflicting outputs between agents

## Example Invocations

**Input:** "Evaluate the security-specialist agent effectiveness"

**Output:** Scorecard: Security (85/100), Architecture (70/100), Ops (75/100), Testing (60/100), Docs (80/100). Total: 370/500. Findings: strong CVE detection but weak test coverage recommendations, documentation quality high but missing escalation follow-through. Top improvement: integrate with testing-specialist for security test gap analysis.
