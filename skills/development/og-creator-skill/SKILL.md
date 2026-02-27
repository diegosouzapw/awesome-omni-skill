---
name: og-creator-skill
description: Create or fix Open Graph metadata for any web project across stacks (React/Vue/Svelte/Next/Nuxt/SSR/SSG/static). Use when a user asks to add OG tags, fix social previews, or standardize OG/Twitter metadata, especially when react-helmet-async is required and og:image must be an absolute URL.
---

# OG Creator Skill

## Overview

Provide a repeatable, web-project-agnostic workflow to add or fix OG/Twitter metadata across all stacks while honoring constraints like react-helmet-async and absolute og:image URLs.

## Workflow

### 1) Confirm scope and constraints

- Ask which stack and rendering mode applies: static HTML, SPA, SPA+runtime head updates, SSR, SSG/pre-render.
- Confirm hard constraints (if any): require react-helmet-async, require absolute og:image URLs, and deployment base URL.
- Request example pages that need custom OG (home, product detail, blog, etc).

### 2) Locate current metadata

- Search for existing OG/Twitter tags in entry HTML, templates, or shared SEO components.
- Identify the head management tool: React Helmet/Head, Next/Nuxt/SvelteKit head APIs, or static templates.
- If SSR/SSG exists, find server/renderer entry points that build the HTML head.

### 3) Implement the minimal viable OG setup

- Keep a static fallback in the entry HTML/template for SPA-first bots and unknown routes.
- Add or update a shared SEO component that sets:
  - `og:type`, `og:title`, `og:description`, `og:url`, `og:image`, `og:site_name`
  - `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- Enforce absolute URLs for `og:image` (and ideally `og:url`), using a base URL constant.

### 4) Make per-page metadata predictable

- Expose `SEO` props for title/description/type/image/url.
- For each route, pass page-specific values; keep defaults centralized.
- Prefer stable, deterministic values for SSG/SSR routes.

### 5) Validate and document

- Provide a quick checklist (see `references/og-checklist.md`) and explain any tradeoffs.
- If the user wants verification, suggest testing with platform validators after deploy.

### 6) Update and refresh share caches

- Verify the live HTML contains the new `og:image` URL.
- Confirm the image URL is publicly accessible (200 OK).
- Refresh platform caches after deploy:
  - Telegram: `@WebpageBot` -> `/refresh <url>`
  - If still stale, add a new query string to the page URL (e.g. `?v=YYYYMMDD`) and refresh again.

## Notes

- If the user mandates react-helmet-async, treat it as the only runtime head tool allowed.
- Absolute `og:image` URLs are mandatory; if the project is not deployed yet, require a temporary base URL or placeholder host.
- Always tailor guidance to the stack's head API or template system while preserving the static fallback HTML.

## Stack-specific guidance (concise)

### React SPA (Vite/CRA) with react-helmet-async

- Keep static fallback OG in `index.html`.
- Wrap app with `HelmetProvider`, create shared `SEO` component, and set per-route tags via props.
- Ensure `og:image` uses an absolute URL built from a base URL constant.

### Next.js (App Router / Pages Router)

- Prefer framework metadata API (`metadata` / `generateMetadata`) or `<Head>` for per-page tags.
- If custom head management is required, centralize defaults and override per page.
- For SSG/SSR, ensure tags render in HTML, not only client runtime.

### Nuxt 3

- Use `useHead`/`useSeoMeta` in pages and layouts.
- Put defaults in `app.vue` or a layout; override in pages.
- Ensure absolute URLs for `og:image`.

### Vue (Vite) SPA

- Use `@vueuse/head` or `vue-meta` for dynamic tags.
- Provide a shared SEO composable and route-level overrides.
- Keep a static fallback in `index.html`.

### SvelteKit

- Use `<svelte:head>` in layout/page files.
- For SSG/SSR, make sure `load` or data props provide stable metadata.
- Keep base URL in a shared constants module.

### Static HTML / Server templates

- Put OG/Twitter tags directly in the HTML/template.
- Use server-side template variables to set per-page metadata.
- Validate absolute `og:image` and `og:url`.
