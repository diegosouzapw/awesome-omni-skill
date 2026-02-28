---
name: adf-format-json-schema
description: Query Atlassian Document Format (ADF) JSON schema definitions to understand ADF node and mark types. Use this skill when implementing ADF dataclass nodes/marks, or when user asks about ADF structure, ADF nodes, ADF marks, or Atlassian Document Format implementation.
---

# ADF JSON Schema Query Skill

This skill provides tools to query the Atlassian Document Format (ADF) JSON schema, which defines the structure of all node and mark types used in Confluence pages and Jira issue fields.

## Purpose

When implementing ADF nodes or marks as Python dataclasses, you need to understand:
- What properties each type has
- Which properties are required vs optional
- The allowed values for each property

This skill helps you query the schema to get this information.

## Commands

### List All Definitions

Lists all available node and mark definitions in the ADF schema:

```bash
.venv/bin/python ./.claude/skills/adf-format-json-schema/scripts/adf_schema_cli.py list_def
```

Output format: `name = <definition_name>, properties = [<property_list>]`

Definitions follow naming conventions:
- `*_node` - Node types (e.g., `paragraph_node`, `heading_node`)
- `*_mark` - Mark types (e.g., `strong_mark`, `em_mark`, `link_mark`)

### Get Definition Details

Gets the full JSON schema for a specific definition:

```bash
.venv/bin/python ./.claude/skills/adf-format-json-schema/scripts/adf_schema_cli.py get_def <definition_name>
```

Example:
```bash
.venv/bin/python ./.claude/skills/adf-format-json-schema/scripts/adf_schema_cli.py get_def strong_mark
```

## Workflow

When implementing an ADF node or mark:

1. **List definitions** to find the exact definition name
2. **Get definition details** to understand the schema structure
3. **Implement the dataclass** based on the schema properties and requirements
