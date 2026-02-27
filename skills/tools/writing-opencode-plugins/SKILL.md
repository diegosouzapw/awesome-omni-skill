---
name: writing-opencode-plugins
description: Guides development of OpenCode plugins including project structure, testing patterns, and publishing. Use when creating or modifying OpenCode plugins.
license: MIT
compatibility: opencode
metadata:
  audience: users
  workflow: general
---

# Writing OpenCode Plugins

## Project structure

Use the shim pattern to separate development from plugin loading:

```
plugin-name/
├── src/
│   └── main.ts              # Main implementation
├── .opencode/
│   ├── package.json         # Plugin dependencies
│   └── plugins/
│       └── shim.ts         # Loads plugin from src/
├── test/
│   ├── unit.test.ts        # Unit tests
│   └── e2e.test.ts        # E2E tests
├── package.json            # Root package.json
└── tsconfig.json           # TypeScript config
```

### Shim file (.opencode/plugins/*.ts)

The shim file is loaded by OpenCode and re-exports from src:

```ts
export { PluginName as PluginNamePlugin } from "../../src/main"
```

Benefits:
- ✅ Development with TypeScript and type safety
- ✅ Easy unit/integration testing
- ✅ Source code not duplicated in .opencode/

## Plugin structure

A plugin exports a function that receives context and returns hooks:

```ts
export const MyPlugin: Plugin = async ({ directory, worktree, project, client, $ }) => {
  return {
    "shell.env": async (input, output) => {
      // input.cwd - current working directory
      // output.env - modify shell environment
    },
  }
}
```

### Context parameters

- `directory` - Current working directory
- `worktree` - Git worktree path
- `project` - Project information
- `client` - OpenCode SDK client
- `$` - Bun shell API

### Available hooks

Common hooks:
- `shell.env` - Inject environment variables before shell execution
- `tool.execute.before` - Intercept tool calls before execution
- `tool.execute.after` - Post-process tool results
- `file.edited` - React to file changes

## Testing strategy

### Unit tests

Test core logic in isolation using `bun test`:

```ts
// test/unit.test.ts
import { test, expect, describe, beforeEach, afterEach } from "bun:test"
import { coreFunction } from "../src/main"
import { tmpdir } from "os"
import { join } from "path"
import { mkdir, writeFile, rm } from "fs/promises"

describe("coreFunction", () => {
  let testDir: string

  beforeEach(async () => {
    testDir = join(tmpdir(), `test-${Date.now()}`)
    await mkdir(testDir, { recursive: true })
  })

  afterEach(async () => {
    await rm(testDir, { recursive: true, force: true })
  })

  test("handles basic case", async () => {
    const result = await coreFunction(testDir)
    expect(result).toBeDefined()
  })
})
```

### E2E tests

Test full plugin integration with OpenCode CLI:

```ts
// test/e2e.test.ts
import { test, expect, describe, beforeEach, afterEach } from "bun:test"
import { $ } from "bun"

describe("E2E", () => {
  test("plugin works with OpenCode CLI", async () => {
    try {
      const result = await $`opencode run 'test command'`.quiet()
      expect(result.exitCode).toBe(0)
      expect(result.stdout).toContain("expected")
    } catch (error) {
      if (error instanceof Error && error.message.includes("ENOENT")) {
        console.log("Skipping: opencode CLI not found")
        return
      }
      throw error
    }
  }, { timeout: 30_000 })
})
```

### Test fixtures

Committed test data in `test/fixtures/`:

```
test/fixtures/
├── basic/           # Basic scenarios
├── advanced/        # Complex scenarios
└── e2e/           # E2E test workspace
```

Test fixtures can contain safe-to-commit test data, encryption keys, etc.

## Dependencies

### Root package.json

Contains dev dependencies and publishing info:

```json
{
  "name": "@scope/plugin-name",
  "version": "0.1.0",
  "main": "./src/main.ts",
  "type": "module",
  "exports": {
    ".": {
      "types": "./src/main.ts",
      "import": "./src/main.ts"
    }
  },
  "scripts": {
    "test": "bun test",
    "test:unit": "bun test test/unit.test.ts",
    "test:e2e": "bun test test/e2e.test.ts"
  },
  "dependencies": {
    "@opencode-ai/plugin": "1.2.10"
  },
  "publishConfig": {
    "access": "public"
  }
}
```

