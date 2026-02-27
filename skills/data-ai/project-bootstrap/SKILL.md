---
name: project-bootstrap
description: "Bootstrap a new or existing project with stack detection and project.json generation. Use when adding a new project, setting up agent system, or generating project manifest. Triggers on: add project, new project, bootstrap project, setup project, detect stack."
---

# Project Bootstrap Skill

Initialize a project with stack-agnostic configuration. For existing projects, auto-detect the tech stack. For new projects, use spec-driven creation with stack recommendations.

---

## The Job

**For Existing Projects:**
1. Auto-detect the technology stack from project files
2. Ask clarifying questions for ambiguous/missing information
3. Collect infrastructure conventions (network, security, AWS, API, testing patterns)
4. Generate `docs/project.json` manifest
5. Set up agent system folder structure (default)
6. Generate ARCHITECTURE.md and CONVENTIONS.md
7. Update the global project registry

**For New Projects (Spec-Driven):**
1. Invoke **spec-analyzer** skill to extract requirements from spec/PRD
2. Invoke **stack-advisor** skill to recommend technology stacks
3. User selects or customizes stack
4. Invoke **project-scaffold** skill to generate boilerplate (future)
5. Generate project agents from templates (future)
6. Generate ARCHITECTURE.md, CONVENTIONS.md, and initial PRD
7. Update the global project registry

## Default Bootstrap Mode (Quick Intake)

Use this as the default for new projects unless the user explicitly asks for advanced/manual setup.

Quick intake asks only:
1. Project name
2. Optional GitHub repo URL
3. One freeform context drop (paste text and optional image attachments)

Assume the user wants the agent system. Do not ask an enable/disable question.

After creation, default immediately to PRD kickoff so Planner can define scope and suggest architecture options.

---

## Step 1: Determine Project Type

Ask the user:

```
═══════════════════════════════════════════════════════════════════════
                         ADD NEW PROJECT
═══════════════════════════════════════════════════════════════════════

Choose an option:

  A. Add existing local project folder
  B. Create new project (quick intake: name + context paste/images)
  C. Create from GitHub repo URL (quick intake + clone)
  D. Advanced/manual stack setup

> _
═══════════════════════════════════════════════════════════════════════
```

