---
name: react-expert
description: Expert React - Hooks, composants, state management, data fetching, testing, styling et patterns modernes
version: 1.0.0
status: active
---

# React Expert Skill

## Position dans l'Architecture

Skill de **NIVEAU 3 : COMMENT** (implémentation) spécialisé dans l'écosystème React.

```
┌─────────────────────────────────────────────────────────────────┐
│  NIVEAU 1 : POURQUOI (direction-technique)                      │
│  → Choisir React ? Architecture ? Patterns globaux ?            │
├─────────────────────────────────────────────────────────────────┤
│  NIVEAU 2 : QUOI (web-dev-process)                              │
│  → Process de dev, workflow tests, conventions équipe           │
├─────────────────────────────────────────────────────────────────┤
│  NIVEAU 3 : COMMENT (react-expert) ← CE SKILL                   │
│  → Code React, hooks, composants, tests, styling                │
└─────────────────────────────────────────────────────────────────┘
```

## Philosophie

Ce skill fournit l'**implémentation concrète** pour React :

- ✅ Code React (hooks, composants, patterns)
- ✅ State management (Context, Zustand, Redux)
- ✅ Data fetching (React Query, SWR)
- ✅ Testing (RTL, mocks)
- ✅ Styling (Tailwind, CSS-in-JS)
- ✅ Performance (memo, code splitting)

Il ne fournit PAS :
- ❌ Décisions "React vs Vue vs Angular" → `direction-technique`
- ❌ Process de code review → `web-dev-process`
- ❌ Conventions d'équipe → `web-dev-process`
- ❌ Next.js spécifique → `nextjs-expert` (skill séparé à créer)

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              react-expert                                    │
│                           (28 agents)                                        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          7 DOMAINES                                  │   │
│  │                                                                       │   │
│  │  hooks/           components/        state/          data/           │   │
│  │   (5)                (5)              (4)             (4)            │   │
│  │                                                                       │   │
│  │  testing/         styling/         performance/                       │   │
│  │   (4)               (3)              (3)                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Domaines et Agents

### 1. hooks/ - React Hooks (5 agents)

Implémentation et patterns de hooks.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination hooks | Routage |
| `state-hooks` | useState, useReducer | Code gestion état local |
| `effect-hooks` | useEffect, useLayoutEffect | Code effets, cleanup |
| `ref-hooks` | useRef, useImperativeHandle | Code refs DOM/valeurs |
| `custom-hooks` | Création de hooks personnalisés | Hooks réutilisables |

**Mots-clés** : useState, useEffect, useRef, useReducer, useCallback, useMemo, hook, custom hook

### 2. components/ - Patterns Composants (5 agents)

Patterns et architecture de composants.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination composants | Routage |
| `functional` | Composants fonctionnels, props, children | Code composants |
| `composition` | Compound, Render Props, HOC | Patterns composition |
| `forms` | Formulaires, validation, controlled/uncontrolled | Code formulaires |
| `error-boundaries` | Gestion d'erreurs, Suspense boundaries | Code error handling |

**Mots-clés** : composant, component, props, children, compound, render props, HOC, form, formulaire

### 3. state/ - Gestion d'État (4 agents)

State management React.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination state | Routage |
| `context` | Context API, providers, consumers | Code Context |
| `zustand` | Stores Zustand, slices, middleware | Code Zustand |
| `redux-toolkit` | Redux Toolkit, slices, thunks | Code RTK |

**Mots-clés** : state, context, provider, zustand, redux, store, slice, global state

### 4. data/ - Data Fetching (4 agents)

Récupération et synchronisation de données.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination data | Routage |
| `react-query` | TanStack Query, queries, mutations | Code React Query |
| `swr` | SWR, revalidation, cache | Code SWR |
| `suspense` | Suspense, lazy, streaming | Code Suspense |

**Mots-clés** : fetch, query, mutation, react-query, tanstack, swr, suspense, cache, data

### 5. testing/ - Tests React (4 agents)

Testing de composants et hooks.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination testing | Routage |
| `rtl` | React Testing Library, queries, events | Code tests composants |
| `hooks-testing` | renderHook, act, async hooks | Code tests hooks |
| `mocking` | Mocks, providers, MSW | Code mocking |

