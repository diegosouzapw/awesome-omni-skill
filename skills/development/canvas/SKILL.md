---
name: canvas
description: Instructions for rendering rich A2UI canvas elements to the chat interface. Auto-injected for web UI sessions.
category: core
triggers: canvas, ui, interface, form, a2ui, interactive
alwaysApply: true
injectionFormat: content
metadata:
  gobby:
    audience: all
    sources: [claude_sdk_web_chat, gemini_sdk_web_chat]
---

# Canvas Frontend Integration

When interacting with a user in the web chat interface, you can render rich interactive elements (forms, layouts, status cards) directly in their chat view using the Canvas tools.

## When to use the Canvas

Use the canvas when you need to:
- Collect multiple pieces of structured input (e.g., a configuration form with checkboxes and text fields).
- Present a complex dashboard or interactive summary.
- Display a rich card with actionable buttons instead of a plain text list.
- Keep the chat clean by replacing long text prompts with a visual form.

## Available Tools

The canvas tools are available through the `gobby-canvas` (or internal MCP) server:

1. `render_surface`: Renders a declarative JSON UI. Pass a flat `components` map (keyed by component ID), a `root_id`, and an initial `data_model`.
2. `update_surface`: Patches an existing canvas with new components or data.
3. `wait_for_interaction`: Pauses execution until the user interacts with the canvas (e.g., clicking a submit button). Returns the action name and the updated data model.
4. `close_canvas`: Manually closes/completes the canvas when interaction is done.
5. `canvas_present`: Present a local HTML file in the Canvas panel sandbox (iframe).

## A2UI Component System

A2UI is a declarative JSON layout system. Components are defined in a flat surface map keyed by unique component IDs. Parent components reference children by ID via `children: { explicitList: ["child-id-1", "child-id-2"] }`.

Key components:
- Layouts: `Column`, `Row`, `Card`, `List`
- Inputs: `TextField`, `CheckBox`
- Displays: `Text`, `Badge`, `Icon`, `Image`
- Actions: `Button`

### Data Binding

Text and labels use `BoundValue` objects:
- Literal text: `{ "literalString": "Hello" }`
- Data-bound: `{ "path": "user/name" }` (resolves from the `data_model`)

### Actions

Buttons define actions as: `"actions": [{ "name": "action_name", "context": { "key": { "path": "field/path" } } }]`

## Example Workflow

```json
// 1. Define a flat component surface map
{
  "components": {
    "root": {
      "type": "Card",
      "label": { "literalString": "Configuration" },
      "children": { "explicitList": ["username-field", "save-btn"] }
    },
    "username-field": {
      "type": "TextField",
      "label": { "literalString": "Username" },
      "value": { "path": "username" }
    },
    "save-btn": {
      "type": "Button",
      "label": { "literalString": "Save" },
      "actions": [{ "name": "save_config", "context": { "username": { "path": "username" } } }]
    }
  },
  "root_id": "root",
  "data_model": { "username": "admin" },
  "blocking": true,
  "timeout": 300
}
```

```python
# 1. Render the surface
result = call_tool("gobby-canvas", "render_surface", {
    "components": {
        "root": {
            "type": "Card",
            "label": {"literalString": "Configuration"},
            "children": {"explicitList": ["username-field", "save-btn"]}
        },
        "username-field": {
            "type": "TextField",
            "label": {"literalString": "Username"},
            "value": {"path": "username"}
        },
        "save-btn": {
            "type": "Button",
            "label": {"literalString": "Save"},
            "actions": [{"name": "save_config", "context": {"username": {"path": "username"}}}]
        }
    },
    "root_id": "root",
    "data_model": {"username": "admin"},
    "blocking": True,
    "timeout": 300
})

# blocking=True means render_surface waits for interaction and returns the result directly
canvas_id = result.get("canvas_id")
action = result.get("action")

if action and action.get("name") == "save_config":
    username = action.get("context", {}).get("username")
    # Process the data...

    # 2. Close the canvas
    call_tool("gobby-canvas", "close_canvas", {"canvas_id": canvas_id})
```
