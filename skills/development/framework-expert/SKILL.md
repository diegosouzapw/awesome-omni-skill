---
name: framework-expert
description: "Unified framework expertise bundle. Lazy-loads relevant framework patterns (React, Vue, Angular, Next.js, Node.js, Python, Laravel, Go, Flutter, Godot) based on detected tech stack."
autoInvoke: true
priority: 50
triggers:
  - "framework"
  - "react"
  - "vue"
  - "angular"
  - "nextjs"
  - "nodejs"
  - "python"
  - "laravel"
  - "go"
  - "flutter"
context: fork
---

# Skill: Framework Expert (Bundle)

**Skill ID:** framework-expert
**Version:** 1.0.0
**Priority:** 50
**Auto-Invoke:** Yes (when framework task detected)

---

## Purpose

Unified entry point for all framework-specific expertise. Lazy-loads only the relevant framework patterns based on detected tech stack, reducing token usage by ~80% compared to loading all 12 individual framework skills.

---

## Triggers

- Framework-specific file detected (.tsx, .vue, .dart, .php, etc.)
- Framework keyword mentioned (React, Vue, Laravel, etc.)
- Project detection identifies framework

---

## Bundles

```toon
framework_bundles[5]{bundle,frameworks,detect_by}:
  web-frontend,"react,vue,angular,nextjs,svelte","package.json deps + file extensions"
  web-backend,"nodejs,python,laravel,go","package.json/requirements.txt/composer.json/go.mod"
  mobile,"react-native,flutter","app.json (expo) or pubspec.yaml"
  game,"godot","project.godot or *.gd files"
  typescript,"typescript","tsconfig.json or .ts files"
```

---

## Lazy Loading Strategy

### Step 1: Detect Framework (from cache)

```javascript
// Use cached detection - no file system scan needed
const detection = getCachedDetection();
// Returns: { framework: "nextjs", packageManager: "pnpm", ... }
```

### Step 2: Load Only Needed Patterns

```toon
load_order[4]{priority,load,when}:
  1,Core patterns (all frameworks),Always (~200 tokens)
  2,Detected framework patterns,Framework detected (~300 tokens)
  3,Secondary framework patterns,Full-stack project (~200 tokens)
  4,Advanced patterns,Complex task (~200 tokens)
```

**Total:** ~500-900 tokens vs ~6000 tokens (loading all 12 skills)

---

## Core Patterns (Always Loaded)

These apply to ALL frameworks:

```toon
core_patterns[8]{pattern,description}:
  File organization,Group by feature not type
  Naming conventions,PascalCase components camelCase functions
  Error handling,Graceful degradation + user feedback
  State management,Minimize global state
  API design,RESTful or GraphQL conventions
  Testing,Unit + integration + e2e coverage
  Performance,Lazy loading + code splitting
  Security,Input validation + sanitization
```

---

## Framework-Specific Patterns

### Web Frontend Bundle

**React** (when detected):
```toon
react_patterns[6]{pattern,rule}:
  Components,Functional with hooks (no class)
  State,useState for local useReducer for complex
  Effects,Cleanup functions + dependency arrays
  Memoization,useMemo/useCallback for expensive ops
  Context,For cross-cutting concerns only
  Error boundaries,Wrap critical UI sections
```

**Vue** (when detected):
```toon
vue_patterns[6]{pattern,rule}:
  Composition API,setup() with ref/reactive
  Composables,Extract reusable logic to use* functions
  Props,Define with types + validators
  Events,emit() for parent communication
  Stores,Pinia for global state
  Slots,Named slots for flexible components
```

**Angular** (when detected):
```toon
angular_patterns[6]{pattern,rule}:
  Components,Standalone preferred
  Services,Injectable with providedIn root
  RxJS,Operators + async pipe
  Signals,Modern reactivity (v17+)
  Modules,Lazy loaded feature modules
  Forms,Reactive forms with validators
```

**Next.js** (when detected):
```toon
nextjs_patterns[6]{pattern,rule}:
  App Router,Server components default
  Data fetching,fetch() in server components
  API routes,Route handlers in app/api/
  Middleware,Edge runtime for auth/redirects
  Caching,revalidate + cache tags
  Metadata,generateMetadata for SEO
```

### Web Backend Bundle

