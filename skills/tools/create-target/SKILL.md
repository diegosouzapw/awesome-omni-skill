---
name: create-target
description: Scaffold a new target type (e.g., Zed, Windsurf) for universal-ai-config
userInvocable: true
argumentHint: "<target-name>"
allowedTools: ["Read", "Write", "Glob", "Grep", "Bash"]
---

Create a new target implementation for universal-ai-config. A target maps universal template frontmatter and hooks to a specific AI coding assistant's configuration format.

## Phase 1 — Research & Approval

1. **Research the target's config format** — find up-to-date documentation online for how the target AI assistant expects its configuration files. Look for:
   - File locations and naming conventions (e.g., `.cursor/rules/*.mdc`, `.github/instructions/*.md`)
   - Frontmatter format for each file type
   - Hook configuration format (JSON structure, event names)

   Look through docs for all supported configuration types:
   - Instructions/rules/"system prompt"
   - Skills/Commands
   - Agents
   - Hooks

2. **Present findings for approval** — before writing any code, present a structured summary of your research to the user. For each supported configuration type, include:
   - **What it maps to**: the universal template type it corresponds to (instructions, skills, agents, hooks)
   - **File location & naming**: where the target expects these files and what extensions/naming conventions it uses
   - **Frontmatter/metadata format**: the exact fields, keys, and structure the target uses (with examples from docs)
   - **Hook format** (if applicable): event names, JSON structure, how handlers are defined
   - **Source links**: direct URLs to the official documentation pages you referenced

   Format as a clear table or grouped list so the user can review each mapping. Ask the user to approve or flag any corrections before proceeding. **Do not continue until the user explicitly approves.**

## Phase 2 — Plan

After the user approves the research findings, **enter plan mode** to design the full implementation before writing any code. The plan should cover:

3. **Review existing targets for patterns** — read through at least one existing target implementation (e.g., `src/targets/claude/index.ts`) and the shared types to understand the exact interfaces and conventions you need to follow.

4. **Draft the implementation plan** — produce a detailed, step-by-step plan that includes:
   - **Target definition**: the full `defineTarget()` structure — `name`, `outputDir`, `supportedTypes`, and for each supported type:
     - `frontmatterMap` — every universal key and what it maps to (string rename or function transform), with reasoning
     - `getOutputPath` — the exact path logic, including how `alwaysApply` or other flags affect routing
   - **Hooks** (if supported): `transform` function logic mapping universal event names → target events, `outputPath`, and `mergeKey` if the hook config nests inside a larger file
   - **Registration**: changes needed in `src/targets/index.ts` and the `Target` type in `src/types.ts`
   - **Config schema**: any changes needed in `src/config/schema.ts` or `src/config/defaults.ts`
   - **Tests**: which fixture(s) to use or create, and what assertions to write
   - **Docs**: any updates needed to README, meta-instruction templates, or seed files

   Reference the `defineTarget()` pattern:

   ```typescript
   import { defineTarget } from "../define-target.js";
   import type { UniversalFrontmatter, UniversalHookHandler } from "../../types.js";

   export default defineTarget({
     name: "<target-name>",
     outputDir: ".<target-dir>",
     supportedTypes: ["instructions", "skills", "agents", "hooks"],
     instructions: {
       frontmatterMap: {
         description: "description",
         globs: "<target-specific-key>",
         alwaysApply: "<target-specific-key>",
       },
       getOutputPath: (name, fm) => `rules/${name}.<ext>`,
     },
     // skills, agents — similar pattern
     // hooks — implement transform function
   });
   ```

   Key decisions to address in the plan:
   - `frontmatterMap`: maps universal keys → target keys. Use a string for direct rename, or a function `(value, fm) => Record<string, unknown>` for transforms
   - `getOutputPath`: determines output file path from template name and frontmatter
   - Hook `transform`: converts universal event names and handler format to target format
   - Hook `mergeKey`: if the target's hook config is nested inside a larger settings file (like Claude's `settings.json` which has a `hooks` key)

5. **Get plan approval** — present the plan to the user for review. **Do not start implementing until the plan is approved.**

## Phase 3 — Implementation

Once the plan is approved, execute it:

6. **Create the target directory** — create `src/targets/<name>/index.ts`

7. **Implement `defineTarget()`** — follow the approved plan to build the full target definition

8. **Register the target** — add to `src/targets/index.ts` and the `Target` type in `src/types.ts`

9. **Add tests** — create integration tests using existing fixtures that generate for the new target, verifying output paths and content

10. **Run `pnpm check`** to verify everything passes
