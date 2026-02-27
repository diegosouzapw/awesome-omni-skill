---
name: sveltekit-latest
description: Quick-reference for SvelteKit + Svelte 5 development (Feb 2026)
version: 1.0.0
tags: [svelte, sveltekit, frontend, typescript, runes]
---

# SvelteKit + Svelte 5 Quick Reference (February 2026)

## Current Versions

| Package | Version | Notes |
|---------|---------|-------|
| Svelte | ~5.50.x | Svelte 5 is the current stable. Svelte 4 is legacy. |
| SvelteKit | ~2.50.x | SvelteKit 2 is current stable. SvelteKit 1 is legacy. |
| sv (CLI) | latest | Replaces `create-svelte`. Use `npx sv` for all scaffolding. |

## Scaffolding a New Project

```bash
# Current recommended command (npm create svelte@latest is DEPRECATED)
npx sv create my-app
cd my-app
npm install
npm run dev
```

### sv create options

| Flag | Purpose |
|------|---------|
| `--template minimal\|demo\|library` | Project template |
| `--types ts\|jsdoc\|--no-types` | TypeScript config |
| `--add [add-ons...]` | Install add-ons (see below) |
| `--no-add-ons` | Skip interactive add-on prompt |
| `--install npm\|pnpm\|yarn\|bun\|deno` | Package manager |

### Available add-ons

`drizzle`, `eslint`, `better-auth`, `tailwindcss`, `prettier`, `vitest`, `playwright`, `storybook`, `mdsvex`, `paraglide`, `sveltekit-adapter`, `mcp`, `devtools-json`

## TypeScript Setup

TypeScript is built-in. Select "ts" when prompted by `npx sv create`, or pass `--types ts`. No extra configuration needed.

- Use `<script lang="ts">` in `.svelte` files
- Types are auto-generated on `npm run dev` or `npx svelte-kit sync`
- Required tsconfig options (set automatically): `verbatimModuleSyntax: true`, `isolatedModules: true`, `noEmit: true`
- `.svelte.ts` files for reactive modules (replaces `.ts` when you need runes)

## Svelte 5 Runes (Core Reactivity)

Runes replace Svelte 4's implicit reactivity (`let`, `$:`) with explicit, portable primitives.

### $state -- Reactive State

```svelte
<script>
  let count = $state(0);           // primitive
  let todos = $state([]);           // deep reactive proxy (arrays/objects)
</script>
<button onclick={() => count++}>{count}</button>
```

- Deeply reactive: mutations to nested objects/arrays are tracked automatically
- Class fields: `done = $state(false);` inside class bodies

#### $state.raw -- Non-reactive (reassign-only)

```js
let items = $state.raw([1, 2, 3]);
// items.push(4)  -- NO EFFECT (not reactive)
items = [...items, 4]; // works (reassignment)
```

Use for large collections you never mutate in place. Better performance.

#### $state.snapshot -- Static copy

```js
let data = $state({ count: 0 });
console.log($state.snapshot(data)); // plain object, no proxy
```

Use when passing state to external libraries or `structuredClone`.

### $derived -- Computed Values

```js
let count = $state(0);
const doubled = $derived(count * 2);
```

#### $derived.by -- Complex derivations

```js
const filtered = $derived.by(() => {
  return items.filter(item => item.active);
});
```

`$derived(expr)` is equivalent to `$derived.by(() => expr)`.

### $effect -- Side Effects

```js
$effect(() => {
  // Runs on mount and whenever dependencies change
  // Dependencies are auto-tracked
  console.log(`count is ${count}`);

  // Optional cleanup (returned function runs before re-execution)
  return () => { /* cleanup */ };
});
```

#### $effect.pre -- Before DOM update

```js
$effect.pre(() => {
  // Runs before DOM is updated. Rare use case.
});
```

**Rule of thumb**: `$derived` for values, `$effect` for actions/side effects.

### $props -- Component Props

```svelte
<script>
  // Replaces "export let"
  let { name, count = 0, class: klass, ...rest } = $props();
</script>
```

- Destructure with defaults, renaming, and rest props
- Props are NOT bindable by default (see $bindable)

### $bindable -- Two-way Binding

```svelte
<script>
  let { value = $bindable('default') } = $props();
</script>
```

Parent can then use `bind:value={something}`.

### $inspect -- Debug

```js
$inspect(count); // logs to console whenever count changes (dev only, stripped in prod)
```

