---
name: rules-vs-skills
description: Explain when to use always-on Rules vs on-demand Skills. Use when the team is confused about where to encode guidance, deciding between rules and skills, or understanding the difference between invariants and procedures.
---

# Rules vs Skills

## Quick Decision Guide

**Use Rules when:**
- ✅ Always-applied invariants (safety, scope, format)
- ✅ Non-negotiable principles (constitution, core policies)
- ✅ Context-independent constraints (file paths, output format)
- ✅ Short, declarative statements

**Use Skills when:**
- ✅ On-demand procedural workflows
- ✅ Multi-step processes (bootstrap, report generation)
- ✅ Explicit invocation needed (user says "go", runs command)
- ✅ Context-dependent procedures

## Rules Overview

### Characteristics
- **Always applied**: `alwaysApply: true` in frontmatter
- **Automatic**: No explicit invocation needed
- **Invariants**: Safety, scope, format constraints
- **Short**: Declarative statements, not procedures
- **File-scoped**: Can use `globs` to target specific files

### Rule File Structure
```markdown
---
description: Brief description
globs: ["**/*"]  # Optional: file patterns
alwaysApply: true
---

## Section
- Rule statement 1
- Rule statement 2
```

### Example Rules

**000-core.mdc** (Core invariants):
```markdown
## Safety (file ops)
- Inventory/Search is read-only.
- Any write action must follow **Plan → Human Approve → Apply**.
- delete is OFF by default (use quarantine).

## Scope guard
- Default edits: `src/**`, `tests/**`
```

**010-tdd.mdc** (TDD principles):
```markdown
- SoT = `plan.md`.
- When user says **go**, select the **next unchecked test** only.
- Loop: **RED → GREEN → REFACTOR**.
```

**015-constitution-cursor.mdc** (Non-negotiable):
```markdown
- The principles in `docs/constitution.md` are non-negotiable.
- If any instruction conflicts, stop and surface the conflict.
```

## Skills Overview

### Characteristics
- **On-demand**: Explicitly invoked (user request or command)
- **Procedural**: Step-by-step workflows
- **Context-dependent**: May require specific conditions
- **Reusable**: Can be called multiple times
- **Detailed**: Comprehensive instructions and examples

### Skill File Structure
```markdown
---
name: skill-name
description: What this skill does and when to use it
---

# Skill Name

## When to Use
- Specific trigger scenarios

## Instructions
1. Step-by-step procedure
2. Commands and examples
3. Integration points
```

### Example Skills

**tdd-go** (TDD workflow):
- Triggered when user says "go"
- Multi-step: RED → GREEN → REFACTOR
- Updates plan.md
- Integrates with implementer agent

**inventory-report** (Report generation):
- Triggered for weekly/monthly audits
- Runs commands to generate reports
- Saves output to `_meta/reports/`
- Can be called on-demand

**plan-gated-apply** (File operations):
- Triggered for file organization
- Multi-step: Plan → Approve → Apply
- Requires approval gate
- Transactional execution

## Decision Matrix

| Aspect | Rules | Skills |
|--------|-------|--------|
| **When applied** | Always (automatic) | On-demand (explicit) |
| **Type** | Invariants, constraints | Procedures, workflows |
| **Length** | Short, declarative | Detailed, step-by-step |
| **Invocation** | Automatic | User request or command |
| **Context** | Context-independent | Context-dependent |
| **Examples** | Safety policies, format | Report generation, setup |

## When to Use Rules

### ✅ Good for Rules

**Safety Invariants:**
- "Any write action must follow Plan → Approve → Apply"
- "delete is OFF by default (use quarantine)"
- "Inventory/Search is read-only"

**Scope Constraints:**
- "Default edits: `src/**`, `tests/**`"
- "Setup allowed: `.cursor/**`, `.github/**`"

**Output Format:**
- "Outputs follow: ExecSummary → Visual → Options → Roadmap"
- "Numbers: 2-dec"
- "KR concise + EN-inline allowed"

**Non-negotiable Principles:**
- "The principles in `docs/constitution.md` are non-negotiable"
- "SoT = `plan.md`"

**TDD Invariants:**
- "When user says **go**, select the **next unchecked test** only"
- "Loop: RED → GREEN → REFACTOR"

### ❌ Not Good for Rules

- Multi-step procedures
- Commands that need to be run
- Context-dependent workflows
- Detailed instructions
- Examples and troubleshooting

## When to Use Skills

### ✅ Good for Skills

**Procedural Workflows:**
- Report generation (`inventory-report`)
- Setup procedures (`repo-bootstrap`, `everything-provider-setup`)
- Validation processes (`plan-validate`, `release-check`)
- TDD cycles (`tdd-go`)

**On-demand Tasks:**
- User says "go" → `tdd-go`
- Weekly audit → `inventory-report`
- Release prep → `release-check`
- File organization → `plan-gated-apply`

**Multi-step Processes:**
- Plan → Approve → Apply workflow
- TDD: RED → GREEN → REFACTOR
- Setup: Install → Configure → Validate

**Integration Procedures:**
- Everything provider setup
- CI/Pre-commit configuration
- Agent selection guidance

### ❌ Not Good for Skills

- Always-applied safety constraints
- Non-negotiable principles
- Short declarative statements
- Context-independent invariants

## Common Patterns

### Pattern 1: Safety + Workflow
- **Rule**: "Any write action must follow Plan → Approve → Apply" (invariant)
- **Skill**: `plan-gated-apply` (how to execute the workflow)

### Pattern 2: TDD Principles + Execution
- **Rule**: "When user says **go**, select the **next unchecked test** only" (invariant)
- **Skill**: `tdd-go` (how to execute TDD cycle)

### Pattern 3: Scope + Procedures
- **Rule**: "Default edits: `src/**`, `tests/**`" (scope constraint)
- **Skill**: `repo-bootstrap` (how to set up project structure)

## Examples from This Project

### Rules Examples

**000-core.mdc:**
```markdown
## Safety (file ops)
- Inventory/Search is read-only.
- Any write action must follow **Plan → Human Approve → Apply**.
- delete is OFF by default (use quarantine).
```
→ **Why Rule**: Always-applied safety invariant

**010-tdd.mdc:**
```markdown
- SoT = `plan.md`.
- When user says **go**, select the **next unchecked test** only.
```
→ **Why Rule**: Always-applied TDD principle

### Skills Examples

**tdd-go:**
- Multi-step TDD workflow
- Detailed instructions for RED/GREEN/REFACTOR
- Integration with agents
→ **Why Skill**: On-demand procedural workflow

**inventory-report:**
- Runs commands to generate reports
- Saves output to specific location
- Can be called for audits
→ **Why Skill**: On-demand report generation

**plan-gated-apply:**
- Multi-step: Plan → Approve → Apply
- Detailed workflow instructions
- Integration with multiple agents
→ **Why Skill**: Complex procedural workflow

## Migration Guide

### Moving from Rule to Skill
If a rule becomes too detailed or procedural:
1. Create new skill file
2. Move detailed instructions to skill
3. Keep invariant in rule (reference skill if needed)
4. Update rule to be declarative only

### Moving from Skill to Rule
If a skill contains always-applied invariants:
1. Extract invariant to rule
2. Keep procedure in skill
3. Reference rule from skill
4. Ensure rule is declarative

## Best Practices

### Rules
- ✅ Keep short and declarative
- ✅ Focus on invariants and constraints
- ✅ Use globs for file-specific rules
- ✅ Group related rules in same file
- ❌ Don't include procedures or commands
- ❌ Don't include examples or troubleshooting

### Skills
- ✅ Provide detailed step-by-step instructions
- ✅ Include examples and use cases
- ✅ Document integration points
- ✅ Include troubleshooting sections
- ❌ Don't duplicate rule invariants
- ❌ Don't include always-applied constraints

## Quick Reference

### Rules Location
```
.cursor/rules/*.mdc
```

### Skills Location
```
.cursor/skills/*/SKILL.md
```

### Rule Naming
- `000-core.mdc` - Core invariants
- `010-tdd.mdc` - TDD principles
- `015-constitution-cursor.mdc` - Non-negotiable
- `030-commits.mdc` - Commit rules
- `040-ci.mdc` - CI rules
- `100-python.mdc` - Python-specific
- `300-inventory-master-domain.mdc` - Domain-specific

### Skill Naming
- Lowercase with hyphens
- Descriptive of function
- Examples: `tdd-go`, `inventory-report`, `plan-gated-apply`

## Additional Resources
- For rule examples: `.cursor/rules/*.mdc`
- For skill examples: `.cursor/skills/*/SKILL.md`
- For agents vs skills: `docs/AGENTS_AND_SKILLS_GUIDE.md`
- For constitution: `docs/constitution.md`
