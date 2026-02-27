---
name: 9d-framework
description: 9D product development framework
---

# 9D Framework

## Overview

The 9D Framework is the complete lifecycle for shipping features from idea to retirement.

```
Discovery → Definition → Design → Development → Documentation → Delivery → Data → Debrief → Deprecation
```

## The Nine Phases

### 1. Discovery
**Purpose**: Understand the problem

**Activities**:
- Research user needs
- Analyze existing solutions
- Identify constraints
- Document assumptions

**Output**: Discovery Brief
- Problem statement
- User context
- Business justification
- Initial scope

**Gate Criteria**:
- Problem clearly articulated
- User need validated
- Aligned with strategy

---

### 2. Definition
**Purpose**: Define what success looks like

**Activities**:
- Write requirements
- Define acceptance criteria
- Create experience spec (UX flows)
- Identify dependencies

**Output**: Requirements + Experience Spec
- Functional requirements
- Non-functional requirements
- UX mockups/flows
- Success metrics

**Gate Criteria**:
- Requirements complete and clear
- Experience designed
- Stakeholder sign-off

---

### 3. Design
**Purpose**: Plan the technical implementation

**Activities**:
- Architecture decisions
- API design
- Data model design
- Risk assessment

**Output**: Architecture Spec + Implementation Plan
- System design
- API contracts
- Data schemas
- Technical risks with mitigations

**Gate Criteria**:
- Architecture reviewed
- Implementation plan approved
- Risks documented

---

### 4. Development
**Purpose**: Build and test the solution

**Activities**:
- Write code
- Write tests
- Code review
- QA validation

**Output**: Working Code
- Feature implementation
- Unit tests
- Integration tests
- PR approved

**Gate Criteria**:
- All tests pass
- Code reviewed
- QA sign-off

---

### 5. Documentation
**Purpose**: Document for users before shipping

**Activities**:
- Write user documentation
- Write API documentation
- Create support materials
- Update runbooks

**Output**: Documentation Package
- User guides
- API reference
- FAQ/troubleshooting
- Internal runbooks

**Gate Criteria**:
- Docs reviewed
- Support team enabled
- Runbooks complete

---

### 6. Delivery
**Purpose**: Release to production safely

**Activities**:
- Staged rollout
- Monitoring setup
- Feature flags
- Communication

**Output**: Feature Live
- Production deployment
- Monitoring active
- Rollback plan ready

**Gate Criteria**:
- Metrics baseline established
- Rollback tested
- Team notified

---

### 7. Data
**Purpose**: Measure impact and validate success

**Activities**:
- Collect metrics
- Analyze usage
- Compare to targets
- Generate insights

**Output**: Data Report
- Usage metrics
- Success metrics vs targets
- User feedback summary
- Recommendations

**Gate Criteria**:
- Metrics collected
- Analysis complete
- Stakeholders informed

---

### 8. Debrief
**Purpose**: Learn and improve

**Activities**:
- Team retrospective
- Document learnings
- Identify improvements
- Update processes

**Output**: Debrief Document
- What went well
- What to improve
- Action items
- Process updates

**Gate Criteria**:
- Team participated
- Learnings captured
- Actions assigned

---

### 9. Deprecation
**Purpose**: End-of-life safely (when needed)

**Activities**:
- Plan migration path
- Communicate timeline
- Support transition
- Clean removal

**Output**: Clean Removal
- Migration guide
- Sunset communication
- Code removed
- Docs archived

**Gate Criteria**:
- Users migrated
- No active usage
- Clean removal

## When to Use 9D

| Scenario | Starting Phase |
|----------|----------------|
| New Feature | Discovery |
| Bug Fix | Development |
| Technical Debt | Definition |
| Experiment | Discovery |
| Feature Retirement | Deprecation |
