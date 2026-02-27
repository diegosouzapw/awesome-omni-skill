---
name: vibe-coding
description: Rapidly prototype and build modern, responsive web applications from scratch using current frameworks and libraries. Use when you want to quickly create a new web app with full local control, creative flow, and modern best practices. Local alternative to Lovable, Bolt, and v0.
user-invocable: true
disable-model-invocation: false
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep]
---

# Vibe Coding

Rapidly prototype modern web applications from scratch with creative flow and quick iteration.

<instructions>

## Inputs

Gather before starting (ask if not provided):
- **App description**: Purpose, key features, target users
- **Tech preferences**: Framework (React/Next.js/Vue/Svelte), styling (Tailwind/CSS modules)
- **Feature requirements**: Core functionality, user flows
- **Design direction**: Style, colors, reference sites
- **Deploy target**: Vercel, Netlify, self-hosted, etc.

## Workflow

Think through the build step-by-step:

1. **Research**: Search official docs for current initialization commands and patterns
   - Never hardcode outdated commands; always verify current best practices
2. **Initialize**: Run framework init, set up TypeScript + ESLint + Prettier + git
3. **Design system**: Set up styling, define tokens (colors, spacing, typography), build base components
4. **Features**: Build incrementally - one feature at a time, test as you go
5. **Polish**: Accessibility (ARIA, keyboard nav), performance (code splitting, image optimization), error boundaries
6. **Document**: README with setup, dev commands, env vars, deployment

## Architecture Principles

- **Component-based**: Small, reusable components
- **Type-safe**: TypeScript throughout
- **Mobile-first**: Responsive by default
- **Accessible**: Semantic HTML, ARIA labels, keyboard navigation
- **Performance-conscious**: Code splitting, lazy loading, optimized images

</instructions>

<scope>

## In Scope
- Full-stack web apps (frontend + API routes/server components)
- Any modern framework (React, Next.js, Vue, Svelte, Astro, etc.)
- Database integration (Prisma, Drizzle with type-safe ORMs)
- Authentication flows
- SEO optimization (meta tags, structured data, sitemaps)

## Out of Scope
- Production infrastructure management
- Complex backend microservices (use separate skill)
- Native mobile apps (iOS/Android)

</scope>

<conventions>

## Styling
- Tailwind CSS recommended for rapid development
- Consistent design tokens for colors, spacing, typography
- Light/dark mode support when appropriate
- Proper contrast ratios (WCAG AA minimum)

## State and Data
- State: React Context / Zustand / Redux based on complexity
- Data fetching: React Query, SWR, or framework built-ins
- Always handle: loading states, error states, empty states

## Backend
- API routes or server components as appropriate
- Type-safe ORMs (Prisma, Drizzle) for databases
- Proper validation at API boundaries

## SEO (public-facing apps)
- Meta tags: title, description, Open Graph, Twitter cards
- Structured data: JSON-LD for rich search results
- Core Web Vitals: optimize LCP, FID, CLS
- Sitemap and robots.txt

</conventions>

<required_artifacts>
- Fully initialized project with all config files
- Clean component and page structure
- Styling implementation (Tailwind config or chosen approach)
- Tests for critical user flows
- README.md with: description, setup, dev commands, env vars, deployment
- Properly configured .gitignore
- package.json with clear scripts
</required_artifacts>

<verification>

```bash
npm install          # or bun install
npm run type-check   # tsc --noEmit
npm run lint         # eslint
npm test             # vitest / jest
npm run build        # production build
npm run dev          # development server
```

Complete when:
- All commands succeed without errors
- App builds and runs in dev mode
- Core features work across screen sizes
- Accessibility checks pass
- Tests cover critical paths
- README is clear and complete

</verification>

<examples>

### SaaS landing page
```
User: "Build a landing page - hero, features, pricing, contact form"

Steps:
  1. Search nextjs.org/docs for current create-next-app command
  2. npx create-next-app@latest --typescript --tailwind --app
  3. Create components: Hero, Features, PricingTable, ContactForm
  4. Implement responsive grid layout, form validation
  5. Add SEO: metadata API, Open Graph images
  6. Deploy config for Vercel
```

### Admin dashboard
```
User: "Prototype an admin dashboard - auth, data tables, charts"

Steps:
  1. Next.js App Router + NextAuth.js for authentication
  2. shadcn/ui for component library
  3. Protected route middleware
  4. Data tables with tanstack-table (sorting, filtering, pagination)
  5. Charts with recharts or chart.js
  6. Tests for auth flow and data display
```

</examples>

<safety>
- Verify package security before adding dependencies (check npm audit)
- Never commit API keys or secrets (use .env.local, document in README)
- If requirements exceed framework capabilities, suggest alternatives
</safety>

## Documentation References

Always search current docs before implementing:
- React: https://react.dev
- Next.js: https://nextjs.org/docs
- Vue: https://vuejs.org/guide
- Svelte: https://svelte.dev/docs
- Tailwind CSS: https://tailwindcss.com/docs
- TypeScript: https://www.typescriptlang.org/docs
