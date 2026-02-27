---
name: lead-dev
description: Lead Développeur - Coordination technique opérationnelle, code review, mentoring et livraison. Pair de web-dev-process au niveau OPÉRATIONS.
version: 1.1.0
status: active
---

# Lead Développeur Skill

## Quick Start

```bash
# 1. Navigation rapide vers un agent
lead-dev/agents/code-review/pr-review          # Valider une PR
lead-dev/agents/team-coordination/task-delegation   # Répartir les tâches
lead-dev/agents/delivery/deployment-check      # Vérifier avant deploy

# 2. Exécuter les tests de validation
cd .claude/skills/lead-dev && npm test

# 3. Questions fréquentes
"Valider cette PR ?"                → code-review/pr-review
"Répartir les tâches du sprint ?"   → team-coordination/task-delegation
"Débloquer un développeur ?"        → team-coordination/blocker-resolution
"Quelle librairie choisir ?"        → technical-decisions/library-selection
"Préparer une release ?"            → delivery/release-planning
```

## Position dans l'Architecture

Ce skill est au **NIVEAU 2 : OPÉRATIONS**, aux côtés de `web-dev-process`. Les deux skills sont complémentaires :

- **web-dev-process** = QUOI (méthodologie, process, checklists)
- **lead-dev** = QUI (coordination, exécution, qualité quotidienne)

