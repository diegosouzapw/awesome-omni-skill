---
name: monorepo-package-workflow
description: "Guides creation of new MCP tool packages in @j0kz/mcp-agents monorepo following modular architecture with constants/, helpers/, utils/ extraction, tools.json registration, and version.json synchron..."
---

# Monorepo Package Creation for @j0kz/mcp-agents

Create new MCP tool packages following established patterns from the 9 existing tools.

## Prerequisites Checklist

- [ ] Tool purpose clearly defined and unique
- [ ] No duplicate functionality in existing tools (check tools.json)
- [ ] Tool fits one category: analysis, generation, refactoring, design, orchestration
- [ ] Name follows pattern: `@j0kz/{tool-name}-mcp`

## Quick Start: Create New Package

```bash
# 1. Create structure
cd packages
mkdir -p your-tool/src/{constants,helpers,utils}
mkdir your-tool/tests

# 2. Initialize package.json (copy template below)
# 3. Create thin mcp-server.ts (<150 LOC)
# 4. Implement main-logic.ts with @j0kz/shared
# 5. Register in tools.json
# 6. Sync version: npm run version:sync
# 7. Build and test
```

## Package Structure (Modular Architecture)

```
your-tool/
├── src/
│   ├── mcp-server.ts      # Thin MCP layer (<150 LOC)
│   ├── main-logic.ts      # Core functionality
│   ├── types.ts           # TypeScript interfaces
│   ├── constants/         # Thresholds, patterns
│   ├── helpers/           # Complex logic (30+ LOC)
│   └── utils/             # Cross-cutting utilities
├── tests/                 # Vitest tests
├── package.json           # With "type": "module"
└── tsconfig.json
```

**Target:** All files <300 LOC (extract when exceeded)

**For detailed structure guide:**
```bash
cat .claude/skills/monorepo-package-workflow/references/package-structure-guide.md
```

## Critical package.json Template

```json
{
  "name": "@j0kz/your-tool-mcp",
  "version": "1.1.0",  // From version.json
  "type": "module",    // CRITICAL: ES modules
  "main": "./dist/index.js",
  "bin": {
    "your-tool-mcp": "dist/mcp-server.js"
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest",
    "test:coverage": "vitest run --coverage"
  },
  "dependencies": {
    "@j0kz/shared": "^1.1.0",  // Must match version
    "@modelcontextprotocol/sdk": "^1.19.1"
  }
}
```

## MCP Server Pattern (Thin Orchestration)

```typescript
// mcp-server.ts - ONLY protocol handling
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { handleYourTool } from './main-logic.js';  // .js extension!

// Setup server (thin, delegates all logic)
// Target: <150 LOC
```

**For complete MCP patterns:**
```bash
cat .claude/skills/monorepo-package-workflow/references/mcp-server-patterns.md
```

## Using @j0kz/shared Utilities

```typescript
import {
  FileSystemManager,   // File ops with caching
  AnalysisCache,       // LRU cache (30min TTL)
  PerformanceMonitor,  // Performance tracking
  MCPPipeline         // Tool chaining
} from '@j0kz/shared';
```

**For shared utilities guide:**
```bash
cat .claude/skills/monorepo-package-workflow/references/shared-utilities-guide.md
```

## Registration & Version Sync

### 1. Register in tools.json

```json
{
  "id": "your-tool",
  "name": "Your Tool Name",
  "package": "@j0kz/your-tool-mcp",
  "category": "analysis",
  "features": ["Feature 1", "Feature 2", "Feature 3"]
}
```

### 2. Sync Version (CRITICAL!)

```bash
# Never manually edit version in package.json!
npm run version:sync          # Updates all packages
npm run version:check-shared  # Verify consistency
```

## Testing with Vitest

```typescript
// tests/main-logic.test.ts
import { describe, it, expect } from 'vitest';
import { handleYourTool } from '../src/main-logic.js';

describe('YourTool', () => {
  it('should handle valid input', async () => {
    const result = await handleYourTool({ filePath: 'test.ts' });
    expect(result.success).toBe(true);
  });
});
```

**Target:** >70% coverage

**For testing setup guide:**
```bash
cat .claude/skills/monorepo-package-workflow/references/testing-setup-guide.md
```

## Import Pattern (ES Modules)

```typescript
// ✅ CORRECT - Always use .js
import { foo } from './bar.js';
import { helper } from './helpers/my-helper.js';

// ❌ WRONG - Will fail
import { foo } from './bar';
import { foo } from './bar.ts';
```

## Build & Validation

```bash
# Build TypeScript
npm run build -w packages/your-tool

# Verify structure
ls dist/  # Should have .js files

# Run tests
npm test -w packages/your-tool

# Check file sizes
find src -name "*.ts" -exec wc -l {} \; | sort -n
# All should be <300 LOC
```

## Complete Registration Checklist

**For detailed checklist with all steps:**
```bash
cat .claude/skills/monorepo-package-workflow/references/registration-checklist.md
```

Quick validation:
- [ ] Package name: `@j0kz/{tool}-mcp`
- [ ] Registered in tools.json
- [ ] Version synced from version.json
- [ ] mcp-server.ts <150 LOC
- [ ] All files <300 LOC
- [ ] Tests >70% coverage
- [ ] Imports use .js extension
- [ ] "type": "module" in package.json

## Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| Manually set version | Use `npm run version:sync` |
| Forget tools.json | Register before publishing |
| Fat mcp-server.ts | Keep <150 LOC, delegate to main-logic |
| Missing .js extension | Add to all relative imports |
| Files >300 LOC | Extract to constants/helpers/utils |

## Reference Examples

Study these well-structured packages:

1. **security-scanner** - Best modular refactoring (395→209 LOC)
2. **smart-reviewer** - AST analysis with fixers/
3. **orchestrator-mcp** - MCPPipeline usage
4. **test-generator** - Code generation patterns

## Publishing Workflow

```bash
# 1. Build all
npm run build

# 2. Publish
npm run publish-all

# 3. Git operations
git add . && git commit -m "feat: add your-tool"
git tag v1.1.0 && git push --tags
```

## Related Skills

- **project-standardization:** Version management and automation
- **modular-refactoring-pattern:** Reducing file complexity
- **code-quality-pipeline:** Ensuring code quality

## Quick Commands

```bash
npm run version:sync       # Sync versions
npm run build              # Build all packages
npm test                   # Run tests
npm run publish-all        # Publish to npm
```

---

**For modular refactoring:** `.claude/skills/modular-refactoring-pattern/SKILL.md`
**For project standards:** `.claude/skills/project-standardization/SKILL.md`