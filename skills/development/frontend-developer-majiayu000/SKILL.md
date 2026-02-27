---
name: frontend-developer
description: Expert en développement front-end moderne - HTML, CSS, JavaScript, TypeScript, frameworks, testing, performance et tooling
version: 1.0.0
status: active
---

# Frontend Developer Skill

## Position dans l'Architecture

Ce skill est un skill de **NIVEAU 3 : COMMENT** (implémentation). Il fournit le code et les configurations concrètes pour le développement front-end.

```
┌─────────────────────────────────────────────────────────────────┐
│  NIVEAU 1 : POURQUOI (direction-technique)                      │
│  → Décisions stratégiques, choix de stack, ADRs                 │
│  → Quand utiliser React vs Vue ? Quel framework CSS ?           │
├─────────────────────────────────────────────────────────────────┤
│  NIVEAU 2 : QUOI (web-dev-process)                              │
│  → Process, workflows, checklists, standards                    │
│  → Comment organiser les tests ? Quel workflow Git ?            │
├─────────────────────────────────────────────────────────────────┤
│  NIVEAU 3 : COMMENT (frontend-developer) ← CE SKILL             │
│  → Implémentation, code, configuration                          │
│  → Comment écrire ce hook React ? Configurer Tailwind ?         │
└─────────────────────────────────────────────────────────────────┘
```

## Philosophie

Ce skill fournit l'**implémentation concrète** pour le développement front-end. Il contient :
- ✅ Du code (React, Vue, TypeScript, CSS...)
- ✅ Des configurations (Vite, ESLint, Tailwind...)
- ✅ Des patterns d'implémentation
- ✅ Des exemples concrets et prêts à l'emploi

Il ne contient PAS :
- ❌ Des décisions stratégiques → `direction-technique`
- ❌ Des processus de travail → `web-dev-process`
- ❌ Des politiques d'équipe → `direction-technique`

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              direction-technique                             │
│                         (POURQUOI - 52 agents)                              │
│                     Décisions stratégiques frontend                          │
│                                                                              │
│  avant-projet/selection-stack → Choix React/Vue/Angular                     │
│  architecture/patterns-design → Patterns d'architecture front               │
│  performance/optimisation-frontend → Stratégie perf (politique)             │
│  qualite/conventions-code → Standards de code (politique)                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              web-dev-process                                 │
│                          (QUOI - 61 agents)                                 │
│                      Process de développement                                │
│                                                                              │
│  design/ui-ux → Principes UX, responsive, accessibility                     │
│  setup/quality-tools → Workflow linting, formatting                         │
│  development/coding-standards → Process de code review                      │
│  testing/orchestrator → Stratégie de test (pyramide)                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            frontend-developer                                │
│                         (COMMENT - 33 agents)                               │
│                      Implémentation concrète                                 │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          8 DOMAINES                                   │   │
│  │                                                                       │   │
│  │  foundations/       javascript/       frameworks/      styling/      │   │
│  │      (5)               (5)               (6)             (4)         │   │
│  │                                                                       │   │
│  │  state-management/   testing/        performance/     tooling/       │   │
│  │      (3)               (4)               (3)            (3)          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Domaines et Agents (33 agents)

### 1. foundations/ - Implémentation HTML/CSS (5 agents)

Code et patterns pour les fondamentaux web.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `html-semantique` | Structure HTML5, SEO | Code HTML, balises, métadonnées |
| `css-moderne` | Grid, Flexbox, variables | Code CSS, layouts |
| `accessibilite` | WCAG, ARIA | Code a11y, attributs ARIA |
| `responsive-design` | Mobile-first | Media queries, clamp() |

### 2. javascript/ - Implémentation JS/TS (5 agents)

Code JavaScript et TypeScript moderne.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `javascript-moderne` | ES6+, async/await | Code JS, patterns |
| `typescript` | Typage, generics | Types, interfaces, configs |
| `dom-manipulation` | DOM API, événements | Code DOM, handlers |
| `api-integration` | Fetch, REST, WS | Code API, clients HTTP |

### 3. frameworks/ - Implémentation React/Vue/WordPress (6 agents)

Code spécifique aux frameworks front-end.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `react-expert` | **Délégation** vers skill `react-expert` | → 28 agents spécialisés |
| `vue-expert` | Composition API, Pinia | Code Vue, composables |
| `nextjs-expert` | **Délégation** vers skill `nextjs-expert` | → 35 agents spécialisés |
| `wordpress-expert` | **Délégation** vers skill `wordpress-gutenberg-expert` | → 41 agents spécialisés |
| `component-patterns` | HOC, Render Props | Patterns réutilisables |

> **Note** : Les agents `react-expert`, `nextjs-expert` et `wordpress-expert` délèguent vers leurs skills autonomes respectifs pour une couverture approfondie.

### 4. styling/ - Implémentation CSS (4 agents)

Code et configuration styling.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `tailwind-expert` | Configuration Tailwind | Config, plugins, classes |
| `css-in-js` | styled-components, Emotion | Code CSS-in-JS |
| `animations` | Transitions, Framer Motion | Code animations |

### 5. state-management/ - Implémentation State (3 agents)

Code de gestion d'état.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `react-state` | useState, Zustand, Redux | Code stores, slices |
| `server-state` | React Query, SWR | Code queries, mutations |

