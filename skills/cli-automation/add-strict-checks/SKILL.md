---
name: add-strict-checks
description: 'Enable stricter TypeScript and linting checks to catch bugs early, especially useful when iterating with AI assistance.'
---

# Add Strict Checks

**Goal:** Enable stricter TypeScript compiler options and linting to catch bugs that standard TypeScript misses.

---

## Task 1: Add Stricter Compiler Options

Add these additional strict options to `tsconfig.json` under `compilerOptions`:

```json
{
  "compilerOptions": {
    // ... existing options ...

    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "forceConsistentCasingInFileNames": true,
    "exactOptionalPropertyTypes": true,
    "useUnknownInCatchVariables": true
  }
}
```

---

## Task 2: Add Check Script

Add a check script to `package.json`:

```json
{
  "scripts": {
    "typecheck:app": "tsc --noEmit -p tsconfig.check.json",
    "typecheck:tests": "tsc --noEmit -p tsconfig.test.json",
    "check": "biome check . && npm run typecheck:app && npm run typecheck:tests"
  }
}
```

---

## Task 3: Fix All Issues

Run `npm run check:write && npm run check` in a loop, fixing issues until it passes.

IMPORTANT: NEVER disable any checks:
- NEVER disable any biome checks,
- NEVER edit tsconfig.json to remove or disable a check
- NEVER edit tsconfig.check.json to remove or disable a check
- NEVER edit package.json to remove a check from a script

NEVER disable checks, even temporarily for any reason (including for easier work with third-party libraries)

Instead, fix the code to not violate the check.

---

## Task 4: Update CLAUDE.md

Update the "Before Committing" section to include `npm run check`:

```bash
npm test && npm run check
```

If tests aren't set up yet, just use:

```bash
npm run check
```

---

## Task 5: Commit

Ask the user if they want to commit the changes.

---

## Task 6: Offer Further Hardening

Ask the user: "Would you like to add backend testing as well? This sets up integration tests with an isolated test database."

If yes, follow the `add-backend-testing` skill.
