---
name: manifest-frontend
description: Work with Manifest's frontend system — building, serving, dev workflow, debugging, and presets. Use when creating pages, components, debugging frontend errors, or configuring the build.
---

# Manifest Frontend

Manifest has a two-layer frontend system: a **framework layer** (serving + building) and **preset extensions** (starter kits with templates and conventions).

---

## How It Works

The framework layer lives in `manifest/frontend.ts` (~90 lines) and handles three things:

1. **Building** — `Bun.build()` bundles `frontend/` into `dist/` with source maps.
2. **Serving** — `manifest/server.ts` serves `dist/` as a fallback after API routes. API routes always win.
3. **Live reload** — In dev mode, an SSE endpoint at `/__dev/reload` pushes reload events to the browser when files change.

The preset extensions provide the actual frontend code and conventions. Check which one is installed:

```bash
ls extensions/ | grep manifest-frontend
```

| Extension | Stack | Best for |
|-----------|-------|----------|
| `manifest-frontend-static` | HTML + Tailwind + vanilla TS | Content sites, dashboards, simple UIs |
| `manifest-frontend-reactive` | SolidJS + Tailwind | Interactive apps, real-time UIs, client-side state |

**Always read the installed extension's `EXTENSION.md`** for preset-specific guidance (component patterns, routing, API fetching).

> **Tailwind v4 + Bun:** Bun's bundler does NOT run the Tailwind compiler. It inlines `@import "tailwindcss"` but generates zero utility classes. Both presets use `bundleCss: false` + Tailwind CLI via `postBuild` in `config/frontend.ts`. If Tailwind classes aren't rendering, check these two settings first.

---

## Config

Read `config/frontend.ts` before making any frontend changes:

```typescript
export default {
  entryPoint: 'frontend/index.ts',  // or index.tsx for reactive
  outputDir: 'dist',
  sourceMaps: true,                  // always true — agents need source maps
  spaFallback: true,                 // serves index.html for all non-file paths
  devReload: true,                   // SSE live reload in dev mode
}
```

- **`entryPoint`** — `.ts` for static preset, `.tsx` for reactive preset.
- **`spaFallback`** — When `true`, any path that doesn't match an API route or a file in `dist/` serves `index.html`. Set to `false` for multi-page sites.

---

## Dev Workflow

One command does everything:

```bash
bun --hot index.ts
```

The server watches `frontend/` for changes, rebuilds automatically, and triggers a browser reload via SSE. No second process needed.

---

## Adding Pages and Components

**Static preset:** Add `.html` files and `.ts` files in `frontend/`. For SPA routing, handle `window.location.pathname` in TypeScript.

**Reactive preset:** Add `.tsx` files in `frontend/`. Import and compose them in `App.tsx`. SolidJS components are plain functions returning JSX.

**Static assets** (images, fonts, favicons): Put them in `frontend/public/`. They're copied to `dist/` as-is during build.

---

## Building for Production

```bash
NODE_ENV=production bun run build
```

Produces minified output with source maps in `dist/`. The `dist/` directory is gitignored.

---

## Debugging Frontend Errors with Source Maps

When a frontend error appears (error tracker, browser console, logs):

1. **Find the bundled location** — e.g., `dist/index.js:142:18`
2. **Read the source map** — `dist/index.js.map` maps bundled lines back to source files
3. **Locate the original file** — e.g., `frontend/App.tsx:28`
4. **Fix the source file**, not the built output
5. **Rebuild and verify** — `bun run build`

The browser dev tools do this mapping automatically. When debugging from server-side error reports, use the `.map` files manually.

---

## When to Suggest Upgrading

If the user is on the **static** preset and asks for:
- Client-side state management
- Dynamic/reactive UI updates
- Component-based architecture
- Real-time data binding

Suggest installing the **reactive** preset (`manifest-frontend-reactive`). Read its `EXTENSION.md` for install steps.

---

## Quick Reference

| Task | Command |
|------|---------|
| Build | `bun run build` |
| Dev (server + watch + reload) | `bun --hot index.ts` |
| Production build | `NODE_ENV=production bun run build` |
| Check config | Read `config/frontend.ts` |
| Check which preset | `ls extensions/ \| grep manifest-frontend` |