**Mots-clés** : test, testing, rtl, testing library, render, screen, userEvent, mock, msw

### 6. styling/ - Styling React (3 agents)

Solutions de styling pour React.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination styling | Routage |
| `tailwind-react` | Tailwind + React, cn, cva | Code Tailwind |
| `css-in-js` | styled-components, Emotion | Code CSS-in-JS |

**Mots-clés** : style, css, tailwind, styled-components, emotion, className, cn

### 7. performance/ - Performance React (3 agents)

Optimisation de performance.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination perf | Routage |
| `memoization` | memo, useMemo, useCallback | Code memo |
| `code-splitting` | lazy, Suspense, dynamic imports | Code splitting |

**Mots-clés** : performance, memo, useMemo, useCallback, lazy, code splitting, optimization

## Règles de Routage

### Par mots-clés

```
SI question contient [useState, useReducer, état local]
   → hooks/state-hooks.md

SI question contient [useEffect, cleanup, effet de bord]
   → hooks/effect-hooks.md

SI question contient [useRef, ref, DOM element, focus]
   → hooks/ref-hooks.md

SI question contient [custom hook, créer un hook, extraire]
   → hooks/custom-hooks.md

SI question contient [composant, props, children, functional]
   → components/functional.md

SI question contient [compound, render props, HOC, composition]
   → components/composition.md

SI question contient [formulaire, form, input, validation, controlled]
   → components/forms.md

SI question contient [Context, Provider, Consumer, useContext]
   → state/context.md

SI question contient [Zustand, store, create, slice]
   → state/zustand.md

SI question contient [Redux, RTK, createSlice, thunk]
   → state/redux-toolkit.md

SI question contient [React Query, useQuery, useMutation, TanStack]
   → data/react-query.md

SI question contient [SWR, useSWR, revalidate]
   → data/swr.md

SI question contient [test, Testing Library, render, screen]
   → testing/rtl.md

SI question contient [renderHook, test hook, act]
   → testing/hooks-testing.md

SI question contient [memo, useMemo, useCallback, re-render]
   → performance/memoization.md

SI question contient [lazy, code splitting, dynamic import]
   → performance/code-splitting.md
```

## Composition avec Autres Skills

### Flux typique : Nouvelle feature

```
1. direction-technique/architecture/patterns-design
   → Décide : "Utiliser le pattern Container/Presenter"

2. web-dev-process/development/coding-standards
   → Définit : "Conventions fichiers, tests obligatoires"

3. react-expert/components/functional
   → Implémente : Code React avec le pattern
```

### Flux typique : Tests

```
1. direction-technique/qualite/metriques-qualite
   → Décide : "Coverage 80%, tests critiques"

2. web-dev-process/testing/unit-tests
   → Définit : "Pyramide tests, quoi tester"

3. react-expert/testing/rtl
   → Implémente : Code tests composants

4. react-expert/testing/hooks-testing
   → Implémente : Code tests hooks
```

## Points d'Escalade

### Vers direction-technique
- Architecture globale (monorepo, micro-frontend)
- Choix de state management (Context vs Zustand vs Redux)
- Trade-offs majeurs

### Vers web-dev-process
- Workflow de PR et code review
- Organisation des tests
- Standards de documentation

### Vers frontend-developer
- Questions CSS/HTML générales
- Questions JavaScript/TypeScript générales
- Outils de build (Vite, Webpack)

### Vers nextjs-expert (skill séparé)
- App Router, Server Components
- Server Actions, API Routes
- Middleware, ISR, SSR

## Skills Associés

| Skill | Niveau | Relation |
|-------|--------|----------|
| `direction-technique` | POURQUOI | Décisions architecture React |
| `web-dev-process` | QUOI | Process de développement |
| `frontend-developer` | COMMENT | JS/TS/CSS génériques (délègue ici pour React) |
| `design-system-foundations` | COMMENT | Tokens et composants design |
| `nextjs-expert` | COMMENT | Implémentation Next.js (à créer) |

## Changelog

### v1.0.0
- Création initiale avec 7 domaines et 28 agents
- Focus sur React 18+ (hooks, Suspense, concurrent features)
- Architecture POURQUOI/QUOI/COMMENT
- Next.js extrait vers skill séparé (à créer)
