---
name: datasette-plugins
description: "Writing Datasette plugins using Python and the pluggy plugin system. Use when Claude needs to: (1) Create a new Datasette plugin, (2) Implement plugin hooks like prepare_connection, register_routes, render_cell, etc., (3) Add custom SQL functions, (4) Create custom output renderers, (5) Add authentication or permissions logic, (6) Extend Datasette's UI with menus, actions, or templates, (7) Package a plugin for distribution on PyPI"
---

# Datasette Plugin Development

## Overview

Datasette plugins extend Datasette's functionality using Python and the [pluggy](https://pluggy.readthedocs.io/) plugin system. Plugins can add SQL functions, custom routes, authentication, UI elements, and more.

## Quick Start: One-off Plugin

Create `plugins/my_plugin.py`:

```python
from datasette import hookimpl

@hookimpl
def prepare_connection(conn):
    conn.create_function("hello_world", 0, lambda: "Hello world!")
```

Run with: `datasette serve mydb.db --plugins-dir=plugins/`

## Installable Plugin Structure

For distributable plugins, use this structure:

```
datasette-my-plugin/
├── pyproject.toml
├── datasette_my_plugin/
│   ├── __init__.py      # Plugin implementation
│   ├── static/          # Optional: JS/CSS files
│   └── templates/       # Optional: Jinja2 templates
└── tests/
    └── test_plugin.py
```

### pyproject.toml

```toml
[project]
name = "datasette-my-plugin"
version = "0.1.0"
description = "My Datasette plugin"
requires-python = ">=3.10"
dependencies = ["datasette"]
[dependency-groups]
dev = [
    "pytest",
    "pytest-asyncio"
]

[project.entry-points.datasette]
my_plugin = "datasette_my_plugin"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

## Core Plugin Hooks

See [references/hooks.md](references/hooks.md) for complete hook documentation.

### Most Common Hooks

| Hook | Purpose |
|------|---------|
| `prepare_connection(conn, database, datasette)` | Register custom SQL functions |
| `register_routes(datasette)` | Add custom URL routes |
| `startup(datasette)` | Initialize on server start |
| `render_cell(row, value, column, table, database, datasette, request)` | Customize cell display |
| `extra_template_vars(...)` | Add template variables |
| `actor_from_request(datasette, request)` | Custom authentication |
| `permission_allowed(datasette, actor, action, resource)` | Custom permissions |

### Example: Custom SQL Function

```python
from datasette import hookimpl
import hashlib

@hookimpl
def prepare_connection(conn):
    conn.create_function("md5", 1, lambda s: hashlib.md5(s.encode()).hexdigest())
```

### Example: Custom Route

```python
from datasette import hookimpl, Response

@hookimpl
def register_routes():
    return [
        (r"^/-/my-page$", my_page_view),
    ]

async def my_page_view(datasette, request):
    return Response.html("<h1>My Custom Page</h1>")
```

### Example: Startup Hook

```python
@hookimpl
def startup(datasette):
    async def inner():
        db = datasette.get_database()
        await db.execute_write("""
            CREATE TABLE IF NOT EXISTS my_table (id INTEGER PRIMARY KEY, data TEXT)
        """)
    return inner
```

## Plugin Configuration

Plugins read configuration from `datasette.yaml`:

```yaml
plugins:
  datasette-my-plugin:
    option1: value1
    option2: value2
```

Access in plugin:

```python
@hookimpl
def startup(datasette):
    config = datasette.plugin_config("datasette-my-plugin") or {}
    my_option = config.get("option1", "default")
```

### Secret Configuration

Use environment variables:

```yaml
plugins:
  datasette-my-plugin:
    api_key:
      $env: MY_API_KEY
```

Or files:

```yaml
plugins:
  datasette-my-plugin:
    api_key:
      $file: /secrets/api-key
```

## Testing Plugins

```python
from datasette.app import Datasette
import pytest

@pytest.mark.asyncio
async def test_plugin_installed():
    datasette = Datasette(memory=True)
    response = await datasette.client.get("/-/plugins.json")
    assert response.status_code == 200
    plugins = {p["name"] for p in response.json()}
    assert "datasette-my-plugin" in plugins

@pytest.mark.asyncio
async def test_custom_route():
    datasette = Datasette(memory=True)
    response = await datasette.client.get("/-/my-page")
    assert response.status_code == 200
    assert "My Custom Page" in response.text
```

Run tests: `pytest`

## Response Types

```python
from datasette import Response

# HTML response
Response.html("<h1>Hello</h1>")

# JSON response
Response.json({"key": "value"})

# Text response
Response.text("Plain text")

# Redirect
Response.redirect("/other-page")

# Custom response
Response(body, content_type="text/plain", status=200, headers={})
```

## URL Design

Use `/-/` prefix to avoid conflicts with database names:

- `/-/my-feature` - Global feature
- `/dbname/-/my-feature` - Database-specific
- `/dbname/tablename/-/my-feature` - Table-specific

## Static Assets & Templates

Static files in `static/` are served at:
`/-/static-plugins/PLUGIN_NAME/filename.js`

Templates in `templates/` override Datasette defaults. Priority:
1. `--template-dir` argument
2. Plugin templates
3. Datasette defaults

## Common Patterns

### Adding Menu Items

```python
@hookimpl
def menu_links(datasette, actor):
    return [{"href": "/-/my-page", "label": "My Feature"}]
```

### Table Actions

```python
@hookimpl
def table_actions(datasette, actor, database, table):
    return [{"href": f"/{database}/{table}/-/action", "label": "My Action"}]
```

### Custom Output Renderer

```python
@hookimpl
def register_output_renderer(datasette):
    return {
        "extension": "csv",
        "render": render_csv,
    }

async def render_csv(datasette, columns, rows):
    # Return Response object
    pass
```

### Event Tracking

```python
@hookimpl
def track_event(datasette, event):
    print(f"Event: {event.name}, Actor: {event.actor}")
```

## Debugging

Enable hook tracing:

```bash
DATASETTE_TRACE_PLUGINS=1 datasette mydb.db
```

## Key Imports

```python
from datasette import hookimpl, Response
from datasette.app import Datasette
from datasette.filters import FilterArguments
from datasette.permissions import Action, Resource, PermissionSQL
import markupsafe  # For safe HTML in render_cell
```
