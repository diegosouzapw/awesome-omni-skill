---
name: frontend-patterns
description: Frontend development patterns for React and modern frameworks
triggers: [context, frontend, react, component, ui]
---

# Frontend Patterns Skill

> **Purpose**: Apply modern frontend development patterns

---

## Overview

This skill provides best practices for building maintainable, performant frontend applications.

---

## Component Architecture

### Atomic Design

```
atoms/          → Button, Input, Label
molecules/      → FormField, SearchInput
organisms/      → LoginForm, Header
templates/      → PageLayout, DashboardLayout
pages/          → LoginPage, DashboardPage
```

### Feature-Based Structure

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── index.ts
│   └── dashboard/
│       ├── components/
│       ├── hooks/
│       └── index.ts
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── App.tsx
```

---

## React Patterns

### Custom Hooks

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

### Compound Components

```tsx
<Select value={selected} onChange={setSelected}>
  <Select.Trigger>Choose option</Select.Trigger>
  <Select.Options>
    <Select.Option value="a">Option A</Select.Option>
    <Select.Option value="b">Option B</Select.Option>
  </Select.Options>
</Select>
```

### Render Props

```tsx
<DataFetcher url="/api/users">
  {({ data, loading, error }) =>
    loading ? <Spinner /> : <UserList users={data} />
  }
</DataFetcher>
```

---

## State Management

| Solution    | Use Case              |
| :---------- | :-------------------- |
| useState    | Local component state |
| useReducer  | Complex local state   |
| Context     | Theme, auth, i18n     |
| Zustand     | Simple global state   |
| Redux       | Complex global state  |
| React Query | Server state          |

---

## Performance

### Memoization

```tsx
// Expensive calculation
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items],
);

// Callback stability
const handleClick = useCallback(() => {
  onClick(id);
}, [id, onClick]);

// Component memoization
const UserCard = memo(({ user }) => <div>{user.name}</div>);
```

---

## Quick Reference

| Pattern      | Usage                  |
| :----------- | :--------------------- |
| Custom Hooks | Reusable logic         |
| Compound     | Flexible APIs          |
| Render Props | Dynamic rendering      |
| HOC          | Cross-cutting concerns |
| Context      | Global state           |
| Memoization  | Performance            |
