---
name: stack-selection
description: Tech stack selection with decision trees, user preferences, and blueprint defaults. Use when choosing frameworks, libraries, or tools for new projects or features.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Tech Stack Selection

> **Philosophy:** Choose tools based on context, requirements, and user preference. No "one size fits all" solutions.
> **Core Principle:** ASK user preferences, SUGGEST based on context, EXPLAIN trade-offs.

---

## When to Use This Skill

Use this skill when:
- Starting a new project (no existing code)
- Adding a new feature that could use different stacks
- User asks "what stack should I use?"
- Migration between technologies is being considered

**Do NOT use this skill when:**
- Project already has established stack (maintain consistency)
- User explicitly specifies what to use
- Only minor adjustments are needed

---

## Question Tool Protocol (MANDATORY)

**ALWAYS use the question tool for stack selection decisions.**

### Multi-Option Format (Preferred)

```javascript
question({
  questions: [{
      question: "Which backend framework would you like to use?",
      header: "Backend Framework",
      options: [
        {
          label: "Fullstack Blueprint (Recommended)",
          description: "FastAPI + SQLAlchemy + Alembic - battle-tested, includes Docker, Redis, Celery"
        },
        {
          label: "FastAPI (Custom)",
          description: "Fast, modern, async - great for APIs"
        },
        {
          label: "Express/Node.js",
          description: "Flexible, huge ecosystem, good for fullstack JavaScript"
        },
        {
          label: "Hono",
          description: "Ultra-fast, edge-ready, TypeScript-first"
        },
        {
          label: "Other",
          description: "Let me know your preference"
        }
      ]
    }]
})
```

### Preference Questions Format

When checking user preferences:

```javascript
question({
  questions: [{
      question: "Do you have any tech stack preferences?",
      header: "Stack Preferences",
      multiple: true,
      options: [
        {
          label: "Use the Fullstack Blueprint",
          description: "I want the standard stack: React/Vite + FastAPI + SQLAlchemy"
        },
        {
          label: "I have specific preferences",
          description: "I'll tell you exactly what I want"
        },
        {
          label: "Suggest based on my requirements",
          description: "Help me choose based on what I'm building"
        }
      ]
    }]
})
```

See `.opencode/rules/MASTER.md` for complete Question Tool Protocol.

---

## Stack Selection Process

### Phase 1: Context Analysis (ALWAYS FIRST)

Before suggesting anything, gather context:

| Question | Why It Matters |
|----------|----------------|
| **What are you building?** | Web app, API, mobile, landing page? |
| **What's your scale?** | Prototype, MVP, production, enterprise? |
| **What's your timeline?** | Rushed, normal, no constraints? |
| **What's your team size/expertise?** | Solo, small team, large team? |
| **Any constraints?** | Edge deployment, legacy systems, compliance? |

> **Reference:** See `@blueprints/fullstack/` for an example of a well-configured fullstack setup. Use it as reference, not requirement.

### Phase 2: Check Existing Stack

| Question | Why It Matters |
|----------|----------------|
| **What are you building?** | Web app, API, mobile, landing page? |
| **What's the scale?** | Prototype, MVP, production, enterprise? |
| **What's your timeline?** | Rushed, normal, no constraints? |
| **What's your team size/expertise?** | Solo, small team, large team? |
| **Any constraints?** | Edge deployment, legacy systems, compliance? |

### Phase 2: Check Existing Stack

**Before suggesting, check if stack exists:**

```bash
# Check for existing backend
ls backend/pyproject.toml backend/package.json backend/go.mod 2>/dev/null

# Check for existing frontend
ls frontend/package.json package.json 2>/dev/null

# Check for config files
ls docker-compose.* vite.config.* next.config.* 2>/dev/null
```

**If stack exists:**
- Maintain consistency unless migration is explicitly requested
- Ask user: "You already use [X]. Should I continue with that stack, or are you considering changes?"

### Phase 3: User Preferences

**ALWAYS ask about preferences:**

