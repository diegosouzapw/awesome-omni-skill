---
name: MCP Tool Design
description: This skill should be used when the user asks to "define a tool", "create tool schema", "design inputSchema", "add tool annotations", "handle file uploads", "tool descriptions", "tool naming", or needs guidance on designing MCP tools for the OpenAI Apps SDK including schemas, annotations, and best practices.
version: 0.1.0
---

# MCP Tool Design for OpenAI Apps SDK

## Overview

Tools are the primary interface between ChatGPT and your MCP server. Well-designed tools enable natural conversations, proper model behavior, and secure operations. This skill covers tool definition patterns, schema design, annotations, and best practices.

## Tool Definition Structure

Every tool requires these components:

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Machine-readable identifier |
| `description` | Yes | Human-readable explanation for the model |
| `inputSchema` | Yes | JSON Schema defining parameters |
| `annotations` | No | Behavioral hints (read-only, destructive, etc.) |
| `_meta` | No | Apps SDK extensions (file params, visibility) |

## Tool Naming Best Practices

**Do:**
- Use verb_noun format: `get_order`, `create_user`, `search_products`
- Be specific and descriptive: `get_order_status` not `get_status`
- Use lowercase with underscores
- Keep names under 64 characters

**Don't:**
- Use promotional language: `best_search`, `official_api`
- Use generic names: `do_thing`, `process`
- Include version numbers: `search_v2`

## Input Schema Design

### Basic Schema

```json
{
  "type": "object",
  "properties": {
    "query": {
      "type": "string",
      "description": "Search query to find products"
    },
    "limit": {
      "type": "integer",
      "description": "Maximum results to return",
      "default": 10
    }
  },
  "required": ["query"]
}
```

### Schema with Enums

```json
{
  "type": "object",
  "properties": {
    "category": {
      "type": "string",
      "enum": ["electronics", "clothing", "books"],
      "description": "Product category to filter by"
    }
  }
}
```

### Nested Objects

```json
{
  "type": "object",
  "properties": {
    "filter": {
      "type": "object",
      "properties": {
        "minPrice": { "type": "number" },
        "maxPrice": { "type": "number" }
      }
    }
  }
}
```

## Tool Annotations

Annotations inform ChatGPT about tool behavior and safety:

| Annotation | Default | Purpose |
|------------|---------|---------|
| `readOnlyHint` | false | Safe to auto-approve, no side effects |
| `destructiveHint` | false | May delete/overwrite data, requires confirmation |
| `openWorldHint` | false | Publishes to external systems |
| `idempotentHint` | false | Repeated calls have no additional effect |

### Python Example

```python
@mcp.tool(
    annotations={
        "readOnlyHint": True,
        "idempotentHint": True
    }
)
def get_user(user_id: str) -> dict:
    """Retrieve user information by ID."""
    return fetch_user(user_id)

@mcp.tool(
    annotations={
        "destructiveHint": True
    }
)
def delete_user(user_id: str) -> dict:
    """Permanently delete a user account."""
    return remove_user(user_id)
```

### TypeScript Example

```typescript
{
  name: "get_user",
  description: "Retrieve user information by ID",
  inputSchema: { /* ... */ },
  annotations: {
    readOnlyHint: true,
    idempotentHint: true
  }
}
```

## Apps SDK _meta Extensions

### Output Template

Link tools to widget UIs:

```python
@mcp.tool()
def get_dashboard() -> dict:
    return {
        "structuredContent": {"data": [...]},
        "_meta": {
            "openai/outputTemplate": "ui://widget/dashboard.html"
        }
    }
```

### Widget Accessibility

Enable widgets to call this tool:

```json
{
  "_meta": {
    "openai/widgetAccessible": true
  }
}
```

### Tool Visibility

Hide tools from the model (only callable from widgets):

```json
{
  "_meta": {
    "openai/visibility": "private"
  }
}
```

### File Parameters

Accept file uploads:

```json
{
  "name": "process_image",
  "inputSchema": {
    "type": "object",
    "properties": {
      "image": {
        "type": "object",
        "properties": {
          "download_url": { "type": "string" },
          "file_id": { "type": "string" }
        }
      }
    }
  },
  "_meta": {
    "openai/fileParams": ["image"]
  }
}
```

## Tool Description Guidelines

Descriptions help the model understand when and how to use tools:

**Good:**
```
"Search for products in the catalog. Returns matching products with prices and availability. Use when the user wants to find or browse products."
```

**Bad:**
```
"Search function"
```

### Description Components

1. **What it does** - Primary action
2. **What it returns** - Output format
3. **When to use** - Context for the model

## Response Patterns

### Standard Response

```python
return {
    "structuredContent": {
        "status": "success",
        "data": result
    },
    "content": [
        {"type": "text", "text": "Operation completed"}
    ]
}
```

### Response with Widget

```python
return {
    "structuredContent": {
        "summary": "Found 5 items"
    },
    "_meta": {
        "items": full_item_list,  # Widget-only data
        "openai/outputTemplate": "ui://widget/list.html"
    }
}
```

### Error Response

```python
return {
    "structuredContent": {
        "status": "error",
        "error": "Item not found"
    },
    "content": [
        {"type": "text", "text": "Could not find the requested item"}
    ],
    "isError": True
}
```

## Tool Filtering

Limit which tools the model sees to reduce context and improve accuracy:

```python
# Responses API configuration
{
    "type": "mcp",
    "server_url": "https://example.com/mcp",
    "allowed_tools": ["search", "get_details"],  # Only expose these
    "require_approval": "never"
}
```

## Validation Best Practices

1. **Validate early** - Check inputs at the start of tool handlers
2. **Return clear errors** - Explain what went wrong
3. **Use strict schemas** - Define all constraints in JSON Schema
4. **Document constraints** - Include limits in descriptions

```python
@mcp.tool()
def search(query: str, limit: int = 10) -> dict:
    """Search products. Query must be 1-100 chars, limit 1-50."""
    if not 1 <= len(query) <= 100:
        return {"error": "Query must be 1-100 characters"}
    if not 1 <= limit <= 50:
        return {"error": "Limit must be 1-50"}
    return perform_search(query, limit)
```

## Additional Resources

### Reference Files

For detailed patterns and examples:
- **`references/schema-patterns.md`** - Advanced JSON Schema patterns
- **`references/annotation-guide.md`** - Complete annotation reference

### Example Files

Working examples in `examples/`:
- **`examples/tool-definitions.py`** - Python tool examples
- **`examples/tool-definitions.ts`** - TypeScript tool examples

### Official Documentation

- Apps SDK Reference: https://developers.openai.com/apps-sdk/reference/
- JSON Schema: https://json-schema.org/understanding-json-schema/
