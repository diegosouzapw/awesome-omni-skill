---
name: add-provider
description: Add a new AI provider to agentconfig.org's comparison system. Use when integrating a new coding assistant (e.g., Cursor, Claude Desktop, GitHub Copilot alternative) with proper type system updates, implementation data, UI components, tests, and documentation.
---

# Add Provider

Add a new AI coding assistant provider to agentconfig.org's provider comparison system.

## Overview

Adding a provider requires coordinated work across **6 parallel work streams**:

| Stream | Work | Duration | Dependencies |
|--------|------|----------|--------------|
| **1. Type System** | Add provider to union types | 2-4 hrs | None |
| **2. Data Layer** | Add implementations for all 11 primitives | 4-6 hrs | Stream 1 |
| **3. UI Components** | Update comparison table | 4-8 hrs | Streams 1-2 |
| **4. Testing** | Update E2E tests | 3-4 hrs | Stream 3 |
| **5. App Integration** | Update site copy & docs | 1-2 hrs | Streams 1-3 |
| **6. LLMs Generation** | Regenerate machine-readable files | 1-2 hrs | All streams |

**Total effort**: ~2-3 hours with parallelization

## When to Use

Use this skill when:
- Integrating a new coding assistant (Cursor, Claude Desktop, Zed with AI, etc.)
- Expanding provider support beyond current offerings
- The provider implements most of the 11 AI primitives
- You want comprehensive comparison data visible to users and AI agents

## Prerequisites

Before starting, gather:
- **Provider capability audit** - Which of the 11 primitives does the provider support?
- **File path documentation** - Where do config files go (global vs project)?
- **Support levels** - `full` (native), `partial` (workarounds), `none` (unavailable), `diy` (custom setup)

## The 11 Primitives

Every provider must map to these primitives:

| Category | Primitives |
|----------|-----------|
| **Execution** | Agent Mode, Skills/Workflows, Tool Integrations (MCP) |
| **Customization** | Persistent Instructions, Global Instructions, Path-Scoped Rules, Slash Commands |
| **Control** | Custom Agents, Permissions & Guardrails, Lifecycle Hooks, Verification/Evals |

## Quick Start

0. **🔍 Research the provider** → See [RESEARCH-GUIDE.md](references/RESEARCH-GUIDE.md) for capability audit template
   - Visit official documentation
   - Document support level for each of the 11 primitives
   - Verify config file locations
   - **Complete this BEFORE writing any code** (see pre-implementation checklist in [CHECKLIST.md](references/CHECKLIST.md))

1. **📋 Read the detailed process** → See [PROCESS.md](references/PROCESS.md) for step-by-step instructions for all 6 streams

2. **📖 Review code examples** → See [EXAMPLES.md](references/EXAMPLES.md) for copy-paste templates for each stream

3. **🎨 Understand patterns** → See [PATTERNS.md](references/PATTERNS.md) for support levels and naming conventions

4. **🐛 Handle errors** → See [ERRORS.md](references/ERRORS.md) for solutions to common issues (including critical generation script updates)

5. **✅ Verify completion** → See [CHECKLIST.md](references/CHECKLIST.md) for verification steps (includes pre-implementation checklist)

## 6-Stream Workflow at a Glance

```
Stream 1: Type System (Add provider to union types)
   ↓
Stream 2: Data Layer (Add implementations for all 11 primitives)
   ├→ Stream 3: UI Components (Update comparison table)
   │    ↓
   │  Stream 4: Testing (Update E2E tests)
   │    ↓
   └→ Stream 5: App Integration (Update site copy/docs) [can run in parallel with 3-4]
        ↓
      Stream 6: LLMs Generation (Regenerate machine-readable files)
```

**Parallel execution**: Start Stream 5 while Streams 3-4 complete. Stream 1-2 are sequential. Stream 6 must run last.

## Key Files to Modify

| Stream | Files |
|--------|-------|
| 1 | `site/src/data/primitives.ts`, `site/src/data/fileTree.ts`, `site/src/data/comparison.ts`, `site/src/components/PrimitiveCards/PrimitiveCard.tsx` |
| 2 | `site/src/data/primitives.ts`, `site/src/data/comparison.ts`, `site/src/data/fileTree.ts` |
| 3 | `site/src/components/ProviderComparison/ComparisonTable.tsx` |
| 4 | `site/tests/e2e/comparison.spec.ts` |
| 5 | `site/src/App.tsx`, `site/src/components/Hero/Hero.tsx`, `README.md` |
| 6 | `.github/skills/generate-llms/scripts/generate-llms-full.ts` (if needed), `site/public/llms-full.txt` |

## Example Prompts

**Add a new provider from scratch:**
```
Use the add-provider skill to add Cursor as a provider to agentconfig.org.
Research Cursor's implementation of all 11 primitives first, then follow all 6 streams.
```

**Skip to a specific stream:**
```
I've completed Stream 1 (types). Now execute Stream 2 (data layer) to add cursor implementations.
```

**Update existing provider data:**
```
Update Cursor's support level from partial to full for Tool Integrations in the comparison.ts and UI.
```

## Success Metrics

✅ Provider added to all type definitions
✅ All 11 primitives have provider implementation data
✅ Comparison table renders with provider column
✅ All E2E tests pass
✅ No TypeScript errors
✅ Production build succeeds
✅ llms-full.txt includes provider data
✅ Responsive design works
✅ Dark mode works

## PR Description Best Practices

When opening your pull request, keep it crisp and focused:

**What to include:**
- **Summary**: One sentence—what provider, what changed
- **Changes**: Organized by stream (Types, Data, UI, Tests, Integration, Docs)
- **Result**: Quick summary of provider's final support coverage
- **Testing**: Concrete steps to verify (run commands, visit site, click features)
- **References**: Link to official provider documentation as sources

**What to avoid:**
- Listing all 11 primitives exhaustively
- Repetitive narrative about each stream
- Verbose technical implementation details

**Example**: See [Cursor provider PR](https://github.com/jonmagic/agentconfig.org/pull/3) for a reference implementation.

## Related Skills

- **[add-primitive](../../add-primitive)** - Add a new AI primitive (expand beyond 11)
- **[generate-llms](../../generate-llms)** - Regenerate llms.txt files
- **[semantic-commit](../../semantic-commit)** - Create semantic commit messages

## References

For detailed information, see:

- **[RESEARCH-GUIDE.md](references/RESEARCH-GUIDE.md)** - How to research a provider before implementation (capability audit template, decision tree, examples)
- **[PROCESS.md](references/PROCESS.md)** - Complete step-by-step instructions for all 6 streams
- **[EXAMPLES.md](references/EXAMPLES.md)** - Copy-paste code examples for each stream
- **[PATTERNS.md](references/PATTERNS.md)** - Support levels, file locations, naming conventions
- **[ERRORS.md](references/ERRORS.md)** - Common issues and solutions (including critical generation script updates)
- **[CHECKLIST.md](references/CHECKLIST.md)** - Comprehensive verification checklists (includes pre-implementation checklist)
