---
name: adding-templates
description: Use when adding new stacks, libraries, or project addons to create-faster CLI tool - covers META entries, template creation, and testing for all addon types
---

# Adding Templates to create-faster

## Overview

Add new content to the create-faster CLI: framework stacks, per-app libraries, or project-level addons. All additions are data-driven through META configuration + Handlebars templates.

**Core principle:** Understand-first, copy-second. Know the library/framework BEFORE writing templates.

## When to Use

Use this skill when:
- Adding a new framework stack (Remix, Astro, SvelteKit, etc.)
- Adding a new library (tRPC, i18n, Storybook, etc.)
- Adding a new project addon (new database, ORM, tooling)

Do NOT use for:
- Fixing existing templates (use `fixing-templates` skill)
- Extracting a library from an external project (use `extracting-templates` skill first, then come here)
- Updating dependency versions

## Architecture Context

Adding content is **data-driven**. Most CLI code operates generically on META configuration. Only source files that define types need changes. Everything else is META entries + template files.

**What does NOT change** when adding content (17+ files are generic):
- `cli.ts`, `flags.ts`, `index.ts` — iterate META entries
- `template-resolver.ts` — scans template directories by convention
- `package-json-generator.ts` — merges from `META.*.packageJson`
- `env-generator.ts` — collects envs from META
- `handlebars.ts` — generic helpers
- `addon-utils.ts` — generic compatibility checks
- All prompts, TUI, summary code

### Three Addon Types

| Type | META location | Template directory | Scope |
|------|--------------|-------------------|-------|
| Stack | `META.stacks` | `templates/stack/{name}/` | Per-app |
| Library | `META.libraries` | `templates/libraries/{name}/` | Per-app |
| Project addon | `META.project.{category}.options` | `templates/project/{category}/{name}/` | Project-level |

## Adding a Stack

### Source Changes

**Only 2-3 source files need code changes:**

#### a. Add to `StackName` union (`types/meta.ts`)

```typescript
export type StackName = 'nextjs' | 'expo' | 'hono' | 'tanstack-start' | 'newstack';
```

All downstream types derive from `StackName` automatically.

#### b. Add stack entry to META (`__meta__.ts`)

```typescript
newstack: {
  type: 'app' | 'server',
  label: 'Framework Name',
  hint: 'Short description',
  packageJson: {
    dependencies: { 'framework-core': '^1.0.0' },
    devDependencies: { typescript: '^5' },
    scripts: {
      dev: 'framework dev --port {{port}}',
      build: 'framework build',
      start: 'framework start --port {{port}}',
    },
  },
},
```

**Scripts with `{{port}}`:** `package-json-generator.ts` resolves placeholders:
- Turborepo: `{{port}}` → `3000 + appIndex`
- Single repo: the `--port {{port}}` portion is stripped entirely

#### c. Add file extensions to KNOWN_EXTENSIONS (if needed, `lib/frontmatter.ts`)

Non-standard extensions (`.svelte`, `.vue`, `.astro`) prevent the stack suffix parser from misidentifying them.

### Template Structure

```
templates/stack/{stackname}/
  {config}.config.ts.hbs    # Vite, Next, etc.
  tsconfig.json.hbs         # TypeScript config
  src/
    {entry-point}.tsx.hbs   # Main entry
    routes/                 # Routing (if file-based)
    components/             # Shared components
  public/                   # Static assets (if needed)
```

### Library Compatibility

Check existing libraries with `support: { stacks: 'all' }`. React libraries shouldn't be available for non-React stacks — change to explicit list:
```typescript
support: { stacks: ['nextjs', 'tanstack-start'] },  // React-only
```

## Adding a Library

### META Entry (`META.libraries`)

