---
name: seeksaas-development
description: Complete SeekSaaS development guide covering project setup, workflows, conventions, and best practices
---

# SeekSaaS Development Guide

## Project Overview

SeekSaaS is a **production-ready full-stack SaaS starter template** built with:
- **Frontend**: React 19 + React Router v7 + Vite 7
- **Backend**: Hono + Drizzle ORM + PostgreSQL
- **Deployment**: Cloudflare Workers + Pages
- **Styling**: Tailwind CSS v4 + Shadcn UI
- **Type Safety**: Full-stack TypeScript

## Monorepo Architecture

SeekSaaS uses a monorepo with **pnpm workspaces** and **Turbo** for efficient development:

```
seeksaas-www-react-router-v7/
├── apps/              # Deployable applications (web)
├── packages/          # Reusable libraries (30+ packages)
├── features/          # Feature modules (ai, auth, cms, mkt, panel, shared)
├── apis/              # API layer (api, api-client, api-services, etc.)
├── auths/             # Authentication (auth, auth-shared, session)
├── database/          # Database (db, dal)
├── mails/             # Email services (mail, mail-templates)
├── payments/          # Payment services (payment, plans)
├── newsletters/       # Newsletter (newsletter, mail-newsletter)
├── configs/           # Configuration (config, env)
├── tools/             # Dev tools (seed, typescript-config)
└── pnpm-workspace.yaml
```

## Development Workflow

### Getting Started

1. **Clone and Install**
```bash
git clone <repository-url>
cd seeksaas-www-react-router-v7
pnpm install
```

2. **Environment Setup**
```bash
cp apps/web/.env.example apps/web/.env
# Edit apps/web/.env with your configuration
```

3. **Database Setup**
```bash
pnpm db:generate  # Generate migrations
pnpm db:migrate   # Run migrations
pnpm db:studio    # Open Drizzle Studio (optional)
```

4. **Start Development**
```bash
pnpm dev          # Start all services
pnpm dev:web      # Start web app only
```

### Common Commands

```bash
# Development
pnpm dev              # Start development server
pnpm dev:web          # Start web app only

# Building
pnpm build            # Build all packages
pnpm check-types      # Type checking

# Code Quality
pnpm lint             # Run linter
pnpm format           # Format code
pnpm check            # Check code quality

# Database
pnpm db:generate      # Generate migrations from schema
pnpm db:migrate       # Apply migrations
pnpm db:push          # Push schema changes (dev only)
pnpm db:studio        # Open database studio
pnpm seed             # Seed database

# Deployment
pnpm cf:preview:web   # Preview on Cloudflare
pnpm cf:deploy:web    # Deploy to Cloudflare
pnpm mail:preview     # Preview email templates

# Email
pnpm mail:preview      # Preview email templates in browser
```

## Code Conventions

### TypeScript

- Use **strict mode** TypeScript
- Prefer explicit types over `any`
- Use interfaces for object shapes, types for unions/primitives
- Export types with `export type` when possible

### React

- Use **functional components** with hooks
- Prefer named exports over default exports
- Use TypeScript for component props
- Follow the composition pattern

### File Naming

- **Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- **Utilities**: `camelCase.ts` (e.g., `formatDate.ts`)
- **Constants**: `UPPER_SNAKE_CASE.ts` (e.g., `API_ROUTES.ts`)
- **Types**: `PascalCase.ts` or `*.types.ts`
- **Tests**: `*.test.ts` or `*.spec.ts`

### Package Dependencies

- Use **workspace protocol**: `"@workspace/package-name": "workspace:*"`
- Use **catalog protocol** for shared dependencies: `"package-name": "catalog:"`
- Follow unidirectional dependencies to avoid cycles

## Architecture Layers

```
┌─────────────────────────────────────┐
│   Application Layer (apps/web)      │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   UI Component Layer                │
│   (packages/ui, features/*)        │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   API Layer                         │
│   (apis/api, apis/api-client)      │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Service Layer                     │
│   (apis/api-services)              │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Data Layer                        │
│   (database/db)                    │
└─────────────────────────────────────┘
```

