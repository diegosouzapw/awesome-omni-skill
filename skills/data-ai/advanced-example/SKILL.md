---
name: "advanced-example"
description: "Advanced example showing all available metadata fields and complex folder structure"
version: "2.0.1"
author: "MCP Skills Team"
created: "2025-01-10"
updated: "2025-10-23"

dependencies:
  python: ["pydantic>=2.0.0", "aiofiles>=23.0.0"]
  system: ["git", "curl"]

category: "advanced-examples"
tags: ["example", "tutorial", "advanced", "complete"]
complexity: "advanced"

when_to_use:
  - "When you need a complete reference for all metadata fields"
  - "When building complex skills with multiple dependencies"
  - "When you want to include multiple example files and templates"
  - "When organizing skills into categories and tags"

related_skills: ["minimal-example", "intermediate-example"]

has_examples: true
example_files:
  - "examples/example1.py"
  - "examples/example2.txt"
  - "templates/template.txt"
---

# Advanced Example Skill

This skill demonstrates **all available metadata fields** and shows how to
organize a complex skill with multiple subdirectories, examples, and templates.

## Complete Folder Structure

```
advanced-example/
├── SKILL.md              ← This file with full metadata
├── examples/             ← Example files directory
│   ├── example1.py       ← Python example
│   └── example2.txt      ← Text example
├── templates/            ← Template files directory
│   └── template.txt      ← Template file
└── README.md             ← Additional documentation (optional)
```

## All Metadata Fields Explained

### Required Fields
- **name**: Unique identifier (kebab-case recommended)
- **description**: Brief, clear description of what the skill does

### Version and Authorship
- **version**: Semantic version (MAJOR.MINOR.PATCH)
- **author**: Creator's name or team
- **created**: Creation date (ISO format: YYYY-MM-DD)
- **updated**: Last update date (ISO format)

### Dependencies

You can specify dependencies in two formats:

**Nested format (recommended for categorized deps):**
```yaml
dependencies:
  python: ["package>=1.0.0", "another-package"]
  system: ["git", "curl"]
```

**Flat format (simpler):**
```yaml
dependencies: ["package1", "package2", "tool1"]
```

### Categorization and Discovery
- **category**: Main category for grouping (e.g., "data-analysis", "document-creation")
- **tags**: Array of tags for search and filtering
- **complexity**: Difficulty level ("beginner", "intermediate", or "advanced")

### Usage Guidance
- **when_to_use**: Array of scenarios when this skill should be used
- **related_skills**: Array of related skill names for cross-referencing

### Examples
- **has_examples**: Boolean flag indicating if examples are included
- **example_files**: Array of paths to example files (relative to skill folder)

## Dependency Management

The server supports tracking dependencies so users know what's required:

### Python Dependencies
```yaml
dependencies:
  python: ["pydantic>=2.0.0", "numpy>=1.20.0"]
```

### System Dependencies
```yaml
dependencies:
  system: ["git", "curl", "jq"]
```

### Mixed Dependencies
```yaml
dependencies:
  python: ["requests>=2.28.0"]
  system: ["ffmpeg"]
  node: ["typescript"]
```

## Using When-To-Use

The `when_to_use` field helps Claude understand when to apply this skill:

```yaml
when_to_use:
  - "When working with complex data transformations"
  - "When you need to generate reports from structured data"
  - "When combining multiple data sources"
```

## Related Skills

Link to other skills to build a knowledge graph:

```yaml
related_skills: ["data-cleaning", "visualization-basics", "export-formats"]
```

## Organizing Example Files

### Examples Directory
Place working examples that demonstrate the skill:
- Code samples (`.py`, `.js`, `.go`, etc.)
- Data files (`.json`, `.csv`, `.xml`)
- Configuration files

### Templates Directory
Include reusable templates:
- File templates
- Code boilerplates
- Configuration templates

### Referencing Examples in Content

You can reference your examples in the markdown content:

- See `examples/example1.py` for a working implementation
- Check `templates/template.txt` for the base template
- Review `examples/example2.txt` for sample data format

## Hot-Reload Support

When hot-reload is enabled, any changes to this SKILL.md file will be
automatically detected and the skill will be reloaded without restarting
the server!

Try it:
1. Edit this file
2. Save it
3. Watch the server logs - you'll see the reload happening

## Advanced Features

### Complexity Levels
- **beginner**: Basic skills, minimal prerequisites
- **intermediate**: Requires some background knowledge
- **advanced**: Complex skills with multiple dependencies

### Category Organization
Skills are grouped by category in the catalog. Choose descriptive categories:
- `document-creation`
- `data-analysis`
- `automation`
- `web-scraping`
- etc.

### Tag Best Practices
Use lowercase tags, be specific but not too narrow:
- Good: `["excel", "formulas", "automation"]`
- Avoid: `["Excel", "EXCEL_FORMULAS", "automation-tool"]`

## Validation

The server validates:
✓ Folder structure (skill must be in dedicated folder)
✓ Required fields (name, description)
✓ YAML syntax
✓ File references (example files must exist)
✓ Complexity values (must be beginner/intermediate/advanced)

## Best Practices Summary

1. **One skill per folder** - Each skill in its own directory
2. **Meaningful names** - Use descriptive, kebab-case names
3. **Complete metadata** - Fill in as many fields as applicable
4. **Organize files** - Use subdirectories (examples/, templates/, docs/)
5. **Version your skills** - Use semantic versioning
6. **Document dependencies** - Be explicit about requirements
7. **Provide examples** - Working examples are invaluable
8. **Use tags wisely** - Make skills discoverable
9. **Link related skills** - Build a knowledge graph
10. **Keep it updated** - Update the `updated` field when you make changes

## Conclusion

This advanced example demonstrates the full power of the MCP Skills Server's
metadata system. You can use as many or as few of these fields as needed for
your specific use case.

Start simple with `minimal-example`, grow with `intermediate-example`, and
reference this `advanced-example` when you need the full feature set!
