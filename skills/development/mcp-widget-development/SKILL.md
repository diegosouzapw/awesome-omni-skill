---
name: MCP Widget Development
description: This skill should be used when the user asks to "build a widget", "create UI component", "ChatGPT UI", "window.openai API", "widget template", "skybridge", "render in ChatGPT", "CSP configuration", or needs guidance on building interactive UI components for OpenAI Apps SDK that render inside ChatGPT.
version: 0.1.0
---

# MCP Widget Development for OpenAI Apps SDK

## Overview

Widgets are interactive UI components that render inside ChatGPT conversations. Built with HTML, CSS, and JavaScript, they run in a sandboxed iframe and communicate with the MCP server through the `window.openai` bridge.

## Widget Architecture

```
Tool Call → Server Returns _meta.openai/outputTemplate →
ChatGPT Loads Widget HTML → Widget Reads window.openai.toolOutput →
Widget Renders UI → User Interacts → Widget Calls Tools (optional)
```

### Key Components

| Component | Purpose |
|-----------|---------|
| HTML Template | Widget markup and styles |
| `window.openai` | Bridge to ChatGPT runtime |
| `_meta` | Widget-only data from server |
| CSP Config | Security allowlists |

## Registering Widget Templates

Widgets are served as MCP resources with the special mime type `text/html+skybridge`.

### Python

```python
@mcp.resource("ui://widget/main.html")
def main_widget() -> str:
    return """<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>/* styles */</style>
</head>
<body>
    <div id="app"></div>
    <script>/* widget code */</script>
</body>
</html>"""
```

### TypeScript

```typescript
server.setRequestHandler(ReadResourceRequestSchema, async (request) => ({
  contents: [{
    uri: request.params.uri,
    mimeType: "text/html+skybridge",
    text: widgetHtml
  }]
}));
```

## The window.openai Bridge

The `window.openai` object provides access to ChatGPT runtime:

### Data Access

```javascript
// Tool input parameters
const input = window.openai.toolInput;

// Tool output (structuredContent + _meta)
const output = window.openai.toolOutput;

// Response metadata
const meta = window.openai.toolResponseMetadata;
```

### Context Information

```javascript
// Theme: "light" or "dark"
const theme = window.openai.theme;

// Display mode: "inline" or "modal"
const displayMode = window.openai.displayMode;

// User's locale (BCP 47)
const locale = window.openai.locale;
```

### Tool Invocation

Call tools from the widget (requires `openai/widgetAccessible: true`):

```javascript
const result = await window.openai.callTool("tool_name", {
  param1: "value1",
  param2: "value2"
});
```

### State Management

```javascript
// Save widget state (persists across conversation turns)
await window.openai.setWidgetState({ key: "value" });

// Read previous state
const prevState = window.openai.toolResponseMetadata?.widgetState;
```

### Layout Control

```javascript
// Request modal display
await window.openai.requestModal();

// Set display mode
await window.openai.requestDisplayMode("modal"); // or "inline"

// Report content height for proper sizing
window.openai.notifyIntrinsicHeight(400);
```

### File Handling

```javascript
// Upload a file
const { fileId } = await window.openai.uploadFile(file);

// Get download URL for a file
const { downloadUrl } = await window.openai.getFileDownloadUrl({ fileId });
```

## Widget Template Structure

### Basic Template

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: system-ui, -apple-system, sans-serif;
            padding: 16px;
            background: var(--bg-color, #ffffff);
            color: var(--text-color, #000000);
        }
        /* Theme support */
        body.dark {
            --bg-color: #1a1a1a;
            --text-color: #ffffff;
        }
    </style>
</head>
<body>
    <div id="app">Loading...</div>

    <script>
        // Apply theme
        if (window.openai?.theme === 'dark') {
            document.body.classList.add('dark');
        }

        // Get data from tool response
        const data = window.openai?.toolOutput?.structuredContent;
        const meta = window.openai?.toolOutput?._meta;

        // Render content
        const app = document.getElementById('app');
        if (data) {
            app.innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
        }

        // Report height
        window.openai?.notifyIntrinsicHeight(document.body.scrollHeight);
    </script>
</body>
</html>
```

### Interactive Widget

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>
        .card { padding: 16px; border: 1px solid #ddd; border-radius: 8px; }
        .btn { padding: 8px 16px; background: #0066cc; color: white; border: none; border-radius: 4px; cursor: pointer; }
        .btn:hover { background: #0055aa; }
    </style>
</head>
<body>
    <div class="card">
        <h2 id="title">Item</h2>
        <p id="description"></p>
        <button class="btn" id="action">Take Action</button>
    </div>

    <script>
        const data = window.openai?.toolOutput?._meta?.item;

        document.getElementById('title').textContent = data?.name || 'Unknown';
        document.getElementById('description').textContent = data?.description || '';

        document.getElementById('action').addEventListener('click', async () => {
            try {
                const result = await window.openai.callTool('process_item', {
                    itemId: data?.id
                });
                alert('Success: ' + JSON.stringify(result));
            } catch (err) {
                alert('Error: ' + err.message);
            }
        });
    </script>
</body>
</html>
```

## Linking Tools to Widgets

### Server-Side

Return the widget URI in `_meta.openai/outputTemplate`:

```python
@mcp.tool()
def get_dashboard() -> dict:
    return {
        "structuredContent": {"summary": "Dashboard loaded"},
        "_meta": {
            "fullData": {...},  # Widget-only
            "openai/outputTemplate": "ui://widget/dashboard.html"
        }
    }
```

### Widget-Accessible Tools

Enable tools to be called from widgets:

```python
@mcp.tool()
def refresh_data() -> dict:
    return {
        "structuredContent": {"data": [...]},
        "_meta": {
            "openai/widgetAccessible": True
        }
    }
```

## CSP Configuration

Configure Content Security Policy for widgets:

```python
@mcp.resource("ui://widget/main.html")
def widget() -> dict:
    return {
        "contents": [{
            "uri": "ui://widget/main.html",
            "mimeType": "text/html+skybridge",
            "text": html_content
        }],
        "_meta": {
            "openai/widgetCSP": {
                "connect_domains": ["api.example.com"],
                "resource_domains": ["cdn.example.com"],
                "frame_domains": ["embed.example.com"]
            }
        }
    }
```

## Theme Support

Handle light and dark themes:

```javascript
const theme = window.openai?.theme || 'light';
document.documentElement.setAttribute('data-theme', theme);
```

```css
:root {
    --bg: #ffffff;
    --text: #000000;
}

[data-theme="dark"] {
    --bg: #1a1a1a;
    --text: #ffffff;
}

body {
    background: var(--bg);
    color: var(--text);
}
```

## Best Practices

1. **Keep widgets lightweight** - Minimize bundle size for fast loading
2. **Handle missing data gracefully** - Check for null/undefined
3. **Support both themes** - Test in light and dark mode
4. **Report content height** - Call `notifyIntrinsicHeight` after render
5. **Use semantic HTML** - Improve accessibility
6. **Avoid external dependencies** - Inline all code when possible

## Additional Resources

### Reference Files

For detailed patterns and examples:
- **`references/window-openai-api.md`** - Complete window.openai API reference
- **`references/csp-guide.md`** - CSP configuration guide

### Example Files

Working examples in `examples/`:
- **`examples/basic-widget.html`** - Simple data display widget
- **`examples/interactive-widget.html`** - Widget with tool calls

### Official Documentation

- Apps SDK UI Guidelines: https://developers.openai.com/apps-sdk/concepts/ui-guidelines/
- Widget Reference: https://developers.openai.com/apps-sdk/reference/
