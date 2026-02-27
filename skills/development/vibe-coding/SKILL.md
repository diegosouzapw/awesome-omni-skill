---
name: vibe-coding
description: Rapidly prototype and build modern, responsive web applications from scratch using current frameworks and libraries. Use when you want to quickly create a new web app with full local control, creative flow, and modern best practices. Local alternative to Lovable, Bolt, and v0.
---
# Skill: Vibe coding

## Purpose

Rapidly prototype and build modern, responsive web applications from scratch using current frameworks, libraries, and best practices. This skill handles everything from initial project setup to implementing features, styling, and basic deployment configuration with a focus on creative flow and quick iteration.

## When to use this skill

- You want to **rapidly prototype** a new web application idea.
- You're in a **creative flow** and want to build something quickly without context switching.
- You want to use **modern frameworks** like React, Next.js, Vue, Svelte, or similar.
- You prefer **local development** with full control rather than hosted builder environments.
- You want to **experiment and iterate fast** on designs and features.

## Key differences from Lovable/Bolt/v0

Unlike hosted vibe-coding tools (Lovable, Bolt, v0):

- **Runs locally**: All code lives on your machine, not in a hosted environment.
- **No infrastructure lock-in**: You control deployment, hosting, and infrastructure choices.
- **Framework flexibility**: Not limited to a specific tech stack; can use any modern framework.
- **Full backend support**: Can integrate with any backend, run local servers, use databases, etc.
- **Version control native**: Built with git workflows in mind from the start.
- **No runtime environment required**: Build from scratch locally, not dependent on their infrastructure.

## Inputs

- **Application description**: Purpose, key features, and target users.
- **Technology preferences**: Desired frameworks (React/Next.js/Vue/Svelte), styling approach (Tailwind/CSS-in-JS/CSS Modules), and any specific libraries.
- **Feature requirements**: Core functionality, user flows, and data models.
- **Design direction**: Style preferences, color schemes, or reference sites.
- **Deployment target**: Where you plan to host (Vercel, Netlify, AWS, self-hosted, etc.).

## Out of scope

- Managing production infrastructure or cloud provider accounts.
- Creating complex backend microservices architecture (use service-integration skill instead).
- Mobile native app development (iOS/Android).

## Conventions and best practices

### Framework selection
- **Always search for the most current documentation** from official sources before implementing.
- Example: Search https://nextjs.org/docs for Next.js, https://react.dev for React, etc.
- **Never hardcode outdated commands or patterns** – always verify current best practices.

### Project initialization
1. Search official docs for the latest initialization command (e.g., `npx create-next-app@latest`).
2. Use TypeScript by default for type safety.
3. Set up ESLint and Prettier for code quality.
4. Initialize git repository from the start.

### Architecture patterns
- **Component-based**: Break UI into small, reusable components.
- **Type-safe**: Use TypeScript throughout for better developer experience.
- **Responsive by default**: Mobile-first design approach.
- **Accessibility first**: Semantic HTML, ARIA labels, keyboard navigation.
- **Performance-conscious**: Code splitting, lazy loading, optimized images.

### Styling approach
- Use modern CSS frameworks (Tailwind CSS recommended for rapid development).
- Implement consistent design system with reusable tokens for colors, spacing, typography.
- Support light/dark mode when appropriate.
- Ensure proper contrast ratios and accessibility.

### State and data
- Choose appropriate state management (React Context, Zustand, Redux) based on complexity.
- Use modern data fetching patterns (React Query, SWR, or framework built-ins like Next.js App Router).
- Implement proper loading, error, and empty states.

### Backend integration
- If backend is needed, set up API routes or server components appropriately.
- For databases, use type-safe ORMs (Prisma, Drizzle) when possible.
- Implement proper error handling and validation.

## Required behavior

1. **Research current best practices**: Before any implementation, search for and reference the latest official documentation.
2. **Initialize properly**: Set up project with all necessary tooling, configs, and directory structure.
3. **Implement features incrementally**: Build and test features one at a time.
4. **Write clean, maintainable code**: Follow framework conventions and best practices.
5. **Handle edge cases**: Loading states, errors, empty states, validation.
6. **Ensure accessibility**: Proper semantic HTML, ARIA labels, keyboard navigation.
7. **Test critical paths**: Write tests for core functionality.
8. **Document setup and usage**: README with setup instructions, environment variables, and deployment notes.

## Required artifacts

- Fully initialized project with all configuration files.
- Clean, well-organized component and page structure.
- Styling implementation (Tailwind config, CSS modules, or chosen approach).
- **Tests** for critical user flows and business logic.
- **README.md** with:
  - Project description and features
  - Setup instructions
  - Development commands
  - Environment variables needed
  - Deployment guidance
