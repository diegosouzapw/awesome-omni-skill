---
name: assumption-validation
description: Test whether assumptions are true before making commitments. Use when assumptions have low certainty and high risk.
---

# Assumption Validation

## Overview
Test whether an assumption is true or false before making commitments based on it.

## When to Use
- When an assumption has low certainty and high risk
- Before major resource commitments
- When stakeholders challenge your reasoning
- During Empathize and Define phases

## How to Apply

### 1. State the Assumption Clearly
Make it testable. Transform vague beliefs into specific claims:
- ❌ "Users want better tools"
- ✅ "Users spend >2 hours/week on manual data entry and would use an automated solution"

### 2. Define What Would Validate or Invalidate
Be explicit about criteria:
- **Validates**: 8+ out of 10 users report >2 hrs/week on manual entry
- **Invalidates**: <5 out of 10 users report this, or they prefer manual control
- **Inconclusive**: Mixed results, need different approach

### 3. Choose Validation Method
Match method to assumption type:

**User behavior/needs**: Interviews, observation, surveys
**Technical feasibility**: Spikes, prototypes, vendor demos
**Market conditions**: Market research, competitor analysis
**Business viability**: Financial modeling, expert consultation

### 4. Execute Validation
Conduct research with focus on disproving, not confirming:
- Ask open-ended questions
- Observe actual behavior, not just stated preferences
- Look for contradictory evidence
- Talk to diverse user types

### 5. Update Status
Record findings in currentstate.json:
- **Validated**: Evidence supports the assumption
- **Invalidated**: Evidence contradicts it
- **Partially validated**: More complex than assumed
- **Needs more research**: Inconclusive

### 6. Act on Findings

**If validated**: Proceed with confidence, but stay alert for new evidence

**If invalidated**: 
- Update problem framing
- Revise approach
- Generate new assumptions
- May need to pivot

**If partially validated**:
- Refine the assumption
- Identify what's true and what's not
- Adjust plans accordingly

## Example

**Assumption**: "Field technicians need offline access"

**Validation Plan**:
- Interview 8 field technicians
- Ask about connectivity at work locations
- Observe their current workarounds
- Ask what happens when connection drops

**Findings**:
- 7/8 work in areas with spotty connectivity
- All have experienced data loss from connection drops
- All use workarounds (paper notes, photos) when offline
- Strong preference for offline-first design

**Result**: VALIDATED — Offline access is a critical requirement

**Action**: Prioritize offline functionality in ideation phase

## Tips
- Seek to disprove, not confirm
- Sample diverse users, not just friendly ones
- Observe behavior, don't just ask
- Document exact evidence, not interpretations
- Update currentstate.json immediately
- One assumption may spawn new assumptions
