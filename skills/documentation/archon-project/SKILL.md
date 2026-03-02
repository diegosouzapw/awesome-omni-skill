---
name: archon-project
description: Gestion du projet MyGGV GPS dans Archon. Utiliser pour la documentation, les tâches, la recherche dans la knowledge base (Supabase docs, MapLibre docs, etc.), et le suivi de projet.
allowed-tools:
  - mcp__archon__find_projects
  - mcp__archon__manage_project
  - mcp__archon__find_tasks
  - mcp__archon__manage_task
  - mcp__archon__find_documents
  - mcp__archon__manage_document
  - mcp__archon__find_versions
  - mcp__archon__manage_version
  - mcp__archon__get_project_features
  - mcp__archon__rag_search_knowledge_base
  - mcp__archon__rag_search_code_examples
  - mcp__archon__rag_get_available_sources
  - mcp__archon__rag_list_pages_for_source
  - mcp__archon__rag_read_full_page
  - mcp__archon__health_check
---

# Archon Project Management - MyGGV GPS

## Objectif

Gérer le projet MyGGV GPS dans Archon : documentation, tâches, recherche dans la knowledge base, et suivi de version.

## Périmètre

### Inclus

- Gestion de projet (création, mise à jour)
- Gestion des tâches (todo, doing, review, done)
- Documentation de projet (specs, notes, guides)
- Recherche RAG dans la knowledge base
- Exemples de code depuis la documentation

### Exclus

- Opérations base de données → utiliser `supabase-database`
- Déploiement → utiliser `netlify-deploy`
- Code de navigation → utiliser `maplibre-navigation`

## IMPORTANT : Recherche de Documentation

### NE PAS utiliser `mcp__supabase__search_docs`

Toujours utiliser Archon pour la recherche de documentation :

```javascript
// CORRECT - Utiliser Archon
mcp__archon__rag_search_knowledge_base({
  query: "vector search pgvector",
  match_count: 5,
});

// INCORRECT - Ne pas utiliser directement
// mcp__supabase__search_docs(...)  // NON !
```

### Sources Disponibles

Pour voir les sources de documentation indexées :

```javascript
mcp__archon__rag_get_available_sources();
```

Sources typiquement disponibles :

- Documentation Supabase
- Documentation MapLibre GL
- Documentation React
- Documentation Vite

## Workflow de Gestion de Projet

### 1. Trouver le projet

```javascript
mcp__archon__find_projects({
  query: "ggv gps",
});
```

### 2. Créer/Mettre à jour le projet

```javascript
mcp__archon__manage_project({
  action: "create",
  title: "MyGGV GPS",
  description: "Application GPS web pour Garden Grove Village",
  github_repo: "https://github.com/user/new-ggv-gps",
});
```

### 3. Gérer les tâches

```javascript
// Créer une tâche
mcp__archon__manage_task({
  action: "create",
  project_id: "<project-id>",
  title: "Implémenter la fonctionnalité",
  description: "Description de la tâche à implémenter",
  status: "todo",
  feature: "navigation",
});

// Mettre à jour le statut
mcp__archon__manage_task({
  action: "update",
  task_id: "<task-id>",
  status: "doing",
});
```

## Recherche RAG

### Requêtes courtes et focalisées

```javascript
// BON - Court et précis
mcp__archon__rag_search_knowledge_base({
  query: "maplibre markers",
  match_count: 5,
});

// MAUVAIS - Trop long
// query: "comment ajouter des markers personnalisés sur une carte maplibre avec des icônes SVG..."
```

### Recherche d'exemples de code

```javascript
mcp__archon__rag_search_code_examples({
  query: "React geolocation hook",
  match_count: 3,
});
```

### Filtrer par source

```javascript
// 1. Obtenir les sources
const sources = mcp__archon__rag_get_available_sources();

// 2. Filtrer la recherche
mcp__archon__rag_search_knowledge_base({
  query: "maplibre markers",
  source_id: "src_maplibre_xxx", // ID de la source MapLibre
  match_count: 5,
});
```

### Lire une page complète

```javascript
mcp__archon__rag_read_full_page({
  url: "https://docs.maplibre.org/guides/navigation/",
});
```

## Documentation de Projet

### Créer un document

```javascript
mcp__archon__manage_document({
  action: "create",
  project_id: "<project-id>",
  title: "Architecture GPS Navigation",
  document_type: "spec",
  content: {
    overview: "...",
    components: ["..."],
    data_flow: "...",
  },
  tags: ["architecture", "navigation"],
});
```

### Types de documents

- `spec` - Spécifications techniques
- `design` - Documents de conception
- `note` - Notes générales
- `prp` - Product Requirements
- `api` - Documentation API
- `guide` - Guides d'utilisation

## Statuts de Tâche

```
todo → doing → review → done
```

- **todo** : À faire
- **doing** : En cours (1 seule tâche à la fois)
- **review** : En attente de validation
- **done** : Terminé

## Bonnes Pratiques

1. **Queries RAG courtes** : 2-5 mots-clés max
2. **Une tâche "doing" à la fois** : Focus sur une seule chose
3. **Documenter les décisions** : Créer des documents pour les choix architecturaux
4. **Versionner** : Utiliser `manage_version` pour les changements importants
