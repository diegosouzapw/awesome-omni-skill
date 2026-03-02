---
name: article-queue
description: Manage article task queue - add, filter, update status, and track multi-language outputs
---

# Article Queue

Manage `.article_writer/article_tasks.json` for article creation.

## File Locations

- **Queue**: `.article_writer/article_tasks.json`
- **Authors**: `.article_writer/authors.json`
- **Schema**: `.article_writer/schemas/article-tasks.schema.json`
- **Backup**: `.article_writer/article_tasks.backup.json`

## Schema Reference

See [references/schema-reference.md](references/schema-reference.md) for fields.

## Key Fields

### Author Reference
```json
{
  "author": {
    "id": "mwguerra",
    "name": "MW Guerra",
    "languages": ["pt_BR", "en_US"]
  }
}
```

If author not specified, first author in authors.json is used.

### Output Files (per language)
```json
{
  "output_folder": "content/articles/2025_01_15_rate-limiting/",
  "output_files": [
    {
      "language": "pt_BR",
      "path": "content/articles/2025_01_15_rate-limiting/rate-limiting.pt_BR.md",
      "translated_at": "2025-01-15T14:00:00Z"
    },
    {
      "language": "en_US",
      "path": "content/articles/2025_01_15_rate-limiting/rate-limiting.en_US.md",
      "translated_at": "2025-01-15T16:00:00Z"
    }
  ]
}
```

### Timestamps
- `created_at`: When task was added to queue
- `written_at`: When primary article was completed
- `published_at`: When article went live
- `updated_at`: Last modification

## Operations

### Status Summary
```bash
bun run "${CLAUDE_PLUGIN_ROOT}"/scripts/queue.ts status
```

### Filter by Author
```javascript
articles.filter(a => a.author?.id === "mwguerra")
```

### Filter by Language
```javascript
articles.filter(a => 
  a.author?.languages?.includes("en_US")
)
```

### Update After Writing
```javascript
article.status = "draft";
article.output_folder = "content/articles/2025_01_15_slug/";
article.output_files = [
  { language: "pt_BR", path: "...", translated_at: "..." }
];
article.written_at = new Date().toISOString();
article.updated_at = new Date().toISOString();
```

### Add Translation
```javascript
article.output_files.push({
  language: "en_US",
  path: "content/articles/.../article.en_US.md",
  translated_at: new Date().toISOString()
});
```

## Status Flow

```
pending → in_progress → draft → review → published
               ↓
           archived
```

## Default Author

When adding tasks without author:
1. Load authors.json
2. Use first author's id, name, languages
3. Store reference in task

```javascript
const authors = JSON.parse(await readFile(".article_writer/authors.json"));
const defaultAuthor = authors.authors[0];
task.author = {
  id: defaultAuthor.id,
  name: defaultAuthor.name,
  languages: defaultAuthor.languages
};
```

## Validation

Before processing:
- Verify author.id exists in authors.json
- Validate languages are subset of author's languages
- Check all required fields present
