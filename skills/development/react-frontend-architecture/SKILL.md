---
name: React Frontend Architecture
description: Comprehensive guidelines and standards for building scalable React applications, including project structure, component design, state management, and styling.
---

# React Frontend Architecture Guide

This skill provides the architectural foundation for React applications in this project. It is designed to ensure maintainability, scalability, and strict separation of concerns.

## 1. Project Structure

Organize the `src` directory as follows. We strictly follow a **Features-First** (Domain-Driven) organization approach.

```
src/
├── app/            # Next.js App Router pages (if using App Router)
├── assets/         # Static assets (images, fonts, global styles)
├── components/     # Shared "Dumb" UI components (Buttons, Inputs, Modals)
│   ├── ui/         # Atomic UI elements
│   └── layout/     # Structural components (Header, Footer, Sidebar)
├── features/       # Business Logic & Complex Features
│   └── [feature]/  # e.g., 'auth', 'dashboard', 'user-profile'
│       ├── components/  # Components specific to this feature
│       ├── hooks/       # Hooks specific to this feature
│       ├── api/         # API calls/services specific to this feature
│       └── types/       # Types specific to this feature
├── hooks/          # Global/Shared custom hooks (useDebounce, useOnClickOutside)
├── lib/            # Third-party library configurations (Axios instance, queryClient)
├── services/       # Global API definitions (if not using feature-based)
├── store/          # Global State Store (Zustand/Redux)
├── utils/          # Pure helper functions (formatDate, currency helpers)
└── types/          # Global TypeScript definitions
```

## 2. Component Architecture: The MVC Pattern

We adhere to a strict **Separation of Concerns** using a ViewModel-like pattern.
**Rule**: UI Components should contain *zero* business logic.

### 2.1 File Structure
Each component is a directory. The structure is non-negotiable for "Smart" (Logic-heavy) components.

```
MyComponent/
├── MyComponent.tsx       # View (Template)
├── MyComponent.hook.ts   # Logic (ViewModel)
├── MyComponent.type.ts   # Type Definitions
├── MyComponent.css       # Styles (CSS Modules preferred)
├── index.ts              # Export Interface
└── components/           # (Optional) Internal sub-components
```

### 2.2 Roles & Responsibilities

#### `[Name].tsx` - The View
*   **Role**: Pure Function of Props/State.
*   **Responsibility**: Rendering JSX, binding events to handlers.
*   **Constraints**:
    *   No `useEffect` allowed (unless purely UI related like animations).
    *   No complex calculations.
    *   Must call `use[Name]` hook to get its state/handlers.

#### `[Name].hook.ts` - The Logic (ViewModel)
*   **Role**: The Brain.
*   **Responsibility**:
    *   Manage local state (`useState`, `useReducer`).
    *   Handle side effects (`useEffect`).
    *   Fetch data (consume `useQuery` or services).
    *   **Process Logic**: Formatting dates, calculating derived values, mapping data structures.
    *   Return a neat interface (`interface Use[Name]Result`) for the View.
*   **Goal**: The `.tsx` file should be reduced to a purely declarative template. If you find yourself writing a `map`, `filter`, or `new Date()` in the `.tsx` file, consider if it belongs in the hook.

#### `[Name].type.ts` - The Type Definitions
*   **Role**: Type Safety & Contract.
*   **Responsibility**:
    *   Define all TypeScript interfaces/types specific to this component.
    *   Separate API response types, component props, and internal state types.
    *   Export types for external consumers (via `index.ts`).
*   **Why Separate**:
    *   Keeps hook and view files focused on logic and rendering.
    *   Makes types easier to import and reuse across features.
    *   Improves testability by isolating type definitions.
*   **Naming**: Use descriptive names with component prefix (e.g., `MyComponentProps`, `MyComponentState`).

#### `api/*.ts` - The Service Layer
*   **Role**: Data Access.
*   **Responsibility**:
    *   Knows HTTP implementation details (GET/POST endpoints).
    *   Returns Promises of typed data.
    *   **Never** contains UI state logic.

