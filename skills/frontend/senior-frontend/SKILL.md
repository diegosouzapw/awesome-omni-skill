---

# === CORE IDENTITY ===
name: senior-frontend
title: Senior Frontend Skill Package
description: Comprehensive frontend development skill for building modern, performant web applications using React, Next.js, TypeScript, Tailwind CSS. Includes component scaffolding, performance optimization, bundle analysis, and UI best practices. Use when developing frontend features, optimizing performance, implementing UI/UX designs, managing state, or reviewing frontend code.
domain: engineering
subdomain: frontend-development

# === TECHNICAL ===
dependencies:
  scripts: []
  references: []
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack:
  - React
  - Next.js
  - TypeScript
  - Tailwind CSS
  - JavaScript
  - HTML5
  - CSS3
  - Webpack
  - ESLint
  - Jest

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-frontend"
    output: "TODO: Add expected output"

# === DISCOVERABILITY ===
tags:
  - analysis
  - design
  - development
  - engineering
  - frontend
  - optimization
  - performance
  - senior
---

# Senior Frontend

Expert frontend development skill with comprehensive tools for building modern, performant, and accessible web applications using React, Next.js, and the latest frontend technologies.

## Overview

This skill provides production-ready frontend development capabilities through three Python automation tools and extensive reference documentation. Whether building React components, optimizing Next.js applications, implementing state management, or improving performance, this skill ensures best practices and scalable architecture.

**What This Skill Provides:**
- Component generation with TypeScript, tests, and stories
- Bundle analysis and optimization recommendations
- Project scaffolding for Next.js and React applications
- Component architecture patterns (Atomic Design, Compound Components)
- State management solutions (Zustand, Context + useReducer)
- Performance optimization techniques (code splitting, memoization)

**Use this skill when:**
- Developing React/Next.js applications
- Building reusable component libraries
- Optimizing frontend performance and bundle size
- Implementing responsive and accessible UIs
- Managing application state
- Testing and documenting components

## Quick Start

### Component Generation

```bash
# Generate component with tests and stories
python scripts/component_generator.py Button --complete

# Generate compound component
python scripts/component_generator.py Select --type compound --complete

# Generate custom hook
python scripts/component_generator.py useDebounce --type hook --tests
```

### Bundle Analysis

```bash
# Analyze Next.js build
npm run build
python scripts/bundle_analyzer.py .next/ --verbose

# Generate HTML report
python scripts/bundle_analyzer.py .next/ --output html --save report.html
```

### Project Scaffolding

```bash
# Create Next.js project
python scripts/frontend_scaffolder.py my-app --framework nextjs --complete

# Start development
cd my-app && npm install && npm run dev
```

## Core Workflows

### 1. Building Component Library

**Steps:**
1. Scaffold project: `python scripts/frontend_scaffolder.py ui-library --framework nextjs`
2. Generate atoms: Button, Input, Label components
3. Generate molecules: FormField, SearchBar using atoms
4. Generate organisms: LoginForm, Navigation using molecules
5. Add tests for each component
6. Document with Storybook stories

**Component Patterns:**
- Atomic Design hierarchy (atoms → molecules → organisms)
- TypeScript for type safety
- Tailwind CSS with CVA for variants
- Testing Library for unit tests
- Storybook for documentation

**See:** [frameworks.md](references/frameworks.md) for component patterns and [templates.md](references/templates.md) for component templates.

### 2. Performance Optimization

**Optimization Workflow:**
1. Build for production: `npm run build`
2. Analyze bundle: `python scripts/bundle_analyzer.py .next/ --verbose`
3. Identify large dependencies and duplicates
4. Implement optimizations:
   - Code splitting with dynamic imports
   - Replace large libraries (moment → date-fns)
   - Use lodash-es for tree-shaking
   - Memoize expensive components
5. Re-analyze and verify improvements

**Optimization Techniques:**
- Lazy loading with React.lazy()
- useMemo and useCallback for expensive operations
- React.memo for component memoization
- Virtual scrolling for long lists
- Image optimization with Next.js Image

