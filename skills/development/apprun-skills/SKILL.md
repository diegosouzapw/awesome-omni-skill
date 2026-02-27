---
name: apprun-skills
description: "End-to-end guidance for AppRun apps in TypeScript using MVU: component patterns, event handling, state management (including async generators), routing/navigation with params and guards, and testing with vitest. Use when designing or reviewing AppRun components, wiring routes, managing state flows, or writing AppRun tests."
---

# AppRun Skills

## Overview

- Build AppRun apps with MVU (Model-View-Update) in TypeScript.
- Prefer pure update functions for testability.
- Use `mounted()` for components embedded in JSX.
- Use `state = async` only for top-level routed pages that must load async data.

## Project Setup

### Recommended Project Structure

```
web/                        # Frontend application root
├── index.html              # Entry HTML file
├── package.json            # Dependencies and scripts
├── vite.config.js          # Vite configuration
├── src/
│   ├── main.tsx            # Application entry point (routes registration)
│   ├── api.ts              # REST API client (optional)
│   ├── styles.css          # Application styles
│   ├── tsconfig.json       # TypeScript configuration
│   ├── components/         # Reusable UI components
│   │   ├── Layout.tsx      # Root layout container
│   │   └── ...             # Other reusable components
│   ├── domain/             # Business logic modules (optional)
│   │   └── ...             # Pure functions and business logic
│   ├── pages/              # Top-level page components
│   │   ├── Home.tsx        # Example: Home page
│   │   └── ...             # Other route pages
│   ├── types/              # TypeScript type definitions
│   │   ├── index.ts        # Shared types
│   │   └── jsx.d.ts        # JSX type declarations
│   └── utils/              # Utility functions
└── public/                 # Static assets (optional)
```

### Vite Configuration

```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    outDir: 'dist',
    emptyOutDir: true,
  },
  server: {
    port: 8080,
    open: true,
    historyApiFallback: true,  // SPA mode
    proxy: {
      // Proxy API requests to backend
      '/api': {
        target: 'http://127.0.0.1:3000',
        changeOrigin: true,
        secure: false
      }
    }
  }
})
```

### Package.json

```json
{
  "name": "my-apprun-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "check": "tsc --noEmit"
  },
  "devDependencies": {
    "apprun": "^3.38.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react",
    "jsxFactory": "app.createElement",
    "jsxFragmentFactory": "app.Fragment",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

**Critical Settings for AppRun:**
- `jsx: "react"` - Enables JSX syntax
- `jsxFactory: "app.createElement"` - Uses AppRun's JSX factory
- `jsxFragmentFactory: "app.Fragment"` - Uses AppRun's Fragment support
- `moduleResolution: "bundler"` - Optimized for Vite

### Entry Points

**HTML Entry (`index.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My AppRun App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="src/main.tsx"></script>
</body>
</html>
```

**Application Entry (`src/main.tsx`):**

```tsx
import app from 'apprun';
import Layout from './components/Layout';
import Home from './pages/Home';
import About from './pages/About';
import './styles.css';

app.render('#root', <Layout />);

app.addComponents('#pages', {
  '/': Home,
  '/about': About,
});
```

**Layout Component (`src/components/Layout.tsx`):**

```tsx
import app from 'apprun';

export default () => (
  <div id="app">
    <div id="pages"></div>
  </div>
);
```

### Styling Options

**Option 1: Vanilla CSS**

```css
/* src/styles.css */
:root {
  --color-primary: #007bff;
  --color-text: #333;
  --spacing-unit: 8px;
}

body {
  font-family: system-ui, -apple-system, sans-serif;
  color: var(--color-text);
  margin: 0;
  padding: 0;
}
```

**Option 2: Tailwind CSS v4**

Install Tailwind v4:
```bash
npm install -D tailwindcss@next @tailwindcss/vite@next
```

Update `vite.config.js`:
```javascript
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()],
  // ... other config
})
```

Import in `src/styles.css`:
```css
@import "tailwindcss";
```

Use in components:
```tsx
<div className="flex items-center gap-4 p-4 bg-white rounded-lg shadow">
  <h1 className="text-2xl font-bold">Hello World</h1>
