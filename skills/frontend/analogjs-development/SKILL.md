---
name: analogjs-development
description: Develop with Analogjs 2.x file-based routing, markdown content management, and SSR/SSG configuration. Use when creating *.page.ts files, contentFilesResource, routeMeta, and prerender settings.
---

# Analogjs Development

Development guide for Analogjs 2.x framework with file-based routing and content management.

## When to Use This Skill

- Creating new pages (*.page.ts)
- Setting up dynamic routes ([param].page.ts)
- Loading and displaying markdown content
- Configuring SSR/SSG (prerender)
- Creating API routes (server/routes/)

**When NOT to use:**

- Creating non-page Angular components → `angular-v21-development`
- Styling only → `tailwindcss-v4-styling`
- UI/UX design application → `material-design-3-expressive`

## Core Principles

- **File-Based Routing:** Routes are defined by file location and naming
- **Default Export:** Page components must use default export
- **Content Resources API:** Use `injectContentFiles()` and `injectContent()` for markdown
- **requestContextInterceptor:** Place as the last interceptor in HttpClient configuration
- **Project Structure:**
  ```
  src/
  ├── app/pages/           # File-based routing
  │   ├── index.page.ts    # / route
  │   ├── about/
  │   │   └── index.page.ts # /about route
  │   └── blog/
  │       ├── index.page.ts     # /blog route
  │       └── [slug].page.ts    # /blog/:slug dynamic route
  ├── content/             # Markdown content
  │   └── blog/
  └── server/routes/       # API endpoints
  ```

## Implementation Guidelines

### Page Component

Key patterns for page components:

1. File must end with `.page.ts` suffix
2. Component must be default exported
3. Can have accompanying `.page.html` and `.page.css` files

→ Details: [Routing Patterns](references/routing-patterns.md)

### Dynamic Routes

Dynamic route patterns:

1. Use bracket syntax for parameter: `[slug].page.ts`
2. Access parameter via `injectContent()` or `ActivatedRoute`
3. Prefer `withComponentInputBinding()` for route params as inputs

→ Details: [Routing Patterns](references/routing-patterns.md#dynamic-routes)

### Content Management

Markdown content handling patterns:

1. Use `injectContentFiles<T>()` for content list
2. Use `injectContent<T>()` for single content by route param
3. Define `PostAttributes` interface for content metadata
4. Configure `provideContent()` with markdown renderer

→ Details: [Content Handling](references/content-handling.md)

### Route Metadata

Route-level configuration patterns:

1. Export `routeMeta` for route configuration
2. Set page title, meta tags, guards
3. Configure SSR/SSG options per route

→ Details: [Routing Patterns](references/routing-patterns.md#route-metadata)

### SSR/SSG Configuration

Server-side rendering and static generation patterns:

1. Configure `prerender.routes` in `vite.config.ts`
2. Use `contentDir` with `transform` for dynamic content routes
3. Set up `provideServerContext()` in `main.server.ts`

→ Details: [Content Handling](references/content-handling.md#prerender-configuration)

### API Routes

Server API route patterns:

1. Create files in `src/server/routes/`
2. Use `defineEventHandler()` from h3
3. File path becomes API endpoint

→ Details: [Routing Patterns](references/routing-patterns.md#api-routes)

## Workflow

1. **Route Planning:** Determine route type (static/dynamic/group)
2. **Create Page File:** Create `*.page.ts` with correct naming
3. **Template Setup:** Create template/style files if needed
4. **Content Connection:** Set up markdown content loading if applicable
5. **Route Metadata:** Configure `routeMeta` export
6. **Prerender Setup:** Add route to prerender configuration for SSG

## Related Skills

- **angular-v21-development:** For component internal implementation
- **tailwindcss-v4-styling:** For page styling
- **material-design-3-expressive:** For UI component application

## Reference Documentation

For detailed patterns and code examples, see:

- [Routing Patterns](references/routing-patterns.md) - File-based routing details
- [Content Handling](references/content-handling.md) - Markdown content management