**See:** [frameworks.md](references/frameworks.md) for performance patterns and memoization examples.

### 3. State Management Implementation

**For Simple State:**
- Use Zustand for lightweight global state
- localStorage persistence with middleware
- TypeScript for type-safe state

**For Complex State:**
- Context + useReducer for complex flows
- Actions and reducers pattern
- Separate context per domain

**See:** [frameworks.md](references/frameworks.md) for state management patterns and [templates.md](references/templates.md) for Zustand setup.

### 4. Next.js Application Development

**Development Workflow:**
1. Scaffold Next.js project with App Router
2. Implement layouts (Dashboard, Auth, Marketing)
3. Create pages with server components
4. Add client components for interactivity
5. Implement API routes for backend logic
6. Configure data fetching (React Query)
7. Add authentication (JWT or NextAuth)
8. Optimize for production

**Next.js Features:**
- Server Components for better performance
- Server-side rendering (SSR) for dynamic content
- Static generation (SSG) for static pages
- API routes for backend endpoints
- Incremental Static Regeneration (ISR)

**See:** [frameworks.md](references/frameworks.md) for Next.js patterns and [templates.md](references/templates.md) for project setup.

### 5. Testing and Quality Assurance

**Testing Strategy:**
1. Unit tests for components (Testing Library)
2. Custom hook tests with renderHook
3. Integration tests for user flows
4. Visual regression tests with Storybook
5. E2E tests with Playwright

**Testing Best Practices:**
- Test user behavior, not implementation
- Use semantic queries (getByRole, getByLabelText)
- Mock external dependencies
- Aim for 80%+ coverage on critical paths

**See:** [templates.md](references/templates.md) for testing examples and patterns.

## Python Tools

### component_generator.py

Automated React component generation with TypeScript, tests, and documentation.

**Key Features:**
- Multiple component types (functional, compound, HOC, hook)
- Atomic design level support (atom, molecule, organism, template)
- Styling options (Tailwind, CSS Modules, styled-components)
- Test generation with Testing Library
- Storybook story generation
- TypeScript definitions with proper types

**Usage:**
```bash
# Complete component with tests and stories
python scripts/component_generator.py Button --complete

# Compound component
python scripts/component_generator.py Select --type compound --complete

# Custom hook with tests
python scripts/component_generator.py useDebounce --type hook --tests
```

**See:** [tools.md](references/tools.md) for complete documentation and generated code examples.

### bundle_analyzer.py

Advanced bundle analysis for identifying optimization opportunities.

**Key Features:**
- Bundle size breakdown by page
- Largest dependencies identification
- Duplicate package detection
- Optimization recommendations
- Interactive treemap visualization
- HTML report generation
- Before/after comparisons

**Usage:**
```bash
# Analyze build
python scripts/bundle_analyzer.py .next/ --verbose

# Generate report
python scripts/bundle_analyzer.py .next/ --output html --save report.html

# Compare builds
python scripts/bundle_analyzer.py .next/ --compare baseline.html
```

**See:** [tools.md](references/tools.md) for analysis features and optimization recommendations.

### frontend_scaffolder.py

Complete project scaffolding for Next.js and React applications.

**Key Features:**
- Next.js 14 (App Router) or React + Vite support
- TypeScript configuration
- Tailwind CSS with custom theme
- State management (Zustand, Redux)
- Form handling (React Hook Form + Zod)
- Testing setup (Jest + Testing Library)
- Storybook configuration
- CI/CD pipelines (GitHub Actions)

**Usage:**
```bash
# Next.js project with all features
python scripts/frontend_scaffolder.py my-app --framework nextjs --complete

# React + Vite project
python scripts/frontend_scaffolder.py my-app --framework react-vite

# Minimal setup
python scripts/frontend_scaffolder.py my-app --minimal
```

**See:** [tools.md](references/tools.md) for project structure and configuration details.

