---
name: fullstory-component-wellbeing
version: v1
description: Expert guidance for monitoring frontend component health, performance, and rendering stability within Fullstory. Framework-agnostic patterns for React, Vue, Angular, Svelte, and React Native.
related_skills:
  - fullstory-stable-selectors
  - fullstory-logging
  - fullstory-privacy-strategy
---

# Fullstory Component Wellbeing & Performance

💡 **STRATEGY**: Application "wellbeing" is defined by the absence of frustration signals (Rage Clicks, Dead Clicks) and the presence of stable, performant component lifecycles. This skill bridges your framework's internal health with Fullstory's external observability.

## Overview

This skill provides patterns to expose framework-specific performance bottlenecks and application errors to Fullstory. The goal is to ensure that when a "Rage Click" occurs, developers can see the exact component state and any underlying "silent" failures—regardless of whether you're using React, Vue, Angular, Svelte, or React Native.

## Core Concepts

### 1. Universal Semantic Attributes

All frameworks render to DOM (or native views). Use `data-*` attributes as your stable contract with Fullstory:

| Attribute | Purpose | Works In |
|-----------|---------|----------|
| `data-component` | Stable anchor for heatmaps and search | All frameworks |
| `data-state` | Tracks `loading`, `error`, `ready`, `stale` | All frameworks |
| `data-testid` | Alternative stable selector | All frameworks |

### 2. Error Boundary Pattern (Framework-Specific)

Every framework has a way to catch rendering/component errors. The pattern is the same: catch the error, extract context, send to Fullstory.

### 3. Render Performance Thresholds

Instead of tracking every render (noise), track when renders exceed acceptable thresholds using the Performance API.

```
┌──────────────────────────────────────────────────────────────┐
│ Component Tree (Any Framework)                                │
│ ├─ <App>                                                      │
│ │  └─ <ErrorBoundary> ──► Reports crash to FS                │
│ │     └─ <Dashboard>                                          │
│ │        └─ <Chart data-state="slow" data-component="Chart">  │
└──────────────────────────────────────────────────────────────┘
```

## API Reference: Semantic Wellbeing

| Attribute / Method | Target | Purpose |
|-------------------|--------|---------|
| `data-component` | Any Component | Creates a stable anchor for heatmaps and performance search |
| `data-state` | Dynamic Containers | Tracks states like `loading`, `error`, `ready`, or `stale` |
| `FS('setVars')` | Error Handlers | Passes component context to Fullstory for debugging |
| `FS('event')` | Error Handlers | Logs framework errors for proactive alerting |

---

## ✅ GOOD Implementation Examples

### Example 1: Global Error Handler

#### Vanilla JavaScript

```javascript
// Works without any framework - catches all uncaught errors
window.addEventListener('error', (event) => {
  FS('setVars', {
    scope: 'page',
    vars: {
      last_error_source: event.filename || 'unknown',
      app_wellbeing_status: 'degraded'
    }
  });
  FS('event', 'Uncaught Error', {
    framework: 'vanilla',
    error_message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno
  });
});

// Also catch unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  FS('setVars', {
    scope: 'page',
    vars: {
      app_wellbeing_status: 'degraded'
    }
  });
  FS('event', 'Unhandled Promise Rejection', {
    framework: 'vanilla',
    error_message: event.reason?.message || String(event.reason)
  });
});
```

#### React (Class or Function Components)

```jsx
class FullstoryErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    FS('setVars', {
      scope: 'page',
      vars: {
        last_error_component: info.componentStack?.split("\n")[1]?.trim() || 'unknown',
        app_wellbeing_status: 'degraded'
      }
    });
    FS('event', 'Component Crash', {
      framework: 'react',
      error_message: error.message,
      component_stack: info.componentStack
    });
  }
  render() { return this.props.children; }
}
```

#### Vue 3

```javascript
// main.js or plugin
app.config.errorHandler = (error, instance, info) => {
  const componentName = instance?.$options?.name || instance?.$.type?.name || 'unknown';
  
  FS('setVars', {
    scope: 'page',
    vars: {
      last_error_component: componentName,
      app_wellbeing_status: 'degraded'
    }
  });
  FS('event', 'Component Crash', {
    framework: 'vue',
    error_message: error.message,
    component_name: componentName,
    lifecycle_hook: info
  });
};
```

