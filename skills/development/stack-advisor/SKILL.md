---
name: stack-advisor
description: Interactive Q&A skill that guides users through tech stack selection for new projects. Use when user runs /speckit.plan without providing tech stack details, or when user is unsure what technologies to use. Asks 3-5 targeted questions about project type, scale, and requirements, then suggests an appropriate tech stack.
---

# Stack Advisor Skill

Guides users through tech stack selection via interactive Q&A for **new projects** where the tech stack hasn't been decided yet.

> **Note**: For **existing projects** with established code, use `tech-stack-detection` skill instead to auto-detect installed technologies.

## When to Activate

This skill activates when:
- User runs `/speckit.plan` without providing tech stack details
- User explicitly asks for help choosing a tech stack
- User says they "don't know" what technologies to use

## Q&A Flow (3-5 Questions)

Ask these questions in order. Each answer narrows down the recommendation.

### Question 1: Project Type (Required)

> **What type of project are you building?**
> 
> 1. **Web Application** - Interactive frontend with user interface
> 2. **API / Backend** - Server-side services, REST/GraphQL APIs
> 3. **CLI Tool** - Command-line application
> 4. **Library / Package** - Reusable code for other projects
> 5. **Full Stack** - Both frontend and backend together
> 6. **Mobile App** - iOS, Android, or cross-platform

### Question 2: Project Scale (Required)

> **What's the expected scale of this project?**
>
> 1. **Prototype / Small** - Quick proof of concept, personal project, learning
> 2. **Medium** - Production app, small team, moderate complexity
> 3. **Large / Enterprise** - Multiple teams, complex requirements, long-term maintenance

### Question 3: Key Requirements (Required)

> **What are the key technical requirements?** (Select all that apply)
>
> 1. **Real-time updates** - WebSockets, live data, collaboration
> 2. **SEO / SSR** - Search engine optimization, server-side rendering
> 3. **Offline-first** - PWA, works without internet
> 4. **High performance** - Sub-second loads, large data sets
> 5. **Simple / Minimal** - Least dependencies, vanilla approach
> 6. **Mobile-responsive** - Must work well on all devices
> 7. **None specific** - Standard requirements

### Question 4: Team Preference (Optional)

> **What's your team's technology preference?**
>
> 1. **Beginner-friendly** - Well-documented, large community, stable
> 2. **Modern standards** - Current best practices, good balance
> 3. **Cutting-edge** - Latest technologies, willing to accept some risk

### Question 5: Constraints (Optional)

> **Any specific constraints or requirements?**
>
> Examples:
> - "Must use Python for ML integration"
> - "Company standard is .NET"
> - "Avoid frameworks, keep it vanilla"
> - "No constraints"

## Tech Stack Decision Matrix

### Web Applications

| Scale + Requirements | Recommended Stack |
|---------------------|-------------------|
| Prototype + Simple | Vanilla HTML/CSS/JS, or Vite + vanilla |
| Prototype + Modern | Vite + React + TailwindCSS |
| Medium + SEO | Next.js + React + TailwindCSS |
| Medium + Real-time | Vite + React + Socket.io + TailwindCSS |
| Medium + Simple | Astro (static) or SvelteKit |
| Large + Enterprise | Next.js + TypeScript + TailwindCSS + Prisma |
| Large + Performance | Remix or SolidStart |

### API / Backend

| Scale + Requirements | Recommended Stack |
|---------------------|-------------------|
| Prototype | Express.js or FastAPI (Python) |
| Medium + Node.js | Express + TypeScript + Prisma |
| Medium + Python | FastAPI + SQLAlchemy + Pydantic |
| Large + Enterprise | NestJS (Node) or Django (Python) |
| High Performance | Go + Gin, or Rust + Axum |

### Full Stack

| Scale + Requirements | Recommended Stack |
|---------------------|-------------------|
| Prototype | Next.js (full stack) or SvelteKit |
| Medium | Next.js + Prisma + PostgreSQL |
| Medium + Real-time | T3 Stack (Next.js + tRPC + Prisma) |
| Large | Separate frontend/backend with shared types |

### CLI Tools

| Language Preference | Recommended |
|--------------------|-------------|
| JavaScript/TypeScript | Commander.js + TypeScript |
| Python | Click or Typer |
| Go | Cobra |
| Rust | Clap |

### Libraries

| Ecosystem | Recommended Setup |
|-----------|------------------|
| npm (JS/TS) | TypeScript + tsup + Vitest |
| PyPI (Python) | Poetry + pytest |
| Crates (Rust) | Cargo (built-in) |

## Standard Additions (Always Recommend)

| Category | Recommendation |
|----------|---------------|
| Testing | Playwright (E2E), Vitest/Jest (unit) |
| Linting | ESLint + Prettier (JS/TS), Ruff (Python) |
| Type Safety | TypeScript (JS), Pydantic (Python) |
| Styling | TailwindCSS (unless vanilla requested) |
| Version Control | Git + Conventional Commits |

## Output Format

After Q&A, present the recommendation:

```markdown
## Recommended Tech Stack

Based on your answers:
- **Project Type**: Web Application
- **Scale**: Medium
- **Requirements**: SEO, Mobile-responsive
- **Preference**: Modern standards

### Suggested Stack

| Category | Technology | Why |
|----------|------------|-----|
| Framework | Next.js 14 | SSR for SEO, React ecosystem |
| Language | TypeScript | Type safety, better DX |
| Styling | TailwindCSS | Rapid development, responsive utilities |
| Testing | Playwright | E2E testing, cross-browser |
| Database | PostgreSQL + Prisma | Type-safe ORM, scalable |

### Project Structure
```
src/
├── app/           # Next.js App Router
├── components/    # React components
├── lib/           # Utility functions
└── styles/        # Global styles
```

**Proceed with this stack?** (Y/n/modify)
```

## Confirmation Flow

1. If user confirms → Proceed to plan generation with this stack
2. If user wants modifications → Ask what to change, update recommendation
3. If user says no → Ask what they'd prefer instead

## Integration with /speckit.plan

When stack is confirmed, pass it to plan generation:

```
User confirmed stack:
- Framework: Next.js 14
- Language: TypeScript
- Styling: TailwindCSS
- Testing: Playwright

Proceeding with plan generation...
```

The plan template will use these values in the Tech Stack section.
