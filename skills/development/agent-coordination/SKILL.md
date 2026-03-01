---
name: agent-coordination
description: Reference documentation for multi-agent coordination framework. Provides protocols, communication standards, and best practices for agent collaboration. Use when understanding agent workflows, coordination protocols, or multi-agent architecture. Triggers on "coordination", "agent protocol", "multi-agent", "workflow rules", "agent communication".
---

# Agent Coordination Framework

## Overview

This skill provides the reference documentation for multi-agent coordination, including communication protocols, workflow standards, and collaboration patterns.

## Mandatory Reporting Protocol

All specialized work MUST follow this communication protocol:

### 1. Before Work (Authorization Request)
```
"Requesting authorization for [task]"
"Scope: [detailed scope]"
"Dependencies: [other skills/tasks]"
"Estimated time: [duration]"
```

### 2. During Work (Progress Updates)
Required every 30-60 minutes:
```
"Progress update"
"Completed: [specific accomplishments]"
"Current: [what's in progress]"
"Next: [next steps]"
"Blockers: [any issues]"
"Commits: [commits made since last update]"
```

### 3. After Work (Completion Report)
```
"Completed [task]"
"Deliverables: [what was produced]"
"Files changed: [list of files]"
"Commits made: [list of commits]"
"Tests: [test status]"
"Ready for: [next steps]"
```

## Skill Ecosystem

### Skill Capabilities

| Skill | Specialization | Tools |
|-------|----------------|-------|
| Orchestration | Multi-agent coordination | All |
| Project Planning | Strategic architecture | Planning tools |
| Backend Development | Firebase, Zustand | Backend tools |
| UI Development | React, Tailwind | Frontend tools |
| Testing Suite | Vitest, Playwright | Testing tools |
| Research | Documentation gathering | Web tools |
| Documentation | Technical writing | Edit tools |
| Version Control | Git, releases | Git tools |
| UX Design | User experience | Read tools |
| Improvement Analysis | System optimization | Analysis tools |

### Coordination Patterns

#### Sequential Workflow
```
Planning → Research → Backend → UI → Testing → Documentation → Version Control
```

#### Parallel Opportunities
```
Research + Planning (parallel)
Backend + early UI prototyping (parallel)
Testing + Documentation (parallel)
```

## Commit Frequency Protocol

### Mandatory Intervals
- **Maximum**: 60 minutes between commits
- **Recommended**: 30-45 minutes for active work
- **Required triggers**:
  - After each component completion
  - After tests added/updated
  - After integration work
  - After documentation updates
  - Before requesting review

### Commit Message Format
```bash
git commit -m "$(cat <<'EOF'
type(scope): description

- Detailed changes
- List modifications

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## Quality Gates

### Phase Transitions
Before progressing between workflow phases:
1. Documentation complete and accurate
2. Types properly defined
3. Tests written and passing
4. Tech stack compliance verified
5. Error handling implemented

### Approval Requirements
- **Standard tasks**: Skill self-validates
- **Complex tasks**: Orchestration approval required
- **Critical changes**: User approval required

## Complexity Assessment

### Scoring System
- Simple file edit: 1
- Single component creation: 2
- Data model creation: 3
- Service implementation: 4
- UI component development: 5
- Testing integration: 6
- Multi-component feature: 8
- Full feature with backend and UI: 9
- System architecture changes: 10

### Orchestration Triggers
- IF complexity_score >= 7 THEN orchestration_required
- IF multiple_skills_needed THEN coordination_required
- IF backend_AND_frontend THEN orchestration_required
- IF testing_mentioned THEN testing_skill_activated

## Conflict Prevention

### Branch Isolation
- Each feature on dedicated branch
- Clear ownership during development phases
- Scheduled merge points
- Independent test suites per feature

### File Ownership
- Explicit ownership during active work
- No concurrent edits to same files
- Clear handoff protocols

## Emergency Protocols

### Work Failure Recovery
1. Immediate pause all related work
2. Root cause analysis
3. Rapid replanning
4. Orchestration review
5. Coordinated resume

### Conflict Resolution
1. Conflict detection
2. Immediate escalation
3. Work stream isolation
4. Resolution planning
5. Validated resume

## Success Metrics

- **Zero Conflicts**: No merge conflicts between parallel work
- **100% Coordination**: All skills working from synchronized context
- **>99% Approval**: All work passes quality review
- **Measurable Outcomes**: Every task has specific success metrics
- **Complete Documentation**: All deliverables properly documented
