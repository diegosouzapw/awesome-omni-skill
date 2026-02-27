---
name: structure
description: Design data structures before implementation. Maps existing code or creates new types.
---

# /structure [path]

Design data structures and architecture. Context-aware: maps existing code or creates new types from plan.

> **No arguments?** Describe this skill and stop. Do not execute.

## First: Activate Workflow

```bash
mkdir -p .claude && echo '{"skill":"structure","started":"'$(date -Iseconds)'"}' > .claude/active-workflow.json
```

## Mode Detection

```
if path points to existing code:
    MODE = "map"      → Analyze and redesign
else if .claude/plans/ has active plan:
    MODE = "create"   → Create types from plan
else:
    ERROR: Need either existing code path or plan
```

---

## Step 0: Load Expert Guidance

Before starting either mode, read these canon skills and apply their principles throughout:

**Always load (base brain — all 10):**
1. `canon/clarity/SUMMARY.md`
2. `canon/pragmatism/SUMMARY.md`
3. `canon/simplicity/SUMMARY.md`
4. `canon/composition/SUMMARY.md`
5. `canon/distributed/SUMMARY.md`
6. `canon/data-first/SUMMARY.md`
7. `canon/correctness/SUMMARY.md`
8. `canon/algorithms/SUMMARY.md`
9. `canon/abstraction/SUMMARY.md`
10. `canon/optimization/SUMMARY.md`

**Auto-detect domain canon (check files, load matches):**

| Check | If found, also read |
|-------|---------------------|
| `*.ts` or `*.js` files in target | `canon/javascript/typescript/SUMMARY.md`, `canon/javascript/js-safety/SUMMARY.md`, `canon/javascript/js-perf/SUMMARY.md`, `canon/javascript/js-internals/SUMMARY.md`, `canon/javascript/functional/SUMMARY.md` |
| `angular.json` in project root | `canon/angular/angular-arch/SUMMARY.md`, `canon/angular/angular-core/SUMMARY.md`, `canon/angular/angular-perf/SUMMARY.md`, `canon/angular/rxjs/SUMMARY.md` |
| `package.json` contains `"react"` | `canon/javascript/react-state/SUMMARY.md`, `canon/javascript/react-test/SUMMARY.md`, `canon/javascript/reactivity/SUMMARY.md` |
| `pom.xml` or `build.gradle` in project | `canon/java/SUMMARY.md` |
| `*.py` files in target | `canon/python/python-advanced/SUMMARY.md`, `canon/python/python-idioms/SUMMARY.md`, `canon/python/python-patterns/SUMMARY.md`, `canon/python/python-protocols/SUMMARY.md` |
| `*.cs` files or `*.csproj` in project | `canon/csharp/csharp-depth/SUMMARY.md`, `canon/csharp/type-systems/SUMMARY.md`, `canon/csharp/async/SUMMARY.md` |
| `.tsx`, `.jsx`, or HTML template files | `canon/ui-ux/components/SUMMARY.md`, `canon/ui-ux/usability/SUMMARY.md`, `canon/ui-ux/tokens/SUMMARY.md` |
| `d3` in package.json or imports | `canon/visualization/d3/SUMMARY.md`, `canon/visualization/charts/SUMMARY.md`, `canon/visualization/dashboards/SUMMARY.md` |
| SQL files or ORM imports | `canon/database/sql/SUMMARY.md`, `canon/database/sql-perf/SUMMARY.md` |
| Auth, tokens, secrets, encryption | `canon/security/security-mindset/SUMMARY.md`, `canon/security/owasp/SUMMARY.md`, `canon/security/web-security/SUMMARY.md` |

If a skill file doesn't exist (not installed in this project), skip it and continue.
List loaded experts in EXPERTS_LOADED. In EXPERT_DECISIONS, show each specific structural decision an expert drove.

### Step 0b: Learn From Past Mistakes