```typescript
newlib: {
  label: 'Library Name',
  hint: 'Short description',
  category: 'UI',  // Groups in interactive prompt (UI, Content, Auth, API, Data Fetching, Forms, Deploy)
  support: { stacks: ['nextjs', 'tanstack-start'] },
  // Optional: require other addons
  require: { orm: ['drizzle', 'prisma'] },
  // Optional: turborepo shared package
  mono: { scope: 'pkg', name: 'libname' },
  packageJson: {
    dependencies: { 'the-lib': '^1.0.0' },
    // Optional: package exports (for shared packages)
    exports: { '.': './src/index.ts' },
  },
  // Optional: environment variables
  envs: [
    { value: 'LIB_SECRET=changeme', monoScope: [{ pkg: 'libname' }, 'app'] },
  ],
},
```

**Key fields:**
- `category` — prompt group name (UI, Content, Auth, API, Data Fetching, Forms, Deploy)
- `support.stacks` — which stacks can use this library (`'all'` or explicit list)
- `require` — dependencies on other addons (e.g., better-auth requires orm)
- `mono` — if set, library gets its own `packages/{name}/` in turborepo
- `envs` — env vars with scope resolution (which `.env.example` files get them)

### Template Structure

```
templates/libraries/{libname}/
  src/lib/{feature}.ts.hbs              # Library setup files
  src/lib/{feature}-client.ts.hbs       # Client-side setup
  src/app/api/{route}/route.ts.nextjs.hbs  # Stack-specific API routes
```

### Handlebars Conditionals for Cross-Library Integration

When a library's behavior changes based on OTHER selected libraries:

```handlebars
{{!-- In the library's own template --}}
{{#if (hasLibrary "other-lib")}}
import { something } from '{{#if (isMono)}}@repo/other{{else}}@/lib/other{{/if}}';
{{/if}}

{{!-- Conditional feature based on project addon --}}
{{#if (has "orm" "drizzle")}}
import { db } from '{{#if (isMono)}}@repo/db{{else}}@/lib/db{{/if}}';
{{/if}}
```

### Modifying Existing Templates

Libraries often need to modify existing stack templates (e.g., `app-providers.tsx.hbs`):

```handlebars
{{!-- Add import --}}
{{#if (hasLibrary "newlib")}}
import { NewLibProvider } from '@/lib/newlib';
{{/if}}

{{!-- Add provider wrapper --}}
{{#if (hasLibrary "newlib")}}
<NewLibProvider>
{{/if}}
  {children}
{{#if (hasLibrary "newlib")}}
</NewLibProvider>
{{/if}}
```

**Wrap order matters** — check existing conditionals and add in the right position.

## Adding a Project Addon

### META Entry (`META.project.{category}.options`)

```typescript
// In META.project.{category}.options:
newoption: {
  label: 'Option Name',
  hint: 'Short description',
  mono: { scope: 'root' },  // or { scope: 'pkg', name: 'pkgname' }
  packageJson: {
    dependencies: { 'the-pkg': '^1.0.0' },
    scripts: { 'task:run': 'the-pkg run' },
  },
  envs: [
    { value: 'DB_URL="connection-string"', monoScope: [{ pkg: 'db' }, 'app'] },
  ],
},
```

### Template Structure

```
templates/project/{category}/{name}/
  config-file.ts.hbs
  scripts/seed.ts.hbs
```

## Programmatic Files — NO Templates

**package.json** — generated from META `packageJson`. No `package.json.hbs`.

**.env.example** — generated from META `envs`. No `.env.example.hbs`.

## Template Authoring Reference

### YAML Frontmatter

```yaml
---
path: src/lib/feature.ts       # Output path (single repo)
mono:
  scope: app | pkg | root      # Monorepo scope (overrides META)
  path: src/feature.ts          # Monorepo path (relative to scope)
only: mono | single             # Repo type filter
---
```

### Stack-Specific File Suffix

`file.ext.{stack}.hbs` — only included when app uses that stack:
```
route.ts.nextjs.hbs → only for Next.js apps
adapter.ts.hono.hbs → only for Hono apps
```