## Snippets (Replace Slots)

Slots are deprecated. Use snippets and `{@render}` instead.

### Default content (children)

```svelte
<!-- Parent -->
<Card>
  <p>This becomes the children snippet</p>
</Card>

<!-- Card.svelte -->
<script>
  let { children } = $props();
</script>
<div class="card">
  {@render children?.()}
</div>
```

### Named snippets

```svelte
<!-- Parent -->
<Card>
  {#snippet header()}
    <h2>Title</h2>
  {/snippet}
  {#snippet footer()}
    <p>Footer</p>
  {/snippet}
  <p>Default children content</p>
</Card>

<!-- Card.svelte -->
<script>
  let { header, footer, children } = $props();
</script>
<div class="card">
  {@render header?.()}
  {@render children?.()}
  {@render footer?.()}
</div>
```

### Snippets with parameters

```svelte
<!-- Parent -->
<List items={users}>
  {#snippet item(user)}
    <span>{user.name}</span>
  {/snippet}
</List>

<!-- List.svelte -->
<script>
  let { items, item } = $props();
</script>
{#each items as entry}
  {@render item(entry)}
{/each}
```

## Event Handling Changes

### DOM events: remove the colon

```svelte
<!-- Svelte 4 -->
<button on:click={handler}>Click</button>
<input on:input={handler} />

<!-- Svelte 5 -->
<button onclick={handler}>Click</button>
<input oninput={handler} />
```

### Component events: callback props replace createEventDispatcher

```svelte
<!-- Svelte 4 child -->
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
</script>
<button on:click={() => dispatch('submit', data)}>Submit</button>

<!-- Svelte 5 child -->
<script>
  let { onsubmit } = $props();
</script>
<button onclick={() => onsubmit(data)}>Submit</button>
```

### Event modifiers are gone

Replace `on:click|preventDefault` with explicit calls:

```svelte
<button onclick={(e) => { e.preventDefault(); handler(e); }}>Click</button>
```

## Component Mounting (Programmatic)

```js
// Svelte 4
const app = new App({ target: document.getElementById('app') });

// Svelte 5
import { mount, unmount } from 'svelte';
const app = mount(App, { target: document.getElementById('app') });
// later: unmount(app);
```

## Cross-Module Reactive State (.svelte.ts files)

```ts
// counter.svelte.ts
let count = $state(0);

// Cannot export reassignable $state directly. Two patterns:

// Pattern 1: Export object (mutate properties)
export const counter = $state({ count: 0 });
export function increment() { counter.count++; }

// Pattern 2: Export getter functions
export function getCount() { return count; }
export function increment() { count++; }
```

> **SSR caveat:** Module-scoped `$state` is a singleton shared across SSR requests. Always reset at render start. See "SSR Behavior for Module-Scoped State" below.

## SSR Behavior for Module-Scoped State

### What runs during SSR vs CSR

| Context | SSR | CSR |
|---------|-----|-----|
| Top-level `<script>` code | YES | YES |
| `$state` initialization | YES | YES |
| `$derived` / `$derived.by` | Compiled as IIFE (computed once at module init) | Reactive (recomputes on dependency change) |
| `$effect` / `$effect.pre` | NO | YES |
| `onMount` | NO | YES |
| Template expressions | YES | YES |

### The $derived SSR trap

`$derived.by(() => ...)` compiles to `const x = (() => ...)()` during SSR -- an immediately-invoked function that runs once at module initialization, not reactively. If module-scoped `$state` is set after the derived is initialized, the derived will NOT reflect the updated values.

**Broken pattern:**
```ts
// store.svelte.ts
let page = $state('');
const snapshot = $derived.by(() => ({ page })); // Frozen at init during SSR!
function setPage(p) { page = p; }
```

**Working pattern:**
```ts
// store.svelte.ts
let page = $state('');
function buildSnapshot() { return { page }; } // Fresh read every call
export const store = {
  get snapshot() { return buildSnapshot(); }, // Works in both SSR and CSR
  setPage(p) { page = p; },
};
```

During CSR, reading `$state` variables inside a getter called from a template still triggers Svelte 5's fine-grained reactivity tracking.

### SSR baseline pattern for layouts/pages

To populate state during SSR, call setters at the top level of `<script>` blocks (not in `$effect` or `onMount`). Use `$effect` for client-side reactive updates:

