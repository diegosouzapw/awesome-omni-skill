---
name: react
description: React development patterns and best practices including hooks, state management, and performance optimization.
globs: ["**/*.jsx", "**/*.tsx", "**/next.config.*", "**/vite.config.*"]
priority: 95
tags: ["framework"]
---

# React Best Practices

## Component Design
- Keep components small and focused (< 200 lines)
- Prefer functional components with hooks
- Use composition over prop drilling
- Lift state only when needed
- Co-locate related code

## Hooks
- Use useState for local state
- Use useEffect for side effects (with cleanup)
- Use useCallback for stable function references
- Use useMemo for expensive computations
- Use useRef for mutable values
- Always include all dependencies in dependency arrays
- Create custom hooks for reusable logic

## Performance
- Use React.memo for expensive pure components
- Lazy load routes with React.lazy
- Use Suspense for loading states
- Use keys properly in lists (never use index as key)
- Avoid inline object/function creation in JSX
- Use React DevTools Profiler

## State Management
- Start with local state + prop drilling
- Use Context for theme/auth/locale
- Use Zustand/Jotai for simple global state
- Use React Query/TanStack Query for server state
- Use Redux Toolkit for complex client state

## Patterns
- Container/Presentational components
- Compound components for flexibility
- Render props for sharing logic
- Custom hooks for logic reuse
