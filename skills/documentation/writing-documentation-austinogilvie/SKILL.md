---
name: writing-documentation
description: Creates technical documentation including READMEs, API references, user guides, architecture docs, ADRs, and runbooks. Use for requests to create, write, generate, or draft documentation. Trigger phrases include "document this", "write docs", "create readme", "API reference", "user guide", "architecture docs", "ADR", "runbook". For updating existing READMEs, use updating-readme instead.
version: 1.0.0
---

# Writing Documentation

## Overview

Expert technical documentation skill for creating clear, accurate, user-friendly documentation. Handles lightweight documentation tasks directly and delegates comprehensive tasks to the writing-documentation agent.

## When to Use This Skill

Use writing-documentation when you need to create:
- **API documentation** from code (OpenAPI, endpoint references, authentication docs)
- **User guides** and tutorials (getting started, feature walkthroughs)
- **Architecture documentation** (system design, ADRs, component diagrams)
- **Operational docs** (runbooks, deployment guides, troubleshooting)
- **Project documentation** (comprehensive READMEs, CONTRIBUTING guides)

## Handoff Rules

### To updating-readme skill
For maintaining existing READMEs (section updates, dependency changes, minor edits), use updating-readme instead. This skill creates; updating-readme maintains.

### To writing-documentation agent
Delegate to the writing-documentation agent via the Task tool when ANY of these apply:
- Output will be multiple files
- User guide will exceed 500 words
- Requires architecture documentation or ADRs
- API documentation requires analyzing more than 3 source files
- Creating documentation from scratch for an undocumented project
- Task requires deep codebase analysis

**How to delegate:**
Use the Task tool with agent `writing-documentation`. Example:
```
Task: writing-documentation
Prompt: Create comprehensive API documentation for the authentication module including all endpoints, request/response schemas, and error codes.
```

### Decision Matrix

| Scenario | Handle Here | Delegate to Agent |
|----------|-------------|-------------------|
| Single README creation | ✓ | |
| API docs for 1-3 endpoints | ✓ | |
| Short user guide (<500 words) | ✓ | |
| Multi-file documentation site | | ✓ |
| Full API reference (10+ endpoints) | | ✓ |
| Architecture docs with ADRs | | ✓ |
| Comprehensive project docs | | ✓ |

## Quick Reference

| Doc Type | Workflow | Template |
|----------|----------|----------|
| API Reference | `workflows/api-docs.md` | `templates/api-reference.md` |
| User Guide | `workflows/user-guide.md` | `templates/user-guide.md` |
| Architecture | `workflows/architecture.md` | `templates/adr.md` |
| Troubleshooting | `workflows/troubleshooting.md` | — |
| Runbook | — | `templates/runbook.md` |
| README | — | `templates/readme-comprehensive.md` |

## Process

When asked to create documentation:

1. **Scope Assessment**: Determine if this should be delegated to the agent (see Decision Matrix above)
2. **If delegating**: Use Task tool with writing-documentation agent
3. **If handling here**:
   - Analyze the code/system to understand what needs documenting
   - **Read the appropriate workflow file:**
     - API docs → Read `workflows/api-docs.md`
     - User guide → Read `workflows/user-guide.md`
     - Architecture → Read `workflows/architecture.md`
     - Troubleshooting → Read `workflows/troubleshooting.md`
   - **Read the matching template:**
     - API docs → Read `templates/api-reference.md`
     - README → Read `templates/readme-comprehensive.md`
     - ADR → Read `templates/adr.md`
     - Runbook → Read `templates/runbook.md`
     - User guide → Read `templates/user-guide.md`
   - Follow the workflow steps
   - Run validation loop
   - Deliver documentation

## Core Principles

### Audience Awareness
Always identify the target audience before writing:
- **End Users**: Simple language, task-oriented, minimal jargon
- **Developers**: Technical details, code examples, API specifics
- **Administrators**: Configuration, deployment, maintenance
- **Contributors**: Architecture, conventions, processes

### Documentation Standards

**Structure**
- Clear hierarchy with consistent heading levels
- Table of contents for documents > 500 words
- Cross-references between related sections
- Progressive disclosure (basic → advanced)

**Content**
- Lead with the most important information
- Use active voice and direct instructions
- Include practical examples for every concept
- Provide both quick-start and comprehensive paths

**Code Examples**
- Validate all code examples for syntax correctness
- Include expected output where applicable
- Show error handling, not just happy path
- Use realistic, not trivial, examples

### Format Guidelines

**Markdown Best Practices**
- Use fenced code blocks with language hints
- Tables for structured comparisons
- Consistent formatting throughout

**API Documentation Standards**
- HTTP method and path clearly visible
- All parameters documented (path, query, body, headers)
- Request and response examples
- Error responses with codes and messages
- Authentication requirements

## Validation Loop (Required)

After generating any documentation, run validation until clean:

1. **Run documentation validation**:
   ```bash
   python .claude/skills/writing-documentation/scripts/validate_docs.py <output_file>
   ```

2. **Run code block syntax validation**:
   ```bash
   python .claude/skills/writing-documentation/scripts/check_code_blocks.py <output_file>
   ```

3. **If ERRORs found in either**:
   - Fix all ERROR items (placeholders, empty code blocks, syntax errors)
   - Re-run both validations

4. **If WARNINGs found**:
   - Address straightforward warnings (missing language hints)
   - Re-run validations

5. **Repeat** until both pass or only acceptable warnings remain.

6. **Never deliver documentation with unresolved ERRORs.**

## Quality Checklist

Before delivering documentation, verify:
- [ ] Target audience is clear
- [ ] All code examples are syntactically correct
- [ ] No placeholder text remains
- [ ] Cross-references are valid
- [ ] Prerequisites are complete
- [ ] Examples are realistic and useful
- [ ] Error cases are documented
- [ ] Document is self-contained (or links to dependencies)
- [ ] Validation script passes

## Integration with updating-readme

After creating a README:
- Future section updates → updating-readme skill
- Future dependency additions → updating-readme skill
- Future config changes → updating-readme skill
- Complete rewrites → back to this skill or agent

## Examples

See `examples/` for complete output samples:
- `examples/api-example.md` — User Management API reference
- `examples/guide-example.md` — CLI tool user guide