### .opencode/package.json

Plugin runtime dependencies only:

```json
{
  "name": "plugin-name-runtime",
  "private": true,
  "dependencies": {
    "@opencode-ai/plugin": "1.2.10"
  }
}
```

OpenCode automatically runs `bun install` in `.opencode/` at startup.

## Publishing

### Using np (recommended)

```bash
# Install dependencies
bun install

# Login to npm
npm login

# Publish with version prompt
bunx np --any-branch
```

### Manual publish

```bash
# Bump version
npm version patch  # or minor, or major

# Publish to npm
npm publish --access public
```

### .npmignore

Exclude development files from npm package:

```
# Testing
test/
coverage/

# OpenCode plugin (local use)
.opencode/
.opencode.json

# Development notes
notes/
AGENTS.md

# Serena
.serena/

# GitHub CI
.github/

# Build/cache
node_modules/
dist/
out/
*.tsbuildinfo
.cache/
.eslintcache

# Logs
logs/
*.log

# Environment files
.env
.env.*
.env.keys
.env.*.keys

# macOS
.DS_Store

# IDE
.idea/
*.swp
*.swo

# npm
bun.lock
```

## TypeScript configuration

Use strict TypeScript with Bun defaults:

```json
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "Preserve",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noPropertyAccessFromIndexSignature": false
  }
}
```

## AGENTS.md (Bun instructions)

Include Bun-specific guidance in AGENTS.md:

```markdown
---
description: Use Bun instead of Node.js, npm, pnpm, or vite.
globs: "*.ts, *.tsx, *.html, *.css, *.js, *.jsx, package.json"
alwaysApply: false
---

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun install` instead of `npm install`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
```

## Development workflow

1. Edit `src/main.ts` - main implementation
2. Run tests: `bun test`
3. Shim in `.opencode/plugins/*.ts` auto-reloads on OpenCode restart

## Plugin installation

### Project-level (development)

Plugin files in `.opencode/plugins/` auto-load on OpenCode restart.

### Global (all projects)

```bash
mkdir -p ~/.config/opencode/plugins
cp .opencode/plugins/*.ts ~/.config/opencode/plugins/
cp .opencode/package.json ~/.config/opencode/package.json
```

### From npm (users)

Add to `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@scope/plugin-name"]
}
```

OpenCode auto-installs npm packages to `~/.cache/opencode/node_modules/`.

## Best practices

### Extract testable functions

Separate core logic from hooks:

```ts
// Bad: Logic inside hook
export const Plugin = async () => ({
  "shell.env": async (input, output) => {
    // All logic here...
  }
})

// Good: Extracted function
async function coreLogic(path: string): Promise<Record<string, string>> {
  // Pure logic, easy to test
}

export const Plugin = async () => ({
  "shell.env": async (input, output) => {
    const result = await coreLogic(input.cwd)
    Object.assign(output.env, result)
  }
})
```

### Error handling

Use try/catch with meaningful errors:

```ts
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error("Operation failed:", error)
  throw new Error(`Failed to process: ${error}`)
}
```

### Async hooks

All hooks are async - use async/await:

```ts
export const Plugin = async () => ({
  "shell.env": async (input, output) => {
    const data = await loadData()
    output.env.VAR = data.value
  }
})
```

### Non-mutating output

Check before setting:

```ts
if (!(key in output.env)) {
  output.env[key] = value
}
```

### Quiet mode

Suppress console output for clean logs:

```ts
someLibrary.configure({ quiet: true })
```

## CI/CD

Example GitHub Actions:

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install
      - run: bun test
```

## References

- [OpenCode Plugins Documentation](https://opencode.ai/docs/plugins/)
- [OpenCode SDK Documentation](https://opencode.ai/docs/sdk/)
- [Bun Documentation](https://bun.sh/docs)
