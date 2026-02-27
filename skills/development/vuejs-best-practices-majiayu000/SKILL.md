---
name: vuejs-best-practices
description: Vue 3 and Nuxt 3 performance optimization and best practices. This skill should be used when writing, reviewing, or refactoring Vue.js code to ensure optimal performance patterns. Triggers on tasks involving Vue components, Nuxt pages, Composition API, Pinia state management, or performance improvements.
license: MIT
metadata:
  author: mine-vibe
  version: "1.0.0"
---

# Vue.js Best Practices

Comprehensive performance optimization guide for Vue 3 and Nuxt 3 applications. Contains 50+ rules across 10 categories, prioritized by impact to guide automated refactoring and code generation.

## When to Apply

Reference these guidelines when:
- Writing new Vue 3 components or Nuxt 3 pages
- Using Composition API and composables
- Implementing data fetching (client or server-side)
- Managing state with Pinia
- Reviewing code for performance issues
- Refactoring existing Vue/Nuxt code
- Optimizing bundle size or load times

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Reactivity Optimization | CRITICAL | `reactivity-` |
| 2 | Bundle Size Optimization | CRITICAL | `bundle-` |
| 3 | Nuxt 3 Server Performance | HIGH | `nuxt-` |
| 4 | Component Design Patterns | HIGH | `component-` |
| 5 | Composition API Patterns | MEDIUM-HIGH | `composable-` |
| 6 | State Management (Pinia) | MEDIUM-HIGH | `pinia-` |
| 7 | Rendering Performance | MEDIUM | `rendering-` |
| 8 | TypeScript Integration | MEDIUM | `typescript-` |
| 9 | Testing Patterns | LOW-MEDIUM | `testing-` |
| 10 | Advanced Patterns | LOW | `advanced-` |

## Quick Reference

### 1. Reactivity Optimization (CRITICAL)

- `reactivity-shallowref` - Use shallowRef for large objects that don't need deep reactivity
- `reactivity-shallowreactive` - Use shallowReactive for flat objects
- `reactivity-computed-cache` - Leverage computed for expensive calculations (auto-caching)
- `reactivity-toraw` - Use toRaw() when passing to external libraries
- `reactivity-markraw` - Use markRaw() for non-reactive objects (classes, third-party instances)
- `reactivity-effectscope` - Use effectScope() to batch cleanup of effects

### 2. Bundle Size Optimization (CRITICAL)

- `bundle-tree-shaking` - Import from 'vue' not internal packages (@vue/reactivity, @vue/runtime-core)
- `bundle-async-components` - Use defineAsyncComponent for heavy components
- `bundle-dynamic-imports` - Dynamic import() for route-level code splitting
- `bundle-icon-imports` - Import icons directly, avoid icon libraries barrel files
- `bundle-lodash-es` - Use lodash-es with specific imports
- `bundle-analyze` - Use rollup-plugin-visualizer to identify bloat
- `bundle-external-cdn` - Externalize large libraries to CDN when appropriate

### 3. Nuxt 3 Server Performance (HIGH)

- `nuxt-usefetch` - Use useFetch/useAsyncData for SSR-friendly data fetching
- `nuxt-lazy-components` - Prefix with Lazy for automatic lazy loading
- `nuxt-payload-reduce` - Minimize payload with pick/transform options
- `nuxt-cache-route` - Use routeRules for caching strategies
- `nuxt-server-components` - Use .server.vue for server-only components
- `nuxt-prerender` - Prerender static pages at build time
- `nuxt-islands` - Use Nuxt Islands for partial hydration
- `nuxt-nitro-cache` - Leverage Nitro caching for API routes

### 4. Component Design Patterns (HIGH)

- `component-sfc-setup` - Always use `<script setup>` syntax
- `component-props-destructure` - Use destructuring with defineProps for reactivity
- `component-emits-typed` - Always define emits with TypeScript
- `component-slots-typed` - Type slots with defineSlots
- `component-expose` - Use defineExpose sparingly, prefer props/emits
- `component-v-once` - Use v-once for static content
- `component-v-memo` - Use v-memo for list item optimization
- `component-teleport` - Use Teleport for modals/tooltips

### 5. Composition API Patterns (MEDIUM-HIGH)

- `composable-naming` - Prefix composables with 'use' (useAuth, useFetch)
- `composable-return-object` - Return object, not array, for better DX
- `composable-cleanup` - Always handle cleanup in onUnmounted
- `composable-async` - Handle async state properly (loading, error, data)
- `composable-ssr-safe` - Check `import.meta.client` for browser-only code
- `composable-provide-inject` - Use provide/inject with InjectionKey for type safety
- `composable-vueuse` - Leverage VueUse before writing custom composables

### 6. State Management - Pinia (MEDIUM-HIGH)

