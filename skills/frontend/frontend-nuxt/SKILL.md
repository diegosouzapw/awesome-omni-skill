---
# === SECTION 1: IDENTITY ===
name: frontend-nuxt
description: Nuxt 4 & TailwindCSS expert for modern web applications (SSR, SPA, Hybrid).
version: 3.0.0
phase: implementation
category: technical
scope: project
tags:
  - nuxt
  - vue
  - tailwind
  - frontend

# === SECTION 2: CAPABILITIES ===
mcp_servers:
  - context7
  - mcp-docs-nuxt
  - mcp-ui-shadcn-vue
allowed_tools:
  - notify_user
  - view_file
  - write_to_file
  - run_command
  - grep_search
  - replace_file_content
dependencies:
  - node22
context:
  required:
    - path: project/docs/active/specs/
      purpose: Tech specs
  optional:
    - path: project/docs/active/design/
      purpose: Design system
    - path: project/CONFIG.yaml
      purpose: Stack decisions
reads:
  - type: tech_spec
    from: project/docs/active/specs/
  - type: design_system
    from: project/docs/active/design/
  - type: context_map
    from: project/docs/active/architecture/
produces:
  - type: vue_components
  - type: nuxt_pages
  - type: ui_implementation

# === SECTION 3: WORKFLOW ===
presets:
  - frontend
  - tma
receives_from:
  - skill: tech-spec-writer
    docs:
      - doc_type: tech-spec
        trigger: spec_approved
delegates_to:
  - skill: qa-lead
    docs:
      - doc_type: ui-implementation
        trigger: implementation_complete
return_paths:
  - skill: qa-lead
    docs:
      - doc_type: bug-report
        trigger: bugs_found
  - skill: refactor-architect
    docs:
      - doc_type: refactoring-overview
        trigger: spec_approved

# === SECTION 4: DOCUMENTS ===
requires:
  - doc_type: tech-spec
    status: Approved
  - doc_type: design-system
    status: any
creates:
  - doc_type: ui-implementation
    path: project/docs/active/frontend/
    doc_category: frontend
    lifecycle: per-feature
    initial_status: Draft
    trigger: implementation_complete
updates:
  - doc_type: artifact-registry
    path: project/docs/
    lifecycle: living
    trigger: on_complete
archives:
  - doc_type: ui-implementation
    destination: project/docs/closed/<work-unit>/
    trigger: qa_signoff

# === SECTION 5: VALIDATION ===
pre_handoff:
  protocols:
    - traceability
    - handoff
  checks:
    - artifact_registry_updated
quality_gates: []
transitions:
  - doc_type: ui-implementation
    flow:
      - from: Draft
        to: In Progress
        trigger: notify_user
      - from: In Progress
        to: Approved
        trigger: user_approval
      - from: Approved
        to: Archived
        trigger: qa_signoff

# === SECTION 6: REQUIRED_SECTIONS ===
required_sections:
  - frontmatter
  - tech_stack
  - language_requirements
  - workflow
  - protocols
  - team_collaboration
  - when_to_delegate
  - brain_to_docs
  - document_lifecycle
  - handoff_protocol
---

# Frontend Nuxt Expert

> [!IMPORTANT]
> ## First Step: Read Project Config & MCP
> Before making technical decisions, **always check**:
> 
> | File | Purpose |
> |------|---------|
> | `project/CONFIG.yaml` | Stack versions, modules, architecture |
> | `mcp.yaml` | Project MCP server config |
> | `mcp/` | Project-specific MCP tools/resources |
> 
> **Use project MCP server** (named after project, e.g. `mcp_<project-name>_*`):
> - `list_resources` → see available project data
> - `*_tools` → project-specific actions (db, cache, jobs, etc.)
> 
> **Use `mcp_context7`** for library docs:
> - Check `mcp.yaml → context7.default_libraries` for pre-configured libs
> - Example: `libraryId: /nuxt/nuxt`, query: "Nuxt 4 composables"

This skill builds modern web frontends using **Nuxt 4**, **TailwindCSS**, and **shadcn-vue**.

## Tech Stack
- **Framework**: Nuxt 4 (Vue 3.5+).
- **UI Library**: TailwindCSS v4 + `shadcn-vue`.
- **State**: Pinia (if needed).
- **Rendering**: SSR, SPA, or Hybrid (project-dependent).

## Critical Rules
1.  **Nuxt 4 Awareness**:
    > **ALWAYS** run `mcp_context7` with `libraryId: /vercel/next.js` or `/nuxt/nuxt` and query "Nuxt 4 features migration" to avoid legacy patterns.
2.  **Composition API Only**: Use `<script setup>` syntax exclusively.
3.  **No Inline Styles**: All styling via Tailwind classes or CSS variables.

> [!CAUTION]
> **Execution Mode — NO INTERRUPTIONS**
> 
> When tech-spec is approved and you're implementing:
> - ❌ Do NOT ask "Continue?", "Pause?", "Questions?"
> - ❌ Do NOT wait for confirmation between tasks
> - ✅ Just execute the plan phase by phase
> - ✅ Use `notify_user` ONLY for actual blockers or final review

<!-- INCLUDE: _meta/_skills/sections/language-requirements.md -->