### Available Handlebars Helpers

| Helper | Purpose |
|--------|---------|
| `isMono` | Check if turborepo |
| `hasLibrary "name"` | Check if current app has library |
| `has "category" "value"` | Check database/orm/tooling/stack |
| `hasContext "key"` | Check if key exists in context |
| `appPort appName` | Get port for app (3000 + index) |
| `eq`, `ne`, `and`, `or` | Logical operators |

### Special Filename Handling

- `__filename` → `.filename` (dotfiles like `.gitignore`)
- `___filename` → `__filename` (literal double underscore)

## Testing

**DO NOT assume templates work. Verify.**

1. Single repo mode:
   ```bash
   bunx create-faster test-single --app test-single:{stack}:{lib1},{lib2}
   ```

2. Turborepo mode:
   ```bash
   bunx create-faster test-turbo --app web:{stack}:{lib1},{lib2} --app api:hono
   ```

3. Verify:
   - All template files present in output
   - `package.json` has correct dependencies
   - Conditional code renders correctly for all combinations
   - `bun install && bun run dev` works

4. Test all library combinations that affect templates:
   - Library alone
   - Library + each cross-library integration
   - Library in single vs turborepo mode

## Checklist

### Stack
- [ ] Added to `StackName` union in `types/meta.ts`
- [ ] Added META entry with `packageJson` (deps + scripts)
- [ ] Added file extension to `KNOWN_EXTENSIONS` (if applicable)
- [ ] Created `templates/stack/{name}/` with all source files
- [ ] Checked library compatibility (`stacks: 'all'` audit)
- [ ] Tested single + turborepo mode
- [ ] Verified dev/build works

### Library
- [ ] Added META entry in `META.libraries` with category/support/require/mono/packageJson/envs
- [ ] Created `templates/libraries/{name}/` with template files
- [ ] Modified existing stack templates for cross-library conditionals
- [ ] Used stack-specific suffix where needed (`.nextjs.hbs`)
- [ ] Tested all library combinations
- [ ] Tested single + turborepo mode

### Project Addon
- [ ] Added META entry in `META.project.{category}.options`
- [ ] Created `templates/project/{category}/{name}/`
- [ ] Tested with different stack/library combinations

## Common Rationalizations — STOP

| Excuse | Reality |
|--------|---------|
| "I need a package.json.hbs" | Package.json is programmatic from META. No template. |
| "I need a .env.example.hbs" | Env files are programmatic from META `envs`. No template. |
| "I'll skip testing combinations" | Library combinations produce different output. Test them. |
| "Close enough, it'll probably work" | Probably = broken. Test it. |
| "I can skip the monorepo test" | Single and turborepo produce different paths. Test both. |
| "I'll use magic comments" | Removed. Use YAML frontmatter. |

## Quick Reference

| What | Where | How |
|------|-------|-----|
| Stack definition | `META.stacks` | `type`, `label`, `hint`, `packageJson` |
| Library definition | `META.libraries` | `label`, `hint`, `category`, `support`, `require`, `mono`, `packageJson`, `envs` |
| Project addon | `META.project.{cat}.options` | `label`, `hint`, `mono`, `packageJson`, `envs` |
| Type union | `StackName` in `types/meta.ts` | Add new literal |
| Stack templates | `templates/stack/{name}/` | `.hbs` files |
| Library templates | `templates/libraries/{name}/` | `.hbs` files |
| Project templates | `templates/project/{cat}/{name}/` | `.hbs` files |
| Package.json | Programmatic | `package-json-generator.ts` reads META |
| Env vars | Programmatic | `env-generator.ts` reads META `envs` |
| File extensions | `KNOWN_EXTENSIONS` | `frontmatter.ts` |
| Frontmatter | YAML in `.hbs` | `path`, `mono`, `only` |
| Helpers | `handlebars.ts` | `isMono`, `hasLibrary`, `has`, `hasContext`, `appPort` |
