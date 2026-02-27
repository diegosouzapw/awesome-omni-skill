---
name: ast-refactor
description: This skill should be used when transforming, refactoring, or replacing code patterns across multiple files. Use for renaming functions/classes/variables, migrating API usage patterns, modernizing syntax, or performing bulk code transformations. Prefer ast-grep over sed/awk for syntax-aware replacements that understand language structure.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Edit
  - Write
  - AskUserQuestion
  - TodoWrite
context: fork
agent: general-purpose
---

# AST-Aware Code Refactoring

## Overview

Transform code patterns systematically using ast-grep, a syntax-aware tool that finds and replaces code based on abstract syntax trees rather than plain text matching.

## When to Use

**Use this skill when:**
- Renaming functions, classes, or variables across the codebase
- Migrating API usage patterns (e.g., `console.log` → `logger.info`)
- Modernizing syntax (e.g., `var` → `const`, `require` → `import`)
- Replacing deprecated function calls with new equivalents
- Restructuring code architecture (moving, splitting, combining modules)
- Performing bulk transformations that require syntax awareness

**Do NOT use this skill for:**
- Searching/exploring code without transformations (use `ast-search` instead)
- Simple text replacements in non-code files (use sed/grep)
- Single-file edits with no pattern repetition (use Edit tool directly)
- Non-structural changes (comments, formatting)
- Bug fixes that don't involve pattern transformation

## Quick Reference

```bash
# Check if ast-grep is installed
which ast-grep

# Find pattern
ast-grep --pattern 'function $FUNC($$$PARAMS) { $$$ }' --lang javascript

# Search and replace
ast-grep run -p 'var $VAR = $VAL' -r 'const $VAR = $VAL' -l javascript

# Interactive mode
ast-grep run -p 'PATTERN' -r 'REPLACEMENT' -l LANG --interactive
```

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `$VAR` | Single AST node (variable, expression, etc.) |
| `$$$ARGS` | Multiple nodes (function arguments) |
| `$$$` | Any sequence of nodes within a block |

## Workflow

### Phase 1: Discovery & Analysis

1. **Understand the Request**: Clarify what structure to refactor
2. **Find Current Usage**: Use ast-grep to discover all instances
3. **Assess Impact**: Count affected files/locations, identify dependencies

```bash
# Example: Find all class definitions
ast-grep --pattern 'class $CLASS { $$$ }' --lang typescript
```

### Phase 2: Planning

1. **Design Transformation**: Define find pattern and replacement pattern
2. **Create Plan**: Summarize what/why/impact with before/after examples

Plan format:
```markdown
## Refactoring Plan: [Title]

**Goal**: [1-2 sentences]
**Impact**: [X files, Y instances]

**Pattern Match**:
```typescript
// Current pattern
```

**Transform To**:
```typescript
// New pattern
```

**Architecture** (if applicable):
```
Current:                    New:
┌─────────┐                ┌─────────┐
│ Module  │ ──────▶        │ Module  │
└─────────┘                └─────────┘
```

**Additional Steps**: [Manual changes needed]
```

### Phase 3: User Feedback

1. Present plan with options if multiple approaches exist
2. Gather feedback using AskUserQuestion for architectural decisions
3. Confirm approach before proceeding

### Phase 4: Execution

1. **Verify Git State**: Ensure clean working tree or user acknowledges changes
2. **Apply Transformations**:
   ```bash
   # Simple pattern replacement
   ast-grep run -p 'console.log($MSG)' -r 'logger.info($MSG)' -l typescript

   # Interactive for careful review
   ast-grep run -p 'PATTERN' -r 'REPLACEMENT' -l LANG --interactive
   ```
3. **Verify Changes**: Run tests, type checkers, linters
4. **Document Results**: Summarize changes and any remaining manual work

## ast-grep Commands

### Basic Search
```bash
ast-grep --pattern 'useState($$$)' --lang typescript
ast-grep -p 'function $F() { $$$ }' -l javascript -A 3 -B 2
```

### Search & Replace
```bash
ast-grep run -p 'var $VAR = $VAL' -r 'const $VAR = $VAL' -l javascript
```

### Rule-Based Refactoring
```yaml
# rule.yml
id: modernize-hasOwnProperty
language: typescript
rule:
  pattern: $OBJ.hasOwnProperty($PROP)
fix: Object.hasOwn($OBJ, $PROP)
message: Use Object.hasOwn instead
```
```bash
ast-grep scan -r rule.yml --update-all
```

### Supported Languages

JavaScript, TypeScript, Python, Go, Rust, Java, C, C++, C#, Kotlin, Swift, Ruby, PHP, and more.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Not checking ast-grep installation | Run `which ast-grep` first, install with `brew install ast-grep` |
| Running on dirty git state | Verify clean state or get user acknowledgment first |
| Skipping interactive mode for large changes | Use `--interactive` for 50+ file changes |
| Pattern doesn't match comments/strings | This is correct behavior - ast-grep is syntax-aware |
| Not running tests after | Always verify with tests/type-checker after refactoring |
| Assuming single pass works | Complex refactoring may need multiple ast-grep passes |

## Safety Guidelines

- Never run destructive refactoring without user approval
- Always show transformation plan with examples before executing
- Use `--interactive` or `--dry-run` modes for significant changes
- Verify tests pass after refactoring
- For large refactorings (50+ files), suggest incremental approach
- Work with clean git state, commit refactoring separately from logic changes

## Troubleshooting

- **Pattern not matching**: Verify language flag, check AST structure
- **Partial matches**: May need multiple patterns for variants
- **Syntax errors after refactor**: Some edge cases need manual intervention
- **Tool not found**: Install with `brew install ast-grep`
