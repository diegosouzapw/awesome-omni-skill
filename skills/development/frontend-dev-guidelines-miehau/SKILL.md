---
name: frontend-dev-guidelines
description: Frontend development guidelines for React/TypeScript applications. Modern patterns including Suspense, lazy loading, useSuspenseQuery, file organization with features directory, MUI v7 styling, TanStack Router, performance optimization, and TypeScript best practices. Use when creating components, pages, features, fetching data, styling, routing, or working with frontend code.
---

# Frontend Development Guidelines

## Purpose

Comprehensive guide for modern Svelte 5 development, emphasizing runes-based reactivity, proper file organization, SvelteKit integration, and performance optimization.

## When to Use This Skill

- Creating new Svelte components or pages
- Building new features with Svelte 5 runes
- Setting up routing with SvelteKit
- Managing state with stores and runes
- Styling components with Tailwind CSS
- Performance optimization
- Organizing frontend code
- TypeScript best practices in Svelte

---

## Quick Start

### New Component Checklist

Creating a component? Follow this checklist:

- [ ] Use `<script lang="ts">` for TypeScript support
- [ ] Define props with `let { prop1, prop2 }: Props = $props()`
- [ ] Use `$state()` rune for reactive state
- [ ] Use `$derived()` for computed values
- [ ] Use `$effect()` for side effects
- [ ] Import aliases: `$lib/`, `$app/`, custom aliases
- [ ] Export component props interface
- [ ] Use `bind:` for two-way data binding when needed
- [ ] Implement proper TypeScript types
- [ ] Use slots/snippets for content projection

### New Feature Checklist

Creating a feature? Set up this structure:

- [ ] Create `src/lib/{feature-name}/` directory
- [ ] Create subdirectories: `stores/`, `components/`, `utils/`, `types/`
- [ ] Create store file: `stores/{feature}.svelte.ts`
- [ ] Set up TypeScript types in `types/`
- [ ] Create route in `src/routes/{feature-name}/+page.svelte`
- [ ] Set up load functions in `+page.ts` or `+page.server.ts`
- [ ] Use proper error boundaries with `+error.svelte`
- [ ] Export public API from feature `index.ts`

---

## Import Aliases Quick Reference

| Alias | Resolves To | Example |
|-------|-------------|---------|
| `$lib` | `src/lib` | `import { userStore } from '$lib/stores/user'` |
| `$app` | SvelteKit runtime | `import { page } from '$app/stores'` |
| `$env` | Environment variables | `import { PUBLIC_API_URL } from '$env/static/public'` |
| Custom | Via `vite.config.js` | `import type { User } from '@types/user'` |

These are SvelteKit's built-in aliases. Custom aliases can be defined in `vite.config.js`.

---

## Common Imports Cheatsheet

```typescript
// Svelte imports
import { onMount, onDestroy, setContext, getContext } from 'svelte';
import { writable, readable, derived, get } from 'svelte/store';

// SvelteKit imports
import { page, navigating } from '$app/stores';
import { goto, invalidate, invalidateAll } from '$app/navigation';
import { browser, dev } from '$app/environment';
import type { PageData, PageLoad } from './$types';

// Component imports
import Button from '$lib/components/Button.svelte';
import Modal from '$lib/components/Modal.svelte';

// Store imports
import { userStore } from '$lib/stores/user.svelte.ts';

// Types
import type { User } from '$lib/types/user';

// Environment variables
import { PUBLIC_API_URL } from '$env/static/public';
import { PRIVATE_API_KEY } from '$env/static/private';
```

---

## Topic Guides

### 🎨 Component Patterns

**Modern Svelte 5 components use:**
- Runes (`$state`, `$derived`, `$effect`) for reactivity
- `<script lang="ts">` for TypeScript support
- Props with `$props()` and `$bindable()`
- Snippets for content projection

**Key Concepts:**
- Use `$state()` for reactive state
- Use `$derived()` for computed values
- Use `$effect()` for side effects
- Component structure: Script → Markup → Style