```
┌─────────────────────────────────────────────────────────────────────┐
│  NIVEAU 1 : STRATÉGIE (direction-technique)                         │
│  → POURQUOI : Décisions, politiques, standards                      │
├─────────────────────────────────────────────────────────────────────┤
│  NIVEAU 2 : OPÉRATIONS                                              │
│  ┌────────────────────────────┐  ┌────────────────────────────┐    │
│  │     web-dev-process        │  │       lead-dev ← CE SKILL  │    │
│  │                            │  │                            │    │
│  │  QUOI : Méthodologie       │  │  QUI : Coordination        │    │
│  │  • 7 phases projet         │  │  • Code review (faire)     │    │
│  │  • Process standards       │  │  • Team coordination       │    │
│  │  • Checklists, workflows   │  │  • Delivery/release        │    │
│  │  • "Comment organiser ?"   │  │  • "Qui fait quoi ?"       │    │
│  └────────────────────────────┘  └────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────┤
│  NIVEAU 3 : IMPLÉMENTATION (skills techniques)                      │
│  → COMMENT : Code, configuration, patterns                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Distinction avec web-dev-process

| Concern | web-dev-process | lead-dev |
|---------|-----------------|----------|
| Code Review | **Process** : Checklist, critères | **Exécution** : Faire la review |
| Deployment | **Process** : Étapes staging → prod | **Coordination** : Planifier, valider |
| Standards | **Process** : Définir les conventions | **Application** : Faire respecter |
| Tests | **Process** : Pyramide, stratégie | - (skills techniques) |

## Philosophie

> Assurer la qualité technique au quotidien, coordonner les développeurs, et garantir des livraisons de qualité.

Le Lead Dev est le **gardien de la qualité technique opérationnelle**. Il :
- ✅ Revoit et valide le code (PRs, architecture locale)
- ✅ Coordonne les tâches entre développeurs
- ✅ Débloque les problèmes techniques
- ✅ Accompagne et forme les développeurs
- ✅ Garantit la qualité des livraisons

Il ne fait PAS :
- ❌ Les choix de stack stratégiques → `direction-technique`
- ❌ Les décisions d'architecture globale → `direction-technique`
- ❌ L'implémentation du code → `frontend-developer`, `backend-developer`
- ❌ Les process d'équipe globaux → `web-dev-process`

## Learning Loop

Avant toute action, consulter les **learnings** :

- 📚 [Patterns](../../../.claude/learnings/patterns/INDEX.md) - Solutions validées
- ⚠️ [Anti-patterns](../../../.claude/learnings/anti-patterns/INDEX.md) - Erreurs à éviter
- 🔀 [Décisions](../../../.claude/learnings/decisions/INDEX.md) - Choix archétypaux

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            direction-technique                              │
│                         (POURQUOI - 52 agents)                              │
│                     Décisions stratégiques                                   │
│                                                                              │
│  avant-projet/selection-stack → Choix de stack                              │
│  architecture/patterns-design → Architecture globale                         │
│  qualite/conventions-code → Standards (politique)                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              lead-dev                                        │
│                       (COORDINATION - 27 agents)                            │
│                    Coordination opérationnelle                               │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          5 DOMAINES                                   │   │
│  │                                                                       │   │
│  │  code-review/      team-coordination/    technical-decisions/        │   │
│  │      (6)                (5)                    (5)                   │   │
│  │                                                                       │   │
│  │  mentoring/            delivery/                                     │   │
│  │      (5)                 (6)                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│                              │                                               │
│              ┌───────────────┼───────────────┐                              │
│              ▼               ▼               ▼                              │
│  ┌─────────────────┐ ┌─────────────┐ ┌─────────────────┐                   │
│  │ frontend-dev    │ │ backend-dev │ │ react-expert    │                   │
│  │ (33 agents)     │ │ (38 agents) │ │ (28 agents)     │                   │
│  └─────────────────┘ └─────────────┘ └─────────────────┘                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Domaines et Agents (27 agents)

### 1. code-review/ - Revue de Code (6 agents)

Assurance qualité du code au quotidien.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination des reviews | Routage |
| `pr-review` | Revue des Pull Requests | Commentaires PR, approbation |
| `architecture-check` | Vérification patterns locaux | Feedback architecture |
| `quality-gate` | Validation standards qualité | Checklist qualité |
| `security-review` | Revue sécurité du code | Alertes sécurité |
| `performance-review` | Revue performance du code | Recommandations perf |

### 2. team-coordination/ - Coordination Équipe (5 agents)

Orchestration du travail quotidien.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination d'équipe | Routage |
| `task-delegation` | Répartition des tâches | Assignations, priorités |
| `standup-prep` | Préparation des daily | Points de blocage, updates |
| `blocker-resolution` | Déblocage technique | Solutions, escalades |
| `sprint-support` | Support technique sprint | Aide au planning |

### 3. technical-decisions/ - Décisions Techniques Projet (5 agents)

Décisions techniques de niveau projet (pas stratégique).

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination décisions | Routage |
| `library-selection` | Choix de librairies | Recommandations, justifications |
| `pattern-choice` | Choix de patterns locaux | Décisions documentées |
| `refactoring-plan` | Planification refactoring | Plan de refactoring |
| `tech-debt-prioritization` | Priorisation dette technique | Backlog dette |

### 4. mentoring/ - Accompagnement (5 agents)

Formation et montée en compétence.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination mentoring | Routage |
| `code-feedback` | Feedback constructif | Retours pédagogiques |
| `best-practices` | Transmission bonnes pratiques | Guidelines, exemples |
| `onboarding-dev` | Intégration nouveaux devs | Parcours onboarding |
| `skill-assessment` | Évaluation compétences | Bilan, plan de progression |

### 5. delivery/ - Livraison Technique (6 agents)

Garantie de livraisons de qualité : **processus et coordination**.

> **Note : Différence avec nextjs-expert/deployment/**
> - `lead-dev/agents/delivery/` = **Processus de release** : planification, vérifications, coordination, documentation
> - `nextjs-expert/deployment/` = **Implémentation technique** : Vercel, Docker, CI/CD *pour Next.js*
>
> Exemple : `lead-dev/agents/delivery/deployment-check` vérifie qu'on est prêt à déployer (tests OK, checklist),
> puis `nextjs-expert/deployment/vercel` effectue le déploiement technique sur Vercel.

| Agent | Responsabilité | Produit |
|-------|----------------|---------|
| `orchestrator` | Coordination livraison | Routage |
| `release-planning` | Planification des releases | Plan de release |
| `merge-strategy` | Stratégie de merge | Guidelines merge |
| `deployment-check` | Vérification pré-déploiement | Checklist deploy |
| `hotfix-coordination` | Coordination des hotfixes | Process hotfix |
| `release-notes` | Notes de version | Changelog, release notes |

**Total : 27 agents spécialisés**

## Règles de Routage

### Par Type d'Action

| Action | Domaine |
|--------|---------|
| Valider une PR, review de code | `code-review/` |
| Répartir les tâches, débloquer un dev | `team-coordination/` |
| Choisir une lib, planifier un refactoring | `technical-decisions/` |
| Former, donner du feedback | `mentoring/` |
| Préparer une release, vérifier avant deploy | `delivery/` |

### Par Mots-Clés

| Mots-clés | Domaine/Agent |
|-----------|---------------|
| PR, pull request, review, merge request | `code-review/pr-review` |
| qualité code, standards, lint | `code-review/quality-gate` |
| sécurité code, vulnérabilité | `code-review/security-review` |
| perf code, N+1, optimisation | `code-review/performance-review` |
| tâche, assignation, qui fait quoi | `team-coordination/task-delegation` |
| daily, standup, blocage | `team-coordination/standup-prep` |
| bloqué, stuck, aide technique | `team-coordination/blocker-resolution` |
| sprint, planning technique | `team-coordination/sprint-support` |
| librairie, package, npm, composer | `technical-decisions/library-selection` |
| pattern, approche, comment faire | `technical-decisions/pattern-choice` |
| refactoring, nettoyer, restructurer | `technical-decisions/refactoring-plan` |
| dette technique, priorité | `technical-decisions/tech-debt-prioritization` |
| feedback, review perso, amélioration | `mentoring/code-feedback` |
| bonnes pratiques, tips, guidelines | `mentoring/best-practices` |
| nouveau dev, onboarding, intégration | `mentoring/onboarding-dev` |
| évaluation, niveau, progression | `mentoring/skill-assessment` |
| release, version, livraison | `delivery/release-planning` |
| merge, branche, git flow | `delivery/merge-strategy` |
| deploy, mise en prod, checklist | `delivery/deployment-check` |
| hotfix, urgence, correctif | `delivery/hotfix-coordination` |
| changelog, notes de version | `delivery/release-notes` |

## Arbre de Décision

```
Requête Lead Dev
│
├─ Concerne la revue de code ?
│  ├─ Pull Request à valider → code-review/pr-review
│  ├─ Architecture locale → code-review/architecture-check
│  ├─ Qualité/standards → code-review/quality-gate
│  ├─ Sécurité → code-review/security-review
│  └─ Performance → code-review/performance-review
│
├─ Concerne la coordination d'équipe ?
│  ├─ Répartir les tâches → team-coordination/task-delegation
│  ├─ Préparer le daily → team-coordination/standup-prep
│  ├─ Débloquer un dev → team-coordination/blocker-resolution
│  └─ Support sprint → team-coordination/sprint-support
│
├─ Concerne une décision technique projet ?
│  ├─ Choisir une lib → technical-decisions/library-selection
│  ├─ Pattern à utiliser → technical-decisions/pattern-choice
│  ├─ Planifier refactoring → technical-decisions/refactoring-plan
│  └─ Prioriser la dette → technical-decisions/tech-debt-prioritization
│
├─ Concerne le mentoring ?
│  ├─ Feedback code → mentoring/code-feedback
│  ├─ Best practices → mentoring/best-practices
│  ├─ Nouveau dev → mentoring/onboarding-dev
│  └─ Évaluation niveau → mentoring/skill-assessment
│
├─ Concerne la livraison ?
│  ├─ Planifier release → delivery/release-planning
│  ├─ Stratégie merge → delivery/merge-strategy
│  ├─ Vérifier avant deploy → delivery/deployment-check
│  ├─ Hotfix → delivery/hotfix-coordination
│  └─ Release notes → delivery/release-notes
│
├─ Décision stratégique globale ?
│  └─ → skill direction-technique
│
└─ Implémentation de code ?
   └─ → skills frontend-developer, backend-developer, react-expert