#### Angular

```typescript
// global-error-handler.ts
@Injectable()
export class FullstoryErrorHandler implements ErrorHandler {
  handleError(error: Error): void {
    const componentName = this.extractComponentName(error);
    
    FS('setVars', {
      scope: 'page',
      vars: {
        last_error_component: componentName,
        app_wellbeing_status: 'degraded'
      }
    });
    FS('event', 'Component Crash', {
      framework: 'angular',
      error_message: error.message,
      stack: error.stack
    });
  }
  
  private extractComponentName(error: any): string {
    // Angular errors often contain component context in the message
    const match = error.message?.match(/at (\w+Component)/);
    return match?.[1] || 'unknown';
  }
}

// app.module.ts
providers: [{ provide: ErrorHandler, useClass: FullstoryErrorHandler }]
```

#### Svelte

```javascript
// hooks.client.js (SvelteKit) or main.js
import { onMount } from 'svelte';

// Global error handler for Svelte apps
window.addEventListener('error', (event) => {
  FS('setVars', {
    scope: 'page',
    vars: {
      last_error_source: event.filename || 'unknown',
      app_wellbeing_status: 'degraded'
    }
  });
  FS('event', 'Component Crash', {
    framework: 'svelte',
    error_message: event.message,
    filename: event.filename
  });
});

// For SvelteKit, use the handleError hook in hooks.client.js
export function handleError({ error, event }) {
  FS('setVars', {
    scope: 'page',
    vars: {
      last_error_route: event.url.pathname,
      app_wellbeing_status: 'degraded'
    }
  });
  FS('event', 'Component Crash', {
    framework: 'svelte',
    meta_framework: 'sveltekit',
    error_message: error.message,
    route: event.url.pathname
  });
}
```

#### React Native

```jsx
// Same as React, plus native crash handling
class FullstoryErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    FS('setVars', {
      scope: 'page',
      vars: {
        last_error_component: info.componentStack?.split("\n")[1]?.trim() || 'unknown',
        app_wellbeing_status: 'degraded'
      }
    });
    FS('event', 'Component Crash', {
      framework: 'react-native',
      error_message: error.message,
      component_stack: info.componentStack
    });
  }
  render() { return this.props.children; }
}
```

---

### Example 2: Hydration/SSR Health

#### React (Next.js / Remix)

```jsx
useEffect(() => {
  const root = document.getElementById('__next') || document.getElementById('root');
  if (root && !root.hasChildNodes()) {
    FS('event', 'Hydration Failure', { 
      framework: 'react',
      meta_framework: 'nextjs',
      path: window.location.pathname
    });
  }
}, []);
```

#### Vue (Nuxt)

```javascript
// plugins/hydration-check.client.js
export default defineNuxtPlugin(() => {
  if (import.meta.client) {
    const root = document.getElementById('__nuxt');
    if (root && root.dataset.serverRendered && !root.hasChildNodes()) {
      FS('event', 'Hydration Failure', {
        framework: 'vue',
        meta_framework: 'nuxt',
        path: window.location.pathname
      });
    }
  }
});
```

#### Angular (Universal)

```typescript
// app.component.ts
@Component({ ... })
export class AppComponent implements AfterViewInit {
  constructor(@Inject(PLATFORM_ID) private platformId: Object) {}
  
  ngAfterViewInit() {
    if (isPlatformBrowser(this.platformId)) {
      const state = document.querySelector('[ng-server-context]');
      if (state && document.body.children.length === 0) {
        FS('event', 'Hydration Failure', {
          framework: 'angular',
          meta_framework: 'universal',
          path: window.location.pathname
        });
      }
    }
  }
}
```

#### Svelte (SvelteKit)

```javascript
// +layout.svelte or +page.svelte
import { onMount } from 'svelte';
import { browser } from '$app/environment';

onMount(() => {
  if (browser) {
    const root = document.getElementById('svelte');
    // Check if hydration produced expected content
    if (root && root.children.length === 0) {
      FS('event', 'Hydration Failure', {
        framework: 'svelte',
        meta_framework: 'sveltekit',
        path: window.location.pathname
      });
    }
  }
});
```

---

### Example 3: Semantic State Attributes (Universal)

Works identically across all frameworks—just different template syntax:

#### Vanilla JavaScript

