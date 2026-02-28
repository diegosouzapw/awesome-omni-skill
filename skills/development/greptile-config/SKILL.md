---
name: greptile-config
description: Generate production-ready Greptile AI code review configuration for any repository. Use this skill whenever the user mentions Greptile, AI code review setup, PR review configuration, automated code review rules, or wants to set up .greptile/ config files. Also trigger when someone says "set up code review", "configure PR reviews", "add review rules", or asks about Greptile configuration — even if they don't say "Greptile" explicitly but describe wanting AI-powered PR review automation. This skill analyzes the actual repository structure and produces tailored config, not generic boilerplate.
---

# Greptile Configuration Generator

Generate optimal Greptile AI code review configuration by analyzing the actual repository — its structure, tech stack, patterns, documentation, and team conventions — then producing tailored `.greptile/` configuration files.

## What Greptile Is

Greptile is an AI code review agent that hooks into GitHub/GitLab PRs. It indexes the full codebase, reads configuration files on each PR (from the source branch), reviews changed files using LLM-powered semantic understanding, and posts inline comments, summaries, confidence scores, and status checks.

The critical insight: Greptile is **not a linter**. It uses LLMs to understand intent, architecture, and cross-file implications. Write rules that leverage semantic understanding — not rules a regex or ESLint could handle. "Service methods must not call HTTP endpoints directly — use the gateway client" is a great Greptile rule. "Use semicolons" is not.

## Workflow

Follow these five phases in order. Do not skip any phase.

### Phase 1: Explore the Repository

Before writing a single line of config, map the territory. Use tools to answer:

1. **Structure** — Is this a monorepo or single-service? What are the top-level directories?
   ```
   Run: ls -la at root, look for packages/, apps/, services/, src/
   ```
2. **Tech stack** — What languages, frameworks, ORMs, and testing tools are in use?
   ```
   Check: package.json, requirements.txt, go.mod, Cargo.toml, build.gradle, etc.
   Look at: tsconfig.json, .eslintrc, prettier config, Dockerfile
   ```
3. **Build artifacts & generated code** — What should be ignored?
   ```
   Look for: dist/, build/, .next/, out/, __generated__/, migrations/
   Check .gitignore for patterns to mirror in ignorePatterns
   ```
4. **Existing documentation** — What context files can the reviewer use?
   ```
   Search for: docs/, architecture.md, ADRs, openapi/, swagger/, prisma/schema.prisma,
   CONTRIBUTING.md, style guides, API specs
   ```
5. **Existing linting & formatting** — What does the toolchain already cover?
   ```
   Check: .eslintrc, .prettierrc, stylelint, rubocop, flake8, golangci-lint
   If strong linting exists → drop "style" from commentTypes
   ```
6. **Team conventions** — Look at recent commits and PR patterns
   ```
   Check: branch naming, commit message style, test patterns, code organization
   ```

### Phase 2: Decide Configuration Strategy

**Which config method?**

| Situation | Method |
|---|---|
| Monorepo with multiple packages/services | `.greptile/` folder with cascading per-directory overrides |
| Single-service repo | `.greptile/` folder (recommended) |
| Different directories need different strictness | Root `.greptile/` + child `.greptile/config.json` in subdirectories |

Always prefer `.greptile/` folder over `greptile.json`. If a `greptile.json` already exists, plan to migrate and delete it (`.greptile/` silently overrides `greptile.json` when both exist).

**Strictness calibration:**

| Level | When to use |
|---|---|
| `1` (Verbose) | Security-critical code, junior-heavy teams, onboarding, early-stage projects |
| `2` (Balanced) | Most production codebases, mixed-seniority teams — **start here** |
| `3` (Critical only) | Senior teams, strong existing linting, pre-release hardening |

For monorepos: set `2` at root, override to `1` in critical paths (payments, auth), override to `3` in low-risk areas (internal tools, scripts).

**Comment types:**

Start with `["logic", "syntax"]`. Add `"style"` only if no Prettier/ESLint. Add `"info"` only if the team wants educational comments (onboarding, junior devs).

### Phase 3: Engineer the Rules

This is the highest-leverage step. Every rule must be:

1. **Specific** — "Functions must not exceed 50 lines" not "Keep functions short"
2. **Measurable** — Pass/fail criteria must be unambiguous
3. **Scoped** — Every rule gets a `scope` array targeting relevant directories. A database rule should not fire on frontend components
4. **Actionable** — The developer must know exactly what to change
5. **Semantic** — Rules that require understanding, not pattern matching. If ESLint can catch it, ESLint should catch it
6. **Identifiable** — Every rule gets a unique `id` so child directories can disable it

**Rule categories to scan for** (check which apply to the repo):

