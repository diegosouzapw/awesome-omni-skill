---
name: rule-making-skill
description: Analyze a specific directory (e.g. frontend/, backend/, e2e/) and generate .claude/rules/ markdown files for it. Use when asked to create rules, analyze a folder for Claude Code, or set up domain memory for a specific part of the codebase.
---

# Rules Generator

Analyze a directory and generate `.claude/rules/` files for Claude Code domain memory.

## Core Concept

Generalized agents fail because they're "amnesiacs with a tool belt." Each session starts with no grounded sense of where we are.

**Solution: Domain Memory** - persistent structured representation containing:
1. **Goals** - What we're trying to achieve, requirements, constraints
2. **State** - What's passing/failing, what's been tried, what broke
3. **Scaffolding** - How to run, test, extend the system

Rules files distill domain knowledge into quick-reference format.

## When to Use

User asks to:
- "Analyze frontend/ and create rules"
- "Set up Claude rules for the backend"
- "Create domain memory for e2e/"

---

## Workflow

### 1. Check Existing Documentation

Read root CLAUDE.md and target directory CLAUDE.md. Pull down what's already documented. We don't want to duplicate everything, but the main CLAUDE.md still have to keep core overview of the application.

### 2. Detect Concerns

Scan the target directory to detect which concerns exist. Only generate rule files for concerns that are actually present.

### 3. Mine Domain Memory

Check planning board and tasks under `/.tasks/` for decisions, gotchas, and patterns related to each detected concern.

Globpattern: ".task-board/done/*.md"

Use tasks older than #150

### 4. Generate Rule Files

Create focused `.md` files only for detected concerns.

---

## Concern Detection Matrix

Scan for these patterns to detect which concerns exist:

| Concern | Detection Signals | Rule File |
|---------|-------------------|-----------|
| **auth** | `**/auth/**`, `AuthContext`, `AuthProvider`, `login`, `logout`, `session`, `token`, `OAuth`, `EasyAuth` | `auth.md` |
| **data** | `**/models/**`, `cosmosdb`, `database`, `mongoose`, `prisma`, `orm`, `migrations`, `Container` | `data.md` |
| **api** | `**/routes/**`, `**/controllers/**`, `router.get`, `router.post`, `express.Router`, `endpoints` | `api.md` |
| **validation** | `**/validators/**`, `zod`, `yup`, `joi`, `schema`, `.parse(`, `.safeParse(` | `validation.md` |
| **state** | `useQuery`, `useMutation`, `QueryClient`, `zustand`, `redux`, `recoil`, `Context.Provider` | `state.md` |
| **components** | `**/ui/**`, `**/components/**`, `.tsx` files with `Props` interfaces, `forwardRef` | `components.md` |
| **styling** | `tokens.css`, `theme`, `tailwind`, `styled-components`, `css modules`, design system files | `styling.md` |
| **charts** | `d3`, `recharts`, `chart.js`, `victory`, `**/charts/**`, `<svg`, `useEffect` with DOM manipulation | `charts.md` |
| **forms** | `**/forms/**`, `useForm`, `handleSubmit`, `<input`, `<form`, form validation patterns | `forms.md` |
| **calculations** | `**/calculation*`, pure functions returning numbers, financial formulas, `Math.` heavy files | `calculations.md` |
| **llm** | `openai`, `langchain`, `anthropic`, `agent`, `tool calls`, `completion`, `langfuse` | `llm.md` |
| **errors** | `**/errors/**`, `AppError`, `ErrorBoundary`, `errorHandler`, custom error classes | `errors.md` |
| **testing** | `*.spec.ts`, `*.test.ts`, `fixtures`, `beforeEach`, `describe(`, `it(`, `expect(` | `testing.md` |
| **middleware** | `**/middleware/**`, `app.use(`, request/response interceptors, `next()` | `middleware.md` |
| **services** | `**/services/**`, business logic classes, dependency injection patterns | `services.md` |
| **onboarding** | `wizard`, `onboarding`, `setup`, multi-step flows, `Step*.tsx` | `onboarding.md` |
| **integrations** | Third-party SDK imports, API clients, webhooks, external service calls | `integrations.md` |

### Detection Algorithm

```
For each concern in matrix:
  1. Glob for folder patterns (**/auth/**, **/models/**, etc.)
  2. Grep for keyword patterns (AuthContext, useQuery, etc.)
  3. If matches found → concern is DETECTED
  4. If no matches → skip this concern
```

**Threshold**: A concern is detected if:
- At least 1 folder pattern matches, OR
- At least 3 keyword matches across files

---

## Rule File Format

```markdown
# [Concern] Rules

## Stack
[One line: key libs/frameworks for this concern]

## Structure
- `/path` - Purpose
- `/path` - Purpose

## Patterns
- Established pattern 1
- Established pattern 2

## Decisions
- Choice X because Y

## Gotchas
- Problem → Solution

## Commands
- `pnpm <command>` - What it does
```

**Not every file needs all sections** - include only what's relevant.

---

## Example: Detected Concerns → Generated Files