```javascript
// Using DOM manipulation
const container = document.querySelector('#search-results');
container.setAttribute('data-component', 'SearchResults');

function updateState(isSearching, results) {
  const state = isSearching ? 'loading' : results.length === 0 ? 'empty' : 'ready';
  container.setAttribute('data-state', state);
}
```

#### React / React Native

```jsx
const SearchResults = ({ results, isSearching }) => (
  <div 
    data-component="SearchResults" 
    data-state={isSearching ? 'loading' : results.length === 0 ? 'empty' : 'ready'}
  >
    {/* content */}
  </div>
);
```

#### Vue

```vue
<template>
  <div 
    data-component="SearchResults" 
    :data-state="computedState"
  >
    <!-- content -->
  </div>
</template>

<script setup>
const computedState = computed(() => 
  isSearching.value ? 'loading' : results.value.length === 0 ? 'empty' : 'ready'
);
</script>
```

#### Angular

```html
<div 
  data-component="SearchResults" 
  [attr.data-state]="isSearching ? 'loading' : results.length === 0 ? 'empty' : 'ready'"
>
  <!-- content -->
</div>
```

#### Svelte

```svelte
<div 
  data-component="SearchResults" 
  data-state={isSearching ? 'loading' : results.length === 0 ? 'empty' : 'ready'}
>
  <!-- content -->
</div>
```

---

### Example 4: Long-Task Render Warning (Universal)

The Performance API works in all browser-based frameworks:

#### Vanilla JavaScript

```javascript
// Generic performance wrapper for any operation
function trackPerformance(operationName, fn) {
  const start = performance.now();
  const result = fn();
  const duration = performance.now() - start;
  
  if (duration > 50) {
    FS('event', 'Long Operation Detected', { 
      operationName, 
      duration, 
      framework: 'vanilla' 
    });
  }
  return result;
}

// Usage
trackPerformance('renderTable', () => {
  // DOM manipulation here
});
```

#### React

```jsx
function usePerformanceTag(componentName) {
  useEffect(() => {
    const start = performance.now();
    return () => {
      const duration = performance.now() - start;
      if (duration > 50) {
        FS('event', 'Long Render Detected', { componentName, duration, framework: 'react' });
      }
    };
  });
}
```

#### Vue

```javascript
// composables/usePerformanceTag.js
export function usePerformanceTag(componentName) {
  let start;
  
  onMounted(() => { start = performance.now(); });
  
  onUpdated(() => {
    const duration = performance.now() - start;
    if (duration > 50) {
      FS('event', 'Long Render Detected', { componentName, duration, framework: 'vue' });
    }
    start = performance.now();
  });
}
```

#### Angular

```typescript
@Directive({ selector: '[fsPerformanceTag]' })
export class PerformanceTagDirective implements AfterViewInit, AfterViewChecked {
  @Input('fsPerformanceTag') componentName: string;
  private start: number;
  
  ngAfterViewInit() { this.start = performance.now(); }
  
  ngAfterViewChecked() {
    const duration = performance.now() - this.start;
    if (duration > 50) {
      FS('event', 'Long Render Detected', { 
        componentName: this.componentName, 
        duration, 
        framework: 'angular' 
      });
    }
    this.start = performance.now();
  }
}
```

#### Svelte

```svelte
<script>
  import { onMount, afterUpdate } from 'svelte';
  
  export let componentName;
  let start;
  
  onMount(() => { start = performance.now(); });
  
  afterUpdate(() => {
    const duration = performance.now() - start;
    if (duration > 50) {
      FS('event', 'Long Render Detected', { componentName, duration, framework: 'svelte' });
    }
    start = performance.now();
  });
</script>
```

---

### Example 5: Portal/Overlay Visibility

#### React

```jsx
const Modal = ({ isOpen, children }) => isOpen ? ReactDOM.createPortal(
  <div data-component="GlobalModal" data-state="visible">{children}</div>,
  document.body
) : null;
```

#### Vue

```vue
<Teleport to="body" v-if="isOpen">
  <div data-component="GlobalModal" data-state="visible">
    <slot />
  </div>
</Teleport>
```

#### Angular

```typescript
// Using CDK Portal
@Component({
  template: `
    <ng-template cdkPortal>
      <div data-component="GlobalModal" data-state="visible">
        <ng-content></ng-content>
      </div>
    </ng-template>
  `
})
export class ModalComponent { }
```