- `pinia-setup-syntax` - Prefer setup stores over options stores
- `pinia-storetorefs` - Use storeToRefs() to maintain reactivity
- `pinia-actions-async` - Use actions for async operations, not getters
- `pinia-persist` - Use pinia-plugin-persistedstate for persistence
- `pinia-reset` - Implement $reset for store cleanup
- `pinia-subscribe` - Use $subscribe for side effects
- `pinia-ssr-hydration` - Handle SSR hydration properly in Nuxt

### 7. Rendering Performance (MEDIUM)

- `rendering-v-show-v-if` - v-show for frequent toggles, v-if for rare conditions
- `rendering-key-attribute` - Always use unique :key in v-for (avoid index as key)
- `rendering-virtual-scroll` - Use virtual scrolling for long lists (100+ items)
- `rendering-keep-alive` - Cache component instances with KeepAlive
- `rendering-suspense` - Use Suspense for async component loading states

### 8. TypeScript Integration (MEDIUM)

- `typescript-props-interface` - Define props with interface, not type
- `typescript-ref-type` - Explicitly type refs: `ref<string>('')`
- `typescript-template-ref` - Type template refs: `ref<HTMLInputElement | null>(null)`
- `typescript-component-type` - Use `ComponentPublicInstance` for component refs
- `typescript-generic-components` - Use generic components for reusable patterns
- `typescript-strict-inject` - Use InjectionKey<T> for type-safe provide/inject
- `typescript-event-handlers` - Type event handlers properly

### 9. Testing Patterns (LOW-MEDIUM)

- `testing-vitest` - Use Vitest for unit testing (Vue ecosystem native)
- `testing-vue-test-utils` - Use @vue/test-utils for component testing
- `testing-composables` - Test composables in isolation
- `testing-pinia` - Use createTestingPinia for store testing
- `testing-msw` - Use MSW for API mocking
- `testing-playwright` - Use Playwright for E2E with Nuxt

### 10. Advanced Patterns (LOW)

- `advanced-render-function` - Use render functions for dynamic component generation
- `advanced-custom-directive` - Create custom directives for DOM manipulation
- `advanced-plugin-pattern` - Structure plugins with proper install function

---

## Decision Trees

### When to use ref vs reactive

```
What type of data?
│
├── Primitives (string, number, boolean)
│   └── ref() ✓
│
├── Objects/Arrays (need deep reactivity)
│   └── reactive() ✓
│
├── Large objects (performance critical)
│   └── shallowRef() or shallowReactive() ✓
│
└── Non-reactive data (class instances, external libs)
    └── markRaw() ✓
```

### When to use useFetch vs $fetch (Nuxt)

```
Where is the code running?
│
├── Component/Page (need SSR + reactivity)
│   └── useFetch() or useAsyncData() ✓
│
├── Server API route
│   └── $fetch() ✓
│
├── Event handler (client-side only)
│   └── $fetch() ✓
│
└── Composable (reusable)
    └── useFetch() with key ✓
```

### Component communication pattern

```
What's the relationship?
│
├── Parent → Child
│   └── Props ✓
│
├── Child → Parent
│   └── Emits ✓
│
├── Siblings
│   └── Pinia store or provide/inject ✓
│
├── Deep nesting (prop drilling)
│   └── provide/inject ✓
│
└── Global state
    └── Pinia store ✓
```

---

## Anti-Patterns to Avoid

### ❌ DON'T:

- Use Options API in new Vue 3 projects
- Mutate props directly
- Use `this` in `<script setup>`
- Create reactive() with primitives
- Use v-if and v-for on same element
- Forget :key in v-for loops
- Use index as :key for dynamic lists
- Access refs without .value in script
- Nest Pinia stores unnecessarily
- Use `$fetch` in components (use `useFetch`)

### ✅ DO:

- Use Composition API with `<script setup>`
- Define props and emits with TypeScript
- Use computed for derived state
- Use watchEffect for side effects
- Leverage VueUse composables
- Use Pinia for global state
- Handle loading/error states
- Clean up effects in onUnmounted
- Use Suspense for async components

---

## Performance Checklist

Before shipping:

- [ ] **Bundle analyzed?** No unexpected large dependencies
- [ ] **Lazy loading?** Routes and heavy components code-split
- [ ] **Reactivity optimized?** Using shallowRef where appropriate
- [ ] **SSR working?** No hydration mismatches
- [ ] **Images optimized?** Using nuxt/image or similar
- [ ] **API calls deduplicated?** Using useFetch with proper keys
- [ ] **State management efficient?** Not over-fetching in stores
- [ ] **List rendering optimized?** Virtual scroll for long lists

---

## Full Compiled Document

For detailed code examples, implementation patterns, and comprehensive explanations: `AGENTS.md`

Each section in AGENTS.md contains:
- Detailed explanations of why patterns matter
- Incorrect vs Correct code comparisons
- Copy-paste ready implementations
- Vue 3 and Nuxt 3 specific examples
- VueUse integration examples