</div>
```

**Option 3: CSS Modules**

```tsx
import styles from './MyComponent.module.css';

export default () => (
  <div className={styles.container}>
    <h1 className={styles.title}>Hello</h1>
  </div>
);
```

### API Client Pattern

```typescript
// src/api.ts
const API_BASE_URL = '/api';

interface RequestOptions extends RequestInit {
  params?: Record<string, string>;
}

async function request<T>(
  endpoint: string,
  options: RequestOptions = {}
): Promise<T> {
  const { params, ...fetchOptions } = options;
  
  let url = `${API_BASE_URL}${endpoint}`;
  if (params) {
    const query = new URLSearchParams(params).toString();
    url += `?${query}`;
  }

  const response = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      ...fetchOptions.headers,
    },
    ...fetchOptions,
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new Error(error.message || `HTTP ${response.status}`);
  }

  return response.json();
}

export const api = {
  get: <T>(endpoint: string, params?: Record<string, string>) =>
    request<T>(endpoint, { method: 'GET', params }),
    
  post: <T>(endpoint: string, data?: unknown) =>
    request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    }),
    
  put: <T>(endpoint: string, data?: unknown) =>
    request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    }),
    
  delete: <T>(endpoint: string) =>
    request<T>(endpoint, { method: 'DELETE' }),
};

export default api;
```

### Quick Start

```bash
# 1. Create project
npm create vite@latest my-apprun-app -- --template vanilla-ts
cd my-apprun-app

# 2. Install AppRun
npm install
npm install -D apprun

# 3. Configure TypeScript (update tsconfig.json with settings above)

# 4. Rename entry file
mv src/main.ts src/main.tsx

# 5. Create basic app structure
# (Add Layout, pages, components as shown above)

# 6. Run development server
npm run dev

# 7. Build for production
npm run build
npm run preview
```

### Why Vite + AppRun?

**Why Vite:**
- Fast development with instant HMR
- Optimized builds with Rollup
- First-class TypeScript support
- Minimal configuration

**Why AppRun:**
- Lightweight (~7KB gzipped)
- Simple MVU pattern
- Direct DOM updates (no virtual DOM)
- Full TypeScript support
- Built-in routing

## Component Patterns - Decision Tree

1. **Manages state + user interactions?** → Stateful Class Component
2. **Popup/modal/overlay?** → Modal Component (use `mounted()`)
3. **Display-only from props?** → Functional Component
4. **10+ events needing type safety?** → Typed Events Pattern

## Stateful Class Component

**Structure Order:** Imports → Interfaces → Helpers → Actions → Component

```typescript
import { app, Component } from 'apprun';

interface Props { data?: any; }
export interface State { 
  loading: boolean; 
  error: string | null; 
  successMessage?: string;
  // ... specific fields
}

const getStateFromProps = (props: Props): State => ({ /* ... */ });

export const saveData = async function* (state: State): AsyncGenerator<State> {
  // Validation
  if (!state.data.name.trim()) {
    yield { ...state, error: 'Name required' };
    return;
  }
  
  // Loading
  yield { ...state, loading: true, error: null };
  
  // API call
  try {
    await api.save(state.data);
    yield { ...state, loading: false, successMessage: 'Saved!' };
    app.run('data-saved');
  } catch (error: any) {
    yield { ...state, loading: false, error: error.message };
  }
};

export default class MyComponent extends Component<State> {
  declare props: Readonly<Props>;
  
  mounted = (props: Props): State => getStateFromProps(props);
  
  view = (state: State) => {
    if (state.loading) return <div>Loading...</div>;
    if (state.error) return <div className="error">{state.error}</div>;
    
    return (
      <form>
        <input $bind="data.name" />
        <button $onclick={[saveData]} disabled={state.loading}>Save</button>
      </form>
    );
  };
}
```

**View Pattern:** Guard clauses → Early returns → Main content

## Modal Component

**CRITICAL:** Must use `mounted()` (embedded in JSX), not `state = async`

```typescript
export default class Modal extends Component<State> {
  declare props: Readonly<Props>;
  
  mounted = (props: Props): State => getStateFromProps(props);
  