```

## Interaction avec les Autres Skills

### Flux Entrants

```
direction-technique/qualite ──► lead-dev/agents/code-review
direction-technique/estimation ──► lead-dev/agents/team-coordination
project-management/pilotage ──► lead-dev/agents/delivery
```

### Flux Sortants

```
lead-dev/agents/code-review ──► frontend-developer (feedback → implémentation)
lead-dev/agents/code-review ──► backend-developer (feedback → implémentation)
lead-dev/agents/technical-decisions ──► direction-technique (escalade stratégique)
lead-dev/agents/delivery ──► project-management (status livraison)
```

## Points d'Escalade

### Vers direction-technique

| Situation | Raison |
|-----------|--------|
| Choix de stack | Décision stratégique |
| Architecture globale | Impact long terme |
| Standards d'équipe | Politique globale |
| Recrutement technique | Stratégie équipe |

### Vers l'humain

| Situation | Raison |
|-----------|--------|
| Conflit technique entre devs | Arbitrage humain requis |
| Performance individuelle | Sensibilité RH |
| Décision avec impact budget | Validation management |
| Incident critique | Responsabilité |

### Vers les Skills d'Implémentation

| Situation | Skill |
|-----------|-------|
| Implémentation React | `react-expert` |
| Implémentation Frontend | `frontend-developer` |
| Implémentation Backend | `backend-developer` |
| Implémentation WordPress | `wordpress-gutenberg-expert` |

## Skills Associés

| Skill | Niveau | Relation |
|-------|--------|----------|
| `direction-technique` | POURQUOI | Reçoit les directives stratégiques |
| `web-dev-process` | QUOI | Suit les process définis |
| `frontend-developer` | COMMENT | Coordonne les devs front |
| `backend-developer` | COMMENT | Coordonne les devs back |
| `react-expert` | COMMENT | Coordonne sur React |
| `project-management` | GESTION | Remonte les status |

## Tests de Validation

Le skill inclut des tests automatisés pour valider sa structure.

```bash
# Exécuter les tests (depuis le dossier du skill)
cd .claude/skills/lead-dev
npm test

