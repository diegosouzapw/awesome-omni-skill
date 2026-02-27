---
name: Implementing Features
description: Execute specification-driven implementation with automatic quality gates, multi-agent orchestration, and progress tracking. Use when building features from specs, fixing bugs with test coverage, or refactoring with validation.
allowed-tools: [Read, Write, Edit, MultiEdit, Bash, Glob, Grep, TodoWrite, Task]
---

# Implementing Features

I help you execute production-quality implementations with auto-detected language standards, intelligent agent orchestration, and specification integration.

## When to Use Me

**Auto-activate when:**
- Invoked via `/quaestor:implement` slash command
- User mentions "build [specific feature]" or "fix [specific bug]" with context
- Continuing implementation after planning phase is complete
- User says "continue implementation" or "resume implementing"
- Coordinating multi-agent implementation of an active specification

**Do NOT auto-activate when:**
- User says only "implement" or "implement it" (slash command handles this)
- User is still in planning/research phase
- Request is vague without feature details

## Supporting Files

This skill uses several supporting files for detailed workflows:

- **@WORKFLOW.md** - 4-phase implementation process (Discovery → Planning → Implementation → Validation)
- **@AGENTS.md** - Agent orchestration strategies and coordination patterns
- **@QUALITY.md** - Language-specific quality standards and validation gates
- **@SPECS.md** - Specification integration and tracking protocols

## My Process

I follow a structured 4-phase workflow to ensure quality and completeness:

### Phase 1: Discovery & Research 🔍

**Specification Integration:**
- Check `.quaestor/specs/active/` for in-progress work
- Search `.quaestor/specs/draft/` for matching specifications
- Move draft spec → active folder (if space available, max 3)
- Update spec status → "in_progress"

**Research Protocol:**
- Analyze codebase patterns & conventions
- Identify dependencies & integration points
- Determine required agents based on task requirements

**See @WORKFLOW.md Phase 1 for complete discovery process**

### Phase 2: Planning & Approval 📋

**Present Implementation Strategy:**
- Architecture decisions & trade-offs
- File changes & new components required
- Quality gates & validation approach
- Risk assessment & mitigation

**MANDATORY: Get user approval before proceeding**

**See @WORKFLOW.md Phase 2 for planning details**

### Phase 3: Implementation ⚡

**Agent Orchestration:**
- **Multi-file operations** → Use researcher + implementer agents
- **System refactoring** → Use architect + refactorer agents
- **Test creation** → Use qa agent for comprehensive coverage
- **Security implementation** → Use security + implementer agents

**Quality Cycle** (every 3 edits):
```
Execute → Validate → Fix (if ❌) → Continue
```

**See @AGENTS.md for complete agent coordination strategies**

### Phase 4: Validation & Completion ✅

**Quality Validation:**
1. Detect project language (Python, Rust, JS/TS, Go, or Generic)
2. Load language-specific standards from @QUALITY.md
3. Run validation pipeline for detected language
4. Fix any issues and re-validate

**Completion Criteria:**
- ✅ All tests passing
- ✅ Zero linting errors
- ✅ Type checking clean (if applicable)
- ✅ Documentation complete
- ✅ Specification status updated

**See @QUALITY.md for dispatch to language-specific standards:**
- `@languages/PYTHON.md` - Python projects
- `@languages/RUST.md` - Rust projects
- `@languages/JAVASCRIPT.md` - JS/TS projects
- `@languages/GO.md` - Go projects
- `@languages/GENERIC.md` - Other languages

## Auto-Intelligence

### Project Detection
- **Language**: Auto-detect → Python|Rust|JS|Generic standards
- **Scope**: Assess changes → Single-file|Multi-file|System-wide
- **Context**: Identify requirements → architecture|security|testing|refactoring

### Execution Strategy
- **System-wide**: Comprehensive planning with multiple agent coordination
- **Feature Development**: Iterative implementation with testing
- **Bug Fixes**: Focused resolution with validation

## Agent Coordination

**I coordinate with specialized agents based on task requirements:**

- **workflow-coordinator** - First! Validates workflow state and ensures planning phase completed
- **implementer** - Builds features according to specification
- **architect** - Designs system architecture when needed
- **security** - Reviews auth, encryption, or access control
- **qa** - Creates comprehensive tests alongside implementation
- **refactorer** - Ensures consistency across multiple files
- **researcher** - Maps dependencies for multi-file changes

**See @AGENTS.md for agent chaining patterns and coordination strategies**

## Specification Integration

**Auto-Update Protocol:**

**Pre-Implementation:**
- Check `.quaestor/specs/draft/` for matching spec ID
- Move spec from draft/ → active/ (max 3 active)
- Declare: "Working on Spec: [ID] - [Title]"
- Update phase status in spec file

**Post-Implementation:**
- Update phase status → "completed"
- Track acceptance criteria completion
- Move spec to completed/ when all phases done
- Create git commit with spec reference

**See @SPECS.md for complete specification integration details**

## Quality Gates

**Code Quality Checkpoints:**
- Function exceeds 50 lines → Use refactorer agent to break into smaller functions
- Nesting depth exceeds 3 → Use refactorer agent to simplify logic
- Circular dependencies detected → Use architect agent to review design
- Performance implications unclear → Use implementer agent to add measurements

**See @QUALITY.md for language-specific quality gates and standards**

## Success Criteria

- ✅ Workflow coordinator validates planning phase completed
- ✅ Specification identified and moved to active/
- ✅ User approval obtained for implementation strategy
- ✅ All quality gates passed (linting, tests, type checking)
- ✅ Documentation updated
- ✅ Specification status updated and tracked
- ✅ Ready for review phase

## Final Response

When implementation is complete:
```
Implementation complete. All quality gates passed.
Specification [ID] updated to completed status.
Ready for review and PR creation.
```

**See @WORKFLOW.md for complete workflow details**

---

*Intelligent implementation with agent orchestration, quality gates, and specification tracking*
