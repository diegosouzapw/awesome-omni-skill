---
name: refactoring-patterns
description: Safe refactoring workflow and common patterns. Invoke when extracting methods, renaming, moving code, or addressing code smells.
---

# Refactoring Patterns Skill

## When to Use

Invoke this skill when:
- Code smells are detected (long functions, duplicated logic, shotgun surgery)
- Before making major structural changes to existing code
- During tech debt sprints or cleanup tasks
- A function exceeds 30 lines and needs decomposition
- Renaming, moving, or reorganizing code across files or packages

## The Golden Rule

```
Tests green -> Refactor -> Tests green
```

Never refactor code that lacks test coverage. If tests don't exist, write them first (characterization tests that capture current behavior), then refactor.

## Safe Refactoring Steps

1. **Verify tests pass** before touching anything. Run the full suite for the affected area.
2. **Make one refactoring move at a time.** Never combine multiple refactorings in a single change.
3. **Run tests after each move.** If anything fails, you know exactly which move broke it.
4. **Commit after each successful step.** Small commits make `git bisect` possible and rollback trivial.
5. **If tests fail, revert the last move.** Do not debug forward -- undo and try a different approach.

```
[tests pass] -> extract method -> [tests pass] -> commit
             -> rename variable -> [tests pass] -> commit
             -> move to new file -> [tests pass] -> commit
             -> [tests FAIL] -> git revert -> rethink
```

## Common Patterns

### Extract Method

**Smell:** Function too long, or comments explaining what a block does.

**Action:** Pull the block into a named function. The function name replaces the comment.

**When:** A function has distinct logical sections separated by blank lines or comments. Each section becomes its own function.

### Extract Class / Module

**Smell:** A class or module has multiple unrelated responsibilities. Changes for different reasons.

**Action:** Split into separate classes/modules, each with a single responsibility.

**When:** You find yourself prefixing method groups (`userCreate`, `userDelete`, `orderCreate`, `orderDelete` in one class) or the file exceeds 500 lines with distinct sections.

### Inline

**Smell:** Indirection adds complexity without clarity. A function just calls another function. A variable is used once and the expression is self-explanatory.

**Action:** Replace the indirection with the direct call or expression.

**When:** The abstraction layer adds no value -- the delegated function is equally readable inline.

### Move

**Smell:** A function or type is referenced more from another package/module than from its current one.

**Action:** Move it to where it belongs. Update all references.

**When:** Feature envy -- a function uses more data from another module than its own. Or the current package has grown a "utils" section that really belongs elsewhere.

### Rename

**Smell:** Name doesn't reveal intent. Requires reading the implementation to understand purpose.

**Action:** Rename to communicate what it does, not how it does it.

**When:** You find yourself re-reading a function body to remember what it does. Variable names like `data`, `result`, `temp`, `val`, or single letters outside tight loops.

### Replace Conditional with Polymorphism

**Smell:** Switch/if chains that check a type field and execute different logic per type.

**Action:** Define an interface with the varying behavior. Each type implements the interface.

**When:** The same type-check conditional appears in multiple places. Adding a new type requires modifying multiple switch statements.

### Introduce Parameter Object

**Smell:** Three or more related parameters always travel together across multiple functions.

**Action:** Group them into a struct, class, or typed object.

**When:** You see the same cluster of parameters in multiple function signatures (`startDate, endDate, timezone` or `host, port, protocol`).

### Replace Magic Number with Named Constant

**Smell:** Literal values embedded in logic with no explanation of their meaning.

**Action:** Extract to a named constant that explains the value's purpose.

**When:** A number or string literal appears in a condition, calculation, or configuration and its meaning is not immediately obvious (`if retries > 3`, `timeout: 30000`).

## Per-Language Tooling

| Language | Rename | Extract / Refactor | Format / Fix |
|----------|--------|---------------------|--------------|
| Go | `gorename`, `gopls rename` | `gopls refactor.extract` | `go fmt`, `goimports` |
| TypeScript | `ts-morph`, IDE rename symbol | IDE extract function/variable | `eslint --fix`, `prettier` |
| Python | `rope`, IDE rename symbol | IDE extract method/variable | `ruff --fix`, `black` |

Prefer IDE-assisted refactoring (rename symbol, extract function) over manual find-and-replace. Automated tools update all references and catch type errors.

## Red Flags -- When NOT to Refactor

- **No tests cover the code being refactored.** Write characterization tests first or accept the risk explicitly.
- **Refactoring across module boundaries without coordination.** If the public API changes, downstream consumers break. Coordinate or version the API.
- **Changing public APIs without versioning.** If external code depends on it, add a new version instead of modifying in place.
- **Mixing refactoring with feature work in the same commit.** Refactoring commits should be behavior-preserving. Feature commits should add new behavior. Mixing makes rollback impossible and code review painful.
- **Refactoring code you don't understand.** Read it first. Write characterization tests. Talk to the original author if available. Refactoring without understanding introduces subtle bugs.

## Pre-Ship Checklist

Before merging refactored code, verify:

1. [ ] All tests pass (unit, integration, E2E for affected area)
2. [ ] No behavior changed -- refactoring is strictly structural
3. [ ] Each commit is a single refactoring move (reviewable in isolation)
4. [ ] No refactoring and feature work mixed in the same commit
5. [ ] Public API signatures are unchanged (or changes are versioned and documented)
6. [ ] No dead code left behind (or marked with `// TODO: AI_DELETION_REVIEW`)
7. [ ] Code review confirms readability improved (the whole point of refactoring)
8. [ ] CI pipeline is green on the final state