# Mode verbose
npm run test:verbose
```

Les tests vérifient :
- ✅ Existence de tous les domaines (5)
- ✅ Présence de tous les agents attendus (27)
- ✅ Frontmatter YAML valide (name, description)
- ✅ Structure des agents (sections requises)
- ✅ Références vers les learnings

## Intégration CI

Les tests sont automatiquement exécutés via GitHub Actions :

- **Workflow** : `.github/workflows/lead-dev-tests.yml`
- **Déclenchement** : Push sur `main` ou PR modifiant `.claude/skills/lead-dev/**`
- **Rapport** : Commentaire automatique sur la PR avec les résultats

| Badge | Description |
|-------|-------------|
| ✅ Pass | Tous les tests passent |
| ❌ Fail | Au moins un test échoue |

```bash
# Vérifier le status localement avant de push
npm test
```

## Changelog

### v1.1.0
- **Clarification hiérarchie** : Positionné au NIVEAU 2 OPÉRATIONS, pair de web-dev-process
- **Distinction claire** : lead-dev = QUI (coordination), web-dev-process = QUOI (process)
- Voir ADR-006 pour la décision complète

### v1.0.0
- Création initiale avec 5 domaines et 27 agents
- Positionnement intermédiaire COORDINATION
- Règles de routage par mots-clés
- Points d'escalade définis
- Intégration avec direction-technique et skills d'implémentation
