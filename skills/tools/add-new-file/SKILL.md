---
name: add-new-file
description: Guide for adding new files to this codebase while respecting architectural principles including Separation of Concerns, Common Closure Principle, small composable functions (max 20 lines), and externalizing constants. Use when creating new modules, utilities, or any new source files in the project.
---

# Add New File

This skill guides you through adding new files to the codebase while maintaining architectural consistency and code quality standards.

## Quick Reference

When adding a new file to this project:

1. **Determine the scope** - What domain/concern does this file address?
2. **Follow module pattern** - Create types.ts, constants.ts, main logic file
3. **Externalize constants** - All magic strings/numbers go in constants.ts
4. **Keep functions small** - Max 20 lines of logic per function
5. **Use proper imports** - `node:` prefix, `import type`, `.ts` extensions
6. **Write co-located tests** - Create `.test.ts` alongside source

See [references/architecture-principles.md](references/architecture-principles.md) for comprehensive guidelines.

## Adding a New Module

When creating a new module (e.g., `cache/`, `auth/`, `logger/`):

### Step 1: Create Module Directory

```
module-name/
├── types.ts          # Type definitions only
├── constants.ts      # Externalized constants
├── module-name.ts    # Core logic
└── module-name.test.ts  # Tests
```

### Step 2: Define Types First

Create `types.ts`:

```typescript
export interface ModuleConfig {
  enabled: boolean;
  timeout: number;
}

export type ModuleState = "idle" | "active" | "error";
```

### Step 3: Externalize Constants

Create `constants.ts`:

```typescript
export const DEFAULT_TIMEOUT_MS = 5000;
export const MODULE_CONFIG_FILE = "module-config.json";
export const ERROR_INVALID_CONFIG = "Invalid module configuration";
```

### Step 4: Implement Core Logic

Create `module-name.ts`:

```typescript
import { existsSync, readFileSync } from "node:fs";
import { join } from "node:path";
import type { ModuleConfig } from "./types.ts";
import { DEFAULT_TIMEOUT_MS, MODULE_CONFIG_FILE } from "./constants.ts";

export const loadModuleConfig = (dir: string): ModuleConfig => {
  const configPath = join(dir, MODULE_CONFIG_FILE);
  if (!existsSync(configPath)) {
    return { enabled: true, timeout: DEFAULT_TIMEOUT_MS };
  }
  return JSON.parse(readFileSync(configPath, "utf-8"));
};
```

### Step 5: Write Tests

Create `module-name.test.ts` following Vitest patterns in the reference doc.

## Adding a Utility File

For standalone utilities that don't need a full module:

1. Place in appropriate existing module or create `utils/` if needed
2. Create `utils/helper.ts` and `utils/helper.test.ts`
3. Extract any constants to the parent module's `constants.ts`
4. Keep functions small and composable

## Pre-Flight Checklist

Before creating files, verify:

- ✅ **Correct location** - File belongs in right module/directory
- ✅ **Constants identified** - Know what values need externalizing
- ✅ **Types defined** - Clear interfaces for inputs/outputs
- ✅ **Single responsibility** - File has one clear purpose
- ✅ **Dependencies mapped** - Know what to import

## Post-Creation Checklist

After creating files, verify:

- ✅ **All imports use `node:` prefix** for built-ins
- ✅ **Type imports use `import type`**
- ✅ **Relative imports include `.ts` extension**
- ✅ **All exports have explicit return types**
- ✅ **Constants externalized to `constants.ts`**
- ✅ **Functions under 20 lines of logic**
- ✅ **Tests created and passing**
- ✅ **Code follows style guide** (see reference doc)
- ✅ **No `console.log` statements** (use `client.tui`)

## Common Patterns

### Configuration Module

```
config/
├── types.ts       # ConfigOptions, ConfigSchema
├── constants.ts   # CONFIG_FILE_NAME, DEFAULT_VALUES
├── config.ts      # loadConfig(), saveConfig()
└── config.test.ts
```

### Business Logic Module

```
processor/
├── types.ts          # ProcessorInput, ProcessorOutput
├── constants.ts      # MAX_RETRIES, TIMEOUT_MS
├── processor.ts      # process(), validate()
└── processor.test.ts
```

### Utility Module

```
utils/
├── string-utils.ts      # String manipulation helpers
├── string-utils.test.ts
├── path-utils.ts        # Path manipulation helpers
└── path-utils.test.ts
```

## Detailed Guidelines

For comprehensive information on:

- Complete constants externalization rules with examples
- Full code style guide (imports, functions, naming, formatting)
- Error handling patterns
- Testing patterns with Vitest
- File system operation patterns

**Read:** [references/architecture-principles.md](references/architecture-principles.md)

## Verification Commands

After creating new files:

```bash
bun run typecheck   # Verify TypeScript types
bun run lint        # Check code style
bun run test        # Run tests
bun run build       # Ensure it builds
```