**Flow by choice:**
- **Option A** → [Step 2a: Existing Project Path](#step-2a-existing-project-path)
- **Option B** → [Step 2b: Quick New Project Creation](#step-2b-quick-new-project-creation)
- **Option C** → [Step 2c: New Project from GitHub Repo](#step-2c-new-project-from-github-repo)
- **Option D** → [Step 2d: Advanced Manual Stack Selection](#step-2d-advanced-manual-stack-selection)

---

## Step 2a: Existing Project Path

If Option A (existing project):

```
Enter the full path to your project:

> _

Example: ~/code/my-project
```

**Validate the path:**
- Check folder exists: `ls <path>`
- Check it's a git repo: `git -C <path> rev-parse --git-dir`

If not a git repo, ask:
```
This folder is not a git repository. Initialize git? (y/n)
```

**Continue to:** [Step 3: Auto-Detect Stack](#step-3-auto-detect-stack)

---

## Step 2b: Quick New Project Creation

If Option B (quick new project):

Ask for only the essentials:

```
Project name: _

Paste project context (goals, scope, constraints). You can also drop image attachments now.
> _
```

Then:
1. Create `~/code/<project-name-kebab>`
2. Initialize git
3. Run spec analysis from the pasted context (+ images)
4. Continue with stack recommendation and project setup

Agent system is always enabled in this flow.

## Step 2c: New Project from GitHub Repo

If Option C (GitHub repo bootstrap):

```
Project name: _
GitHub repo URL: _

Paste project context (goals, scope, constraints). You can also drop image attachments now.
> _
```

Then:
1. Clone repository to `~/code/<project-name-kebab>`
2. Read existing codebase for stack detection
3. Merge user context with detected stack/capabilities
4. Generate docs manifests and agent system files

Agent system is always enabled in this flow.

## Step 2d: Advanced Manual Stack Selection

If Option D (manual stack selection):

Use the existing advanced stack interview flow below.

---

## Step 2e: Spec-Driven Creation (Detailed)

If user explicitly requests deeper spec-driven analysis beyond quick intake, run the detailed flow below.

### 2b.1: Invoke Spec Analyzer

Load and invoke the `spec-analyzer` skill:

```
═══════════════════════════════════════════════════════════════════════
                      NEW PROJECT FROM SPEC
═══════════════════════════════════════════════════════════════════════

Let's start with your project requirements. This helps me recommend
the best technology stack for your needs.

I'll analyze your spec/PRD and extract:
  • Product type (SaaS, API, CLI, etc.)
  • Features needed (auth, payments, realtime, etc.)
  • Scale expectations
  • Technical constraints
  • Core entities and user stories

───────────────────────────────────────────────────────────────────────

[spec-analyzer skill takes over — see spec-analyzer/SKILL.md]

───────────────────────────────────────────────────────────────────────
```

The spec-analyzer will:
1. Acquire the spec (paste, file, URL, or interactive)
2. Extract structured requirements
3. Present findings for confirmation
4. Output a `RequirementsManifest` JSON

### 2b.2: Invoke Stack Advisor

Once spec analysis is complete, invoke the `stack-advisor` skill:

```
───────────────────────────────────────────────────────────────────────

Requirements analysis complete! Now let me recommend some tech stacks...

[stack-advisor skill takes over — see stack-advisor/SKILL.md]

───────────────────────────────────────────────────────────────────────
```

The stack-advisor will:
1. Load the stack database from `~/.config/opencode/data/stacks.yaml`
2. Score archetypes against requirements
3. Present top 3 recommendations with trade-offs
4. Allow user to select or customize
5. Output a `StackDecision` JSON

### 2b.3: Project Creation

Once stack is selected:

```
Project name: _
(will be converted to kebab-case for folder name)

Parent directory: ~/code
(press Enter to accept default, or enter a different path)
```

Create the project:
```bash
mkdir -p <parent>/<project-name-kebab>
cd <parent>/<project-name-kebab>
git init
```

### 2b.4: Scaffold Generation

Based on the selected archetype from `StackDecision`, invoke the `project-scaffold` skill to generate boilerplate code:

```
───────────────────────────────────────────────────────────────────────

Stack selected! Now generating project scaffold...

[project-scaffold skill takes over — see project-scaffold/SKILL.md]

───────────────────────────────────────────────────────────────────────
```

The project-scaffold will:
1. Select the appropriate scaffold template based on archetype (e.g., `nextjs-prisma`, `go-chi-postgres`)
2. Process template variables from `StackDecision` and `RequirementsManifest`
3. Generate all boilerplate files (package.json, config files, source code)
4. Create database schema from entities (if applicable)
5. Run post-scaffold commands (npm install, prisma generate, git init)
6. Output the list of generated files

### Available Scaffolds

| Archetype | Scaffold | Description |
|-----------|----------|-------------|
| `nextjs-supabase` | `nextjs-supabase` | Next.js 15 + Supabase + Tailwind v4 |
| `nextjs-prisma` | `nextjs-prisma` | Next.js 15 + Prisma + NextAuth.js + Tailwind v4 |
| `go-api-postgres` | `go-chi-postgres` | Go Chi + PostgreSQL + JWT auth |

**If scaffold not available:** Skip scaffold generation and continue to Step 2b.5. The user will need to manually set up their project structure.

### 2b.5: Generate Project Agents

Generate project-specific agent definitions from templates based on the selected stack. These agents contain project-tailored guidance that the global routers (critic, tester) will use instead of generic agents.

**Templates Directory:** `~/.config/opencode/agent-templates/`

#### Template Selection Logic

Based on `docs/project.json` values, select applicable templates:

| project.json Path | Template | Output File |
|-------------------|----------|-------------|
| `stack.languages` contains "typescript" | `critics/typescript.md` | `docs/agents/typescript-critic.md` |
| `stack.languages` contains "go" | `critics/go.md` | `docs/agents/go-critic.md` |
| `stack.languages` contains "python" | `critics/python.md` | `docs/agents/python-critic.md` |
| `apps.*.framework` is "nextjs", "remix", or "react" | `frontend/react.md` | `docs/agents/react-dev.md` |
| `apps.*.framework` is "vue" or "nuxt" | `frontend/vue.md` | `docs/agents/vue-dev.md` |
| `apps.*.framework` is "svelte" or "sveltekit" | `frontend/svelte.md` | `docs/agents/svelte-dev.md` |
| `stack.runtime` is "go" + Chi detected | `backend/go-chi.md` | `docs/agents/go-dev.md` |
| `stack.runtime` is "node" + Express detected | `backend/node-express.md` | `docs/agents/express-dev.md` |
| `stack.runtime` is "python" + FastAPI detected | `backend/python-fastapi.md` | `docs/agents/fastapi-dev.md` |
| `styling.framework` is "tailwind" | `styling/tailwind.md` | `docs/agents/tailwind.md` |
| `testing.unit` is "jest" + React | `testing/jest-react.md` | `docs/agents/react-tester.md` |
| `testing.unit` is "jest" (backend only) | `testing/jest-tester.md` | `docs/agents/jest-tester.md` |
| `testing.unit` contains "go" tests | `testing/go-test.md` | `docs/agents/go-tester.md` |
| `testing.unit` is "pytest" | `testing/pytest.md` | `docs/agents/pytest-tester.md` |
| `testing.e2e` is "playwright" | `testing/playwright.md` | `docs/agents/playwright-tester.md` |

#### Template Rendering

Templates use Handlebars-style syntax. Render them with context from `project.json` and `CONVENTIONS.md`:

**Template Variables:**
```javascript
const context = {
  PROJECT: projectJson,                    // Full project.json object
  CONVENTIONS: conventionsMarkdown,        // Raw CONVENTIONS.md content
  PROJECT_NAME: projectJson.name,
  PROJECT_PATH: projectPath,
  
  // Computed booleans for conditionals
  HAS_DARK_MODE: projectJson.styling?.darkMode?.enabled,
  DARK_MODE_STRATEGY: projectJson.styling?.darkMode?.strategy,
  USES_TAILWIND: projectJson.styling?.framework === 'tailwind',
  TAILWIND_VERSION: projectJson.styling?.version || '4',
  USES_TYPESCRIPT: projectJson.stack?.languages?.includes('typescript'),
  USES_SUPABASE: projectJson.database?.client === 'supabase',
  USES_PRISMA: projectJson.database?.client === 'prisma',
  USES_DRIZZLE: projectJson.database?.client === 'drizzle',
  DATABASE_TYPE: projectJson.database?.type,
  TESTING_FRAMEWORK: projectJson.testing?.unit,
  E2E_FRAMEWORK: projectJson.testing?.e2e,
};
```

**Conditional Syntax:**
```handlebars
{{#if HAS_DARK_MODE}}
## Dark Mode
Always include dark: variants for colors.
{{/if}}

{{#if DARK_MODE_STRATEGY == 'class'}}
Use .dark class on html element.
{{else if DARK_MODE_STRATEGY == 'media'}}
Use prefers-color-scheme media query.
{{/if}}

{{#if USES_PRISMA}}
Use Prisma client for database queries.
{{else if USES_DRIZZLE}}
Use Drizzle ORM for type-safe queries.
{{else if USES_SUPABASE}}
Use Supabase client for database operations.
{{/if}}
```

#### Generation Process

1. **Create agents directory:**
   ```bash
   mkdir -p docs/agents
   ```

2. **For each applicable template:**
   - Read template from `~/.config/opencode/agent-templates/<category>/<template>.md`
   - Replace `{{VARIABLE}}` placeholders with context values
   - Process `{{#if CONDITION}}...{{else}}...{{/if}}` blocks
   - Write rendered agent to `docs/agents/<output-name>.md`

3. **Create agents manifest** at `docs/agents/manifest.json`:
   ```json
   {
     "generated": "<timestamp>",
     "fromStack": {
       "languages": ["typescript", "go"],
       "framework": "nextjs",
       "styling": "tailwind",
       "testing": "jest"
     },
     "agents": [
       {
         "name": "typescript-critic",
         "template": "critics/typescript.md",
         "output": "docs/agents/typescript-critic.md"
       },
       {
         "name": "react-dev",
         "template": "frontend/react.md",
         "output": "docs/agents/react-dev.md"
       }
     ]
   }
   ```

4. **Update project.json** to indicate agents were generated:
   ```json
   {
     "agents": {
       ...existing config...,
       "projectAgents": "docs/agents/",
       "agentsManifest": "docs/agents/manifest.json"
     }
   }
   ```

**Continue to:** [Step 9: Agent System Setup](#step-9-agent-system-setup)

---

## Step 2d: Manual Stack Selection

If Option D (manual stack selection):

User knows their stack and just wants to set up the project structure.

```
═══════════════════════════════════════════════════════════════════════
                      MANUAL PROJECT SETUP
═══════════════════════════════════════════════════════════════════════

Project name: _
(will be converted to kebab-case for folder name)

Parent directory: ~/code
(press Enter to accept default, or enter a different path)
```

Create the project:
```bash
mkdir -p <parent>/<project-name-kebab>
cd <parent>/<project-name-kebab>
git init
```

Then ask about stack choices one by one:

```
───────────────────────────────────────────────────────────────────────
FRONTEND
───────────────────────────────────────────────────────────────────────

  1. Next.js (React, full-stack)
  2. Remix (React, edge-focused)
  3. Nuxt (Vue, full-stack)
  4. SvelteKit (Svelte, full-stack)
  5. Vite + React (SPA)
  6. Vite + Vue (SPA)
  7. Astro (content-focused)
  8. None (API only)

> _
```

Continue through:
- **Backend** (if not fullstack frontend)
- **Database** — Supabase, Postgres, PlanetScale, MongoDB, etc.
- **Auth** — Based on previous choices
- **Styling** — Tailwind, CSS Modules, etc.
- **Testing** — Jest/Vitest, Playwright/Cypress

After collecting choices, **continue to:** [Step 9: Agent System Setup](#step-9-agent-system-setup)

---

## Step 3: Auto-Detect Stack

Scan the project directory for common files and infer the stack.

### Detection Rules

Run these checks in parallel for speed:

```bash
# Check for various config files
ls package.json 2>/dev/null          # Node.js/JavaScript/TypeScript
ls go.mod 2>/dev/null                 # Go
ls Cargo.toml 2>/dev/null             # Rust
ls pyproject.toml setup.py requirements.txt 2>/dev/null  # Python
ls pom.xml build.gradle 2>/dev/null   # Java
ls Gemfile 2>/dev/null                # Ruby
ls composer.json 2>/dev/null          # PHP
ls *.csproj *.sln 2>/dev/null         # C#/.NET
```

### package.json Analysis

If `package.json` exists, read it and detect:

| File/Dependency | Detection |
|-----------------|-----------|
| `dependencies.next` | Framework: Next.js |
| `dependencies.react` without next | Framework: React (CRA/Vite) |
| `dependencies.vue` | Framework: Vue |
| `dependencies.@angular/core` | Framework: Angular |
| `dependencies.svelte` | Framework: Svelte/SvelteKit |
| `dependencies.express` | Framework: Express |
| `dependencies.fastify` | Framework: Fastify |
| `devDependencies.typescript` | Language: TypeScript |
| `devDependencies.tailwindcss` | Styling: Tailwind |
| `devDependencies.jest` | Testing: Jest |
| `devDependencies.vitest` | Testing: Vitest |
| `devDependencies.@playwright/test` | E2E: Playwright |
| `devDependencies.cypress` | E2E: Cypress |
| `devDependencies.eslint` | Linting: ESLint |
| `devDependencies.prettier` | Linting: Prettier |
| `devDependencies.biome` | Linting: Biome |
| `dependencies.@supabase/supabase-js` | Integration: Supabase |
| `dependencies.prisma` or `@prisma/client` | DB Client: Prisma |
| `dependencies.drizzle-orm` | DB Client: Drizzle |
| `dependencies.stripe` | Integration: Stripe |
| `dependencies.resend` | Integration: Resend |
| `dependencies.openai` | Integration: OpenAI |
| `workspaces` field exists | Structure: Monorepo |

### go.mod Analysis

If `go.mod` exists, detect:

| Pattern | Detection |
|---------|-----------|
| `github.com/gin-gonic/gin` | Framework: Gin |
| `github.com/go-chi/chi` | Framework: Chi |
| `github.com/labstack/echo` | Framework: Echo |
| `github.com/gofiber/fiber` | Framework: Fiber |
| `github.com/jackc/pgx` | DB Client: pgx |
| `gorm.io/gorm` | DB Client: GORM |

### Directory Structure Detection

```bash
# Check for monorepo patterns
ls apps/ packages/ 2>/dev/null        # Turborepo/monorepo style
ls src/ 2>/dev/null                   # Single app style

# Check for specific frameworks
ls app/ 2>/dev/null                   # Next.js App Router
ls pages/ 2>/dev/null                 # Next.js Pages Router
ls routes/ 2>/dev/null                # Remix/SvelteKit

# Check for existing agent system
ls docs/prd-registry.json 2>/dev/null
ls docs/project.json 2>/dev/null

# Check for database
ls supabase/ 2>/dev/null              # Supabase
ls prisma/ 2>/dev/null                # Prisma
ls drizzle/ 2>/dev/null               # Drizzle
ls migrations/ 2>/dev/null            # Generic migrations

# Check for config files
ls tailwind.config.* 2>/dev/null      # Tailwind
ls jest.config.* 2>/dev/null          # Jest
ls vitest.config.* 2>/dev/null        # Vitest
ls playwright.config.* 2>/dev/null    # Playwright
ls .eslintrc* eslint.config.* 2>/dev/null  # ESLint
```

### Build Detection Summary

After scanning, compile a detection summary:

```javascript
const detected = {
  languages: [],           // ['typescript', 'go']
  runtime: null,           // 'node' | 'go' | 'python' | etc
  packageManager: null,    // 'npm' | 'yarn' | 'pnpm' | 'bun'
  framework: null,         // 'nextjs' | 'express' | 'chi' | etc
  frameworkVersion: null,
  structure: null,         // 'monorepo' | 'single-app'
  styling: null,           // 'tailwind' | 'css-modules' | etc
  database: null,          // 'postgres' | 'mysql' | etc
  databaseClient: null,    // 'supabase' | 'prisma' | etc
  testing: {
    unit: null,            // 'jest' | 'vitest' | etc
    e2e: null              // 'playwright' | 'cypress' | etc
  },
  linting: [],             // ['eslint', 'prettier']
  integrations: [],        // ['stripe', 'resend', 'openai']
  hasAgentSystem: false,
  confidence: {}           // Track confidence per detection
};
```

---

## Step 4: Present Detection Results

Show what was auto-detected and ask for confirmation/corrections:

```
═══════════════════════════════════════════════════════════════════════
                      DETECTED PROJECT STACK
═══════════════════════════════════════════════════════════════════════

I scanned your project and detected the following:

  ✅ Languages:      TypeScript, Go
  ✅ Runtime:        Node.js
  ✅ Package Mgr:    npm
  ✅ Framework:      Next.js 15
  ✅ Structure:      Monorepo (apps/, packages/)
  ✅ Styling:        Tailwind CSS v4
  ✅ Database:       Postgres (via Supabase)
  ✅ Unit Testing:   Jest
  ✅ E2E Testing:    Playwright
  ✅ Linting:        ESLint, Prettier
  ✅ Integrations:   Supabase, Stripe, Resend, OpenAI

  ⚠️  Could not detect:
     - Dark mode strategy (class vs media query)
     - Multi-tenant architecture
     - Development server port

═══════════════════════════════════════════════════════════════════════

Is this correct? (y/n, or enter numbers to fix specific items)
> _
```

---

## Step 5: Clarifying Questions

For items that couldn't be auto-detected or need confirmation, ask targeted questions.

### Question Format

Use lettered options for quick responses:

```
1. Does your app support dark mode?
   A. Yes, using class strategy (.dark on html)
   B. Yes, using media query (prefers-color-scheme)
   C. Yes, using system preference with toggle
   D. No dark mode

2. What port does your dev server run on?
   A. 3000 (default)
   B. 5000
   C. 5001
   D. Other: ___
   E. No local runtime (code-only project, can't run locally)

3. Is this a multi-tenant application?
   A. Yes (data isolated per organization/tenant)
   B. No (single tenant or user-based only)

4. What git workflow should agents use?
   A. Trunk-based (commit directly to main)
   B. Feature branches with PRs
   C. Feature branches without PRs
```

**User can respond:** `1A, 2C, 3A, 4A`

---

## Step 6: App Structure Discovery

For monorepos or complex projects, discover the app structure:

```
I found multiple apps in your project:

  apps/
  ├── web/          (Next.js frontend)
  └── api/          (Go backend)

  packages/
  └── types/        (shared)

Should I map these? (y/n)
> _
```

If yes, for each app ask:
- Entry point directory
- Key structure directories (components, hooks, handlers, etc.)
- Development port

Or attempt auto-detection:
```bash
# For Next.js apps
ls app/ pages/ 2>/dev/null            # Entry point
ls components/ 2>/dev/null
ls hooks/ lib/ utils/ 2>/dev/null

# For Go apps
ls cmd/ 2>/dev/null                   # Entry point
ls internal/ pkg/ 2>/dev/null
```

---

## Step 7: Commands Discovery

Detect available commands from package.json scripts or Makefile:

```javascript
// From package.json
const scripts = packageJson.scripts || {};
const commandMapping = {
  'dev': scripts.dev || scripts.start || scripts.serve,
  'build': scripts.build,
  'test': scripts.test,
  'testUnit': scripts['test:unit'] || scripts.test,
  'testE2E': scripts['test:e2e'] || scripts.e2e,
  'typecheck': scripts.typecheck || scripts['type-check'] || scripts.tsc,
  'lint': scripts.lint,
  'lintFix': scripts['lint:fix'] || scripts.fix,
  'format': scripts.format || scripts.prettier
};
```

Present for confirmation:
```
Detected commands (from package.json):

  dev:       npm run dev
  build:     npm run build
  test:      npm run test
  typecheck: npm run typecheck
  lint:      npm run lint
  
  ⚠️  Not found: test:e2e, format

Are these correct? (y/n)
> _
```

---

## Step 8: Features Detection

Ask about higher-level features:

```
Which features does your project include?

  [x] Authentication (detected: @supabase/ssr)
  [x] Payments (detected: stripe)
  [x] Email (detected: resend)
  [x] AI/LLM (detected: openai)
  [ ] Internationalization (i18n)
  [x] Dark mode
  [ ] Marketing pages
  [ ] Support documentation
  [ ] Real-time updates
  [x] API (detected: app/api/)

Enter letters to toggle, or press Enter to confirm:
  A=Auth, B=Payments, C=Email, D=AI, E=i18n, F=DarkMode, 
  G=Marketing, H=SupportDocs, I=Realtime, J=API

> _
```

---

## Step 8b: Infrastructure Conventions

For projects with backend/API components, ask about infrastructure patterns that critics and dev agents need to know:

```
═══════════════════════════════════════════════════════════════════════
                    INFRASTRUCTURE CONVENTIONS
═══════════════════════════════════════════════════════════════════════

AI agents can review network, security, and API patterns more accurately 
if they know your conventions. Answer what applies to your project:

───────────────────────────────────────────────────────────────────────
NETWORK & HTTP
───────────────────────────────────────────────────────────────────────

1. Do you have a standard HTTP client wrapper with built-in retries/timeouts?
   A. Yes (I'll provide the path)
   B. No, we use fetch/axios/http directly
   C. N/A (no external HTTP calls)

2. What are your timeout conventions? (Enter to skip if not standardized)
   Internal API connect/read: ___/___ms  (e.g., 5000/30000)
   External API connect/read: ___/___ms
   Database timeout: ___ms

> _
```

If they have a wrapper, ask for the path:
```
Path to HTTP client wrapper (relative to project root):
> lib/http/client.ts
```

Continue with security if they have backend code:

```
───────────────────────────────────────────────────────────────────────
SECURITY
───────────────────────────────────────────────────────────────────────

3. Where is your authentication middleware defined?
   > _ (e.g., src/middleware/auth.ts, or "Supabase auth" for managed)

4. What CSRF protection strategy do you use?
   A. Double-submit cookie
   B. Synchronizer token
   C. SameSite cookie only
   D. None / N/A (API-only with token auth)

5. Where is CORS configured?
   > _ (e.g., src/config/cors.ts, or "Next.js middleware")

6. What input validation library do you use?
   A. Zod
   B. Yup  
   C. Joi
   D. class-validator
   E. None / manual validation
   F. Other: ___

> _
```

Continue with AWS if detected:

```
───────────────────────────────────────────────────────────────────────
AWS (detected: aws-sdk usage)
───────────────────────────────────────────────────────────────────────

7. Do you have a standard AWS client wrapper?
   A. Yes (I'll provide the path)
   B. No, direct SDK usage
   
8. Do AWS services run locally in development?
   A. Yes (LocalStack, DynamoDB Local, etc.)
   B. No, we use real AWS in dev
   C. Mixed (some local, some real)

9. What infrastructure-as-code tool do you use?
   A. CDK
   B. CloudFormation
   C. Terraform
   D. SAM
   E. Serverless Framework
   F. None

> _
```

Continue with API conventions:

```
───────────────────────────────────────────────────────────────────────
API DESIGN
───────────────────────────────────────────────────────────────────────

10. What pagination style do you use?
    A. Offset-based (page, pageSize)
    B. Cursor-based (cursor, limit)
    C. Limit-offset (offset, limit)
    D. None / not applicable

11. Do you use a standard response envelope?
    A. Yes: { data: ..., meta?: ... }
    B. Yes: { result: ... }
    C. No envelope (return data directly)
    D. Other (I'll describe in CONVENTIONS.md)

> _
```

---

## Step 8c: Documentation & AI Tools Systems

If the project has support documentation or AI features, ask about the systems:

```
───────────────────────────────────────────────────────────────────────
DOCUMENTATION SYSTEM (for docs-writer, support-article-writer)
───────────────────────────────────────────────────────────────────────

12. What documentation system do you use for user-facing docs?
    A. Markdown files (in docs/ or content/)
    B. Docusaurus
    C. VitePress
    D. Database-backed (stored in Supabase/Postgres)
    E. Notion
    F. None yet

If database-backed:
    Table name for articles: ___
    
> _
```

If AI features detected:

```
───────────────────────────────────────────────────────────────────────
AI TOOLS SYSTEM (for tools-writer)
───────────────────────────────────────────────────────────────────────

13. What AI tool system do you use for chatbot/agent tools?
    A. OpenAI function calling
    B. LangChain tools
    C. MCP (Model Context Protocol)
    D. Custom system
    E. None yet

If a system is selected:
    Tool schema file: ___
    Tool implementation file: ___

> _
```

---

## Step 8d: Testing Conventions

For projects with test frameworks detected:

```
───────────────────────────────────────────────────────────────────────
TESTING CONVENTIONS
───────────────────────────────────────────────────────────────────────

14. Where are your tests located?
    A. Co-located (*.test.ts next to source)
    B. Co-located in __tests__/ directories
    C. Centralized (test/ or tests/ directory)
    D. Mixed

15. What should be mocked in tests?
    (Select all that apply)
    A. External HTTP APIs
    B. Time/dates
    C. Database (we use test DB)
    D. AWS services (we use LocalStack)
    E. Nothing specific

16. What should NOT be mocked?
    (Select all that apply)
    A. Database (test against real/local DB)
    B. AWS services (test against local services)
    C. Internal services (test integration)
    
> _
```

If E2E tests detected:

```
17. For E2E tests, what selector strategy do you prefer?
    A. Role-based (getByRole) - recommended
    B. Test IDs (getByTestId)
    C. Text content (getByText)
    D. Mixed / no strong preference

18. How do E2E tests handle authentication?
    A. Storage state from global setup
    B. Mock auth API responses
    C. Real login flow each test
    D. Not sure yet

> _
```

Store all infrastructure convention answers for use when generating CONVENTIONS.md.

---

## Step 8e: Git Workflow

Ask about branching strategy:

```
───────────────────────────────────────────────────────────────────────
GIT WORKFLOW
───────────────────────────────────────────────────────────────────────

19. What branching strategy do you use?

    A. Trunk-based
       All work happens on main with short-lived feature branches (< 1 day).
       No long-running branches. Merge directly to main.
       → Best for: Solo devs, small teams, rapid iteration, CI/CD-heavy
       
    B. GitHub Flow
       Feature branches → Pull Request → Merge to main → Deploy from main.
       Simple and effective for continuous deployment.
       → Best for: Most SaaS projects, continuous deployment
       
    C. Git Flow
       Feature branches → develop branch → release branches → main.
       Structured workflow with develop for integration, release/* for 
       stabilization, and main for production.
       → Best for: Projects with formal release cycles, multiple environments
       
    D. Release Branches
       Feature branches → develop → release/* → main.
       Similar to Git Flow but with emphasis on scheduled releases.
       Release branches progress through test → stage → production.
       → Best for: Enterprise, regulated industries, scheduled releases

    Default: A (Trunk-based)

> _
```

If they choose C (Git Flow) or D (Release Branches), ask follow-up:

```
20. What is your integration branch called?
    This is where feature branches merge before release.
    Default: develop
    > _

21. What pattern do you use for release branches?
    Default: release/*
    > _
```

---

## Step 9: Agent System Setup

Always enable PRD-based agent system for bootstrap flows.

Create:
```bash
mkdir -p docs/prds docs/drafts docs/completed docs/bugs docs/memory docs/abandoned
```

And create registry files (see Step 10).

Do not ask an enable/disable question for agent system setup.

---

## Step 9a: Seed Architecture Guardrails and Bounded-Context Docs

During bootstrap, generate baseline architecture automation artifacts by default.

1. Seed architecture guardrails from detected structure:
   - import boundary rules
   - layer constraints (UI/app/domain/data)
   - restricted direct-access rules
2. Seed bounded-context docs:
   - `docs/architecture/bounded-contexts.md`
   - optional `docs/architecture/contexts/*.md`
3. Persist default policies in `docs/project.json`:

```json
{
  "architecture": {
    "guardrails": {
      "enabled": true,
      "strictness": "standard",
      "exceptionsPath": "docs/architecture/guardrail-exceptions.md"
    },
    "boundedContexts": {
      "enabled": true,
      "policy": "strict",
      "docsPath": "docs/architecture/bounded-contexts.md"
    }
  }
}
```

Supported strictness values for guardrails: `fast`, `standard`, `strict`.

Only ask users to override these defaults when they explicitly request policy customization.

---

## Step 9b: Documentation Templates

Generate both documentation templates by default:

- `docs/ARCHITECTURE.md`
- `docs/CONVENTIONS.md`

Do not prompt for this in quick intake mode.

Templates are located at:
- `~/.config/opencode/templates/ARCHITECTURE.md`
- `~/.config/opencode/templates/CONVENTIONS.md`

When generating, replace placeholders with detected values:

### Basic Placeholders

| Placeholder | Source |
|-------------|--------|
| `{{PROJECT_NAME}}` | User-provided or detected |
| `{{DESCRIPTION}}` | User-provided |
| `{{PROJECT_ROOT}}` | Project path |
| `{{STRUCTURE}}` | Auto-detected directory tree |
| `{{DATABASE_TYPE}}` | `database.type` from detection |
| `{{DATABASE_CLIENT}}` | `database.client` from detection |
| `{{STYLING_FRAMEWORK}}` | `styling.framework` from detection |
| `{{DARK_MODE_STRATEGY}}` | `styling.darkMode.strategy` from detection |
| `{{LANGUAGE}}` | Primary language (tsx, ts, go, etc.) |
| `{{DEV_PORT}}` | Detected or user-provided |
| `{{DATE}}` | Current date |

### Infrastructure Convention Placeholders

| Placeholder | Source |
|-------------|--------|
| `{{HTTP_CLIENT_WRAPPER}}` | Step 8b Q1 (path or "none") |
| `{{INTERNAL_CONNECT_TIMEOUT}}` | Step 8b Q2 |
| `{{INTERNAL_READ_TIMEOUT}}` | Step 8b Q2 |
| `{{EXTERNAL_CONNECT_TIMEOUT}}` | Step 8b Q2 |
| `{{EXTERNAL_READ_TIMEOUT}}` | Step 8b Q2 |
| `{{RETRY_POLICY}}` | Step 8b (inferred from wrapper or default) |
| `{{AUTH_MIDDLEWARE_PATTERN}}` | Step 8b Q3 |
| `{{CSRF_PATTERN}}` | Step 8b Q4 |
| `{{CORS_PATTERN}}` | Step 8b Q5 |
| `{{VALIDATION_LIBRARY}}` | Step 8b Q6 |
| `{{AWS_CLIENT_WRAPPER}}` | Step 8b Q7 |
| `{{AWS_LOCAL_DEV}}` | Step 8b Q8 |
| `{{AWS_IAC_TOOL}}` | Step 8b Q9 |
| `{{PAGINATION_STYLE}}` | Step 8b Q10 |
| `{{API_RESPONSE_ENVELOPE}}` | Step 8b Q11 |
| `{{DOCS_SYSTEM_TYPE}}` | Step 8c Q12 |
| `{{AI_TOOLS_SYSTEM}}` | Step 8c Q13 |
| `{{UNIT_TESTS_LOCATION}}` | Step 8d Q14 |
| `{{MOCK_TARGETS}}` | Step 8d Q15 |
| `{{NO_MOCK_TARGETS}}` | Step 8d Q16 |
| `{{E2E_SELECTOR_PRIORITY}}` | Step 8d Q17 |
| `{{E2E_AUTH_PATTERN}}` | Step 8d Q18 |

For sections that can't be auto-filled, leave the placeholder with a 
`<!-- BOOTSTRAP NOTE: ... -->` comment explaining what to add.

---

## Step 9c: Generate Project-Specific Agents

Generate project-specific agent definitions from templates. These agents are tailored to the project's stack and conventions, and are used by routers (critic, tester) instead of generic global agents.

```
═══════════════════════════════════════════════════════════════════════
                    PROJECT-SPECIFIC AGENTS
═══════════════════════════════════════════════════════════════════════

I can generate project-specific AI agents that understand your exact 
stack and conventions. These replace generic agents with tailored ones:

  Based on your stack, I'll generate:
  
    ✅ typescript-critic.md   (TypeScript code review)
    ✅ react-dev.md           (React development patterns)
    ✅ react-tester.md        (Jest + React Testing Library)
    ✅ tailwind.md            (Tailwind CSS patterns)
    ⬜ go-critic.md           (not applicable - no Go detected)
    
  These agents know:
    • Your database: Supabase with Prisma
    • Your styling: Tailwind v4 with class-based dark mode
    • Your testing: Jest + Playwright
    • Your conventions from CONVENTIONS.md

Generate project-specific agents?

  A. Yes, generate all applicable agents (recommended)
  B. Let me select which agents to generate
  C. No, use global agents only

> _
═══════════════════════════════════════════════════════════════════════
```

### Agent Selection (if Option B)

If user wants to select specific agents:

```
Select agents to generate (enter letters, e.g., A,B,D):

  Critics (code review):
    A. typescript-critic.md  - TypeScript-specific review patterns
    B. python-critic.md      - Python-specific review patterns  
    C. go-critic.md          - Go-specific review patterns

  Development (implementation guidance):
    D. react-dev.md          - React/Next.js patterns
    E. vue-dev.md            - Vue/Nuxt patterns
    F. svelte-dev.md         - Svelte/SvelteKit patterns
    G. express-dev.md        - Express.js patterns
    H. fastapi-dev.md        - FastAPI patterns
    I. go-dev.md             - Go Chi patterns

  Styling:
    J. tailwind.md           - Tailwind CSS patterns

  Testing:
    K. react-tester.md       - Jest + React Testing Library
    L. jest-tester.md        - Jest for backend TypeScript
    M. go-tester.md          - Go testify patterns
    N. pytest-tester.md      - Python pytest patterns
    O. playwright-tester.md  - Playwright E2E testing

> _
```

### Agent Generation Process

1. **Create agents directory:**
   ```bash
   mkdir -p docs/agents
   ```

2. **For each selected template:**

   a. Read template from `~/.config/opencode/agent-templates/<category>/<name>.md`
   
   b. Build context object:
   ```javascript
   const context = {
     PROJECT: projectJson,
     PROJECT_NAME: projectJson.name,
     PROJECT_PATH: projectPath,
     
     // From project.json
     HAS_DARK_MODE: projectJson.styling?.darkMode?.enabled,
     DARK_MODE_STRATEGY: projectJson.styling?.darkMode?.strategy,
     USES_TAILWIND: projectJson.styling?.framework === 'tailwind',
     TAILWIND_VERSION: projectJson.styling?.version || '4',
     USES_TYPESCRIPT: projectJson.stack?.languages?.includes('typescript'),
     DATABASE_TYPE: projectJson.database?.type,
     DATABASE_CLIENT: projectJson.database?.client,
     TESTING_FRAMEWORK: projectJson.testing?.unit,
     E2E_FRAMEWORK: projectJson.testing?.e2e,
     
     // Computed from detection/selection
     USES_SUPABASE: projectJson.database?.client === 'supabase',
     USES_PRISMA: projectJson.database?.client === 'prisma',
     USES_DRIZZLE: projectJson.database?.client === 'drizzle',
     USES_ZOD: projectJson.security?.inputValidation === 'zod',
     USES_NEXT_APP_ROUTER: projectJson.apps?.web?.framework === 'nextjs' && projectJson.apps?.web?.router === 'app',
   };
   ```
   
   c. Process template:
   - Replace `{{VARIABLE}}` with context values
   - Evaluate `{{#if CONDITION}}...{{else}}...{{/if}}` blocks
   - Remove unfulfilled conditional blocks entirely
   
   d. Write to `docs/agents/<output-name>.md`

3. **Create manifest** at `docs/agents/manifest.json`:
   ```json
   {
     "generated": "2026-02-19T10:30:00Z",
     "fromStack": {
       "languages": ["typescript"],
       "framework": "nextjs",
       "styling": "tailwind",
       "testing": "jest"
     },
     "agents": [
       {
         "name": "typescript-critic",
         "template": "critics/typescript.md",
         "output": "docs/agents/typescript-critic.md",
         "purpose": "TypeScript code review"
       }
     ]
   }
   ```

4. **Update project.json:**
   ```json
   {
     "agents": {
       "gitWorkflow": "trunk-based",
       "autoCommit": true,
       "projectAgents": "docs/agents/",
       "agentsManifest": "docs/agents/manifest.json"
     }
   }
   ```

### Template Syntax Reference

Templates use Handlebars-style syntax:

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{VAR}}` | Simple substitution | `{{PROJECT_NAME}}` → "MyApp" |
| `{{PROJECT.path.to.value}}` | Nested access | `{{PROJECT.database.type}}` → "postgres" |
| `{{#if VAR}}...{{/if}}` | Conditional include | Include section if VAR is truthy |
| `{{#if VAR == 'value'}}...{{/if}}` | Equality check | Include if VAR equals 'value' |
| `{{else}}` | Else branch | Alternative content |
| `{{else if COND}}` | Else-if chain | Multiple conditions |

**Example template snippet:**

```markdown
## Database Queries

{{#if USES_PRISMA}}
Use Prisma client for all database operations:
\`\`\`typescript
import { prisma } from '@/lib/prisma';
const users = await prisma.user.findMany();
\`\`\`
{{else if USES_DRIZZLE}}
Use Drizzle ORM for type-safe queries:
\`\`\`typescript
import { db } from '@/lib/db';
const users = await db.select().from(users);
\`\`\`
{{else if USES_SUPABASE}}
Use Supabase client for database operations:
\`\`\`typescript
import { createClient } from '@/lib/supabase/server';
const supabase = await createClient();
const { data: users } = await supabase.from('users').select();
\`\`\`
{{/if}}
```

---

## Step 10: Generate Files

### Data Sources

The data for file generation comes from different sources depending on how the project was created:

| Source | For Existing Projects | For Spec-Driven New Projects | For Manual New Projects |
|--------|----------------------|------------------------------|------------------------|
| Stack info | Auto-detected from files | `StackDecision` from stack-advisor | User selections |
| Features | Auto-detected + user confirmation | `RequirementsManifest` from spec-analyzer | User selections |
| Entities | N/A (existing code) | `RequirementsManifest.entities` | N/A |
| User stories | N/A | `RequirementsManifest.userStories` | N/A |
| Infrastructure | Step 8b-8d questions | Defaults based on stack, can customize later | Step 8b-8d questions |

### 10a: Generate `docs/project.json`

Compile all detected and confirmed information into the manifest:

```json
{
  "$schema": "https://opencode.ai/schemas/project.json",
  
  "name": "<Project Name>",
  "description": "<user-provided or auto-generated>",
  
  "stack": {
    "languages": [/* detected or from StackDecision */],
    "runtime": "<detected or from StackDecision>",
    "packageManager": "<detected or from StackDecision>"
  },
  
  "apps": {/* discovered structure or generated from scaffold */},
  "packages": {/* discovered packages */},
  
  "database": {/* detected or from StackDecision */},
  "styling": {/* detected or from StackDecision */},
  "testing": {/* detected or from StackDecision */},
  "linting": {/* detected or defaults */},
  
  "git": {
    "branchingStrategy": "<from Step 8e Q19: trunk-based|github-flow|git-flow|release-branches>",
    "defaultBranch": "main",
    "developBranch": "<from Step 8e Q20 if applicable, default: develop>",
    "releaseBranchPattern": "<from Step 8e Q21 if applicable, default: release/*>"
  },
  
  "commands": {/* discovered or scaffold defaults */},
  "qualityGates": {/* inferred from commands */},
  
  "features": {
    "authentication": true,
    "multiTenant": false,
    "payments": true,
    "email": true,
    "i18n": false,
    "darkMode": true,
    "marketing": false,
    "api": true,
    "realtime": false,
    "documentation": {
      "system": "<from Step 8c Q12: markdown|docusaurus|database|none>",
      "userDocsPath": "<if provided>",
      "supportArticlesPath": "<if provided>",
      "databaseTable": "<if database-backed>"
    },
    "aiTools": {
      "system": "<from Step 8c Q13: openai-functions|langchain|mcp|none>",
      "schemaPath": "<if provided>",
      "implementationPath": "<if provided>"
    }
  },
  
  "integrations": [/* detected */],
  
  "aws": {
    "services": ["dynamodb", "s3", "sqs"],
    "sdkVersion": "v3",
    "infrastructure": "<from Step 8b Q9: cdk|cloudformation|terraform|none>",
    "infrastructurePath": "<if provided>",
    "clientWrapperPath": "<from Step 8b Q7 if provided>",
    "localDevelopment": "<from Step 8b Q8: true|false>"
  },
  
  "security": {
    "authMiddleware": "<from Step 8b Q3>",
    "csrfProtection": "<from Step 8b Q4: double-submit|synchronizer-token|samesite-cookie|none>",
    "corsConfigPath": "<from Step 8b Q5>",
    "inputValidation": "<from Step 8b Q6: zod|yup|joi|class-validator>"
  },
  
  "agents": {
    "autoCommit": true,
    "autoPush": true,
    "browserVerification": "<has frontend>",
    "prReviewRequired": false
  },
  
  "context": {
    "architecture": "docs/ARCHITECTURE.md",
    "conventions": "docs/CONVENTIONS.md",
    "designSystem": null
  }
}
```

**Note:** Only include sections that apply to the project. Omit `aws` if no AWS services detected, omit `security` fields that weren't answered, etc.

### 10b: Generate `docs/prd-registry.json` (if agent system)

```json
{
  "version": "1.0",
  "prds": [],
  "completed": []
}
```

### 10c: Generate `docs/session-locks.json` (if agent system)

```json
{
  "sessions": []
}
```

### 10d: Generate `docs/ARCHITECTURE.md`

Use the template from `~/.config/opencode/templates/ARCHITECTURE.md`.

Replace placeholders with detected values. For a Next.js + Supabase project:

```markdown
# Architecture

> This document describes the high-level architecture of Example Scheduler.
> It helps AI agents and new developers understand how the codebase is organized.

## Overview

Scheduling application for flooring businesses.

## Directory Structure

\`\`\`
example-scheduler/
├── apps/
│   └── web/                    # Next.js 15 frontend
│       ├── app/               # App Router pages
│       ├── components/        # React components
│       └── lib/               # Utilities
├── packages/
│   └── types/                 # Shared TypeScript types
├── supabase/
│   └── migrations/            # Database migrations
└── docs/                      # Documentation and PRDs
\`\`\`

## Database Schema

**Type:** PostgreSQL
**Client:** Supabase
**Migrations:** `supabase/migrations/`

<!-- Continue filling in detected values... -->
```

### 10e: Generate `docs/CONVENTIONS.md`

Use the template from `~/.config/opencode/templates/CONVENTIONS.md`.

Replace placeholders with detected values. For a TypeScript + Tailwind project:

```markdown
# Conventions

> This document describes the coding conventions and patterns used in Example Scheduler.
> AI agents should follow these patterns to maintain consistency.

## File Naming

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useAuth.ts` |
| Utilities | camelCase | `formatDate.ts` |

## Styling

### Framework: Tailwind CSS v4

### Dark Mode

**Strategy:** class-based (.dark on html)

\`\`\`tsx
<div className="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  Content adapts to theme
</div>
\`\`\`

<!-- Continue filling in detected values... -->
```

### 10f: Generate/Update `AGENTS.md` (if doesn't exist)

Create a minimal AGENTS.md with key information:

```markdown
# <Project Name>

## Development

\`\`\`bash
<detected dev command>    # Start dev server
<detected test command>   # Run tests
<detected build command>  # Production build
\`\`\`

## Documentation

For detailed information, see:
- [Architecture](docs/ARCHITECTURE.md) - System overview and data flow
- [Conventions](docs/CONVENTIONS.md) - Coding patterns and style guidelines

## Project Structure

\`\`\`
<discovered structure>
\`\`\`

## Tech Stack

- **Frontend**: <detected>
- **Backend**: <detected>
- **Database**: <detected>
- **Testing**: <detected>
```

---

## Step 10g: Generate Initial PRD (for spec-driven new projects)

If the project was created via spec-driven flow (Option B), generate an initial PRD from the extracted user stories:

### From RequirementsManifest

The `RequirementsManifest` contains:
- `entities` — Core domain objects
- `userStories` — Extracted user stories with priorities

### Generate `docs/drafts/prd-mvp.md`

Use the `prd` skill to format, or generate directly:

```markdown
# PRD: MVP - <Project Name>

> Initial PRD generated from project spec analysis.

## Overview

<Description from RequirementsManifest or user input>

## Entities

| Entity | Description |
|--------|-------------|
{{#each entities}}
| {{name}} | {{description}} |
{{/each}}

## User Stories

{{#each userStories}}
### {{id}}: {{title}}

**Priority:** {{priority}}

{{description}}

**Acceptance Criteria:**
- [ ] TBD — refine with @planner

{{/each}}

## Technical Notes

**Selected Stack:** {{stackDecision.archetype}}
- Frontend: {{stackDecision.stack.frontend.framework}}
- Backend: {{stackDecision.stack.backend.framework}}
- Database: {{stackDecision.stack.database.provider}}
- Auth: {{stackDecision.stack.auth.provider}}

## Open Questions

{{#if requirementsManifest.openQuestions}}
{{#each requirementsManifest.openQuestions}}
- [ ] {{question}}
{{/each}}
{{else}}
None identified during spec analysis.
{{/if}}
```

### Update `docs/prd-registry.json`

Add the generated PRD:

```json
{
  "version": "1.0",
  "prds": [
    {
      "id": "prd-mvp",
      "name": "MVP",
      "status": "draft",
      "filePath": "docs/drafts/prd-mvp.md",
      "createdAt": "<timestamp>",
      "stories": [/* extracted story IDs */]
    }
  ],
  "completed": []
}
```

### Save Analysis Files

Also save the analysis outputs for reference:

- `docs/requirements.json` — The RequirementsManifest
- `docs/stack-decision.json` — The StackDecision

These can be referenced later for understanding why certain choices were made.

---

## Step 10h: Generate Project-Specific Skills (US-009)

After generating the project manifest, auto-invoke meta-skill generators based on detected capabilities.

### Skill Trigger Mapping

Load the capability-to-skill mapping from `~/.config/opencode/data/meta-skill-triggers.json`:

```json
{
  "capabilityTriggers": {
    "authentication": { "metaSkill": "auth-skill-generator", "generates": "auth-flow" },
    "multiTenant": { "metaSkill": "multi-tenant-skill-generator", "generates": "tenant-context" },
    "api": { "metaSkill": "api-endpoint-skill-generator", "generates": "api-patterns" },
    "email": { "metaSkill": "email-skill-generator", "generates": "email-patterns" },
    "ai": { "metaSkill": "ai-tools-skill-generator", "generates": "ai-tools" }
  },
  "integrationTriggers": {
    "stripe": { "metaSkill": "stripe-skill-generator", "generates": "payments" }
  },
  "alwaysGenerate": [
    { "metaSkill": "crud-skill-generator", "generates": "crud-patterns", "condition": "hasDatabase" },
    { "metaSkill": "database-migration-skill-generator", "generates": "migrations", "condition": "hasDatabase" },
    { "metaSkill": "form-skill-generator", "generates": "form-patterns", "condition": "hasFrontend" },
    { "metaSkill": "table-skill-generator", "generates": "table-patterns", "condition": "hasFrontend" }
  ]
}
```

### Determine Skills to Generate

Based on the detected/confirmed capabilities and integrations:

```javascript
const skillsToGenerate = [];

// Check capabilities from project.json.capabilities (or features for legacy)
for (const [capability, config] of Object.entries(capabilityTriggers)) {
  if (projectJson.capabilities?.[capability] || projectJson.features?.[capability]) {
    skillsToGenerate.push(config);
  }
}

// Check integrations
for (const [integration, config] of Object.entries(integrationTriggers)) {
  if (projectJson.integrations?.includes(integration)) {
    skillsToGenerate.push(config);
  }
}

// Always-generate skills based on conditions
for (const config of alwaysGenerate) {
  if (config.condition === 'hasDatabase' && projectJson.database) {
    skillsToGenerate.push(config);
  }
  if (config.condition === 'hasFrontend' && projectJson.apps?.web) {
    skillsToGenerate.push(config);
  }
}
```

### Ask User Permission

```
═══════════════════════════════════════════════════════════════════════
                    PROJECT-SPECIFIC SKILLS
═══════════════════════════════════════════════════════════════════════

Based on your project's capabilities, I can generate these skills:

  ✅ auth-flow         Authentication patterns (capabilities.authentication)
  ✅ tenant-context    Multi-tenant isolation (capabilities.multiTenant)
  ✅ api-patterns      API endpoint conventions (capabilities.api)
  ✅ payments          Stripe integration patterns (integrations: stripe)
  ✅ crud-patterns     Entity CRUD operations (database detected)
  ✅ migrations        Database migration patterns (database detected)
  ✅ form-patterns     Form handling (frontend detected)
  ✅ table-patterns    Data tables (frontend detected)

These skills help AI agents understand YOUR project's specific patterns.

Generate project-specific skills?

  Y. Yes, generate all (recommended)
  S. Select which ones to generate
  N. Skip skill generation

> _
═══════════════════════════════════════════════════════════════════════
```

### Generate Skills

For each skill to generate:

1. **Create the skills directory:**
   ```bash
   mkdir -p docs/skills
   ```

2. **Invoke the meta-skill generator:**
   The meta-skill generator (e.g., `auth-skill-generator`) will:
   - Analyze the codebase for existing patterns
   - Ask any necessary clarifying questions
   - Generate `docs/skills/<skill-name>/SKILL.md`
   
   ```
   Generating auth-flow skill...
   
   [auth-skill-generator runs, may ask questions about auth patterns]
   
   ✅ Created: docs/skills/auth-flow/SKILL.md
   ```

3. **Track generated skills:**
   After each skill is generated, record it in `project.json`:

   ```json
   {
     "skills": {
       "projectSkillsPath": "docs/skills/",
       "generated": [
         {
           "name": "auth-flow",
           "generatedFrom": "auth-skill-generator",
           "generatedAt": "2026-02-20",
           "triggeredBy": "capabilities.authentication"
         },
         {
           "name": "tenant-context",
           "generatedFrom": "multi-tenant-skill-generator",
           "generatedAt": "2026-02-20",
           "triggeredBy": "capabilities.multiTenant"
         }
       ]
     }
   }
   ```

### Output

After skill generation completes:

```
═══════════════════════════════════════════════════════════════════════
                    SKILLS GENERATED
═══════════════════════════════════════════════════════════════════════

Generated 6 project-specific skills:

  docs/skills/
  ├── auth-flow/SKILL.md         Authentication patterns
  ├── tenant-context/SKILL.md    Multi-tenant isolation
  ├── api-patterns/SKILL.md      API endpoint conventions
  ├── payments/SKILL.md          Stripe integration
  ├── crud-patterns/SKILL.md     Entity CRUD operations
  └── migrations/SKILL.md        Database migrations

These skills will be automatically loaded when agents work on
related tasks. You can customize them in docs/skills/.

═══════════════════════════════════════════════════════════════════════
```

---

## Step 11: Update Global Registry

Add the project to `~/.config/opencode/projects.json`:

```json
{
  "id": "<kebab-case-name>",
  "name": "<Display Name>",
  "path": "<full-path>",
  "description": "<description>",
  "devPort": <port-number-or-null>,
  "hasAgentSystem": true,
  "projectManifest": "docs/project.json",
  "prdRegistry": "docs/prd-registry.json" or null,
  "sessionLocks": "docs/session-locks.json" or null
}
```

**devPort assignment:**
- Use the port from user's answer to question 2 (e.g., 3000, 5000, 5001, or custom)
- If user selected "No local runtime" (option E), set `devPort: null`
- When `devPort` is null, E2E tests and dev server startup are skipped for this project

Set as `activeProject`.

---

## Step 13: Summary

Display completion summary based on flow used:

### For Existing Projects

```
═══════════════════════════════════════════════════════════════════════
                      PROJECT SETUP COMPLETE
═══════════════════════════════════════════════════════════════════════

✅ Created: docs/project.json (with stack, features, integrations)
✅ Created: docs/prd-registry.json
✅ Created: docs/session-locks.json
✅ Created: docs/ARCHITECTURE.md
✅ Created: docs/CONVENTIONS.md (with infrastructure conventions)
✅ Created: docs/agents/ (project-specific agents)
✅ Created: AGENTS.md
✅ Updated: ~/.config/opencode/projects.json

Project "<Name>" is now ready!

📦 Generated agents (in docs/agents/):
   - typescript-critic.md    TypeScript code review
   - react-dev.md            React/Next.js patterns
   - react-tester.md         Jest + RTL testing
   - tailwind.md             Tailwind CSS styling

🎯 Generated skills (in docs/skills/):
   - auth-flow/              Authentication patterns
   - tenant-context/         Multi-tenant isolation
   - api-patterns/           API endpoint conventions
   - crud-patterns/          Entity CRUD operations
   (Skills are generated based on detected capabilities)

📝 Next steps (default):
  1. Start PRD kickoff now with @planner to define project scope
  2. Include architecture options/tradeoffs in the PRD as part of scope definition
  3. Move PRD to ready
  4. Start implementation with @builder

💡 Infrastructure conventions collected:
   - Network/HTTP: <summary of what was configured>
   - Security: <auth middleware, CSRF, validation>
   - AWS: <services, local dev, IaC tool>
   - API Design: <pagination, response envelope>
   - Testing: <locations, mocking conventions>

💡 The ARCHITECTURE.md and CONVENTIONS.md files have placeholders 
   marked with <!-- BOOTSTRAP NOTE: ... --> comments. Fill these in 
   to help AI agents better understand your codebase.

═══════════════════════════════════════════════════════════════════════
```

### For Spec-Driven New Projects

```
═══════════════════════════════════════════════════════════════════════
                      PROJECT CREATED FROM SPEC
═══════════════════════════════════════════════════════════════════════

✅ Analyzed: Your project spec/PRD
✅ Selected: <Archetype Name> stack
✅ Created:  <project-path>/

Files generated:
  docs/project.json          Project manifest
  docs/requirements.json     Extracted requirements
  docs/stack-decision.json   Stack selection rationale
  docs/drafts/prd-mvp.md     Initial PRD with <N> user stories
  docs/prd-registry.json     PRD registry
  docs/session-locks.json    Session coordination
  docs/ARCHITECTURE.md       Architecture overview
  docs/CONVENTIONS.md        Coding conventions
  docs/agents/               Project-specific agents
  docs/skills/               Project-specific skills
  AGENTS.md                  Quick reference

Stack selected:
  Frontend:   <frontend>
  Backend:    <backend>
  Database:   <database>
  Auth:       <auth>
  Hosting:    <hosting>

📦 Generated agents (in docs/agents/):
   - typescript-critic.md    TypeScript code review
   - react-dev.md            React/Next.js patterns
   - react-tester.md         Jest + RTL testing
   - tailwind.md             Tailwind CSS styling
   - playwright-tester.md    E2E testing

🎯 Generated skills (in docs/skills/):
   - <list based on detected capabilities>
   (e.g., auth-flow, api-patterns, crud-patterns)

Project "<Name>" is ready for development!

📝 Next steps (default):
  1. Continue PRD kickoff with @planner to lock scope + architecture direction
  2. Move PRD to ready: @planner move prd-mvp to ready
  3. Start implementation: @builder (will claim the ready PRD)

💡 The PRD contains <N> user stories extracted from your spec.
   Use @planner to refine acceptance criteria before building.

═══════════════════════════════════════════════════════════════════════
```

### For Manual New Projects

```
═══════════════════════════════════════════════════════════════════════
                      PROJECT CREATED
═══════════════════════════════════════════════════════════════════════

✅ Created: <project-path>/
✅ Stack:   <selected stack summary>

Files generated:
  docs/project.json          Project manifest
  docs/prd-registry.json     PRD registry
  docs/session-locks.json    Session coordination
  docs/ARCHITECTURE.md       Architecture overview
  docs/CONVENTIONS.md        Coding conventions
  docs/agents/               Project-specific agents
  docs/skills/               Project-specific skills
  AGENTS.md                  Quick reference

📦 Generated agents (in docs/agents/):
   - <list based on selected stack>

🎯 Generated skills (in docs/skills/):
   - <list based on detected capabilities>

Project "<Name>" is ready!

📝 Next steps (default):
  1. Start PRD kickoff with @planner to define project scope
  2. Add architecture recommendations in the PRD
  3. Move PRD to ready and start implementation with @builder

═══════════════════════════════════════════════════════════════════════
```

---

## Error Handling

### Path doesn't exist
```
❌ Path not found: ~/code/nonexistent

Please check the path and try again.
```

### Not a git repo (and user declines init)
```
⚠️  Project must be a git repository for agent coordination.

You can:
  A. Let me initialize git now
  B. Cancel and initialize manually

> _
```

### project.json already exists
```
⚠️  This project already has docs/project.json

What would you like to do?
  A. Overwrite with fresh detection
  B. Keep existing and just add to registry
  C. Cancel

> _
```

---

## Quick Mode

For experienced users, support a quick mode:

```bash
# In the future, could support:
# @bootstrap /path/to/project --quick
```

This would:
1. Auto-detect everything possible
2. Use sensible defaults for unknowns
3. Skip all confirmation prompts
4. Generate files immediately

---

## Output

Return a summary of what was created and the path to the project, so the calling agent can proceed with displaying the status dashboard.
