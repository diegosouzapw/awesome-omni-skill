---
name: spec-kit-agent-orchestrator
description: Generate workflow prompts and scripts for multi-agent orchestration using Speckit methodology. Create templates for agent handoffs, task delegation, and workflow management for agentic development processes.
---

# Spec-Kit & Agent Orchestrator

## Overview
Generate workflow prompts and scripts for multi-agent orchestration using Speckit methodology for Phase 5 agentic development processes.

## Core Components

### 1. Speckit Commands Guide
How to run Speckit commands for project management.

```markdown
# agents/workflow-guide.md

# Speckit Commands Guide

## Core Speckit Commands

### 1. /speckit.clarify
Purpose: Clarify project requirements, specifications, and constraints.

Usage:
```
/speckit.clarify
```

When to use:
- At the beginning of a project to clarify requirements
- When specifications are ambiguous or incomplete
- When encountering conflicting requirements
- When needing to validate assumptions

What it does:
- Reviews speckit.constitution for guiding principles
- Clarifies speckit.specify requirements
- Identifies gaps in specifications
- Suggests clarifications for unclear requirements

### 2. /plan
Purpose: Generate detailed implementation plan based on specifications.

Usage:
```
/plan
```

When to use:
- After requirements are clarified
- Before starting implementation work
- When modifying existing architecture
- When adding new features

What it does:
- Creates detailed implementation plan (speckit.plan)
- Outlines architectural components
- Defines data flows and dependencies
- Identifies risks and mitigation strategies

### 3. /tasks
Purpose: Generate specific, actionable tasks for implementation.

Usage:
```
/tasks
```

When to use:
- After plan is approved
- To break down complex features into manageable tasks
- When assigning work to team members
- For tracking progress

What it does:
- Creates granular task breakdown (speckit.tasks)
- Assigns priorities to tasks
- Defines task dependencies
- Links tasks to specifications and plans
```

### 2. Multi-Agent Handoff Templates
Templates for delegating work between specialized agents.

```markdown
# Multi-Agent Handoff Templates

## General Handoff Template
```
Delegate to [AGENT_ROLE]: [SPECIFIC_TASK_DESCRIPTION]

Context:
- Current task: [TASK_ID]
- Related to: [SPECIFICATION_SECTION]
- Deadline: [DATE_IF_APPLICABLE]

Requirements:
- Follow: [CONSTITUTION_PRINCIPLE]
- Reference: [PLAN_SECTION]
- Output format: [EXPECTED_FORMAT]

Acceptance Criteria:
- [CRITERIA_1]
- [CRITERIA_2]
- [CRITERIA_3]
```

## Specific Handoff Examples

### Handoff to Dapr Specialist
```
Delegate to Dapr Specialist: Generate Dapr component configurations for Kafka integration

Context:
- Current task: T-101
- Related to: speckit.specify §3.2 (Event-driven architecture)
- Deadline: Tomorrow

Requirements:
- Follow: Constitution Principle 2 (Dapr full abstraction)
- Reference: Plan §2.1 (Dapr Components)
- Output format: ```yaml dapr-components/pubsub-kafka.yaml

Acceptance Criteria:
- Uses Redpanda brokers
- Includes proper authentication
- Follows security best practices
- Integrates with existing pubsub pattern
```

### Handoff to Infrastructure Specialist
```
Delegate to Infrastructure Specialist: Create Kubernetes deployment manifests

Context:
- Current task: T-102
- Related to: speckit.specify §4.1 (Deployment requirements)
- Deadline: End of sprint

Requirements:
- Follow: Constitution Principle 4 (Security first)
- Reference: Plan §4.2 (K8s deployment strategy)
- Output format: ```yaml kubernetes/deployments/chat-api.yaml

Acceptance Criteria:
- Includes Dapr sidecar annotations
- Has proper resource limits
- Implements health checks
- Follows naming conventions
```

### Handoff to Features Engineer
```
Delegate to Advanced Features Engineer: Implement recurring task functionality

Context:
- Current task: T-103
- Related to: speckit.specify §2.3 (Feature requirements)
- Deadline: Next milestone

Requirements:
- Follow: Constitution Principle 5 (Performance)
- Reference: Plan §3.1 (Feature implementation)
- Output format: ```python services/recurring/main.py

Acceptance Criteria:
- Implements recurrence patterns
- Handles task completion events
- Integrates with Dapr pubsub
- Follows async patterns
```

