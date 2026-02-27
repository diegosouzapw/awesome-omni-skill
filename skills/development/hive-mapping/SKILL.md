---
name: hive-mapping
description: Schema mappings for auto-syncing embedded documents
globs:
  - src/autoMap/schemaMappings.json
alwaysApply: false
---

# Schema Mappings

Auto-sync embedded references when source documents change.

Location: `/src/autoMap/schemaMappings.json`

## How It Works

1. You embed document references in schemas (e.g., `user: { _id, fullName }`)
2. You register the mapping in `schemaMappings.json`
3. When source document updates, all referencing documents sync automatically

## Config Format

```json
{
  "tasks": {
    "user": {
      "schema": "users"
    },
    "project": {
      "schema": "projects"
    }
  }
}
```

Means: In `tasks` collection, fields `user` and `project` reference `users` and `projects` collections.

## Example

**Schema with embedded reference:**
```javascript
// tasks.schema.js
project: z
  .object({
    _id: z.string(),
    name: z.coerce.string().nullable().optional(),
    logoUrl: z.coerce.string().nullable().optional(),
  })
  .nullable()
  .optional(),
```

**Mapping config:**
```json
{
  "tasks": {
    "project": {
      "schema": "projects"
    }
  }
}
```

**Result:** When project's `name` or `logoUrl` changes, all tasks with that project update automatically.

## Adding New Mapping

1. Define embedded reference in schema (include fields to sync)
2. Add entry to `schemaMappings.json`:

```json
{
  "yourResource": {
    "fieldName": {
      "schema": "sourceSchema"
    }
  }
}
```

## Rules

- Only fields defined in the embedded object shape are synced
- Works for both single references and arrays
- Changes sync on source document `updated` event