### 6. testing/ - Implémentation Tests (4 agents)

Code de tests front-end.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `unit-testing` | Jest, Vitest | Code tests unitaires |
| `component-testing` | RTL, Vue Test Utils | Code tests composants |
| `e2e-testing` | Playwright, Cypress | Code tests E2E |

### 7. performance/ - Implémentation Perf (3 agents)

Code et configuration performance.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `core-web-vitals` | LCP, CLS, INP | Code optimisation |
| `bundle-optimization` | Code splitting, lazy | Config bundler |

### 8. tooling/ - Configuration Outils (3 agents)

Configuration des outils de développement.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination | Routage |
| `build-tools` | Vite, Webpack | Fichiers config |
| `linting-formatting` | ESLint, Prettier | Fichiers config |

## Règles de Routage

### Depuis direction-technique (POURQUOI)

| Question stratégique | Ce skill fournit |
|---------------------|------------------|
| "Quel framework choisir ?" | → direction-technique décide, puis ce skill implémente |
| "Architecture micro-frontend ?" | → direction-technique décide, puis ce skill code |
| "Stratégie de performance ?" | → direction-technique définit, ce skill optimise |

### Depuis web-dev-process (QUOI)

| Process défini | Ce skill implémente |
|----------------|---------------------|
| "Pyramide de tests" | → Code des tests (unit, component, e2e) |
| "Code review checklist" | → Implémente les bonnes pratiques |
| "Workflow CI/CD" | → Configure les outils |

### Routage interne par mots-clés

```
SI question contient [code HTML, balises, SEO on-page]
   → agents/foundations/html-semantique.md

SI question contient [code CSS, Grid, Flexbox, layout]
   → agents/foundations/css-moderne.md

SI question contient [code ARIA, attributs a11y, lecteur écran]
   → agents/foundations/accessibilite.md

SI question contient [code JS, ES6, async, Promise, modules]
   → agents/javascript/javascript-moderne.md

SI question contient [code TypeScript, types, interface, generic]
   → agents/javascript/typescript.md

SI question contient [code React, hook, composant, useState]
   → agents/frameworks/react-expert.md

SI question contient [code Vue, ref, reactive, composable]
   → agents/frameworks/vue-expert.md

SI question contient [config Tailwind, classes, plugin]
   → agents/styling/tailwind-expert.md

SI question contient [code store, Zustand, Redux, slice]
   → agents/state-management/react-state.md

SI question contient [code test, Jest, Vitest, expect]
   → agents/testing/unit-testing.md

SI question contient [config Vite, Webpack, bundler]
   → agents/tooling/build-tools.md
```

## Composition avec Autres Skills

### Exemple 1 : Nouvelle feature React

```
1. direction-technique/architecture/patterns-design
   → Décide : "Utiliser le pattern Container/Presenter"

2. web-dev-process/development/coding-standards
   → Définit : "Convention de nommage, structure fichiers"

3. frontend-developer/frameworks/react-expert
   → Implémente : Code React avec le pattern
```

### Exemple 2 : Optimisation performance

```
1. direction-technique/performance/optimisation-frontend
   → Décide : "Prioriser LCP, budget < 2.5s"

2. web-dev-process/testing/performance
   → Définit : "Process de mesure, outils, seuils"

3. frontend-developer/performance/core-web-vitals
   → Implémente : Code d'optimisation LCP
```

### Exemple 3 : Tests composants

```
1. direction-technique/qualite/metriques-qualite
   → Décide : "Coverage minimum 80%"

2. web-dev-process/testing/unit-tests
   → Définit : "Pyramide de tests, quoi tester"

3. frontend-developer/testing/component-testing
   → Implémente : Code des tests RTL
```

## Points d'Escalade

### Vers direction-technique

- Choix de framework (React vs Vue vs Angular)
- Architecture globale (monolith vs micro-frontend)
- Décisions impactant toute l'équipe
- Trade-offs majeurs (performance vs maintenabilité)

### Vers web-dev-process

- Organisation du workflow de test
- Process de code review
- Standards de documentation
- Conventions d'équipe

### Vers l'humain

- Intégration avec systèmes legacy non documentés
- Contraintes techniques inhabituelles
- Bugs complexes sans solution évidente

## Skills Associés

| Skill | Niveau | Relation |
|-------|--------|----------|
| `direction-technique` | POURQUOI | Définit les décisions stratégiques |
| `web-dev-process` | QUOI | Définit les processus |
| `react-expert` | COMMENT | Implémentation React (28 agents) - délégation |
| `nextjs-expert` | COMMENT | Implémentation Next.js (35 agents) - délégation |
| `design-system-foundations` | COMMENT | Tokens et composants design |
| `wordpress-gutenberg-expert` | COMMENT | Implémentation WordPress |

## Changelog

### v1.1.0
- Délégation Next.js vers skill `nextjs-expert` (35 agents)
- 7 domaines Next.js : app-router, server-components, data, rendering, optimization, deployment, testing

### v1.0.0
- Création initiale avec 8 domaines et 33 agents
- Positionnement POURQUOI/QUOI/COMMENT
- Règles de composition avec direction-technique et web-dev-process
- Délégation React vers skill `react-expert` (28 agents)
- Délégation WordPress vers skill `wordpress-gutenberg-expert` (41 agents)