## Key Technologies

### Frontend Stack

- **React 19** - Latest React with concurrent features
- **React Router v7** - File-system routing with data loading
- **Vite 7** - Fast build tool with HMR
- **Tailwind CSS v4** - Utility-first CSS
- **Shadcn UI** - Accessible component primitives

### Backend Stack

- **Hono** - Lightweight web framework for edge computing
- **Cloudflare Workers** - Serverless deployment
- **Drizzle ORM** - Type-safe database ORM
- **PostgreSQL** - Relational database (Neon)
- **Better Auth** - Authentication solution

### Developer Tools

- **TypeScript** - Type safety
- **Biome** - Linting and formatting (replaces ESLint + Prettier)
- **Turbo** - Monorepo build system
- **pnpm** - Package manager with workspace support
- **Husky** - Git hooks
- **lint-staged** - Pre-commit checks

## Feature Modules

### Core Features

- **Authentication** (`features/auth`) - Login, signup, password reset
- **CMS** (`features/cms`) - Content management system
- **Marketing** (`features/mkt`) - Landing pages, about, contact
- **Panel** (`features/panel`) - User dashboard and admin panel
- **AI** (`features/ai`) - AI features (chat, image generation)
- **Shared** (`features/shared`) - Shared components (Logo, background, 404)

### Integration Modules

- **Payments** (`payments/*`) - Stripe integration, plan management
- **Email** (`mails/*`) - Email service with Resend
- **Newsletter** (`newsletters/*`) - Newsletter subscription
- **Storage** (`packages/storage`) - S3/R2 storage

## Environment Variables

### Required Variables

```bash
# Database
DATABASE_URL=postgresql://user:password@host:port/db

# Authentication
BETTER_AUTH_SECRET=your-secret-key
BETTER_AUTH_URL=http://localhost:5173

# Optional Services
OPENAI_API_KEY=sk-xxx              # AI features
RESEND_API_KEY=re_xxx              # Email
STRIPE_PK=pk_test_xxx              # Payments
STRIPE_SK=sk_test_xxx
```

### Environment Files

- **Development**: `apps/web/.env`
- **Production**: Set in Cloudflare Pages/Workers

## Testing

### Unit Tests

```bash
# Run tests in a specific package
pnpm test --filter <package-name>
```

### E2E Tests

```bash
pnpm e2e
```

## Best Practices

### Component Development

1. Keep components **small and focused**
2. Use **composition** over inheritance
3. Implement **proper error boundaries**
4. Add **loading states** for async operations
5. Use **React Query** for server state

### API Development

1. Use **Hono** for API routes
2. Implement **proper validation** with Zod
3. Add **OpenAPI** documentation
4. Use **API client** on frontend (Hono RPC)
5. Handle **errors gracefully**

### Database

1. Define **clear schema** with Drizzle
2. Use **migrations** for schema changes
3. Add **indexes** for performance
4. Use **prepared statements**
5. Handle **transactions** properly

## Troubleshooting

### Common Issues

1. **Database connection fails**
   - Check `DATABASE_URL` is correct
   - Verify PostgreSQL service is running
   - Check network/firewall settings

2. **Build fails with type errors**
   - Run `pnpm check-types` to see details
   - Ensure all packages are built
   - Check `tsconfig.json` settings

3. **Port already in use**
   - Check which process is using port 5173/3000
   - Kill the process or use different port

4. **Environment variables not loading**
   - Verify `.env` file exists in `apps/web/`
   - Check variable names are correct
   - Restart development server

## Resources

- **Documentation**: `/packages/cms/content/docs/`
- **AGENTS.md**: Project-specific guidelines
- **Package READMEs**: Each package has its own README
- **External Docs**:
  - [React Router v7](https://reactrouter.com)
  - [Hono](https://hono.dev)
  - [Drizzle ORM](https://orm.drizzle.team)
  - [Better Auth](https://better-auth.com)
  - [Cloudflare Workers](https://workers.cloudflare.com)
