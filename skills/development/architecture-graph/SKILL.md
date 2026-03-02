---
name: Architecture Graph
description: This skill should be used when the user asks about "entities", "relations", "architecture graph", "spec-driven development", "code annotations", "@graph anchors", mentions ".graph/" directory, asks to "add entity", "create relation", "verify sync", "check drift", discusses "code_refs", "architecture specification", or is working with YAML entity files in .graph/entities/. Provides guidance for maintaining architecture-as-code consistency.
version: 0.1.0
---

# Architecture Graph Skill

## Overview

Cartographer enables **spec-driven development** where architecture specifications in `.graph/` define the source of truth for entities, relations, and code structure. The code follows the spec, not the other way around.

## Core Concepts

### Entities

Entities are defined in `.graph/entities/*.yaml`:

```yaml
name: User
description: Application user account
fields:
  - name: id
    type: uuid
    primary: true
  - name: email
    type: string
    unique: true
  - name: name
    type: string
relations:
  - name: todos
    entity: Todo
    type: has_many
code_refs:
  model:
    anchor: "@graph:User.model"
  schema:
    anchor: "@graph:User.schema"
```

**Key properties:**
- `name`: PascalCase entity identifier
- `description`: Human-readable purpose
- `fields`: Typed fields with constraints (primary, unique, nullable)
- `relations`: Connections to other entities
- `code_refs`: Links to implementation code via anchors

### Relations

Relations define how entities connect:

| Type | Description | Example |
|------|-------------|---------|
| `has_many` | One-to-many | User has_many Todos |
| `belongs_to` | Many-to-one (inverse) | Todo belongs_to User |
| `has_one` | One-to-one | User has_one Profile |
| `has_many_through` | Many-to-many via junction | User has_many Categories through TodoCategories |

### Code References (code_refs)

Code is linked to specs via anchor comments in source files:

```typescript
// @graph:User.model
export interface User {
  id: string;
  email: string;
  name: string;
}
// @end:User.model
```

**Anchor syntax:**
- Start: `// @graph:EntityName.refType`
- End: `// @end:EntityName.refType`
- Works in any language with comments

## Spec-Driven Workflow

When implementing or modifying code, **always follow this workflow**:

### 1. Check the Spec First

Before writing any code, read the entity YAML in `.graph/entities/`:

```bash
# Use /graph:entity command
/graph:entity User
```

Or read the file directly to understand:
- What fields are expected
- What relations exist
- Where code should be anchored

### 2. Follow the Spec

Implement exactly what's specified:
- Use the same field names and types
- Implement all defined relations
- Place code in the anchored locations

### 3. Add Anchors

Wrap implementations with `@graph:` comments:

```typescript
// @graph:User.model
// Implementation here
// @end:User.model
```

### 4. Verify Sync

Run `/graph:check` to validate that code matches spec.

## Available Commands

| Command | Purpose |
|---------|---------|
| `/graph:check` | Verify spec-code synchronization |
| `/graph:entity <name>` | Query entity details |
| `/graph:impact <entity>` | Analyze change impact |

## MCP Tools

The Cartographer MCP server provides these tools:

| Tool | Purpose |
|------|---------|
| `scan` | Analyze codebase, find anchors |
| `query` | Get entity details |
| `validate` | Check spec compliance |
| `impact` | Analyze change dependencies |
| `get_relations` | Fetch entity relations |

## Best Practices

### 1. Spec-First Development

Always define in `.graph/` before implementing:

```
1. Create/update .graph/entities/new-entity.yaml
2. Implement code with anchors
3. Run /graph:check
```

### 2. Atomic Anchors

One anchor per logical unit:
- `model` - Data structure/interface
- `schema` - Validation rules
- `handler` - API endpoint handler
- `repository` - Database access layer

### 3. Keep in Sync

Run `/graph:check` after every modification to catch drift early.

### 4. Document Relations

Make entity connections explicit in both entities:

```yaml
# user.yaml
relations:
  - name: todos
    entity: Todo
    type: has_many

# todo.yaml
relations:
  - name: user
    entity: User
    type: belongs_to
```

## Common Tasks

### Adding a New Entity

1. Create `.graph/entities/new-entity.yaml` with all fields
2. Add relations to connected entities
3. Create code files with anchor comments
4. Run `/graph:check` to verify

### Adding a Relation

1. Update both entity YAML files
2. Add foreign key field if needed
3. Update both code_refs sections
4. Verify with `/graph:check`

### Modifying a Field

1. Run `/graph:impact Entity.field` first
2. Update the entity YAML
3. Modify all code_refs locations
4. Run `/graph:check`

### Checking Impact Before Changes

Before modifying an entity:

```
/graph:impact User.email
```

This shows:
- All code locations that need updates
- Related entities affected
- Suggested action items

## Field Types Reference

Common field types in entity definitions:

| Type | Description |
|------|-------------|
| `uuid` | Universally unique identifier |
| `string` | Text data |
| `integer` | Whole numbers |
| `float` | Decimal numbers |
| `boolean` | True/false |
| `timestamp` | Date and time |
| `date` | Date only |
| `json` | JSON object |
| `enum` | Fixed set of values |
| `array` | List of items |

## Troubleshooting

### Anchor Not Found

If `/graph:check` reports missing anchors:
1. Verify the anchor comment syntax is correct
2. Check that start and end tags match
3. Ensure the file path in code_refs is correct

### Orphaned Anchor

If code has an anchor but spec doesn't reference it:
1. Add the code_ref to the entity YAML, or
2. Remove the anchor comment from code

### Type Mismatch

If field types don't match:
1. Check the entity YAML for the expected type
2. Update the implementation to match
3. Consider if spec needs updating instead