### Handoff to Project Coordinator
```
Delegate to Project Coordinator: Validate integration between components

Context:
- Current task: T-104
- Related to: speckit.specify §5.1 (Integration requirements)
- Deadline: Before release

Requirements:
- Follow: Constitution Principle 1 (Decoupled systems)
- Reference: Plan §6.1 (Integration testing)
- Output format: ```markdown integration-validation.md

Acceptance Criteria:
- Verifies component compatibility
- Tests end-to-end workflows
- Documents integration points
- Identifies potential issues
```
```

### 3. CLAUDE.md Updates Template
Template for maintaining the CLAUDE.md file with agent instructions.

```markdown
# CLAUDE.md Updates Template

## Adding New Agent Roles
```
## [AGENT_NAME]
Role: [ROLE_DESCRIPTION]
Responsibilities:
- [RESPONSIBILITY_1]
- [RESPONSIBILITY_2]
- [RESPONSIBILITY_3]

Rules:
- [RULE_1]
- [RULE_2]
- [RULE_3]

Output Format:
- [FORMAT_1]
- [FORMAT_2]

References:
- [CONSTITUTION_PRINCIPLE]
- [PLAN_SECTION]
```

## Updating Existing Agent Instructions
```
Current Instructions:
[EXISTING_INSTRUCTIONS]

Updated Instructions:
[NEW_INSTRUCTIONS]

Reason for Change:
[JUSTIFICATION]

Impact Assessment:
- [IMPACT_1]
- [IMPACT_2]
```

## Adding New Workflow Patterns
```
## [WORKFLOW_NAME]
Trigger: [WHEN_TO_USE]
Steps:
1. [STEP_1]
2. [STEP_2]
3. [STEP_3]

Agents Involved:
- [AGENT_1]: [ROLE]
- [AGENT_2]: [ROLE]

Expected Output:
- [OUTPUT_1]
- [OUTPUT_2]
```
```

### 4. Agent Orchestration Workflows
Workflows for coordinating multiple agents.

```markdown
# Agent Orchestration Workflows

## Feature Development Workflow
```
1. Project Coordinator initiates feature development
2. /speckit.clarify to validate requirements
3. /plan to create implementation strategy
4. /tasks to break down work
5. Delegate to Advanced Features Engineer for business logic
6. Delegate to Dapr Specialist for integration components
7. Delegate to Infrastructure Specialist for deployment
8. Project Coordinator validates integration
9. Quality Assurance reviews implementation
10. Project Coordinator approves for deployment
```

## Bug Fix Workflow
```
1. Quality Assurance identifies bug
2. Project Coordinator assesses impact
3. /tasks to create fix tasks
4. Delegate to appropriate specialist based on component
5. Specialist implements fix
6. Quality Assurance verifies fix
7. Project Coordinator approves for deployment
```

## Architecture Review Workflow
```
1. Project Coordinator initiates architecture review
2. /speckit.clarify to validate requirements
3. /plan to create architecture plan
4. Delegate to Infrastructure Specialist for infrastructure design
5. Delegate to Dapr Specialist for integration design
6. All specialists review cross-component impacts
7. Project Coordinator approves architecture
8. /tasks to break down implementation
```

## Release Workflow
```
1. Project Coordinator initiates release process
2. Quality Assurance confirms all tests pass
3. Infrastructure Specialist validates deployment readiness
4. Dapr Specialist confirms component compatibility
5. Advanced Features Engineer confirms feature completeness
6. Project Coordinator approves release
7. GitHub Actions executes deployment
8. Monitoring confirms successful deployment
```
```

### 5. Speckit Task Management
Guidelines for managing tasks through the Speckit system.

```markdown
# Speckit Task Management

## Creating Tasks
When creating tasks via /tasks:

1. **Task ID Format**: Use T-XXX format (e.g., T-101, T-102)
2. **Task Description**: Clear, specific, and actionable
3. **Dependencies**: Identify any prerequisite tasks
4. **Priority**: High, Medium, or Low
5. **Estimate**: Time estimate in hours/days
6. **Assignee**: Relevant agent role
7. **Acceptance Criteria**: Clear definition of done

Example Task:
```
Task: T-101
Description: Implement recurring task creation endpoint
Dependencies: T-100 (Database schema)
Priority: High
Estimate: 4 hours
Assignee: Advanced Features Engineer
Acceptance Criteria:
- Endpoint accepts recurring task parameters
- Validates recurrence pattern
- Creates task in database
- Publishes creation event via Dapr pubsub
- Follows async patterns
```

## Task Status Tracking
- **Pending**: Task created, waiting for assignment
- **In Progress**: Work actively being done
- **Blocked**: Waiting on dependencies or clarifications
- **Review**: Completed, awaiting review
- **Done**: Approved and completed
- **Cancelled**: No longer needed

## Task Handoff Protocol
1. Ensure all context is documented
2. Update task status to reflect handoff
3. Notify receiving agent
4. Provide clear acceptance criteria
5. Establish timeline expectations
```

