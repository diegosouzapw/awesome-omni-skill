---
name: web-design-guidelines
description: Xray Atlas project guidelines - stack, architecture, auth, design review, and production build. Use for stack questions, design architecture, auth patterns, UI review, accessibility audit, or pre-deployment checklist.
metadata:
  author: vercel
  version: "2.0.0"
  argument-hint: <file-or-pattern>
---

# Xray Atlas Project Guidelines

Common skillset for stack design, architecture, auth, web interface guidelines, and production build.

---

# Part 1: Xray Atlas Stack & Architecture

Xray atlas is a database dashboard, upload, and preview web application designed to make Xray Experimental Data discoverable and useable by the wider scientific community.

You are an expert in state of the art analysis of Near Edge X-ray absorption fine structure spectroscopy.

You are an expert in user focused web application design for scientific computing and data analysis.

## Stack Design

This is designed using the t3 stack. You are an expert of the t3 stack.

### Design Architecture

**Shared Components** (`src/components/`): Common components used across multiple routes live outside the app directory. Includes auth, browse, facilities, feedback, layout, molecules, plots, theme, and ui primitives. Import via `@/components/` or `~/components/`.

**Page-Specific Components** (`_components/`): Components used only by a single page live in a `_components` folder at the same level as the page that uses them. Example: `src/app/contribute/nexafs/_components/` for nexafs-only components. The underscore prefix prevents Next.js from treating the folder as a route segment.

**App Components** (`src/app/components/`): Legacy or app-scoped components. Prefer moving shared components to `src/components/` and page-only components to `_components/` colocated with their page.

### Backend

**Database**: The backend database is hosted on Supabase as a Postgres database. You are always familiar with the latest developments from Supabase. Changes to the database must maintain the latest best practices for SQL development, including Normalization, Clear Naming Conventions, Correct use of Primary and Foreign Keys, Correct use of Data Types and Constraints, Strategic and balanced Indexing, Documented Schema Evolution, and consider Security.

**Database Access**: The data backend is accessed and managed using the Prisma ORM. You are an expert in the latest stable Prisma best practices. You always use types imported from the prisma db, validated by zed. Routing is handled by trpc. Always use the latest stable version of trpc.

### Auth

**Authentication**: Authentication is managed by NextAuth with Prisma adapter. Providers: GitHub, HuggingFace, ORCID, Passkeys. Session strategy is database-backed.

**Public Routes**: Browse, about, molecules, facilities, and sign-in pages are publicly accessible. No auth required to view data.

**Protected Routes**: All `/contribute/*` pages require sign-in. Pages check `useSession()` and render a sign-in prompt or redirect when unauthenticated. Mutations (create, update, delete) use tRPC `protectedProcedure`.

**tRPC Procedures**: Use `publicProcedure` for read-only endpoints that serve anonymous users. Use `protectedProcedure` for any endpoint that mutates data or requires user identity. Protected procedures throw `UNAUTHORIZED` when `ctx.userId` is null.

**API Auth**: Server-side use `auth()` from `~/server/auth` or `requireAuth()` when a session is mandatory. API routes (passkeys, link-account) call `auth()` and return 401 when unauthenticated.

### Server

**Typescript**: You are a type script expert who follows all of the best practices of type script. You are familiar with the current ESLint rules, and ensure that all code follows the current rules. React components follow the design architecture above. You always favor server side optimization whenever possible.

**Bun Runtime**: You are an expert in Bun and use Bun as the fast TypeScript and JavaScript package manager, bundler, test runner, all in one toolkit, and runtime. You use the latest stable version of bun and are an expert in all of it's features.

**Next.js Page Router**: The website uses the NextJs Page router design. You always use the latest version of Next.js Page router, and are familiar with the latest features and best practices.

### Front End

**Tailwind Styling**: CSS styling always uses Tailwind primitives. You are an expert in Tailwind and always use the latest stable version of Tailwind.

