---
name: project-logger
description: SQLite-based project documentation logger for tracking API references, components, and project progress. Use this skill when documenting code changes, adding API documentation, recording component updates, or tracking project milestones. Automatically invoked when user mentions documentation, changelog, API docs, component docs, or project updates.
disable-model-invocation: false
user-invocable: true
allowed-tools: Bash(python:*), Read, Write
argument-hint: [action] [type] [args...]
---

# Project Logger Skill

A SQLite-based documentation system for managing project documentation through Agent interactions. This replaces traditional markdown-based documentation with a structured database approach.

## Overview

This skill provides a programmatic way to manage three types of documentation:

| Type | Description | Table |
|------|-------------|-------|
| **API** | HTTP endpoints, request/response formats, status codes | `api_docs` |
| **Component** | React components, props, events, usage examples | `components` |
| **Project** | Milestones, changes, progress tracking | `projects` |

## Quick Start

### Initialize Database

```bash
python ~/skills/project-logger/scripts/logger.py init
```

### Add Documentation

```bash
# Add API documentation
python ~/skills/project-logger/scripts/logger.py add api --name "Chat API" --path "/api/chat" --method "POST" --description "AI chat endpoint"

# Add Component documentation  
python ~/skills/project-logger/scripts/logger.py add component --name "ChatPanel" --description "Main chat interface component"

# Add Project milestone
python ~/skills/project-logger/scripts/logger.py add project --title "v1.0 Release" --event "Added" --description "Initial release"
```

### Query Documentation

```bash
# List all entries
python ~/skills/project-logger/scripts/logger.py list api
python ~/skills/project-logger/scripts/logger.py list component
python ~/skills/project-logger/scripts/logger.py list project

# Search entries
python ~/skills/project-logger/scripts/logger.py search "chat"

# Get specific entry
python ~/skills/project-logger/scripts/logger.py get api --id 1
```

### Update Documentation

```bash
python ~/skills/project-logger/scripts/logger.py update api --id 1 --description "Updated description"
```

### Export to Markdown (Optional)

```bash
python ~/skills/project-logger/scripts/logger.py export --format markdown --output ./docs/
```

## Database Schema

The SQLite database is stored at `~/skills/project-logger/data/project_docs.db`

### Tables

- `api_docs` - API endpoint documentation
- `components` - React component documentation  
- `projects` - Project changelog and milestones
- `doc_tags` - Tags for categorization
- `doc_tag_relations` - Many-to-many tag relationships

## Additional Resources

For detailed information, see:

- [Database Schema](./schema.md) - Complete table definitions
- [API Reference](./api-reference.md) - Full CLI command documentation
- [Examples](./examples.md) - Usage examples and patterns
- [Templates](./templates/) - Documentation templates

## Usage Patterns

### When Adding New Features

1. Add project entry with event "Added"
2. Add component entries for new UI components
3. Add API entries for new endpoints

### When Updating Existing Features

1. Add project entry with event "Updated"
2. Update component/API entries with new details

### When Removing Features

1. Add project entry with event "Removed"
2. Mark component/API entries as deprecated

## Integration with CI/CD

The logger can be integrated into your CI/CD pipeline:

```yaml
# ~/workflows/docs.yml
- name: Generate docs
  run: python ~/skills/project-logger/scripts/logger.py export --format markdown
```

## Best Practices

1. **Always timestamp entries** - The system auto-generates timestamps
2. **Use consistent naming** - Follow naming conventions for entries
3. **Add tags for searchability** - Tag entries for easier discovery
4. **Keep descriptions concise** - Detailed info goes in specific fields
5. **Link related entries** - Reference component IDs in API docs when relevant