### 2.3 Implementation Example

**MyComponent.type.ts**
```typescript
// Component Props
export interface MyComponentProps {
    initialCount?: number;
}

// State Shape
export interface MyComponentState {
    count: number;
    isSubmitting: boolean;
}

// Action Handlers
export interface MyComponentActions {
    increment: () => void;
    submit: () => Promise<void>;
}

// Hook Return Interface
export interface UseMyComponentResult {
    state: MyComponentState;
    actions: MyComponentActions;
}

// API Request/Response Types
export interface SubmitDataRequest {
    count: number;
}

export interface SubmitDataResponse {
    success: boolean;
    timestamp: string;
}
```

**MyComponent.hook.ts**
```typescript
import { useState, useCallback } from 'react';
import { useSubmitData } from './api/useSubmitData';
import type {
    MyComponentProps,
    MyComponentState,
    MyComponentActions,
    UseMyComponentResult,
    SubmitDataRequest
} from './MyComponent.type';

export const useMyComponent = ({ initialCount = 0 }: MyComponentProps): UseMyComponentResult => {
    const [count, setCount] = useState(initialCount);
    const mutation = useSubmitData();

    const increment = useCallback(() => setCount(c => c + 1), []);

    const submit = async () => {
        const request: SubmitDataRequest = { count };
        try {
            await mutation.mutateAsync(request);
        } catch (error) {
            console.error(error);
        }
    };

    return {
        state: { count, isSubmitting: mutation.isLoading },
        actions: { increment, submit }
    };
};
```

**MyComponent.tsx**
```tsx
import React from 'react';
import styles from './MyComponent.module.css';
import { useMyComponent } from './MyComponent.hook';
import type { MyComponentProps } from './MyComponent.type';

export const MyComponent: React.FC<MyComponentProps> = (props) => {
    const { state, actions } = useMyComponent(props);

    return (
        <div className={styles.container}>
            <h1>Count: {state.count}</h1>
            <button onClick={actions.increment} disabled={state.isSubmitting}>
                Increment
            </button>
            <button onClick={actions.submit}>
                {state.isSubmitting ? 'Saving...' : 'Save'}
            </button>
        </div>
    );
};
```

## 3. Naming Conventions (Strict)

Consistency is key to preventing code rot.

*   **Directories**:
    *   Components/Pages: `PascalCase` (e.g., `UserProfile`, `EditButton`).
    *   Utilities/Hooks/Libs: `camelCase` (e.g., `utils`, `hooks`).
*   **Files**:
    *   React Components: `PascalCase.tsx` (e.g., `UserProfile.tsx`).
    *   Hooks: `use` prefix + `camelCase.ts` (e.g., `useUserProfile.ts`).
    *   Logic Files: `[Component].hook.ts`.
    *   Type Files: `[Component].type.ts`.
    *   Utilities: `camelCase.ts` (e.g., `formatDate.ts`).
    *   Styles: `[Component].module.css` (preferred for scoping).
*   **Variables/Functions**:
    *   Boolean props/vars: `is`, `has`, `should` prefix (e.g., `isLoading`, `hasError`).
    *   Handlers: `handle` prefix (e.g., `handleSubmit`, `handleInputChange`).
    *   Props passing handlers: `on` prefix (e.g., `onClick`, `onSubmit`).

## 4. Next.js Specific Guidelines

*   **Client vs Server Components**:
    *   The `*.hook.ts` pattern is specifically for **Client Components** (`'use client'`).
    *   **Server Components** (RSC) should be used for initial data fetching where appropriate to reduce client JS bundles.
    *   **Boundary**: Keep RSC at the "Page" level and pass data down to "Smart" Client components.

## 5. CSS & Styling Architecture

*   **Avoid Global CSS**: Only use `globals.css` for resets and root variables.
*   **Z-Index Management**: Define z-indices in a global variable map or strictly in CSS variables to avoid stacking context wars.
*   **Design System**: Reusable values (colors, spacing) MUST come from CSS variables or Tailwind utility classes. Do not use magic hex codes in component files.