- **.gitignore** properly configured.
- **Package.json** with clear scripts and dependencies.

## Implementation checklist

### 1. Discovery and planning
- [ ] Search for current best practices for chosen framework
- [ ] Review official documentation for initialization commands
- [ ] Understand feature requirements and user flows
- [ ] Plan component hierarchy and data flow

### 2. Project initialization
- [ ] Run current framework initialization command
- [ ] Set up TypeScript, ESLint, Prettier
- [ ] Initialize git repository
- [ ] Create basic directory structure

### 3. Design system setup
- [ ] Set up styling solution (Tailwind, CSS-in-JS, etc.)
- [ ] Define color palette and design tokens
- [ ] Create base components (Button, Input, Card, etc.)
- [ ] Implement responsive layout system

### 4. Feature implementation
- [ ] Build pages and routes
- [ ] Implement components with proper typing
- [ ] Add state management where needed
- [ ] Integrate data fetching and APIs
- [ ] Handle loading, error, and empty states

### 5. Quality and polish
- [ ] Add accessibility features (ARIA, keyboard nav)
- [ ] Optimize performance (code splitting, image optimization)
- [ ] Write tests for critical flows
- [ ] Add error boundaries and fallbacks
- [ ] Review responsive behavior on different screen sizes

### 6. Documentation and deployment
- [ ] Write comprehensive README
- [ ] Document environment variables
- [ ] Add deployment configuration for target platform
- [ ] Create development and build scripts

## Verification

Run the following commands (adjust based on package manager and setup):

```bash
# Install dependencies
npm install

# Type checking
npm run type-check   # or tsc --noEmit

# Linting
npm run lint

# Tests
npm test

# Build verification
npm run build

# Run locally
npm run dev
```

The skill is complete when:

- All commands run successfully without errors.
- The application builds and runs in development mode.
- Core features work as specified across different screen sizes.
- Accessibility checks pass (use browser dev tools or axe extension).
- Tests cover critical user paths.
- Documentation is clear and complete.

## Safety and escalation

- **External dependencies**: Always verify package security and maintenance status before adding dependencies.
- **Environment secrets**: Never commit API keys, secrets, or credentials. Use `.env.local` and document in README.
- **Framework limitations**: If requirements exceed framework capabilities, suggest alternatives or clarify constraints.
- **Performance concerns**: If the app requires complex state or data handling, consider suggesting more robust solutions.

## Dynamic documentation references

Always search for and reference the most current documentation:

- **React**: https://react.dev
- **Next.js**: https://nextjs.org/docs
- **Vue**: https://vuejs.org/guide
- **Svelte**: https://svelte.dev/docs
- **Tailwind CSS**: https://tailwindcss.com/docs
- **TypeScript**: https://www.typescriptlang.org/docs
- **Vite**: https://vitejs.dev/guide

Before implementing any feature, search these docs to ensure you're using current APIs and best practices.

## Example workflows

### Vibing a landing page
```
User: "Let's vibe a landing page for a SaaS product - hero section, features, pricing, contact form"

Droid will:
1. Search for current Next.js initialization best practices
2. Initialize Next.js project with TypeScript and Tailwind
3. Create component structure (Hero, Features, Pricing, Contact)
4. Implement responsive design with proper accessibility
5. Add form validation and error handling
6. Set up basic SEO with metadata
7. Provide deployment instructions for Vercel/Netlify
```

### Quick dashboard prototype
```
User: "I want to quickly prototype an admin dashboard - auth, data tables, charts"

Droid will:
1. Research current patterns for Next.js App Router with authentication
2. Initialize project with appropriate dependencies
3. Set up authentication flow (NextAuth.js or similar)
4. Create protected route structure
5. Implement data fetching with loading states
6. Add table and chart components with proper typing
7. Include tests for auth and data flows
8. Document setup including database requirements
```

## SEO and web vitals

For public-facing applications, automatically implement:

- **Meta tags**: Title, description, Open Graph, Twitter cards
- **Structured data**: JSON-LD for rich search results
- **Performance**: Image optimization, code splitting, lazy loading
- **Core Web Vitals**: Optimize LCP, FID, CLS metrics
- **Sitemap**: Generate sitemap.xml for better indexing
- **Robots.txt**: Configure crawler behavior

## Integration with other skills

This skill can be combined with:

- **Service integration**: When the web app needs to call existing backend services.
- **Internal tools**: When building internal admin panels or dashboards.
- **Data querying**: When the app needs to display analytics or reports.