```svelte
<script>
  import { page } from '$app/stores';
  import { myStore } from '$lib/stores/my.svelte';

  let { data } = $props();

  // SSR + CSR: runs on every render
  myStore.setPage(data.title);

  // CSR only: reactive updates on navigation
  $effect(() => {
    myStore.setPage(data.title);
  });
</script>
```

### Avoiding state_referenced_locally warnings

When you need `$derived` values for SSR baseline code, DON'T read the `$derived` at top level. Instead, compute the value directly from the source:

```svelte
<!-- BAD: warns because $derived is read outside reactive context -->
<script>
  let isAdmin = $derived(data.user?.role === 'admin');
  if (isAdmin) { store.setAuth({...}); } // Warning!
</script>

<!-- GOOD: compute from source directly -->
<script>
  let isAdmin = $derived(data.user?.role === 'admin');
  // SSR baseline: use source data, not $derived
  if (data.user?.role === 'admin') { store.setAuth({...}); }
  // $effect for CSR reactivity uses $derived normally
  $effect(() => { if (isAdmin) { store.setAuth({...}); } });
</script>
```

### Module-scoped $state and SSR safety

Module-scoped `$state` is shared across all SSR requests (it's a singleton). SvelteKit SSR is synchronous per-request, so call `reset()` at the top of the root layout to prevent cross-request pollution:

```svelte
<!-- +layout.svelte (root) -->
<script>
  import { myStore } from '$lib/stores/my.svelte';
  myStore.reset(); // Clear state before each render
</script>
```

## SvelteKit 2 Key Differences from SvelteKit 1

- `error()` and `redirect()` no longer need to be thrown -- just call them
- `cookies.set/delete/serialize` require explicit `path` parameter
- Import `vitePreprocess` from `@sveltejs/vite-plugin-svelte` (not from SvelteKit)
- `resolvePath` replaced by `resolveRoute` from `$app/paths`
- Paths are consistently relative or absolute based on `paths.relative` config
- Requires Node 18.13+

## Migration Commands

```bash
# Migrate Svelte 4 -> Svelte 5
npx sv migrate svelte-5

# Migrate SvelteKit 1 -> SvelteKit 2
npx sv migrate sveltekit-2
```

## Project Structure (SvelteKit)

```
src/
  lib/           # Shared code, importable as $lib/
  routes/        # File-based routing
    +page.svelte        # Page component
    +page.ts            # Page load function (universal)
    +page.server.ts     # Server-only load / form actions
    +layout.svelte      # Layout component
    +layout.ts          # Layout load function
    +layout.server.ts   # Server-only layout load
    +error.svelte       # Error page
    +server.ts          # API endpoint (GET, POST, etc.)
  app.html       # HTML template
  app.d.ts       # Auto-generated types
svelte.config.js # SvelteKit configuration
vite.config.ts   # Vite configuration
```

## Common Patterns

### Form Actions (server-side form handling)

```svelte
<!-- +page.svelte -->
<form method="POST" action="?/create">
  <input name="title" />
  <button>Create</button>
</form>
```

```ts
// +page.server.ts
import type { Actions } from './$types';

export const actions = {
  create: async ({ request }) => {
    const data = await request.formData();
    const title = data.get('title');
    // handle creation
  }
} satisfies Actions;
```

### Load Functions

```ts
// +page.ts (runs on server AND client)
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch }) => {
  const res = await fetch('/api/data');
  return { items: await res.json() };
};
```

```ts
// +page.server.ts (runs ONLY on server)
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
  return { user: locals.user };
};
```

## Sources

- [Svelte 5 Migration Guide](https://svelte.dev/docs/svelte/v5-migration-guide)
- [Svelte Runes: $state](https://svelte.dev/docs/svelte/$state)
- [Svelte Runes: $derived](https://svelte.dev/docs/svelte/$derived)
- [Svelte Runes: $effect](https://svelte.dev/docs/svelte/$effect)
- [Svelte Runes: $props](https://svelte.dev/docs/svelte/$props)
- [sv create docs](https://svelte.dev/docs/cli/sv-create)
- [SvelteKit 2 Migration Guide](https://svelte.dev/docs/kit/migrating-to-sveltekit-2)
- [Creating a SvelteKit project](https://svelte.dev/docs/kit/creating-a-project)
- [What's new in Svelte: February 2026](https://svelte.dev/blog/whats-new-in-svelte-february-2026)