#### Svelte

```svelte
<script>
  import { portal } from 'svelte-portal';
  export let isOpen = false;
</script>

{#if isOpen}
  <div use:portal={'body'} data-component="GlobalModal" data-state="visible">
    <slot />
  </div>
{/if}
```

#### React Native

```jsx
// React Native uses Modal component, not portals
<Modal visible={isOpen} data-component="GlobalModal" data-state="visible">
  {children}
</Modal>
```

---

### Example 6: Decoupled API Health (Universal)

Tracking when a component is waiting for data vs. a failure:

#### Vanilla JavaScript

```javascript
const container = document.querySelector('#user-profile');
container.setAttribute('data-component', 'UserProfile');

async function loadProfile(userId) {
  container.setAttribute('data-state', 'loading');
  try {
    const user = await fetchUser(userId);
    container.setAttribute('data-state', 'ready');
    renderProfile(user);
  } catch (error) {
    container.setAttribute('data-state', 'error');
    renderError(error.message);
  }
}
```

#### React

```jsx
const Profile = ({ user, error }) => (
  <div 
    data-component="UserProfile" 
    data-state={error ? 'error' : user ? 'ready' : 'loading'}
  >
    {error ? <ErrorMessage message={error} /> : <UserDetail user={user} />}
  </div>
);
```

#### Vue

```vue
<template>
  <div 
    data-component="UserProfile" 
    :data-state="error ? 'error' : user ? 'ready' : 'loading'"
  >
    <ErrorMessage v-if="error" :message="error" />
    <UserDetail v-else :user="user" />
  </div>
</template>
```

#### Angular

```html
<div 
  data-component="UserProfile" 
  [attr.data-state]="error ? 'error' : user ? 'ready' : 'loading'"
>
  <app-error-message *ngIf="error" [message]="error"></app-error-message>
  <app-user-detail *ngIf="!error" [user]="user"></app-user-detail>
</div>
```

#### Svelte

```svelte
<div 
  data-component="UserProfile" 
  data-state={error ? 'error' : user ? 'ready' : 'loading'}
>
  {#if error}
    <ErrorMessage message={error} />
  {:else}
    <UserDetail {user} />
  {/if}
</div>
```

---

## ❌ BAD Implementation Examples

### Example 1: Dumping Raw State (All Frameworks)

**Why it's bad:** Creates massive noise and risks leaking PII.

```javascript
// BAD: Do not dump entire state object into Fullstory

// Vanilla JS
const state = getAppState();
FS('setVars', { scope: 'page', vars: { ...state } });

// React
const { state } = useMyState();
useEffect(() => {
  FS('setVars', { scope: 'page', vars: { ...state } });
}, [state]);

// Vue
watch(state, (newState) => {
  FS('setVars', { scope: 'page', vars: { ...newState } });
});

// Angular
this.store.subscribe(state => {
  FS('setVars', { scope: 'page', vars: { ...state } });
});

// Svelte
$: FS('setVars', { scope: 'page', vars: { ...$store } });
```

### Example 2: Unstable Selectors (All Frameworks)

**Why it's bad:** Framework-generated class names change every build, breaking Fullstory Search.

```javascript
// BAD: These selectors are brittle
// Vanilla (CSS-in-JS): <div class="css-abc123">
// React (styled-components): <div class="sc-ax123">
// Vue (scoped): <div class="data-v-abc123">
// Angular (ViewEncapsulation): <div class="_ngcontent-abc-123">
// Svelte (scoped): <div class="svelte-xyz789">

// GOOD: Use data-component for stable identification
<div data-component="SearchResults">...</div>
```

### Example 3: Redundant Click Events (All Frameworks)

**Why it's bad:** Fullstory autocaptures clicks; manual events create redundant data.

```jsx
// BAD: Manual event tracking for basic interactions

// Vanilla
button.addEventListener('click', () => {
  FS('event', 'Login Button Clicked'); // Redundant!
  handleLogin();
});

// React
<button onClick={() => {
  FS('event', 'Login Button Clicked'); // Redundant! FS already knows this.
  handleLogin();
}}>Login</button>

// Vue
<button @click="FS('event', 'Login Button Clicked'); handleLogin()">Login</button>

// Angular
<button (click)="logClick(); handleLogin()">Login</button>

// Svelte
<button on:click={() => { FS('event', 'Login Button Clicked'); handleLogin(); }}>Login</button>
```