### Target: `backend/`

**Detection results:**
- ✅ auth → `middleware/auth.ts`, `routes/authRoutes.ts`
- ✅ data → `config/cosmosdb.ts`, `models/*.ts`
- ✅ api → `routes/*.ts`, `controllers/*.ts`
- ✅ validation → `validators/*.ts`, zod imports
- ✅ services → `services/*.ts`
- ✅ calculations → `services/calculationService.ts`
- ✅ llm → `services/importAgentService.ts`, openai imports
- ✅ errors → `errors/AppError.ts`, `middleware/errorHandler.ts`
- ✅ middleware → `middleware/*.ts`
- ❌ components → not found
- ❌ styling → not found
- ❌ charts → not found

**Generated files:**
```
backend/.claude/rules/
├── auth.md
├── data.md
├── api.md
├── validation.md
├── services.md
├── calculations.md
├── llm.md
├── errors.md
└── middleware.md
```

### Target: `components/`

**Detection results:**
- ✅ components → `ui/**`, `cards/**`, `layout/**`
- ✅ styling → `styles/tokens.css`
- ✅ charts → `charts/*.tsx`, d3 imports
- ✅ forms → `forms/*.tsx`
- ✅ errors → `system/ErrorBoundary`
- ❌ auth → not found
- ❌ data → not found
- ❌ api → not found

**Generated files:**
```
components/.claude/rules/
├── components.md
├── styling.md
├── charts.md
├── forms.md
└── errors.md
```

### Target: `e2e/`

**Detection results:**
- ✅ testing → `*.spec.ts`, fixtures
- ✅ auth → login helpers, auth fixtures
- ❌ everything else → not found

**Generated files:**
```
e2e/.claude/rules/
├── testing.md
└── auth.md
```

---

## Content Guidelines

### What to Include

| Section | Source |
|---------|--------|
| Stack | `package.json` deps for this concern |
| Structure | Folder layout for this concern only |
| Patterns | Code analysis + task-board history |
| Decisions | Task-board "decided to..." entries |
| Gotchas | Task-board "had issues with..." entries |
| Commands | `package.json` scripts for this concern |

### What NOT to Include

- Anything in root `CLAUDE.md` (no duplication)
- Generic patterns (only project-specific)
- Obvious things (React uses JSX, etc.)

---

## Example Rule Files

### `backend/.claude/rules/data.md`
```markdown
# Data Rules

## Stack
CosmosDB (NoSQL), @azure/cosmos SDK

## Structure
- `/config/cosmosdb.ts` - Connection, container getters
- `/models/` - Document type definitions

## Patterns
- Containers: `users` (partition: /id), `portfolios` (partition: /userId)
- Documents are denormalized (snapshots store full account data)
- Use singleton pattern for database instance

## Decisions
- Denormalized snapshots for historical accuracy (accounts change over time)
- Co-locate user data by userId partition for fast queries

## Gotchas
- Date strings "dd.MM.yyyy" don't sort correctly in CosmosDB
- Always sort dates in JS using `compareDatesAsc` from dateUtils.ts
- Zod strips unknown fields - add to schema or they're dropped

## Commands
- `pnpm --filter backend seed` - Seed demo data
- `pnpm --filter backend seed:reset` - Reset database
```

### `components/.claude/rules/charts.md`
```markdown
# Charts Rules

## Stack
D3.js for all visualizations

## Structure
- `/charts/AreaChart` - Single line/area
- `/charts/StackedAreaChart` - Multiple stacked areas
- `/charts/DonutChart` - Pie/donut charts

## Patterns
- SVG-based, responsive via viewBox
- Data prop: `{ date: string, value: number }[]`
- Colors from design tokens (--muted-sage, --pale-blue, etc.)
- Tooltips via D3 mouse events

## Gotchas
- D3 selections in useEffect with cleanup
- Don't mix D3 DOM manipulation with React state
- Mobile: increase touch targets for tooltips
```

### `e2e/.claude/rules/testing.md`
```markdown
# Testing Rules

## Stack
Playwright

## Structure
- `/tests/*.spec.ts` - Test files
- `/tests/fixtures.ts` - Shared helpers, constants

## Patterns
- `PROTECTED_PAGES` array for auth-required pages
- `login()` helper handles demo authentication
- `clearAuthState()` between tests

## Decisions
- Integration + E2E only, no unit tests
- Sanity checks over comprehensive coverage
- Test user flows, not implementation details

## Gotchas
- Demo login has rate limiting (5 req/min)
- Always clearAuthState() in beforeEach
- Mobile tests use fixtures/mobile-viewports.ts

## Commands
- `pnpm test:e2e` - Run all tests
- `pnpm test:e2e --ui` - Interactive mode
```

---

## Best Practices

1. **Detect first** - Only create files for concerns that exist
2. **Mine task-board** - Decisions and gotchas are gold
3. **No duplication** - If root CLAUDE.md has it, skip it
4. **Be specific** - "Port 3000" not "default port"
5. **Keep short** - Each file <50 lines
6. **Update on discovery** - Rules are living documentation