Read both lessons files if they exist:
1. `.claude/universal-lessons.md` — universal patterns (ships with skills, applies to all projects)
2. `.claude/lessons.md` — project-specific patterns (accumulated from this project's runs)

Apply relevant lessons to your structural design:

- **DESIGN** entries → avoid these architectural mistakes in your target state
- **LOGIC** entries → design types/interfaces that make these bug patterns impossible (e.g., validated newtypes instead of raw strings for paths/names)
- **AI_SMELL** entries → do not create speculative types/interfaces without consumers; avoid over-decomposition

If a file doesn't exist, skip it and continue.

### Step 0c: Load Quality Contracts

Read `.claude/rubric/contracts.md`. This defines 7 abstract types for boundary enforcement. Use them during boundary analysis in both modes.

---

## MODE: Map (Existing Code)

Analyze existing code, diagram relationships, design improvements.

### Step 1: Map Current Architecture

Read all files in target. Output a diagram:

```
## Architecture: [target]

CURRENT_STATE:
┌─────────────┐     ┌─────────────┐
│ UserService │────▶│  AuthStore  │
│             │     │             │
│ - getUser() │     │ - token     │
│ - saveUser()│     │ - validate()│
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────┐     ┌─────────────┐
│   User      │     │   Session   │
│ (interface) │     │ (interface) │
└─────────────┘     └─────────────┘

DEPENDENCIES:
- UserService → AuthStore (tight coupling)
- UserService → User (data)
- AuthStore → Session (data)

ISSUES_FOUND:
- [issue]: [why it matters]

PURITY_CHECK:
- [module]: pure ✓ | impure ✗ (calls [I/O function] directly)
  For each impure module: can business logic be extracted into pure functions
  that take data and return data, with I/O pushed to callers?
```

### Step 2: Apply Canon Wisdom

Review against principles from the masters:

| Principle | Check |
|-----------|-------|
| **Data-first** | Are data structures driving the design? |
| **Composition** | Inheritance depth > 2? Refactor to composition. |
| **Single responsibility** | Does each class do one thing? |
| **Interface segregation** | Are interfaces minimal? |
| **Dependency inversion** | Depend on abstractions, not concretions? |
| **Pure core / impure shell** | Is business logic free of I/O? Core functions should take data and return data — no direct calls to filesystem, network, or database. I/O belongs in a thin outer layer that calls the pure core. If a function both computes and persists, split it. |
| **Impossible states** | Can invalid states be represented? |
| **Quality contracts** | Are boundaries enforced with validated types, or do raw strings pass through? |

### Step 3: Design Target State

```
TARGET_STATE:
┌─────────────┐     ┌─────────────┐
│ UserService │     │ AuthService │
│             │     │ (new)       │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────────────────────────┐
│         UserRepository          │
│         (extracted)             │
└─────────────────────────────────┘

CHANGES_NEEDED:
- Extract AuthService from AuthStore
- Create UserRepository interface
- Inject dependencies via constructor

EXPERTS_LOADED: [list of skill names actually read]
EXPERT_DECISIONS:
- [expert-skill]: [specific structural decision it drove]
```

### Map Mode Output

```markdown
## Structure: [target]

MODE: map

CURRENT_STATE:
[diagram]

ISSUES_FOUND:
- [issue]: [description]

TARGET_STATE:
[diagram]

CHANGES_NEEDED:
- [change 1]
- [change 2]

QUALITY_CONTRACTS:
| Boundary | Abstract Type | Contract | Construction Check |
|----------|--------------|----------|--------------------|
| {boundary} | {type} | {what must be true} | {missing contract = issue} |

EXPERTS_LOADED: [list of skill names actually read]
EXPERT_DECISIONS:
- [expert-skill]: [specific decision it drove]

STRUCTURE_COMPLETE
```

---

## MODE: Create (New Types from Plan)

Create actual type files from an approved plan.

### Requirements

1. **CREATE TYPE FILES** - Use Write tool to create actual .ts files
2. **EVERY TYPE FROM PLAN** - Create all types listed in plan's TYPES section
3. **NO PLACEHOLDER TYPES** - Every field must have a real type, not 'any' or 'unknown'
4. **INVARIANTS AS COMMENTS** - Document invariants as JSDoc comments

### Craft Standards

Types must look like they were designed by a skilled human engineer:

**Reject:**
- Over-engineered type hierarchies
- Generic wrappers when simple types work
- Speculative fields "in case we need them later"
- `Partial<T>` abuse creating unclear contracts

**Embrace:**
- Types that document the domain
- Minimal fields - only what's actually used
- Self-documenting names
- Impossible states made unrepresentable

### Elegance Principles

| Principle | Apply It |
|-----------|----------|
| **Minimal surface** | Only essential methods/fields |
| **Consistent naming** | Same operation = same name everywhere |
| **Orthogonal operations** | Operations compose cleanly |
| **Interface over implementation** | Return `List<T>` not `ArrayList<T>` |
| **Immutability by default** | Prefer readonly. Mutation is explicit. |
| **Pure core / impure shell** | Business logic takes data, returns data. I/O is a separate layer. |
| **Fail-fast** | Validate at boundaries |

### Boundary Analysis (Create Mode)

For each function/module being created, identify boundaries using the detection signals from `.claude/rubric/contracts.md`:
- Where does user input enter? → ValidatedInput
- Where are file paths constructed? → SafePath
- Where are errors caught and re-thrown? → CausedError
- Where are secrets handled? → Secret
- Where is external data read? → ExternalData
- Where could operations hang or grow unbounded? → BoundedOperation
- Where are state mutations that could be retried? → IdempotentAction

Output a QUALITY_CONTRACTS table in the structure output.

### Create Mode Output

```markdown
## Structure: [feature]

MODE: create

TYPES_CREATED:
- src/types/user.ts: User, UserPublic, CreateUserInput
- src/types/auth.ts: AuthToken, TokenPayload

INVARIANTS_DOCUMENTED:
- User: email must be unique, password_hash never exposed
- AuthToken: expires_at must be in future

QUALITY_CONTRACTS:
| Boundary | Abstract Type | Contract | Construction Check |
|----------|--------------|----------|--------------------|
| {where data enters/leaves} | {abstract type} | {what must be true} | {EXPORT_FUNCTION or EXPORT_TYPE to verify} |

EXPERTS_LOADED: [list of skill names actually read]
EXPERT_DECISIONS:
- [expert-skill]: [specific decision it drove]

STRUCTURE_COMPLETE
```

---

## FORBIDDEN (Phase FAILS if detected)

- Using 'any' or 'unknown' types
- TODO comments in types
- Describing without creating (create mode)
- Skipping diagram (map mode)
- No STRUCTURE_COMPLETE marker

## 🛑 MANDATORY STOP

After completing structure analysis/creation:
- DO NOT proceed to `/implementation`
- DO NOT start writing implementation code

**Your turn ends here.** Output STRUCTURE_COMPLETE and STOP.
