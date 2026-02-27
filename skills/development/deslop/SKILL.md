---
name: deslop
description: Use when removing AI-generated slop, cleaning up verbose code, or removing unnecessary defensive checks. Triggers on "remove slop", "clean up AI code", "deslop".
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
---

# Remove AI Code Slop

Check the diff against main and remove all AI-generated slop introduced in this branch.

## Usage

```
/deslop
```

## What It Removes

- **Extra comments** that a human wouldn't add or are inconsistent with the rest of the file
- **Extra defensive checks** or try/catch blocks that are abnormal for that area of the codebase (especially if called by trusted/validated codepaths)
- **Casts to `any`** to get around type issues
- **Over-engineered patterns** that don't match the file's existing style
- **Unnecessary type annotations** on obvious types
- **Verbose error handling** where simpler patterns exist
- **Redundant null checks** in already-validated paths
- **Excessive logging** beyond what's normal for the codebase

## What It Preserves

- Actual bug fixes and intended functionality
- Comments that provide genuine value
- Defensive code at trust boundaries (user input, external APIs)
- Type annotations that improve clarity

## Process

1. Gets diff between current branch and main
2. Reviews each changed file for AI slop patterns
3. Removes slop while preserving intended changes
4. Reports a 1-3 sentence summary of what was changed

## Output

A brief summary like:
> "Removed 3 unnecessary try/catch blocks in `query.py`, deleted 5 redundant comments in `handler.ts`, and removed 2 `any` casts that were hiding type issues."