### 6. Quality Assurance Checklist
Checklist for validating agent work.

```markdown
# Quality Assurance Checklist

## Before Accepting Agent Output

### For Code Generation (Advanced Features Engineer):
- [ ] Follows async patterns (async def, await)
- [ ] Uses Dapr client properly (localhost:3500)
- [ ] Includes proper error handling
- [ ] Has type hints and documentation
- [ ] Follows security best practices
- [ ] References correct constitution principles
- [ ] Matches plan specifications

### For Infrastructure (Infrastructure Specialist):
- [ ] YAML syntax is valid
- [ ] Proper resource limits and requests
- [ ] Security contexts applied
- [ ] Health checks configured
- [ ] Dapr annotations present
- [ ] Follows naming conventions
- [ ] Matches plan specifications

### For Dapr Components (Dapr Specialist):
- [ ] Component syntax is correct
- [ ] Proper authentication configured
- [ ] Security best practices followed
- [ ] References secrets properly
- [ ] Matches plan specifications
- [ ] Follows constitution principles

### For All Outputs:
- [ ] Task ID properly referenced
- [ ] Sources properly cited (constitution, plan, specify)
- [ ] Output format matches requirements
- [ ] No hardcoded values where dynamic expected
- [ ] Follows project conventions
```

### 7. Communication Protocols
Protocols for agent communication and coordination.

```markdown
# Communication Protocols

## Agent-to-Agent Communication
When agents need to communicate:

1. **Use Task References**: Always reference the relevant task (T-XXX)
2. **Be Specific**: Clearly state what information is needed
3. **Provide Context**: Include relevant sections from constitution/specify/plan
4. **Set Expectations**: Define timeline and format requirements

Example:
```
Hi Dapr Specialist,

Regarding task T-105 (Implement pubsub for task events):

I need the pubsub component configuration that will allow my service to publish task events. 

Context:
- From: speckit.specify §3.2
- References: Plan §2.1
- Constitution: Principle 2 (Dapr full abstraction)

Requirements:
- Should support task-events topic
- Need to handle authentication
- Should be compatible with Redpanda

Could you provide the pubsub-kafka.yaml component file by EOD tomorrow?

Thanks,
Advanced Features Engineer
```

## Escalation Procedures
1. **Technical Disagreements**: Escalate to Project Coordinator
2. **Timeline Conflicts**: Escalate to Project Coordinator
3. **Specification Ambiguities**: Use /speckit.clarify
4. **Resource Constraints**: Escalate to Project Coordinator
5. **Quality Concerns**: Escalate to Quality Assurance

## Status Reporting
Agents should report status:
- **Daily Standup**: Brief progress update
- **Blockers**: Immediately when encountered
- **Task Completion**: Within 1 hour of completion
- **Critical Issues**: Immediately
```

## Implementation Guidelines

### 1. No Manual Code Creation
All code must be generated through proper task delegation:
- Create task via /tasks
- Delegate to appropriate agent
- Validate output through QA checklist
- Never write code directly without task reference

### 2. Consistent Referencing
Always reference:
- speckit.constitution for principles
- speckit.specify for requirements
- speckit.plan for implementation details
- speckit.tasks for work breakdown

### 3. Agent Specialization
Respect agent boundaries:
- Don't ask Infrastructure Specialist to write business logic
- Don't ask Features Engineer to create Kubernetes manifests
- Don't ask Dapr Specialist to implement algorithms
- Use Project Coordinator for cross-cutting concerns

## Best Practices

### 1. Incremental Development
- Break large features into smaller tasks
- Validate early and often
- Use iterative refinement
- Maintain working state at each step

### 2. Documentation
- Keep CLAUDE.md updated
- Document decisions and rationale
- Maintain clear task descriptions
- Record integration points

### 3. Validation
- Use QA checklists consistently
- Validate against specifications
- Test integration points
- Verify security compliance

This orchestrator ensures that all development work flows through the proper channels with appropriate specialization, validation, and documentation.