**[📖 Complete Guide: resources/component-patterns.md](resources/component-patterns.md)**

---

### 📊 Data Fetching

**PRIMARY PATTERNS:**
- SvelteKit load functions (`+page.ts`, `+page.server.ts`)
- Native `fetch` with proper error handling
- Stores for client-side state
- Form actions for mutations

**Data Loading:**
- Use `load` functions for SSR/CSR data
- Return data from `+page.server.ts` for server-only
- Use `invalidate()` to refresh data
- Handle errors with `+error.svelte`

**[📖 Complete Guide: resources/data-fetching.md](resources/data-fetching.md)**

---

### 📁 File Organization

**lib/ vs routes/:**
- `src/lib/`: Reusable code (components, stores, utils)
- `src/routes/`: Page routes and layouts

**Library Structure:**
```
src/lib/
  components/     # Reusable components
  stores/         # Global stores
  utils/          # Helper functions
  types/          # TypeScript types
  server/         # Server-only code
```

**[📖 Complete Guide: resources/file-organization.md](resources/file-organization.md)**

---

### 🎨 Styling

**Styling Options:**
- Scoped styles in `<style>` blocks
- Tailwind CSS classes (if configured)
- CSS modules with `.module.css`
- Global styles in `app.css`

**Best Practices:**
- Use scoped styles by default
- CSS custom properties for theming
- Conditional classes with class directive

**Example:**
```svelte
<div class="container" class:active={isActive}>
<style>
  .container { padding: 1rem; }
  .active { background: var(--primary); }
</style>
```

**[📖 Complete Guide: resources/styling-guide.md](resources/styling-guide.md)**

---

### 🛣️ Routing

**SvelteKit File-Based Routing:**
- Routes defined by file structure
- `+page.svelte` for pages
- `+layout.svelte` for layouts
- `+server.ts` for API endpoints

**Route Files:**
```
src/routes/
  +page.svelte         # /
  about/+page.svelte   # /about
  blog/[slug]/
    +page.svelte       # /blog/:slug
    +page.ts           # Load function
```

**Load Functions:**
```typescript
// +page.ts
export const load: PageLoad = async ({ params }) => {
    const post = await fetchPost(params.slug);
    return { post };
};
```

**[📖 Complete Guide: resources/routing-guide.md](resources/routing-guide.md)**

---

### ⏳ Loading & Error States

**Loading States:**
- Use `{#await}` blocks for promises
- Show loading indicators during data fetch
- Streaming SSR with `+page.server.ts`

**Error Handling:**
```svelte
{#await promise}
  <p>Loading...</p>
{:then data}
  <p>{data}</p>
{:catch error}
  <p>Error: {error.message}</p>
{/await}
```

**Error Boundaries:**
- `+error.svelte` for route errors
- `app.html` for app-wide errors
- Form validation with enhance

**[📖 Complete Guide: resources/loading-and-error-states.md](resources/loading-and-error-states.md)**

---

### ⚡ Performance

**Optimization Patterns:**
- `$derived` for expensive computations
- Svelte's built-in reactivity optimization
- Lazy loading with dynamic imports
- Debounced actions with timers
- `onDestroy` for cleanup

**Key Features:**
- Compile-time optimizations
- No virtual DOM overhead
- Fine-grained reactivity
- Automatic batching

**[📖 Complete Guide: resources/performance.md](resources/performance.md)**

---

### 📘 TypeScript

**Standards:**
- Use `lang="ts"` in script tags
- Define props interfaces
- Type imports: `import type { User } from '$lib/types'`
- Generic components with `generics` attribute

**Component Types:**
```typescript
<script lang="ts">
  interface Props {
    name: string;
    age?: number;
  }
  let { name, age = 0 }: Props = $props();
</script>
```

**[📖 Complete Guide: resources/typescript-standards.md](resources/typescript-standards.md)**

---

### 🔧 Common Patterns

