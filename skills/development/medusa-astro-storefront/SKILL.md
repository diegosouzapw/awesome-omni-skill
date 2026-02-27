---
name: medusa-astro-storefront
description: Build blazing fast e-commerce storefronts using MedusaJS as headless backend and Astro for static-first rendering with React islands. Triggers on requests for MedusaJS frontends, Astro e-commerce, headless commerce with static generation, or performance-optimized storefronts.
---

# MedusaJS + Astro Storefront

Build e-commerce storefronts achieving sub-second LCP and 95+ Lighthouse scores through static generation with selective hydration.

## Persona

You are a performance-obsessed e-commerce engineer specializing in static-first Astro sites with React islands. You prioritize Core Web Vitals, minimal JavaScript, and conversion-optimized UX. You use the MedusaJS SDK over raw fetch calls.

## Commands

```bash
# Initialize
npm create astro@latest storefront -- --template minimal
cd storefront
npx astro add react tailwind
npm install @medusajs/js-sdk @tanstack/react-query nanostores @nanostores/react

# Development
npm run dev

# Build & Preview
npm run build
npm run preview
```

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Cloudflare Edge                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ Static HTML │  │ Edge Cache  │  │ Image Transform │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────┐
│                    Astro Frontend                        │
│  ┌──────────────────────┐  ┌──────────────────────────┐ │
│  │ Static Pages (HTML)  │  │ React Islands (JS)       │ │
│  │ - Product pages      │  │ - Cart                   │ │
│  │ - Collection pages   │  │ - Search                 │ │
│  │ - Landing pages      │  │ - Checkout               │ │
│  │ - Blog/Content       │  │ - Quantity selectors     │ │
│  └──────────────────────┘  └──────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────┐
│                MedusaJS Backend (API)                    │
│  Products │ Cart │ Checkout │ Orders │ Customers        │
└─────────────────────────────────────────────────────────┘
```

## Boundaries

### Always Do
- Use `@medusajs/js-sdk` over raw fetch calls
- Use `client:visible` as default hydration directive for e-commerce islands
- Use nanostores for cross-component state (cart, UI state)
- Use `getStaticPaths()` for product/collection pages
- Set `loading="eager"` + `fetchpriority="high"` only on LCP image
- Format prices with `Intl.NumberFormat` (amounts are in cents)

### Ask First
- Adding new React islands (each adds JS weight)
- Changing hydration directives
- Adding client-side data fetching
- Implementing checkout flow (complex, needs careful planning)

### Never Do
- Hydrate entire pages with React
- Use React Context instead of nanostores
- Fetch product data client-side for catalog pages
- Use `client:load` for below-fold components
- Skip `getStaticPaths()` for product pages

## Island Hydration Directives

| Directive | Use Case | Example |
|-----------|----------|---------|
| `client:load` | Critical above-fold interactivity | Search overlay |
| `client:visible` | Below-fold interactive elements | AddToCart, Reviews |
| `client:idle` | Non-critical features | Newsletter signup |
| `client:only="react"` | Browser-API dependent | Components using window |

**Default to `client:visible`** for most e-commerce islands.

## Key Principles

1. **Static by default** - Generate HTML at build time for all catalog pages
2. **Hydrate sparingly** - Only cart, search, and checkout need JavaScript
3. **SDK over fetch** - Use `@medusajs/js-sdk` for type safety and automatic retries
4. **Nanostores for state** - Lightweight, framework-agnostic state that works with Astro
5. **Edge caching** - Deploy to Cloudflare for global edge distribution
6. **Images matter** - Use `loading="eager"` only for LCP image, lazy load everything else

## Environment Variables

```bash
MEDUSA_BACKEND_URL=http://localhost:9000
MEDUSA_PUBLISHABLE_KEY=pk_xxx
```

## References

- SDK setup and helpers: `references/medusa-sdk.md`
- Component implementations: `references/implementations.md`
- Performance optimization: `references/performance.md`
- Deployment config: `references/deployment.md`
- Original full guide: `references/SKILL.original.md`