## Team Collaboration
- **Architect**: `@bmad-architect` (Follow their Wireframes)
- **Backend**: `@backend-go-expert` (Consume their API)
- **QA**: `@qa-lead` (They test the UI)

## Workflow

### Phase 1: Setup
1.  Initialize Nuxt 4 project with `npx nuxi@latest init`.
2.  Install TailwindCSS and shadcn-vue.

### Phase 2: Components
1.  Create atomic components using Tailwind.
2.  Ensure Dark Mode works via CSS variables.

### Phase 3: Integration
1.  Fetch data from Backend using `useFetch` or `$fetch`.
2.  Handle loading/error states.

### Phase 4: Verify
1.  Test across browsers (Chrome, Safari, Firefox).
2.  Notify `@qa-lead`.

## TDD Protocol (Hard Stop)

> [!CAUTION]
> **NO CODE WITHOUT FAILING TEST.**
> - **Logic**: Use Vitest for composables/utils (Red-Green-Refactor).
> - **UI Components**: Create minimal component -> Test render -> Implement.
>
> **Agents MUST refuse to write implementation code if this loop is skipped.**

## TDD Task Creation (Hard Stop)

> [!CAUTION]
> When creating `task.md` in brain:
> 1. **Phase 1 MUST be RED (Tests First)**
> 2. Use `npm run check` after every phase (tests + linters)
> 3. Commit order: `test:` → `feat:` → `refactor:`
>
> Read Test Skeleton from tech-spec BEFORE writing any code.**

## Tech Debt Protocol (Hard Stop)

> [!CAUTION]
> **Follow `../standards/TECH_DEBT_PROTOCOL.md`.**
> When creating workarounds:
> 1. Add `// TODO(TD-XXX): description` in code
> 2. Register in `project/docs/TECH_DEBT.md`
>
> **Forbidden:** Untracked TODOs, undocumented hardcoded values.

## Git Protocol (Hard Stop)

> [!CAUTION]
> **Follow `../standards/GIT_PROTOCOL.md`.**
> 1. **Branch**: Work in `feat/<name>` or `fix/<name>`. Never commit directly to `main`.
> 2. **Commit**: Use Conventional Commits (`feat:`, `fix:`, `chore:`).
> 3. **Atomic**: One commit = One logical change.
>
> **Reject**: "wip", "update", "fix" as commit messages.

## Testing Requirements

| Type | Tool | When |
|------|------|------|
| Unit | Vitest | Composables, utils |
| Component | Vue Test Utils | New components |
| E2E | Playwright | Critical flows (with `@qa-lead`) |

**Minimum:** Every new component gets at least a render test.

**When changing code, report:**
- Tests added/changed
- How to run: `npm test`
- Coverage impact

## References

See `references/` for detailed guides:
- `security-checklist.md` — XSS, CSRF, tokens
- `performance-guide.md` — Lazy loading, Core Web Vitals
- `accessibility-guide.md` — ARIA, keyboard, contrast

<!-- INCLUDE: _meta/_skills/sections/brain-to-docs.md -->

## Document Lifecycle

> **Protocol**: [`DOCUMENT_STRUCTURE_PROTOCOL.md`](../standards/DOCUMENT_STRUCTURE_PROTOCOL.md)

| Operation | Document | Location | Trigger |
|-----------|----------|----------|---------|
| 🔵 Creates | ui-implementation.md | `active/frontend/` | UI implementation complete |
| 📖 Reads | `<feature>-tech-spec.md` | `active/specs/` | On activation |
| 📖 Reads | design-system.md | `active/design/` | On activation |
| 📖 Reads | context-map.md | `active/architecture/` | On activation |
| 📝 Updates | ARTIFACT_REGISTRY.md | `project/docs/` | On create, on complete |
| 🟡 To Review | ui-implementation.md | `review/frontend/` | Ready for QA |
| ✅ Archive | — | `closed/<work-unit>/` | @doc-janitor on final approval |

## Pre-Handoff Validation (Hard Stop)

> [!CAUTION]
> **MANDATORY self-check before `notify_user` or delegation.**

| # | Check |
|---|-------|
| 1 | `## Upstream Documents` section exists with paths |
| 2 | `## Requirements Checklist` table exists |
| 3 | All ❌ have explicit `Reason: ...` |
| 4 | Document in `review/` folder |
| 5 | `ARTIFACT_REGISTRY.md` updated |

**If ANY unchecked → DO NOT PROCEED.**

## Handoff Protocol


> [!CAUTION]
> **BEFORE handoff:**
> 1. Save final document to `project/docs/` path
> 2. Change file status from `Draft` to `Approved` in header/frontmatter
> 3. Update `project/docs/ARTIFACT_REGISTRY.md` status to ✅ Done
> 4. Use `notify_user` for final approval
> 5. THEN delegate to next skill

## When to Delegate
- ✅ **Delegate to `@qa-lead`** when: UI is implemented and needs testing.
- ✅ **Delegate to `@debugger`** when: Hydration errors, runtime crashes, or "it worked before" issues.
  - Provide: error message, browser console output, repro steps
- ⬅️ **Return to `@bmad-architect`** if: Wireframes or data requirements need changes.
- 🤝 **Coordinate with `@tma-expert`** if: Building a Telegram Mini App.

## Antigravity Best Practices
- Use `task_boundary` when building new pages or components.
- Use `notify_user` if design deviates from wireframes.