**Covered Topics:**
- Form handling with `use:enhance`
- Store patterns and subscriptions
- Two-way binding with `bind:`
- Context API usage
- Component composition with slots

**[📖 Complete Guide: resources/common-patterns.md](resources/common-patterns.md)**

---

### 📚 Complete Examples

**Full working examples:**
- Modern component with all patterns
- Complete feature structure
- API service layer
- Route with lazy loading
- Suspense + useSuspenseQuery
- Form with validation

**[📖 Complete Guide: resources/complete-examples.md](resources/complete-examples.md)**

---

## Navigation Guide

| Need to... | Read this resource |
|------------|-------------------|
| Create a component | [component-patterns.md](resources/component-patterns.md) |
| Fetch data | [data-fetching.md](resources/data-fetching.md) |
| Organize files/folders | [file-organization.md](resources/file-organization.md) |
| Style components | [styling-guide.md](resources/styling-guide.md) |
| Set up routing | [routing-guide.md](resources/routing-guide.md) |
| Handle loading/errors | [loading-and-error-states.md](resources/loading-and-error-states.md) |
| Optimize performance | [performance.md](resources/performance.md) |
| TypeScript types | [typescript-standards.md](resources/typescript-standards.md) |
| Forms/Auth/DataGrid | [common-patterns.md](resources/common-patterns.md) |
| See full examples | [complete-examples.md](resources/complete-examples.md) |

---

## Core Principles

1. **Runes for Reactivity**: Use `$state`, `$derived`, `$effect`
2. **File-Based Routing**: SvelteKit's file structure defines routes
3. **Server-First**: Leverage SSR and server-side data loading
4. **Type Safety**: TypeScript in all components
5. **Scoped Styles**: Component styles are scoped by default
6. **Built-in Optimization**: Svelte compiles to optimal JavaScript
7. **Progressive Enhancement**: Works without JavaScript
8. **Form Actions**: Server-side form handling

---

## Quick Reference: File Structure

```
src/
  lib/
    components/
      Button.svelte           # Reusable component
      Modal.svelte           
    stores/
      user.svelte.ts         # Global state
      auth.svelte.ts
    utils/
      api.ts                  # Utility functions
      format.ts
    types/
      index.ts                # TypeScript types
    server/
      db.ts                   # Server-only code

  routes/
    +layout.svelte            # Root layout
    +page.svelte              # Home page
    about/
      +page.svelte            # About page
    blog/
      +page.svelte            # Blog list
      [slug]/
        +page.svelte          # Blog post
        +page.ts              # Load function
    api/
      posts/
        +server.ts            # API endpoint
```

---

## Modern Component Template (Quick Copy)

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import type { User } from '$lib/types';
  
  interface Props {
    id: number;
    onAction?: () => void;
  }
  
  let { id, onAction }: Props = $props();
  
  let user = $state<User | null>(null);
  let loading = $state(true);
  let count = $state(0);
  
  const doubled = $derived(count * 2);
  
  $effect(() => {
    console.log(`Count changed to ${count}`);
  });
  
  onMount(async () => {
    const response = await fetch(`/api/users/${id}`);
    user = await response.json();
    loading = false;
  });
  
  function handleClick() {
    count++;
    onAction?.();
  }
</script>

{#if loading}
  <p>Loading...</p>
{:else if user}
  <div class="container">
    <h2>{user.name}</h2>
    <button onclick={handleClick}>
      Count: {count} (doubled: {doubled})
    </button>
  </div>
{/if}

<style>
  .container {
    padding: 1rem;
    border-radius: 8px;
    background: var(--surface);
  }
</style>
```

For complete examples, see [resources/complete-examples.md](resources/complete-examples.md)

---

## Related Skills

- **error-tracking**: Error tracking with Sentry (applies to frontend too)
- **backend-dev-guidelines**: Backend API patterns that frontend consumes

---

**Skill Status**: Modular structure with progressive loading for optimal context management