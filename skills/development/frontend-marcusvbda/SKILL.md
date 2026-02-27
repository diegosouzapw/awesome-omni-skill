---
name: frontend
description: Defines frontend standards for Next.js App Router, React 19, React Query, shadcn/ui. Use when building components, pages, hooks, or frontend state management.
---

# Frontend Architecture & UX

## Stack

- Next.js App Router
- React 19
- React Query (the only remote state manager)
- shadcn/ui + Tailwind CSS

---

## Core Architecture Rules

### Rendering Model

- Server Components by default
- Client Components **only** for interactivity
- No Client Component by convenience
- Client Components must be explicitly justified

### Client Component Boundaries

Client Components **must not**:

- Import Server Components
- Access environment variables
- Perform direct data fetching
- Contain business or domain logic

Client Components **may only**:

- Handle user interaction
- Manage local UI state
- Consume hooks
- Render presentation logic

---

## Layered Responsibilities

- **UI Components**
  - Pure presentation
  - No business rules
  - No data normalization

- **Hooks**
  - Compose UI state
  - Integrate React Query
  - No business decisions

- **Domain / Services**
  - Business rules
  - Decision making
  - Data validation

- **API Layer**
  - HTTP adaptation
  - Contract enforcement
  - Error translation

---

## Data & State Management

- React Query is the **only** remote state manager
- No direct data mutations without React Query
- No fetch calls outside the API layer

### React Query Rules

- Every query must have a stable `queryKey`
- Mutable objects are forbidden in `queryKey`
- All mutations must explicitly invalidate or update cache
- `refetchOnWindowFocus` is disabled by default
- Errors must always be handled explicitly
- Silent failures are forbidden

---

## UX Best Practices

### Accessibility

- Accessibility first
- ARIA roles where applicable
- Full keyboard navigation support
- Respect `prefers-reduced-motion`

### Visual Consistency

- Clear visual hierarchy
- Consistent spacing and typography
- Design tokens must be respected
- Avoid layout shifts (CLS)

### States Handling

Every async interaction **must** define:

- Loading state (prefer skeletons over spinners)
- Error state (user-friendly, not technical)
- Empty state (intentional, not accidental)

Technical errors must **never** be shown directly to users.

---

## Performance Rules

- Minimize client-side JavaScript
- Memoize only when measured or clearly justified
- Defensive memoization is forbidden
- Avoid unnecessary re-renders
- Lazy-load non-critical components only
- Dynamic imports must not affect critical rendering paths

---

## Developer Experience (DX)

- Reusable components over duplication
- Clear, explicit naming
- Prefer composition over configuration
- Avoid deep prop drilling

### Props Discipline

- More than 5 props requires design reconsideration
- Boolean explosion is forbidden (`isA`, `isB`, `isC`)
- Prefer explicit components over conditional flags

---

## Non-Negotiables

- No business logic in UI
- No direct backend calls from components
- No architectural shortcuts
- Predictability over flexibility

Architectural violations are treated as bugs.
