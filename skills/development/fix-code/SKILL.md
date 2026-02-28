---
name: fix-code
description: Properly fix code issues by following the quality assurance guidelines. Ensures comprehensive fixes across all related locations.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Code Fix Helper

A skill for properly fixing code issues. Follows the core principles in `.claude/rules/fix-guidelines.instructions.md` to perform comprehensive and efficient fixes.

## Fix Flow

Follow the common flow in `.claude/rules/fix-guidelines.instructions.md`.

## Code-Specific Fix Guidelines

### Code Deletion & Refactoring

1. Search all instances of the target pattern (verify with Grep)
2. Identify affected tests and documentation
3. Fix all related locations at once
4. Run build and tests

### Variable & Function Name Changes

1. Search all usage locations of the target (Grep + file type filter)
2. Check reference documentation as well
3. Change all at once
4. Verify build

### Error Handling & Logic Fixes

1. Search for the same pattern elsewhere
2. Check related call sites
3. Fix all at once
4. Run tests

## Self-Review (Code-Specific)

After fixing, verify the following:

- [ ] All affected locations have been fixed
- [ ] The fix is correct
- [ ] No build errors
- [ ] All tests pass
- [ ] Follows existing code style
- [ ] Documentation needs updating as well?

## Practical Example

```
❌ Bad example: Fix only 1 location
- Fix only the reported location

✅ Good example: Search and fix all locations
1. Use Grep to search all instances of the same pattern
2. Fix the same pattern across all found files at once
3. Fix related tests as well
4. Run build and tests
```
