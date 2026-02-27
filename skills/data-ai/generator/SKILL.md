---
name: generator
description: Générateur de Skill - Crée de nouveaux fichiers SKILL.md depuis les définitions YAML d'agents
version: "1.0"
---

# Générateur de Skill

Génère des fichiers SKILL.md compatibles OpenSkills depuis les définitions d'agents YAML existants.

## Objectif

Convertit les fichiers `.claude-portable/agents/common/*.agent.yaml` en format `.claude-portable/skills/*/SKILL.md` pour compatibilité universelle avec :
- Claude Code
- Cursor
- Windsurf
- Codex
- Tout agent lisant AGENTS.md

## Utilisation

```
@generator "Convertir tous les agents en format SKILL.md"
@generator "Créer SKILL.md pour l'agent security-hardener"
```

## Processus de Conversion

### Entrée (agent.yaml)
```yaml
meta:
  id: error-handler
  name: "Error Handler"
  version: "1.0"
  description: "Ajoute la gestion d'erreurs chirurgicalement"
  token_budget:
    max_input: 500
    max_output: 400

role: |
  Expert en gestion d'erreurs...

workflow:
  1_analyze:
    action: "Identifier les points de défaillance"
  2_implement:
    action: "Ajouter try/catch"

constraints:
  - "Pas de refactoring"
  - "Gestion d'erreurs uniquement"
```

### Sortie (SKILL.md)
```markdown
---
name: error-handler
description: Ajoute la gestion d'erreurs chirurgicalement
version: "1.0"
---

# Error Handler

Expert en gestion d'erreurs...

## Workflow

### 1. Analyser
Identifier les points de défaillance...

### 2. Implémenter
Ajouter try/catch...

## Constraints

- Pas de refactoring
- Gestion d'erreurs uniquement
```

## Template

```markdown
---
name: {{ meta.id }}
description: {{ meta.description }}
version: {{ meta.version }}
---

# {{ meta.name }}

{{ role }}

## Votre Expertise

{{ expertise | bullet_list }}

## Vos Responsabilités

{{ responsibilities | numbered_list }}

## Workflow

{{ workflow | format_steps }}

## Format de Sortie

{{ output.format | describe }}

## Constraints

{{ constraints | bullet_list }}

## Exemple

{{ examples[0] | format_example }}
```

## Emplacement des Fichiers Générés

```
.claude-portable/skills/
├── AGENTS.md              # Fichier de découverte (auto-mis à jour)
├── architect/SKILL.md
├── developer/SKILL.md
├── conductor/SKILL.md
├── error-handler/SKILL.md
├── ...
```

## Commande de Sync

Pour régénérer tous les skills depuis les agents :

```bash
# Conceptuel - serait implémenté comme script
for agent in .claude-portable/agents/common/*.agent.yaml; do
  generate_skill "$agent"
done
update_agents_md
```

## Mise à Jour AGENTS.md

Après génération des skills, le fichier de découverte est mis à jour :

```xml
<available_skills>
  <skill>
    <name>{{ skill.name }}</name>
    <description>{{ skill.description }}</description>
    <location>project</location>
  </skill>
  ...
</available_skills>
```