1. **Primary question:**
   ```javascript
   question({
     questions: [{
         question: "Do you have any tech stack preferences?",
         header: "Stack Preferences",
         multiple: true,
         options: [
           {
             label: "I have specific preferences",
             description: "I'll tell you exactly what I want"
           },
           {
             label: "Suggest based on my requirements",
             description: "Help me choose based on what I'm building"
           }
         ]
       }]
   })
   ```

2. **If user wants suggestions:** Proceed to Phase 4 with decision trees

### Phase 4: Suggest Based on Requirements

If user wants suggestions, provide options with trade-offs:

**Example Backend Suggestion:**

| Option | Best For | Trade-offs |
|--------|----------|------------|
| FastAPI | APIs, async, Python expertise | Smaller ecosystem than Node.js |
| Express | Fullstack JS, flexibility | Slower, callback hell risk |
| Hono | Edge, performance, TypeScript | Newer, smaller community |
| Go | Performance, concurrency | Slower development, verbose |

---

## Fullstack Blueprint (Example Stack)

### Blueprint Stack Details

**Note:** This is an example of a well-configured fullstack setup used in `@blueprints/fullstack/`. It's not mandatory - use it as reference, not requirement.

**Frontend:**
```
- React 18: Modern, component-based
- Vite: Fast dev server, optimized builds
- TanStack Query: Server state management (caching, refetching)
- TanStack Router: Type-safe routing
- Tailwind CSS: Utility-first styling
- ShadcnUI: Accessible, customizable components
```

**Backend:**
```
- FastAPI: Async, modern Python, automatic OpenAPI docs
- SQLAlchemy: ORM, migrations via Alembic
- PostgreSQL: Robust relational database
- Redis: Caching, Celery broker
- Celery: Async task queue
- Docker: Containerization
```

**Infrastructure:**
```
- Docker Compose: Dev and production setups
- Poetry: Python dependency management
```

---

## Backend Decision Tree

### Framework Selection

```
What runtime do you prefer?
├── Python
│   ├── Need async/high performance?
│   │   ├── Yes → FastAPI (recommended)
│   │   └── No → Django (batteries included) or Flask (lightweight)
│   └── Need full features?
│       ├── Yes → Django
│       └── No → FastAPI or Flask
├── Node.js
│   ├── Edge deployment?
│   │   ├── Yes → Hono or Bun
│   │   └── No → Fastify (performance) or Express (flexible)
│   └── TypeScript fullstack?
│       └── Consider tRPC with Next.js
└── Go/Rust
    └── Performance critical → FastAPI is usually sufficient
```

### Database Selection

```
What type of data?
├── Relational data, complex relationships
│   ├── Production → PostgreSQL (recommended)
│   ├── Edge/Serverless → Neon (serverless PG)
│   └── Simple/Local → SQLite
├── Vector search/AI
│   └── PostgreSQL + pgvector or dedicated vector DB
├── Global distribution
│   └── Turso (edge SQLite) or PlanetScale (MySQL)
└── Key-value cache
    └── Redis or Upstash (serverless)
```

### ORM Selection

```
Python:
├── FastAPI → SQLAlchemy (recommended) or Tortoise ORM
├── Django → Django ORM (built-in)
└── Simple → Pydantic models + direct SQL

Node.js:
├── PostgreSQL → Prisma or Drizzle
└── Performance → Drizzle (edge-ready)
```

---

## Frontend Decision Tree

### Framework Selection

```
What type of application?
├── Single Page App (SPA)
│   ├── React ecosystem?
│   │   ├── Yes → React + Vite (recommended in blueprint)
│   │   └── No → Svelte + SvelteKit or Vue + Vite
│   └── Build tools → Vite (fastest) or Next.js (SSR)
├── Server-Side Rendering (SSR)
│   ├── Need SEO/crawling?
│   │   ├── Yes → Next.js (recommended)
│   │   └── No → Consider SPA
│   └── Fullstack TypeScript?
│       └── Next.js + tRPC (type-safe)
└── Static Site
    └── Next.js (static export) or Astro
```

### State Management

```
What state do you need?
├── Server data (API calls)
│   └── TanStack Query (recommended) or SWR
├── Client state (UI state)
│   ├── Simple → React built-in (useState, useReducer)
│   ├── Complex but local → Zustand
│   └── Global/shared → Context API or Redux Toolkit
└── URL state (shareable)
    └── URL params + useSearchParams
```