  view = (state: State) => (
    <div className="modal-backdrop" onclick={closeModal}>
      <div className="modal-content" onclick={(e) => e.stopPropagation()}>
        <button onclick={closeModal}>×</button>
        {/* content */}
      </div>
    </div>
  );
}
```

**Requirements:** Close button + backdrop click + stopPropagation

## Functional Component

```typescript
export interface Props {
  data: DataType[];
  onItemClick?: (item: DataType) => void;
}

export default function DisplayComponent({ data, onItemClick }: Props) {
  if (!data?.length) return <div>No items</div>;
  return (
    <ul>
      {data.map(item => (
        <li onclick={() => onItemClick?.(item)}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**Pattern:** Destructure → Guard clauses → Main render

## Typed Events Pattern

**Payload Rules:**
1. Single value → `payload: string` | Call: `$onclick={['delete', id]}`
2. Multiple values → `payload: { id: string; name: string }` | Call: `$onclick={['edit', { id, name }]}`
3. No payload → `payload: void` | Call: `$onclick="save"`
4. Input events → `payload: { target: { value: string } }`

```typescript
// types/events.ts
export type MyEvents =
  | { name: 'save'; payload: void }
  | { name: 'delete'; payload: string }
  | { name: 'edit'; payload: { id: string; name: string } };

export type MyEventName = MyEvents['name'];

// Component
class MyComponent extends Component<State, MyEventName> {
  override update = myHandlers;
}

// Handlers (OBJECT format, not array)
export const myHandlers: Update<State, MyEventName> = {
  save: (state): State => ({ ...state, saved: true }),
  delete: (state, id: string): State => ({ 
    ...state, 
    items: state.items.filter(i => i.id !== id) 
  }),
  edit: (state, { id, name }: { id: string; name: string }): State => ({ 
    ...state, 
    editing: { id, name } 
  })
};
```

**stopPropagation:** Add event as last parameter

```typescript
'click-item': (state, id: string, e?: Event): State => {
  e?.stopPropagation();
  return { ...state, selected: id };
}
```

## Event Directives

### AppRun Directives (Trigger Update Handlers)

| Directive | Use Case | Example |
|-----------|----------|---------|
| `$bind="field"` | Two-way binding (PREFERRED for forms) | `<input $bind="name" />` |
| `$bind="nested.field"` | Nested property | `<input $bind="user.profile.name" />` |
| `$onclick="action"` | String action | `<button $onclick="save" />` |
| `$onclick={['action', data]}` | Action with params | `<button $onclick={['delete', id]} />` |
| `$onclick={[func]}` | Direct function | `<button $onclick={[saveData]} />` |
| `$oninput="handler"` | Custom input handling | `<input $oninput="validate" />` |

**Other directives:** `$onchange`, `$onsubmit`, `$onfocus`, `$onblur`, `$onkeydown`

### Standard HTML Events (DOM Manipulation)

Use `onclick`, `oninput`, etc. for direct DOM manipulation only:

```tsx
<div onclick={(e) => e.stopPropagation()}>Content</div>
```

### When to Use What

- ✅ **`$bind`** - Simple form fields (no handler needed)
- ✅ **`$oninput`** - Validation, transformation, debouncing
- ✅ **`$onclick`** - Trigger update handlers
- ❌ **Never** - `$onclick={() => app.run('action')}`

**Validation Example:**

```typescript
$oninput="validate-email"

'validate-email': (state, e: Event) => {
  const email = (e.target as HTMLInputElement).value;
  const valid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  return { ...state, email, emailError: valid ? null : 'Invalid' };
}
```

## Update Handlers

**Sync:** Return new state

```typescript
'increment': (state) => ({ ...state, count: state.count + 1 })
```

**Async:** Use `async`

```typescript
'load': async (state) => {
  this.setState({ ...state, loading: true });
  const data = await api.fetch();
  return { ...state, data, loading: false };
}
```

**Generator:** Multi-step with intermediate renders (PREFERRED for complex flows)

```typescript
'save': async function* (state) {
  yield { ...state, loading: true };
  await api.save(state.data);
  yield { ...state, loading: false, success: true };
}
```

**Side Effects:** No return = no re-render

```typescript
'navigate': (state) => {
  window.location.href = '/path';
  // No return - no re-render
}
```

## Component Communication

| Pattern | Use Case | Implementation |
|---------|----------|----------------|
| Props | Parent → Child | Pass data via props |
| Callbacks | Child → Parent | Pass function via props |
| Global Events | Any → Any | `is_global_event = () => true` |

**Global Events:**

```typescript
// Modal component
class Modal extends Component {
  is_global_event = () => true;
  
  update = {
    'open-modal': (state, data) => ({ ...state, visible: true, data }),
    'close-modal': (state) => ({ ...state, visible: false })
  };
}

// Any component can trigger
<button onclick={() => app.run('open-modal', data)}>Open</button>
```

## Critical Rules

### State Initialization

| Component Type | Use | Example |
|----------------|-----|---------|
| JSX Embedded | `mounted()` | `mounted = (props) => getStateFromProps(props)` |
| Top-Level Routed | `state = async` | `state = async () => { const data = await api.fetch(); return { data }; }` |

**❌ NEVER mix both** `mounted()` and `state = async`

### State Updates

**Returning state triggers re-render:**
- Immutable (recommended): `return { ...state, field: value }`
- Mutable (allowed): `state.field = value; return state`
- Side effects only: Don't return (no re-render)

### Required State Properties

```typescript
interface State {
  loading: boolean;        // For async operations
  error: string | null;    // For error messages
  successMessage?: string; // For success feedback
  // ... specific fields
}
```

### Deep Cloning

```typescript
// Nested object update
return { 
  ...state, 
  user: { 
    ...state.user, 
    profile: { 
      ...state.user.profile, 
      name 
    } 
  } 
};
```

## Anti-Patterns

**❌ DON'T:**

```typescript
// Don't use $onclick with arrow functions calling app.run
$onclick={() => app.run('action')}

// Don't forget error handling in async
async function save() { await api.save(); }  // No try-catch!

// Don't use manual input when $bind available
$oninput={(e) => setState({ ...state, field: e.target.value })}

// Don't use state = async for JSX embedded components
class Modal extends Component {
  state = async () => { /* WRONG */ };
}

// Don't forget defensive programming
messages.map()  // messages might be undefined - use messages?.map()

// Don't use array format for update handlers
update = [['event', handler]]  // WRONG - use object format

// Don't mutate state directly
state.count++;  // WRONG
```

## Routing, Linking, and Component Registration

This section explains how AppRun applications handle routing, page navigation, and component registration.

### Overview

The app uses AppRun's built-in routing system without any external router libraries. Routes are defined declaratively, and navigation uses standard HTML anchor tags or programmatic methods.

### 1. Component Registration

Routes are registered centrally in `main.tsx`:

```tsx
import app from 'apprun';
import Layout from './components/Layout';
import Home from './pages/Home';
import World from './pages/World';

app.render('#root', <Layout />);
app.addComponents('#pages', {
  '/': Home,
  '/World': World,
  // '/Agent': Agent,      // commented out
  // '/Settings': Settings, // commented out
});
```

**How It Works:**

- **`app.render('#root', <Layout />)`**: Renders the top-level Layout component into the `#root` DOM element
- **`app.addComponents('#pages', {...})`**: Registers route-to-component mappings
  - Key: Route path (e.g., `'/'`, `'/World'`)
  - Value: Component class (e.g., `Home`, `World`)
  - Components are rendered into the `#pages` container defined in Layout

### 2. Layout Container

The Layout component provides the rendering container for routed pages:

```tsx
// web/src/components/Layout.tsx
export default () => <div id="main" className="w-full min-h-screen">
  <div id="pages"></div>
</div>
```

- Minimal wrapper with full-width, full-height container
- The `#pages` div is where route components are dynamically rendered
- AppRun automatically swaps components based on the current route

### 3. Page Linking (Declarative Navigation)

The app uses **standard HTML anchor tags** for navigation:

**Example from Home Component:**

```tsx
// Navigate to a specific world
<a href={'/World/' + worldName}>
  <button className="btn btn-primary">
    Enter {worldName}
  </button>
</a>
```

**Example from World Component:**

```tsx
// Navigate back to home
<a href="/">
  <button className="back-button" title="Back to Worlds">
    <span className="world-back-icon">←</span>
  </button>
</a>
```

**How It Works:**

- Standard `<a href="">` links trigger AppRun's routing
- AppRun intercepts link clicks and updates the route without full page reload
- Route parameters (like world name) are included in the URL path
- No special Link component required—just plain HTML

### 4. Programmatic Navigation

Components can navigate programmatically using `window.location.href`:

**Example from Home Component Update Handler:**

```tsx
update = {
  'enter-world': (state: HomeState, world: World): void => {
    // Navigate to the world page
    window.location.href = '/World/' + world.name;
  }
}
```

**When to Use:**

- Inside event handlers that need to navigate after logic
- When navigation is a side effect (return `void` instead of new state)
- For conditional navigation based on user actions

### 5. Route Parameters

Routes can include dynamic parameters in the path:

**URL Pattern:**

```
/World/:worldName
```

**Parsing Parameters:**

Components can access route parameters from the URL:

```tsx
// Example: /World/MyWorld
const worldName = window.location.pathname.split('/')[2];  // "MyWorld"
```

**Route Handler Pattern:**

```tsx
update = {
  '/World': async (state, worldName: string) => {
    // worldName is parsed from the URL
    return {
      ...state,
      worldName,
      // ... load world data
    };
  }
}
```

### 6. Component Architecture (MVU Pattern)

Page components follow AppRun's **Model-View-Update** pattern:

```tsx
export default class PageComponent extends Component<StateType> {
  
  // 1. STATE: Initial data and loading states
  state = {
    loading: true,
    data: null,
    // ...
  };

  // 2. VIEW: Render function that returns JSX
  view = (state: StateType) => {
    return <div>
      {/* JSX markup */}
    </div>;
  };

  // 3. UPDATE: Event handlers
  update = {
    'event-name': (state, payload) => {
      // Return new state to trigger re-render
      return { ...state, newData: payload };
    },
    
    'navigation-event': (state) => {
      // Return void for side effects (no re-render)
      window.location.href = '/path';
    }
  };
}
```

**Key Principles:**

- **State**: Plain object with component data
- **View**: Pure function that converts state to JSX
- **Update**: Event handlers that return new state or void
- **Immutability**: Always return new state objects, never mutate

### 7. Event System

**Local vs Global Events:**

Components can be configured to listen to global events:

```tsx
export default class WorldComponent extends Component {
  // Make all events global (visible across components)
  override is_global_event = () => true;
}
```

**Event Propagation:**

- **Local events**: Only visible within the component
- **Global events**: Can be triggered from child components or other parts of the app
- Use `app.run('event-name', payload)` to trigger events programmatically

**Event Handler Types:**

```tsx
update = {
  // Returns new state → triggers re-render
  'update-data': (state, newData) => ({
    ...state,
    data: newData
  }),
  
  // Returns void → no re-render (side effect only)
  'navigate': (state) => {
    window.location.href = '/path';
  }
}
```

### 8. Best Practices

**Navigation:**

- ✅ Use `<a href="">` for simple links
- ✅ Use `window.location.href` for programmatic navigation
- ✅ Include route parameters in the path: `/World/${name}`
- ❌ Don't use client-side routing for external URLs

**Component Registration:**

- ✅ Register all routes in a single place (`main.tsx`)
- ✅ Use clear, semantic route paths
- ✅ Keep the route structure flat and simple
- ❌ Don't nest routes deeply

**Event Handling:**

- ✅ Return new state to trigger re-render
- ✅ Return void for navigation or side effects
- ✅ Use descriptive event names: `'load-world'`, `'delete-chat'`
- ❌ Don't mutate state directly

**URL Structure:**

```
/                      → Home page (world selection)
/World/:name          → World page (chat interface)
/Agent/:id            → Agent page (currently disabled)
/Settings             → Settings page (currently disabled)
```

### 9. Example Flow: Entering a World

**Step 1: User clicks "Enter World" button on Home page**

```tsx
// Home.tsx
<a href={'/World/' + world.name}>
  <button className="btn btn-primary">
    Enter {world.name}
  </button>
</a>
```

**Step 2: AppRun intercepts the link and updates route**

- URL changes to `/World/MyWorld`
- AppRun's router detects the route change
- Router looks up the registered component for `/World`

**Step 3: World component is mounted and initialized**

```tsx
// World.tsx
update = {
  '/World': async (state, worldName: string) => {
    // Load world data from API
    const world = await api.getWorld(worldName);
    const messages = await api.getMessages(worldName);
    
    return {
      ...state,
      worldName,
      world,
      messages,
      loading: false
    };
  }
}
```

**Step 4: World component renders with loaded data**

- View function receives the updated state
- Chat interface displays with agents and messages
- Component is now interactive and listening for events

### 10. Debugging Tips

**Check Current Route:**

```tsx
console.log(window.location.pathname);  // "/World/MyWorld"
```

**Monitor Route Changes:**

```tsx
app.on('//', (route) => {
  console.log('Route changed to:', route);
});
```

**Verify Component Registration:**

```tsx
// Check if component is registered for a route
// Look for the component rendering in #pages container
console.log(document.querySelector('#pages').innerHTML);
```

### Summary

- **Registration**: `app.addComponents('#pages', { path: Component })`
- **Navigation**: Use `<a href="">` or `window.location.href`
- **Route Params**: Parsed from URL path in route handlers
- **Component Pattern**: MVU (Model-View-Update)
- **Events**: Local by default, can be made global with `is_global_event()`
- **No Router Library**: AppRun's built-in routing handles everything

## Testing (Vitest)

- Unit test pure update functions.
- Iterate async generators to capture each yield.
- Mock APIs with `vi.mock`.

```typescript
import { describe, it, expect, vi } from 'vitest';
import { save } from './Form';
import api from '../api';

vi.mock('../api');

describe('save', () => {
  it('yields validation then stops', async () => {
    const state = { loading: false, error: null, form: { name: '' } } as State;
    const gen = save(state);
    const first = await gen.next();
    expect(first.value?.error).toBe('Name is required');
  });
});
```

## Development Checklist

### Component Structure
- [ ] Imports at top
- [ ] Props interface with `?` for optional
- [ ] State interface (exported)
- [ ] Helper functions
- [ ] Action functions (exported for `$onclick` and testing)
- [ ] Component class with `mounted` or `state = async`

### TypeScript Types
- [ ] Props interface
- [ ] State interface exported
- [ ] Event types for 10+ events (discriminated union)
- [ ] Generic types: `Component<State, EventName>`
- [ ] Async generators: `AsyncGenerator<State>`

### View Method
- [ ] Guard clauses first (loading, error, success)
- [ ] Early returns for special states
- [ ] Main content last
- [ ] Defensive programming (`data?.map()`, defaults)

### State Management
- [ ] Include `loading`, `error`, `successMessage?`
- [ ] Return new state to re-render
- [ ] Use `mounted()` for JSX embedded
- [ ] Use `state = async` only for routed pages
- [ ] Never mix both

### Event Handling
- [ ] Use `$bind` for simple forms
- [ ] Use `$onclick` (not `onclick={() => app.run()}`)
- [ ] Export action functions for reusability
- [ ] Use async generators for multi-step
- [ ] Add try-catch in async functions

### Error Handling
- [ ] Try-catch in async operations
- [ ] Error state in interface
- [ ] Error display in view
- [ ] Loading states during async
- [ ] Success messages

### Best Practices
- [ ] Keep update logic pure when possible
- [ ] Use global events for cross-component
- [ ] Add catch-all route for 404
- [ ] Test update logic and error paths
- [ ] Use descriptive event names

## Quick Reference

**Component Selection:**
- State + interactions → Stateful Class
- Modal/popup → Modal Component (`mounted()`)
- Display only → Functional
- 10+ events → Typed Events

**State Init:**
- JSX embedded → `mounted()`
- Routed page → `state = async`

**Events:**
- `$bind` for forms (preferred)
- `$onclick` for actions
- Typed for large components

**Updates:**
- Return state → re-render
- No return → side effect
- Generators → multi-step

**Communication:**
- Props: parent → child
- Callbacks: child → parent
- Global: any → any
