---
name: python-typing
description: Migrate Python codebases to strict type checking with pyright. Use when user wants to add types, fix type errors, set up strict mode, or run a typing migration. Provides setup automation, fix patterns, discipline enforcement, and optional iteration loop support.
---

# Python Typing

Migrate Python codebases to strict type checking using pyright.

## Onboarding

On first invocation, run the setup flow:

### 1. Determine Strictness Level

If user explicitly said "strict typing", use strict. Otherwise ask:

```
What level of type checking?
  A) strict - Maximum type safety (recommended for new projects)
  B) standard - Balanced (recommended for existing codebases)
  C) basic - Minimal, just obvious bugs
```

Run pyright at each level to show baselines:
```bash
npx pyright --outputjson 2>/dev/null | jq '.summary.errorCount' # strict
npx pyright -p pyrightconfig-standard.json --outputjson 2>/dev/null | jq '.summary.errorCount'
```

### 2. Choose Setup Type

```
Setup options:
  A) Recommended - Full setup with tracking and rules
  B) Minimal - Just pyright config and rules
  C) Custom - Choose each component
```

### 3. Run Setup

**For Recommended (A):**
```bash
python3 <SKILL_PATH>/scripts/init_typing.py --level {strict|standard|basic} --full
```

This installs:
- pyrightconfig-{level}.json
- .long-task-harness/ with typing rules
- typing-findings.md
- Pre-commit hook
- long-task-harness (if not present)
- ralph-wiggum plugin (Claude Code only)

**For Minimal (B):**
```bash
python3 <SKILL_PATH>/scripts/init_typing.py --level {strict|standard|basic}
```

**For Custom (C):** Ask which components, then run with appropriate flags.

### 4. Explain What Was Installed

After setup, explain each component:
- What pyrightconfig does
- What the rules enforce and why
- What long-task-harness provides (link to docs)
- How ralph-wiggum works (if installed)

### 5. Show Fix Strategy

```
Recommended approach - fix in layers:

1. Quick wins: unused imports, missing return types, generic args
2. Annotations: parameter types, class attributes
3. Type safety: None checks, narrowing, unions
4. Structural: conditional imports, TypedDict, Protocol
5. External: missing stubs, third-party workarounds
6. Edge cases: complex generics, metaprogramming

Run analysis first:
  python3 <SKILL_PATH>/scripts/analyze_typing.py
```

### 6. Offer Iteration Loop (if ralph-wiggum installed)

Show the prompt template and confirm before user runs it.

---

## Fixing Errors

### Workflow

1. Run `npx pyright` to see current errors
2. Pick 5-10 related errors (same file or type)
3. Fix properly (no shortcuts)
4. Run pyright to verify
5. Commit with descriptive message
6. Repeat

### Rules (Critical)

1. **No `# type: ignore`** - Fix the actual issue. If truly unfixable, document in typing-findings.md first.

2. **No `assert x is not None`** - Use proper patterns:
   - Early return: `if x is None: return`
   - Conditional: `if x is not None: x.method()`
   - Raise with context: `if x is None: raise ValueError("x required")`

3. **Avoid `Any`** - Use specific types, TypeVar, Union, Protocol, or object.

4. **Avoid `cast()`** - Use isinstance() narrowing or TypeGuard.

### Common Patterns

See `references/patterns.md` for detailed fix patterns:
- Conditional imports (try/except)
- TypedDict for dict shapes
- Protocol for duck typing
- Third-party stubs
- Complex generics

### When Stuck

1. Check patterns.md for similar issues
2. Check typing-findings.md for documented workarounds
3. For third-party libs: install stubs or document limitation
4. Ask for help rather than using type:ignore

---

## Progress Tracking

### With long-task-harness

Update `.long-task-harness/long-task-progress.md` with:
- Error count at start/end of session
- Files/modules fixed
- Patterns discovered
- Decisions made

### Every 50 Errors

- Review work for consistency
- Document new patterns in typing-findings.md
- Note any recurring issues

---

## Ralph-Wiggum Loop

For unattended iteration (Claude Code only):

```
/ralph-loop "Fix pyright strict mode errors.

## Setup
Run: npx pyright
Baseline: {ERROR_COUNT} errors

## Rules (CRITICAL)
1. Do NOT use # type: ignore - fix the actual issue
2. Do NOT use assert x is not None - use proper conditionals
3. Avoid Any - use specific types
4. Avoid cast() - use isinstance() narrowing

## Workflow
1. Run pyright, note error count
2. Pick 5-10 related errors (same file or same type)
3. Fix them properly
4. Run pyright to verify
5. Commit with descriptive message
6. Repeat

## Every 50 Errors
- Review for consistency
- Document unfixable issues in typing-findings.md
- Note patterns

## If Stuck
- Check typing-findings.md for similar issues
- For third-party libs: document and use minimal workaround
- Ask for help rather than using type: ignore

## Completion
When pyright reports 0 errors: <promise>TYPING_COMPLETE</promise>" --completion-promise "TYPING_COMPLETE" --max-iterations 300
```

---

## References

- `references/patterns.md` - Common fix patterns by category
- `references/faq.md` - Detailed FAQ for common questions