## Reference Documentation

### Architecture Frameworks ([frameworks.md](references/frameworks.md))

Comprehensive patterns and best practices:

- **Component Patterns:** Atomic Design, Compound Components, Render Props
- **State Management:** Zustand, Context + useReducer, React Query
- **Performance Optimization:** Code splitting, memoization, virtual scrolling
- **Next.js Patterns:** Server Components, data fetching, API routes
- **Testing Patterns:** Component tests, hook tests, integration tests
- **Styling Patterns:** CSS Modules, Tailwind + CVA, styled-components
- **Error Handling:** Error boundaries, error states, fallback UIs

### Implementation Templates ([templates.md](references/templates.md))

Production-ready code templates:

- **Project Setup:** Next.js + TypeScript + Tailwind configuration
- **Component Templates:** Button, Input, Modal, Form components
- **Custom Hooks:** useDebounce, useAsync, useLocalStorage
- **Layout Templates:** Dashboard, Auth, Marketing layouts
- **Data Fetching:** React Query setup, custom hooks
- **Testing Templates:** Component tests, hook tests, integration tests
- **Environment Configuration:** .env setup and feature flags

### Python Tools Guide ([tools.md](references/tools.md))

Complete tool documentation:

- **component_generator.py:** Component types, options, generated structure
- **bundle_analyzer.py:** Analysis features, optimization recommendations
- **frontend_scaffolder.py:** Framework options, project structure, features

## Tech Stack

**Core:** React 18, Next.js 14, TypeScript 5
**Styling:** Tailwind CSS, CSS Modules, class-variance-authority
**State:** Zustand, React Query, Context API
**Forms:** React Hook Form, Zod validation
**Testing:** Jest, Testing Library, Playwright
**Docs:** Storybook
**Build:** Webpack, Turbopack, Vite
**Linting:** ESLint, Prettier
**CI/CD:** GitHub Actions

## Best Practices Summary

### Component Design
- Use TypeScript for type safety
- Follow Atomic Design principles
- Keep components small and focused
- Use composition over inheritance
- Implement proper prop types and defaults

### Performance
- Code split routes and heavy components
- Memoize expensive computations
- Use React.memo for pure components
- Implement virtual scrolling for long lists
- Optimize images with Next.js Image
- Lazy load below-the-fold content

### State Management
- Keep state close to where it's used
- Use Zustand for simple global state
- Context + useReducer for complex flows
- React Query for server state
- Avoid prop drilling with composition

### Testing
- Test user behavior, not implementation
- Use semantic queries (getByRole, getByLabelText)
- Mock external dependencies
- Test error states and edge cases
- Maintain 80%+ coverage on critical paths

### Accessibility
- Use semantic HTML elements
- Implement proper ARIA labels
- Ensure keyboard navigation
- Maintain color contrast ratios
- Test with screen readers

## Common Commands

```bash
# Development
pnpm run dev          # Start dev server
pnpm run build        # Build for production
pnpm start            # Start production server

# Testing
pnpm test             # Run tests
pnpm run test:watch   # Watch mode
pnpm run test:coverage  # Coverage report

# Quality
pnpm run lint         # ESLint
pnpm run format       # Prettier
pnpm run type-check   # TypeScript

# Storybook
pnpm run storybook    # Start Storybook
pnpm run build-storybook  # Build static Storybook
```

## Integration Points

This skill integrates with:
- **Backend Skills:** API consumption, authentication
- **Design Skills:** Figma integration, design tokens
- **QA Skills:** E2E testing, visual regression
- **DevOps Skills:** Docker deployment, CI/CD
- **Analytics:** Google Analytics, custom events

## Getting Help

1. **Component patterns:** See [frameworks.md](references/frameworks.md)
2. **Code templates:** See [templates.md](references/templates.md)
3. **Tool usage:** See [tools.md](references/tools.md) or run with `--help`
4. **Project setup:** Use frontend_scaffolder.py to generate boilerplate
