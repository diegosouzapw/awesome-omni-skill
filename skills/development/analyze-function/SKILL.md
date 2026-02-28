---
name: analyze-function
description: Deep line-by-line analysis of a function or method. Explains what each line does, why it's written that way, performance implications, edge cases, and design patterns. Use when user says "analyze-function", "analyze {function}", "deep dive on {function}", or "explain {function} line by line".
compatibility: Requires filesystem read access to source files.
---

# Analyze Function

Perform a thorough line-by-line analysis of a function, explaining not just what the code does but why it's written this way.

## When to Use

- Understanding unfamiliar or complex code
- Onboarding to a new codebase
- Pre-refactor deep understanding
- Performance investigation
- Code archaeology (understanding legacy code intent)
- Learning from well-written code

## Process

### 1. Parse Request

Extract:
- File path containing the function
- Function/method name to analyze

Formats to recognize:
- `{file}:{function_name}`
- `{function_name} in {file}`
- Just `{function_name}` (search for it)

### 2. Locate the Function

1. Read the file
2. Find the function definition
3. Identify the full scope (including nested functions, closures)
4. Note the line numbers

### 3. Gather Context

Before line-by-line analysis, understand:
- What module/class is this part of?
- What calls this function? (if easily determinable)
- What does this function call?
- Any relevant types, interfaces, or data structures

### 4. Analyze Each Line

Open [ANALYSIS-GUIDE.md](ANALYSIS-GUIDE.md) for the detailed framework.

For every line, consider:
- **What**: Technical explanation of what this line does
- **Why**: Rationale for this approach (not just restating the what)
- **Note**: Performance implications, edge cases, or gotchas (when relevant)

Do not skip "boring" lines. Every line exists for a reason.

### 5. Identify Critical Details

After line-by-line, explicitly list things that might be missed on casual reading:
- Implicit assumptions the code makes
- Non-obvious behavior or side effects
- Hidden dependencies
- Subtle edge cases
- Error conditions that aren't obvious

### 6. Note Design Patterns

If the function uses notable patterns or techniques, name them:
- Design patterns (Factory, Observer, etc.)
- Language idioms
- Performance optimization techniques
- Error handling patterns

### 7. Suggest Improvements (If Any)

Only if genuine concerns exist:
- Potential bugs or edge cases not handled
- Performance issues
- Readability improvements
- Missing error handling

Do not invent issues. If the code is solid, say so.

## Output Format

```markdown
## Function Analysis: {function_name}

**File**: `{path/to/file.ext}`
**Lines**: {start}-{end}
**Language**: {language}

### Context & Purpose

{2-3 sentences: What this function does, where it fits in the system, why it exists}

### Line-by-Line Analysis

**Line {N}**: `{code}`
- **What**: {technical explanation}
- **Why**: {rationale for this approach}
- **Note**: {performance implication, edge case, or gotcha}

**Line {N+1}**: `{code}`
- **What**: {explanation}
- **Why**: {rationale}

{Continue for all lines...}

### Critical Details

Things that might be missed on casual reading:

- {Non-obvious behavior or assumption}
- {Hidden dependency or side effect}
- {Subtle edge case}

### Design Patterns

- **{Pattern name}**: {How it's applied here}

### Potential Improvements

{If any genuine issues exist:}
- {Concern}: {explanation}

{Or if code is solid:}
No significant issues identified. The implementation is {brief positive assessment}.
```

## Guidelines

- **Be thorough**: Every line matters. Don't summarize or skip.
- **Explain why, not just what**: "Initializes counter to 0" is useless. "Starts at 0 because indexes are zero-based and we're tracking position" is useful.
- **Be honest about uncertainty**: If you're not sure why something is done a certain way, say so.
- **Connect to context**: How does this line relate to the function's purpose?
- **Note the non-obvious**: What would trip up someone reading this quickly?
