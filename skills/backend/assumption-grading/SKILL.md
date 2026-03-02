---
name: assumption-grading
description: Assess assumptions on certainty and risk to prioritize validation efforts. Use at project start or before phase transitions.
---

# Assumption Grading

## Overview
Systematically assess assumptions to identify which ones need validation and prioritize research efforts.

## When to Use
- At project start to grade initial assumptions
- During Define phase when framing the problem
- Before phase transitions to assess remaining risks
- When deciding what to research next

## How to Apply

### 1. List Assumptions
Identify what you're assuming to be true. Common sources:
- Beliefs about user behavior
- Market or business conditions
- Technical feasibility claims
- Resource availability
- Timeline estimates

### 2. Grade Each Assumption
Assess on two dimensions:

**CERTAINTY** — How confident are we this is true?
- **High**: Strong evidence, validated with users/data
- **Medium**: Some evidence, but not thoroughly validated
- **Low**: Gut feeling, no validation yet

**RISK** — What's the impact if we're wrong?
- **High**: Project fails, major wasted resources, wrong direction
- **Medium**: Significant rework needed, delays, budget impact
- **Low**: Minor adjustments, easy to correct

### 3. Prioritize Validation
Focus on assumptions that are:
1. **Low certainty + High risk** — VALIDATE IMMEDIATELY
2. **Low certainty + Medium risk** — Validate before major commitments
3. **Medium certainty + High risk** — Validate before proceeding
4. **High certainty + Low risk** — Can proceed with monitoring

### 4. Plan Validation
For each high-priority assumption, define:
- What would prove this true or false?
- What's the cheapest/fastest way to test it?
- Who can provide evidence?
- When do we need to know?

### 5. Update currentstate.json
Record each assumption with:
```json
{
  "id": "a1",
  "description": "Users want automated reporting",
  "certainty": "low",
  "risk": "high",
  "validation_plan": "Interview 5 users about reporting needs and current workflows",
  "status": "open"
}
```

## Example Assessment

**Assumption**: "Users will adopt mobile app over desktop"
- **Certainty**: Low (based on industry trends, not our users)
- **Risk**: High (entire platform strategy depends on this)
- **Priority**: VALIDATE IMMEDIATELY
- **Validation Plan**: Interview current users about device usage patterns and preferences

**Assumption**: "API can handle 1000 requests/second"
- **Certainty**: Medium (vendor specs, but not tested)
- **Risk**: High (performance is core requirement)
- **Priority**: Validate before prototyping
- **Validation Plan**: Load testing with production-like data

**Assumption**: "Users understand technical jargon"
- **Certainty**: Low (no evidence)
- **Risk**: Low (can adjust language easily)
- **Priority**: Test during iteration, easy to fix

## Tips
- Be explicit about what you're assuming
- Don't confuse hope with certainty
- High certainty still means "could be wrong"
- Update grades as you learn
- Re-grade assumptions at phase transitions