| Category | Signal to look for |
|---|---|
| Security | SQL queries, user input handling, auth code, PII |
| Architecture | Controller/service/repository layers, module boundaries |
| Error handling | try-catch patterns, error logging, error response shapes |
| API contracts | OpenAPI specs, shared API types, cross-service calls |
| Dependencies | Shared libraries, internal packages, design systems |
| Performance | Database queries in loops, N+1 patterns, unbounded fetches |
| Naming | Component naming, hook prefixes, file naming conventions |
| Migration | JS→TS migration, legacy patterns being phased out |
| Compliance | PII handling, logging restrictions, audit requirements |
| Tauri IPC | `#[tauri::command]` handlers, invoke calls, capability/permission scoping, state management with Mutex/RwLock |
| Tauri Security | CSP config, shell plugin usage, filesystem scope, input validation in command handlers, no panics in commands |
| MCP Protocol | Tool/resource/prompt handler compliance, input schema validation with Zod, structured error responses, transport correctness |
| MCP Security | Command injection prevention in tool handlers, file system access controls, secrets not logged, timeout enforcement |
| mcp-use Framework | MCPClient/MCPAgent session lifecycle, async context managers, tool call error handling, multi-server routing, connection cleanup |
| Next.js Website | Server Component boundaries, metadata/SEO, ISR/revalidation, image optimization, Core Web Vitals, structured data |
| Next.js Dashboard | Auth middleware on protected routes, server action input validation, role-based access, data table pagination, error boundaries |
| Next.js Shared | Server vs client component misuse, environment variable exposure, route handler authentication, CSRF via server actions |

**For each rule you write**, explain why it exists by referencing what you found in the repo. Generic rules without repo-specific justification are noise.

### Phase 4: Map Context Files

Scan the repo for documentation that would help the reviewer:

| File type | What it provides | Example scope |
|---|---|---|
| Architecture docs | System boundaries, service topology | All files |
| API specs (OpenAPI/Swagger) | Endpoint contracts | `src/api/**`, `src/routes/**` |
| Database schemas (Prisma, migrations) | Model relationships | `src/db/**`, `src/models/**` |
| ADRs | Decision rationale | All files |
| Style/contribution guides | Team conventions | All files |
| Type definitions / shared contracts | Cross-service types | Service-specific |

Scope context files to relevant directories. A Prisma schema is useless context when reviewing React components.

### Phase 5: Generate and Validate

Generate the configuration files, then run the validation checklist before outputting.

**Files to generate:**

1. `.greptile/config.json` — Primary configuration (review behavior, rules, filters, output format)
2. `.greptile/rules.md` — Prose rules with code examples (only if rules need narrative context)
3. `.greptile/files.json` — Context file mappings (only if documentation exists to reference)
4. Child `.greptile/config.json` files — For monorepo subdirectories that need different settings

**Validation checklist — run before every output:**

- [ ] All JSON is syntactically valid (no trailing commas, no comments)
- [ ] Every `scope` is an **array** of strings, never a comma-separated string
- [ ] `ignorePatterns` is a **newline-separated string** (`\n`), never an array
- [ ] `strictness` is integer 1, 2, or 3
- [ ] `commentTypes` only contains: `"logic"`, `"syntax"`, `"style"`, `"info"`
- [ ] `severity` values only: `"high"`, `"medium"`, `"low"`
- [ ] `patternRepositories` use `org/repo` format, never full URLs
- [ ] Every disableable rule has a unique `id`
- [ ] Every rule is specific and measurable — no vague platitudes
- [ ] Every high-noise rule has a `scope`
- [ ] `fileChangeLimit` is >= 1 (0 skips all PRs)
- [ ] `files.json` paths point to files that actually exist in the repo
- [ ] No `.greptile/` and `greptile.json` coexistence (if migrating, note to delete old)

**Output format:**

For every configuration you produce, include:

1. **File tree** showing exactly which files go where
2. **Each config file** as a complete, valid JSON (or markdown) code block
3. **Reasoning annotations** after each file explaining WHY each major decision was made, referencing specific repo context
4. **Canary test** — a simple verification step to confirm the config is working
5. **Migration notes** if moving from `greptile.json` to `.greptile/`

## Reference Files

Read these when you need detailed specifications:

- **`references/config-spec.md`** — Complete parameter reference for all config files, data types, cascading behavior, and monorepo inheritance rules. Read this when you need to check a specific parameter's format or understand how child configs interact with parent configs.

- **`references/anti-patterns.md`** — Common mistakes, the troubleshooting reasoning chain, and the testing/verification protocol. Read this before finalizing any output to catch errors. Also useful when debugging why a Greptile config isn't working.

- **`references/scenarios.md`** — Complete example configurations for TypeScript backend, React frontend, and monorepo setups. Read these for inspiration, but never copy them verbatim — every rule must be justified by the actual repository context.

## Key Gotchas (Keep in Mind)

- Config is read from the **source branch** of the PR, not the target branch
- Changes take effect on the **next PR**, not retroactively
- `ignorePatterns` skips **review** only — files are still **indexed**
- `includeAuthors: []` means **all authors** (not none)
- `fileChangeLimit: 0` means **skip all PRs** (minimum is 1)
- To suppress "X files reviewed, no comments" messages, use `statusCheck: true` (not `statusCommentsEnabled: false`)
- After ~10 PRs, Greptile auto-suggests rules — duplicates of existing rules may appear (this is normal)