### Example 4: Prop-Drilling FS Instance (React/Vue/Svelte)

**Why it's bad:** The `FS` global is already available. Prop-drilling makes components harder to test and maintain.

```jsx
// BAD: Unnecessary prop complexity
const DeepChild = ({ fsInstance }) => (
  <button onClick={() => fsInstance('event', 'child_click')}>Submit</button>
);
```

### Example 5: High-Frequency Updates (All Frameworks)

**Why it's bad:** Will hit FS rate limits or crash the browser tab.

```javascript
// BAD: Flooding the API with high-frequency updates
window.addEventListener('mousemove', (e) => {
  FS('setVars', { scope: 'page', vars: { mouseX: e.clientX } });
});

// React
const [pos, setPos] = useState({ x: 0, y: 0 });
useEffect(() => {
  FS('setVars', { scope: 'page', vars: { mouse_x: pos.x } });
}, [pos]);

// Svelte
$: FS('setVars', { scope: 'page', vars: { mouse_x: mouseX } }); // Reactive on every change!
```

### Example 6: Over-Masking Root (All Frameworks)

**Why it's bad:** Masking the root element makes session replay useless for diagnosing wellbeing.

```html
<!-- BAD: Don't mask the root! Use targeted masking instead. -->

<!-- Vanilla -->
<div id="app" class="fs-mask">...</div>

<!-- React -->
<div id="root" class="fs-mask">...</div>

<!-- Vue -->
<div id="app" class="fs-mask">...</div>

<!-- Angular -->
<app-root class="fs-mask">...</app-root>

<!-- Svelte -->
<div id="svelte" class="fs-mask">...</div>
```

---

## Key Takeaways for Agent

1. **Prioritize Error Boundaries/Handlers**: Always recommend framework-appropriate error catching that calls `FS('setVars')` and `FS('event')`.

2. **Stable Selectors for Performance**: Use `data-component` attributes. Fullstory can then aggregate "Time to Interactive" for specific components.

3. **Monitor "Loading" States**: Use `data-state="loading"`. This allows searching for sessions where users waited for content for more than N seconds.

4. **Avoid Redundancy**: Don't manually log clicks or navigation unless adding specific metadata that autocapture misses.

5. **Hydration Awareness**: If using SSR frameworks (Next.js, Nuxt, SvelteKit, Angular Universal), log hydration failures as events to detect "broken" entry points.

6. **Performance API is Universal**: `performance.now()` works in all browser environments for measuring render times.

```javascript
// Quick Agent logic check:
if (isFrontendApp && isPerformanceIssue) {
  recommend("Stable Selectors (data-component) on suspected bottlenecks");
  recommend("Global error handler with framework-specific API");
  recommend("data-state='loading' for measuring async delays");
}
```

---

## Framework Error Handler Quick Reference

| Framework | Error Handler | Component Context |
|-----------|---------------|-------------------|
| Vanilla JS | `window.addEventListener('error')` | `event.filename` |
| React | `componentDidCatch(error, info)` | `info.componentStack` |
| Vue 3 | `app.config.errorHandler` | `instance.$options.name` |
| Angular | `ErrorHandler.handleError()` | Parse from error message |
| Svelte | `handleError` hook (SvelteKit) | `event.url.pathname` |
| React Native | `componentDidCatch(error, info)` | `info.componentStack` |

---

## Fullstory Search Examples

### Find Sessions with Component Crashes (Any Framework)

```
Event where name = "Component Crash"
```

### Filter by Framework

```
Event where name = "Component Crash" and framework = "vue"
```

```
Event where name = "Component Crash" and framework = "svelte"
```

### Find Sessions with Degraded Wellbeing

```
Page where app_wellbeing_status = "degraded"
```

### Find Components Stuck in Loading State

```
Clicked on element where data-state = "loading"
```

### Find Long Renders

```
Event where name = "Long Render Detected" and duration > 100
```

### Find Hydration Failures

```
Event where name = "Hydration Failure"
```

### Find Hydration Failures by Meta-Framework

```
Event where name = "Hydration Failure" and meta_framework = "nuxt"
```

```
Event where name = "Hydration Failure" and meta_framework = "sveltekit"
```