**HeroUI**: Whenever and wherever possible save time and code by using HeroUI component primitives. You always use the latest stable version of HeroUI, and are familiar with the latest features and best practices.

---

# Part 2: Web Interface Guidelines

Review files for compliance with Web Interface Guidelines and Apple Human Interface Guidelines.

## How It Works

1. Read the relevant guideline references for the review type
2. Read the specified files (or prompt user for files/pattern)
3. Check against all applicable rules
4. Output findings in the terse `file:line` format

## Guidelines Sources

### Vercel Web Interface Guidelines

Fetch fresh guidelines before each review:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Use WebFetch to retrieve the latest rules. The fetched content contains all the rules and output format instructions.

### Apple Human Interface Guidelines

Reference the local HIG guideline files in the `references/` directory for platform-specific design patterns. Each reference includes links to the official Apple HIG documentation.

## Reference Documents

The following Human Interface Guidelines references are available:

| Reference                                        | Description                                          | Apple HIG Link                                                                                    |
| ------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [Accessibility](references/hig-accessibility.md) | WCAG compliance, screen readers, keyboard navigation | [HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility) |
| [Color](references/hig-color.md)                 | Semantic colors, contrast, data visualization        | [HIG: Color](https://developer.apple.com/design/human-interface-guidelines/color)                 |
| [Dark Mode](references/hig-dark-mode.md)         | Appearance modes, surface elevation, adaptation      | [HIG: Dark Mode](https://developer.apple.com/design/human-interface-guidelines/dark-mode)         |
| [Icons](references/hig-icons.md)                 | Icon sizing, accessibility, animation                | [HIG: SF Symbols](https://developer.apple.com/design/human-interface-guidelines/sf-symbols)       |
| [Layout](references/hig-layout.md)               | Spacing, grids, responsive design                    | [HIG: Layout](https://developer.apple.com/design/human-interface-guidelines/layout)               |
| [Motion](references/hig-motion.md)               | Animation timing, easing, reduced motion             | [HIG: Motion](https://developer.apple.com/design/human-interface-guidelines/motion)               |
| [Typography](references/hig-typography.md)       | Type scale, hierarchy, scientific typography         | [HIG: Typography](https://developer.apple.com/design/human-interface-guidelines/typography)       |

## Usage

When a user provides a file or pattern argument:

1. Fetch Vercel guidelines from the source URL above
2. Read the relevant HIG reference files from `references/`
3. Read the specified files
4. Apply all rules from both guideline sources
5. Output findings using the format specified in the guidelines

### Review Types

**General UI Review** - Use all references:

- Accessibility, Color, Dark Mode, Icons, Layout, Motion, Typography

**Accessibility Audit** - Focus on:

- Accessibility reference
- Color reference (contrast requirements)
- Typography reference (legibility)

**Design System Review** - Focus on:

- Color reference
- Typography reference
- Icons reference

**Animation Review** - Focus on:

- Motion reference
- Accessibility reference (reduced motion)

**Responsive/Layout Review** - Focus on:

- Layout reference
- Typography reference (responsive scaling)

If no files specified, ask the user which files to review.

## Apple HIG Quick Links

Core guidelines from Apple Human Interface Guidelines:

- **Foundations**
  - [Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
  - [Color](https://developer.apple.com/design/human-interface-guidelines/color)
  - [Dark Mode](https://developer.apple.com/design/human-interface-guidelines/dark-mode)
  - [Icons](https://developer.apple.com/design/human-interface-guidelines/sf-symbols)
  - [Layout](https://developer.apple.com/design/human-interface-guidelines/layout)
  - [Motion](https://developer.apple.com/design/human-interface-guidelines/motion)
  - [Typography](https://developer.apple.com/design/human-interface-guidelines/typography)

- **Additional Resources**
  - [Materials](https://developer.apple.com/design/human-interface-guidelines/materials)
  - [Right to Left](https://developer.apple.com/design/human-interface-guidelines/right-to-left)
  - [Spatial Layout](https://developer.apple.com/design/human-interface-guidelines/spatial-layout)
  - [Playing Haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)

---

# Part 3: Production Build & Deployment (Bun)

Pre-deployment checklist for Next.js production builds.

## Phase 1: Environment & Dependencies

First, verify your environment is production-ready. Run `bun audit` to check for security vulnerabilities in your dependencies. Address any critical or high-severity issues before proceeding. Ensure your `.env.production` file exists with all necessary environment variables, and confirm no sensitive data is committed to version control by checking your `.gitignore` includes `.env*` files.

## Phase 2: Code Quality & Linting

Navigate to your project root and run `bunx eslint . --ext .js,.jsx,.ts,.tsx --max-warnings 0` to enforce zero ESLint warnings. If you need to check specific directories, run `bunx eslint ./src/components --ext .js,.jsx,.ts,.tsx` for each directory. To see the active ESLint configuration, use `bunx eslint --print-config src/components/SomeComponent.tsx`. For auto-fixable issues, run `bunx eslint . --fix --ext .js,.jsx,.ts,.tsx`.

If you're using TypeScript, run `bunx tsc --noEmit` to catch type errors without generating output files. This ensures type safety across your entire codebase.

## Phase 3: Testing & Validation

Run your full test suite with `bun test`. Ensure all unit tests, integration tests, and end-to-end tests pass. If using Playwright or Cypress, run `bunx playwright test` or `bunx cypress run` for E2E tests. Check test coverage with `bun test --coverage` and aim for at least 80% coverage on critical paths.

## Phase 4: Accessibility & Interface Guidelines

For Apple Human Interface Guidelines compliance, manually review components for proper semantic HTML, touch target sizes (minimum 44x44 points), color contrast ratios (WCAG AA minimum), and proper focus management. Run `bunx @axe-core/cli http://localhost:3000` on your development build to catch automated accessibility issues. Consider using `eslint-plugin-jsx-a11y` in your ESLint config to catch accessibility issues during development.

## Phase 5: Build Verification

Run `bun run build` to create your production build. This will trigger Next.js optimization, tree-shaking, and bundle creation. Pay attention to bundle size warnings. After the build completes, analyze the output in `.next/analyze` if you have `@next/bundle-analyzer` configured, or add it by running `bun add @next/bundle-analyzer` and adding it to your `next.config.js`.

Test the production build locally with `bun run start` and verify all routes, API endpoints, and functionality work as expected. Check for console errors in your browser developer tools.

## Phase 6: Performance & Optimization

Run Lighthouse audits using Chrome DevTools on key pages to ensure Performance, Accessibility, Best Practices, and SEO scores are above 90. Address any Critical or High priority issues. Verify your images are using Next.js Image component with proper width, height, and alt attributes. Check that fonts are optimized using `next/font`.

## Phase 7: Security Checks

Verify Content Security Policy headers are configured in your `next.config.js`. Ensure no API keys or secrets are exposed in client-side code. Run `bun run build` and inspect the output for any accidentally bundled environment variables. Check that your API routes have proper authentication and rate limiting.

## Additional Items

Confirm your `robots.txt` and `sitemap.xml` are properly configured for SEO. Verify social media meta tags (Open Graph, Twitter Cards) are present on relevant pages. Test your application in multiple browsers (Chrome, Safari, Firefox, Edge) and ensure your error boundaries are working by deliberately triggering errors in development. Set up proper logging and monitoring (Sentry, LogRocket, etc.) before deployment. Verify your database migrations are up to date and backup procedures are in place.

## Final Build Command

Once all checks pass:

```bash
bun run lint && bun run type-check && bun test && bun run build
```

Or create a custom script in your `package.json`:

```json
"scripts": {
  "pre-deploy": "bun run lint && tsc --noEmit && bun test",
  "deploy": "bun run pre-deploy && bun run build"
}
```

Then run `bun run deploy` to execute all checks before building.
