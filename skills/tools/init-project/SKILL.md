---
name: init-project
description: Initialise un nouveau projet. Analyse la stack, génère des agents spécialisés, crée les règles clean code/archi, cherche des skills sur SkillsMP/GitHub, crée les issues GitHub. Lance ce skill au démarrage de chaque projet.
user-invocable: true
---

Tu initialises le projet. Le setup est **entièrement automatique** — tu analyses le projet et tu configures tout.

## Contexte du projet
!`cat project.md`

---

## Phase 1 — Valider project.md

- Vérifie que toutes les sections obligatoires sont remplies (pas de placeholders)
- Sections obligatoires : Nom, Description, Stack technique, Structure, User Stories
- Si des sections sont incomplètes → demande à l'utilisateur de les remplir
- Vérifie le format des US : `- [US-XX] Titre | Description | Priorité | Dépendances (optionnel)`

---

## Phase 1.5 — Brainstorm : enrichir les US et structurer le projet

**C'est la phase clé.** Tu ne te contentes pas de recopier les US — tu **réfléchis au projet** et tu l'améliores.

### 1.5.1 Analyser la cohérence du projet

- Les US couvrent-elles tous les aspects de la description du projet ?
- Manque-t-il des US évidentes ? (Ex: le projet mentionne "authentification" mais aucune US d'auth)
- Y a-t-il des US trop grosses qui devraient être découpées ?
- Y a-t-il des US trop vagues qui méritent d'être précisées ?

**Si des US manquent ou sont mal découpées** → propose des ajouts/modifications à l'utilisateur :
```
Analyse du projet :

  US manquantes détectées :
    - Le projet mentionne "authentification" mais aucune US ne couvre le login/register
    - Pas d'US pour le setup initial (DB, config, CI)

  US trop grosses à découper :
    - US-03 "Dashboard complet" → suggère US-03a "Layout + navigation" + US-03b "Widgets stats"

  Voulez-vous que j'ajoute/modifie ces US ?
```

### 1.5.2 Enrichir chaque US

Pour chaque US, génère :

**Critères d'acceptance** — conditions vérifiables pour considérer l'US comme terminée :
```markdown
### Critères d'acceptance
- [ ] L'utilisateur peut se connecter avec email/password
- [ ] Un token JWT est généré et stocké en cookie httpOnly
- [ ] Les routes protégées redirigent vers /login si non authentifié
- [ ] Les erreurs de login affichent un message clair
```

**Sous-tâches techniques** — décomposition en tâches concrètes :
```markdown
### Sous-tâches techniques
1. Créer le schéma User (Prisma/Mongoose/...)
2. Implémenter l'endpoint POST /api/auth/register
3. Implémenter l'endpoint POST /api/auth/login
4. Créer le middleware d'authentification
5. Créer les pages /login et /register
6. Écrire les tests d'auth (unit + e2e)
```

**Fichiers impactés** — estimation des fichiers à créer/modifier :
```markdown
### Fichiers impactés (estimation)
- Créer : src/lib/auth.ts, src/app/api/auth/[...]/route.ts
- Créer : src/app/(auth)/login/page.tsx, src/app/(auth)/register/page.tsx
- Modifier : prisma/schema.prisma (ajouter modèle User)
- Créer : src/middleware.ts (auth middleware)
```

### 1.5.3 Détecter les dépendances automatiquement

Analyse TOUTES les US entre elles pour détecter les dépendances **même si l'utilisateur ne les a pas spécifiées** :

**Règles de détection automatique :**

| Signal | Type de dépendance |
|--------|--------------------|
| US-B mentionne un modèle/table créé dans US-A | `après:US-A` |
| US-B utilise un endpoint/service de US-A | `après:US-A` |
| US-B a besoin de l'auth et US-A = auth | `après:US-A` |
| US-A et US-B modifient les mêmes fichiers | `partage:US-A` |
| US-B ajoute une fonctionnalité à ce que US-A construit | `enrichit:US-A` |
| US-B = page qui affiche des données et US-A = API qui fournit ces données | `après:US-A` |

**Exemple concret :**
```
US déclarées par l'utilisateur :
  - [US-01] Auth utilisateur | Login/register | haute
  - [US-02] Dashboard | Stats et graphiques | haute
  - [US-03] API publique | CRUD endpoints | moyenne
  - [US-04] Export CSV | Exporter les données | basse

Dépendances détectées automatiquement :
  ✓ US-02 → après:US-01 (le dashboard a besoin de l'auth pour les routes protégées)
  ✓ US-02 → après:US-03 (le dashboard affiche des données de l'API)
  ✓ US-04 → enrichit:US-03 (l'export utilise les mêmes données que l'API)
  ✓ US-03 → après:US-01 (l'API a besoin de l'auth pour sécuriser les endpoints)
```

### 1.5.4 Proposer un US-00 "Setup initial" si nécessaire

Si le projet n'a pas encore de structure de base, propose une US-00 :
```
[US-00] Setup initial | Config projet, DB, CI, structure de base | haute
  - Init du projet (npm init, tsconfig, etc.)
  - Configurer la base de données (schema initial, connexion)
  - Configurer le linter et le formatter
  - Configurer les tests (framework, config)
  - Configurer la CI (GitHub Actions)
  - Créer la structure de dossiers
```

**Toutes les autres US dépendront de US-00** si le projet n'existe pas encore.

### 1.5.5 Valider avec l'utilisateur

Présente le brainstorm complet et demande validation :

```
═══════════════════════════════════════════════
  BRAINSTORM — Analyse du projet
═══════════════════════════════════════════════

  US enrichies :
    [US-01] Auth utilisateur (haute)
      → 4 critères d'acceptance
      → 6 sous-tâches techniques
      → Dépendances : aucune (racine)

    [US-02] Dashboard (haute)
      → 5 critères d'acceptance
      → 8 sous-tâches techniques
      → Dépendances détectées : après:US-01, après:US-03

  US ajoutées (suggérées) :
    [US-00] Setup initial (haute) — recommandé si projet vierge

  Dépendances détectées automatiquement :
    US-01 ──→ US-02, US-03
    US-03 ──→ US-02, US-04

  Graphe d'exécution :
    US-00 → US-01 → US-03 → US-02
                            → US-04

  Valider ce plan ? (oui / modifier)
═══════════════════════════════════════════════
```

**YOU MUST** obtenir la validation de l'utilisateur avant de continuer.

---

## Phase 2 — Analyser le projet

### 2.1 Détecter la stack

Lis `project.md` section "Stack technique" et identifie :

- **Langage principal** : TypeScript, JavaScript, Python, Rust, Go, Java...
- **Framework** : Next.js, Express, Fastify, NestJS, Django, FastAPI, Actix...
- **Base de données** : PostgreSQL, MongoDB, MySQL, Redis, SQLite...
- **ORM/ODM** : Prisma, Drizzle, Mongoose, TypeORM, SQLAlchemy...
- **Tests** : Jest, Vitest, Playwright, Cypress, pytest...
- **Linter** : ESLint, Biome, Prettier, Ruff...
- **UI** : Tailwind, Shadcn, MUI, Chakra...
- **Auth** : NextAuth, Auth0, Clerk, Supabase Auth...
- **Autre** : Docker, Stripe, S3, Redis, GraphQL, tRPC...

### 2.2 Déterminer le type de projet

| Pattern détecté | Type de projet |
|-----------------|----------------|
| Next.js + App Router | Full-stack SSR |
| Express/Fastify + DB | API Backend |
| React/Vue seul | SPA Frontend |
| Next.js + API externe | BFF + Frontend |
| CLI + pas de framework web | Outil CLI |
| Monorepo (packages/) | Monorepo |

### 2.3 Identifier les domaines fonctionnels

Analyse les US pour détecter les domaines :
- Auth/Users, Dashboard/UI, API/Endpoints, Database/Models
- Integrations (Stripe, S3, email...), Admin, etc.

---

## Phase 3 — Générer les règles clean code & architecture

**Crée des fichiers de règles spécifiques au projet dans `.claude/rules/`.**

### 3.1 Architecture (`.claude/rules/architecture.md`)

Génère des règles d'architecture **spécifiques à la stack détectée**. Sois opinionated, pas générique.

Le fichier DOIT contenir :
- **Structure** : organisation des dossiers et responsabilités de chaque couche
- **Patterns obligatoires** : patterns du framework à utiliser systématiquement
- **Séparation des responsabilités** : qui fait quoi dans quelle couche
- **Gestion des données** : comment accéder à la DB, validation, DTOs
- **Gestion d'erreurs** : stratégie d'error handling par couche

**Exemples de contenu selon la stack :**

Pour **Next.js + Prisma** :
- App Router only, Server Components par défaut, `'use client'` uniquement si nécessaire
- Route Handlers pour l'API, Server Actions pour les mutations
- Validation zod sur tous les inputs
- DB queries uniquement côté serveur
- Error boundaries au niveau layout, Suspense pour loading states

Pour **Express + MongoDB** :
- Controllers → Services → Repositories → Models
- Middleware chain pour cross-cutting concerns
- Error handling middleware en dernier dans la chaîne
- Mongoose lean() pour les lectures, indexes sur les FK

Pour **FastAPI + PostgreSQL** :
- Routers → Services → Repositories → Models (SQLAlchemy)
- Pydantic models pour validation
- Dependency injection pour les services
- Alembic pour les migrations

**Adapte TOUJOURS au projet réel.** Ne copie pas ces exemples tels quels.

### 3.2 Clean Code (`.claude/rules/clean-code.md`)

Génère des règles clean code **adaptées au langage et framework** :

Le fichier DOIT contenir :
- **Nommage** : conventions précises par type de fichier/classe/fonction
- **Anti-patterns interdits** : liste avec ❌ et alternative avec ✅
- **Patterns recommandés** : idiomes du framework
- **Organisation des imports** : ordre et groupes
- **Taille** : limites de taille pour fichiers/fonctions/classes
- **Typage** : règles strictes de typage pour le langage

### 3.3 Testing (`.claude/rules/testing.md`)

Génère des conventions de test **adaptées au framework de test** :

Le fichier DOIT contenir :
- **Structure** : où placer les tests, convention de nommage des fichiers
- **Conventions** : patterns (AAA, describe/it), nommage des tests
- **Couverture** : cas nominaux, limites, erreurs à couvrir
- **Mocks** : quand mocker, quand ne pas mocker
- **Commandes** : scripts npm/yarn pour lancer les tests

### 3.4 Mettre à jour le stabilizer

Mets à jour `.claude/rules/stability.md` avec les commandes spécifiques au projet détectées :
```bash
# Adapter les commandes au package manager et aux outils détectés
# npm / yarn / pnpm
# vitest / jest / pytest
# eslint / biome / ruff
# tsc / pyright / mypy
```

---

## Phase 4 — Chercher des skills communautaires (SkillsMP / GitHub)

**Cherche des skills existants qui matchent la stack du projet.**

```bash
bash scripts/search-skills.sh --stack
```

**Évalue les résultats :**
- ★ > 10 stars → pertinent, suggérer à l'utilisateur
- Match exact avec un besoin → recommander fortement
- Trop générique ou mauvaise qualité → ignorer

**Présente les résultats à l'utilisateur :**
```
Skills communautaires trouvés pour votre stack :

  [RECOMMANDÉ] ★ 45 | nextjs-testing-skill
    Tests patterns pour Next.js App Router avec Vitest
    → bash scripts/install-skill.sh owner/repo

  [SUGGÉRÉ] ★ 23 | prisma-migration-skill
    Gestion des migrations Prisma avec rollback
    → bash scripts/install-skill.sh owner/repo

Installer certains skills ? (listez les numéros ou "skip")
```

**Si l'utilisateur veut installer :**
```bash
bash scripts/install-skill.sh <github-owner/repo> [skill-name]
```

---

## Phase 5 — Générer les agents spécialisés

### 5.1 Déterminer les rôles nécessaires

Analyse la stack et les US pour créer les bons agents :

| Stack / Besoin | Agent à générer |
|----------------|-----------------|
| Framework frontend (Next.js, React...) | `frontend-dev` |
| Framework backend (Express, Fastify...) | `api-dev` |
| Full-stack simple | `fullstack-dev` (un seul agent dev) |
| Base de données + ORM | `db-architect` |
| Auth (NextAuth, Auth0...) | `auth-dev` |
| Intégrations (Stripe, S3...) | `integrations-dev` |
| Tests unitaires | `unit-tester` |
| Tests E2E (Playwright, Cypress) | `e2e-tester` |
| UI complexe (Dashboard, forms) | `ui-dev` |
| DevOps/CI | `devops` |

**Règle** : ne génère PAS d'agents inutiles. Un projet simple = 2-3 agents spécialisés + core (forge, stabilizer, reviewer).

### 5.2 Créer les SKILL.md pour chaque agent

Pour chaque agent, crée `.claude/skills/<agent-name>/SKILL.md` avec :

```yaml
---
name: <agent-name>
description: <description-spécifique-au-projet>
user-invocable: true
---
```

Le contenu de chaque agent DOIT inclure :
1. **Contexte technique** : framework, langage, stack pertinente
2. **Expertise spécifique** : ce que cet agent sait faire en détail
3. **Conventions du projet** : importées des règles générées en Phase 3
4. **Patterns à suivre** : idiomes spécifiques au framework
5. **Anti-patterns à éviter** : erreurs courantes dans cette stack
6. **Mission** : ce qu'on attend de l'agent (`$ARGUMENTS` pour la tâche)

**Chaque agent doit être un EXPERT de son domaine dans la stack du projet.** Il connaît les bonnes pratiques, les pièges, les patterns idiomatiques. Il n'est pas un développeur générique — il est spécialisé.

### 5.3 Mettre à jour team.md

Réécris `.claude/team.md` avec les agents réels du projet :

```markdown
# Équipe du projet <nom>

## Agents core (toujours présents)
- **forge** — Team Lead orchestrateur
- **stabilizer** — Quality gate (build, tests, lint, types)
- **reviewer** — Revue de code qualité + sécurité

## Agents spécialisés (générés pour ce projet)
- **<agent-1>** — <description>
- **<agent-2>** — <description>
...

## Composition d'équipe par type de tâche
| Type | Équipe |
|------|--------|
| Feature frontend | frontend-dev, unit-tester, stabilizer |
| Feature API | api-dev, db-architect, unit-tester, stabilizer |
| Feature full-stack | frontend-dev, api-dev, e2e-tester, reviewer, stabilizer |
| Bug fix | <dev-pertinent>, unit-tester, stabilizer |
```

---

## Phase 6 — Auto-assigner les agents aux US

Pour chaque US, analyse sa description et assigne les agents pertinents :

1. Identifie les domaines de l'US (frontend, API, DB, auth...)
2. Assigne les agents spécialisés correspondants
3. Ajoute toujours `stabilizer` en dernier
4. Ajoute `reviewer` si priorité haute ou domaine critique (auth, payment)

Écris l'assignation dans :
- Le body de chaque issue GitHub
- `project.md` section "Équipe agentique par feature" (auto-générée)

---

## Phase 7 — Analyser les dépendances entre US

1. Parse les dépendances explicites (`après:`, `partage:`, `enrichit:`)
2. Détecte les dépendances **implicites** :
   - Même scope dans les agents assignés → risque de fichiers partagés
   - US dont la description mentionne une autre US
   - US de priorité basse qui étend une US de priorité haute
3. Vérifie qu'il n'y a pas de **cycles**
4. Détermine l'ordre d'exécution optimal :
   - Racines du graphe d'abord
   - Puis US dont toutes les dépendances sont satisfaites
   - Priorité (haute → moyenne → basse) à dépendances égales

Types de relations :
| Relation | Signification | Impact |
|----------|---------------|--------|
| `après:US-XX` | Dépendance stricte | US-XX doit être Done |
| `partage:US-XX` | Mêmes fichiers/scope | Ne pas travailler en parallèle |
| `enrichit:US-XX` | Étend une feature | US-XX doit être en cours ou Done |

---

## Phase 8 — Créer les labels et issues GitHub

### Labels
```bash
gh label create "task" --description "US pas encore commencée" --color "0075ca" --force
gh label create "in-progress" --description "US en cours" --color "e4e669" --force
gh label create "done" --description "US terminée et stabilisée" --color "0e8a16" --force
gh label create "bug" --description "Bug détecté" --color "d73a4a" --force
gh label create "blocked" --description "US bloquée par une dépendance" --color "b60205" --force
gh label create "haute" --description "Priorité haute" --color "d93f0b" --force
gh label create "moyenne" --description "Priorité moyenne" --color "fbca04" --force
gh label create "basse" --description "Priorité basse" --color "c5def5" --force
```

### Issues

Pour chaque US, crée une issue (ordre topologique) :

```markdown
## Description
[Description de la US]

## Dépendances
<!-- "Aucune — peut démarrer immédiatement" si pas de dépendances -->
- **Bloquée par** : #<numero> ([US-YY] Titre) — type: après
- **Partage le scope avec** : #<numero> ([US-ZZ] Titre) — type: partage

## Équipe agentique
Agents : <agent-1>, <agent-2>, ..., stabilizer

## Priorité
[haute/moyenne/basse]
```

---

## Phase 9 — Résumé

Affiche un rapport complet :

```
═══════════════════════════════════════════════
  PROJET INITIALISÉ : <nom-projet>
═══════════════════════════════════════════════

  Stack détectée :
    Langage    : <langage>
    Framework  : <framework>
    DB         : <db> + <orm>
    Tests      : <test-framework>
    UI         : <ui-lib>

  Règles générées :
    .claude/rules/architecture.md  — Patterns <framework>
    .claude/rules/clean-code.md    — Conventions <langage> strict
    .claude/rules/testing.md       — Stratégie <test-framework>

  Agents spécialisés créés :
    <agent-1>  — <description-courte>
    <agent-2>  — <description-courte>
    ...

  Agents core :
    forge      — Team Lead orchestrateur
    stabilizer — Quality gate
    reviewer   — Code review

  Skills communautaires :
    [installé] <skill-1> (★ XX)
    [suggéré]  <skill-2> (★ XX) — non installé

  US créées : X
    Haute    : X
    Moyenne  : X
    Basse    : X

  Graphe de dépendances :
    US-01 ──→ US-02 ──→ US-04
    US-03 (indépendante)

  Ordre d'exécution recommandé :
    1. US-01 (haute, racine)
    2. ...

═══════════════════════════════════════════════
  Prochaine étape : /forge
═══════════════════════════════════════════════
```