**Node.js** (when detected):
```toon
nodejs_patterns[6]{pattern,rule}:
  Structure,Controllers + services + repos
  Async,async/await (no callbacks)
  Validation,Zod or Joi schemas
  Error handling,Custom error classes
  Middleware,Auth + logging + rate limit
  Database,ORM (Prisma/TypeORM) or query builder
```

**Python** (when detected):
```toon
python_patterns[6]{pattern,rule}:
  Type hints,All function signatures
  Async,asyncio for I/O bound
  Validation,Pydantic models
  Structure,Domain-driven design
  Testing,pytest with fixtures
  Dependencies,Virtual env + requirements.txt
```

**Laravel** (when detected):
```toon
laravel_patterns[6]{pattern,rule}:
  Controllers,Single responsibility
  Models,Eloquent with scopes
  Validation,Form requests
  Services,Business logic extraction
  Events,Decouple with event/listener
  Queues,Background job processing
```

**Go** (when detected):
```toon
go_patterns[6]{pattern,rule}:
  Structure,cmd/ internal/ pkg/
  Error handling,Explicit error returns
  Interfaces,Accept interfaces return structs
  Concurrency,Goroutines + channels
  Testing,Table-driven tests
  Dependencies,Go modules
```

### Mobile Bundle

**React Native** (when detected):
```toon
rn_patterns[6]{pattern,rule}:
  Navigation,React Navigation v6+
  Styling,StyleSheet or NativeWind
  State,Zustand or Redux Toolkit
  Platform,Platform.select() for differences
  Performance,FlatList + memo
  Testing,Detox for e2e
```

**Flutter** (when detected):
```toon
flutter_patterns[6]{pattern,rule}:
  Widgets,Composition over inheritance
  State,Riverpod or BLoC
  Navigation,go_router
  Styling,Theme extensions
  Testing,Widget + integration tests
  Platform,Platform channels for native
```

### Game Bundle

**Godot** (when detected):
```toon
godot_patterns[6]{pattern,rule}:
  Scenes,Composition with child scenes
  Scripts,GDScript with static typing
  Signals,Decouple with custom signals
  Resources,Custom resource classes
  Autoload,Singletons for global state
  Export,Multi-platform builds
```

---

## Detection to Pattern Mapping

```toon
detection_map[12]{detection,load_patterns,tokens}:
  react,core + react_patterns,~500
  vue,core + vue_patterns,~500
  angular,core + angular_patterns,~500
  nextjs,core + nextjs_patterns + react_patterns,~700
  nodejs,core + nodejs_patterns,~500
  python,core + python_patterns,~500
  laravel,core + laravel_patterns,~500
  go,core + go_patterns,~500
  react-native,core + rn_patterns + react_patterns,~700
  flutter,core + flutter_patterns,~500
  godot,core + godot_patterns,~500
  typescript,core + ts_patterns,~400
```

---

## Integration with Project Cache

```javascript
// This skill reads from project cache
const { getCachedDetection } = require('./hooks/lib/af-project-cache.cjs');

const detection = getCachedDetection();
if (detection && detection.framework) {
  // Load only detected framework patterns
  loadPatterns(detection.framework);
}
```

---

## Full-Stack Projects

For full-stack projects (frontend + backend detected):

```toon
fullstack_loading[3]{scenario,loads,tokens}:
  Next.js + API routes,nextjs_patterns + nodejs_patterns,~900
  Vue + Laravel,vue_patterns + laravel_patterns,~900
  React + Django,react_patterns + python_patterns,~900
```

---

## Related Files

- Individual framework skills (legacy, still available):
  - `skills/react-expert/SKILL.md`
  - `skills/vue-expert/SKILL.md`
  - `skills/nextjs-expert/SKILL.md`
  - etc.
- `hooks/lib/af-project-cache.cjs` - Project detection cache
- `rules/context-management.md` - Token optimization rules

---

## Migration Note

This bundle skill is an **optimization layer** on top of existing framework skills. The individual skills (`react-expert`, `vue-expert`, etc.) remain available for:
- Direct invocation when needed
- Detailed reference documentation
- Edge cases requiring full skill context

The framework-expert bundle is the **default** for normal usage.

---

**Version:** 1.0.0 | **Last Updated:** 2026-01-21