### UI Library Selection

```
What do you need?
├── Blueprint default → ShadcnUI (recommended)
├── Full-featured component library
│   ├── Radix UI (unstyled primitives)
│   ├── Mantine (opinionated)
│   ├── Chakra UI (simple)
│   └── Material UI (Google design)
├── Design system building
│   └── Headless UI or Radix UI
└── Complete control
│   └── Custom CSS or Tailwind only
```

### Styling

```
What approach?
├── Blueprint default → Tailwind CSS (recommended)
├── CSS-in-JS
│   ├── Styled Components
│   └── Emotion
├── CSS Modules
│   └── CSS file per component
└── Traditional CSS
│   └── Plain CSS with variables
```

---

## Stack Decision Template

When documenting stack decisions, use this format:

### Backend Stack

| Component | Choice | Why? |
|-----------|--------|------|
| Runtime | [e.g., Python 3.12] | Team expertise, async support |
| Framework | [e.g., FastAPI] | Performance, automatic docs |
| Database | [e.g., PostgreSQL] | Relational data, ACID compliance |
| ORM | [e.g., SQLAlchemy] | Migration support, type hints |
| Task Queue | [e.g., Celery] | Async jobs, Python ecosystem |

### Frontend Stack

| Component | Choice | Why? |
|-----------|--------|------|
| Framework | [e.g., React 18] | Team expertise, ecosystem |
| Build Tool | [e.g., Vite] | Fast dev experience |
| Routing | [e.g., TanStack Router] | Type-safe, file-based |
| State Management | [e.g., TanStack Query] | Server state, caching |
| UI Library | [e.g., ShadcnUI] | Accessible, customizable |
| Styling | [e.g., Tailwind CSS] | Developer experience |

### Infrastructure

| Component | Choice | Why? |
|-----------|--------|------|
| Containerization | [e.g., Docker] | Reproducibility, deployment |
| Deployment | [e.g., Docker Compose] | Simple setup, production-ready |

---

## Selection Checklist

Before finalizing stack selection:

- [ ] **Asked user about preferences?**
- [ ] **Checked for existing stack?**
- [ ] **Offered blueprint as option?**
- [ ] **Explained trade-offs of options?**
- [ ] **Documented rationale?**
- [ ] **Team expertise considered?**
- [ ] **Constraints considered?**

---

## Anti-Patterns

**DON'T:**
- Force a stack without asking user
- Assume "modern" = "best" for every project
- Ignore existing code/stack
- Choose based on trends instead of requirements
- Skip trade-off explanations

**DO:**
- Always ask user preferences first
- Explain why you're recommending each option
- Consider team expertise and constraints
- Document rationale for decisions
- Offer blueprint as default, not mandate

---

## Related Skills

| Need | Skill |
|------|-------|
| Backend patterns | `@[skills/python-patterns]` or `@[skills/api-patterns]` |
| Frontend patterns | `@[skills/frontend-design]` or `@[skills/nextjs-react-expert]` |
| Architecture | `@[skills/architecture]` |
| Database design | `@[skills/database-design]` |

---

## Usage Examples

### Example 1: New Project, No Preferences

**User:** "I want to build a web application for task management"

**Process:**
1. Ask: "Do you want to use the Fullstack Blueprint (React + FastAPI + SQLAlchemy)?"
2. If yes → Use blueprint stack
3. If no → Ask about preferences or suggest based on requirements

### Example 2: Existing Project

**User:** "Add a new feature to the existing app"

**Process:**
1. Check existing stack (e.g., FastAPI + React)
2. Maintain consistency
3. Ask: "You're using FastAPI + React. Continue with that stack?"

### Example 3: Migration

**User:** "I want to migrate to a different stack"

**Process:**
1. Understand motivation (performance, team expertise, etc.)
2. Suggest alternatives with trade-offs
3. Document migration plan

---

> **Remember:** There's no "perfect" stack. The best stack is the one that fits the project requirements, team expertise, and constraints. Always involve the user in decisions.
