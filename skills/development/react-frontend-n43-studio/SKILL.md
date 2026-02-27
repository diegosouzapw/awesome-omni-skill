---
name: react-frontend
description: Provides best practices for building React applications with TypeScript and Tailwind CSS, covering component design, state management, data fetching, forms, performance, and accessibility. Use when building or modifying React frontend components, hooks, or pages.
---

# React Frontend Best Practices

## Quick Start

### Component Design

- Use functional components with TypeScript interfaces for props
- Keep components small and focused (single responsibility)
- Use compound components for complex UI patterns

```tsx
interface CardProps {
  title: string
  value: number
  onSelect?: () => void
}

function Card({ title, value, onSelect }: CardProps) {
  return (
    <div className="rounded border p-4" onClick={onSelect}>
      <h3>{title}</h3>
      <span>{value}</span>
    </div>
  )
}
```

### State Management

| State Type        | Solution                    |
| ----------------- | --------------------------- |
| Server/async data | TanStack Query              |
| Form state        | react-hook-form or useState |
| Local UI state    | useState                    |
| Shared UI state   | Context or Zustand          |
| URL state         | React Router                |

### Data Fetching (TanStack Query)

```tsx
function useData() {
  return useQuery({ queryKey: ["data"], queryFn: fetchData })
}
```

### Styling with Tailwind

- Use `cn()` utility for conditional classes
- Define variant objects for component variants
- Never use inline styles when Tailwind classes exist

### File Naming

| Type       | Convention              | Example               |
| ---------- | ----------------------- | --------------------- |
| Components | PascalCase              | `DashboardCard.tsx`   |
| Hooks      | camelCase, `use` prefix | `useDashboardData.ts` |
| Utilities  | camelCase               | `formatDate.ts`       |
| Constants  | SCREAMING_SNAKE_CASE    | `API_BASE_URL`        |

### Performance

- Memoize expensive computations with `useMemo`
- Memoize callbacks passed to children with `useCallback`
- Don't memoize prematurely — measure first

## Additional Resources

- For complete patterns including forms, routing, error handling, testing, and accessibility, see [reference.md](reference.md)
