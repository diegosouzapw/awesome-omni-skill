---
name: mcp-mcp-apps-kit
description: Guide for implementing MCP Apps (SEP-1865) - interactive UI extensions for MCP servers. Use when building MCP servers that need to return rich, interactive HTML-based UIs alongside tool results for conversational AI hosts like Claude Desktop or ChatGPT.
---

# MCP Apps Builder

## Overview

This skill provides comprehensive guidance for implementing **MCP Apps** - an extension to the Model Context Protocol (SEP-1865) that enables MCP servers to deliver interactive user interfaces to conversational AI hosts.

**Use this skill when:**

- Building MCP servers that need to return rich, interactive UIs alongside tool results
- Adding visual data presentation capabilities to existing MCP tools
- Creating interactive dashboards, forms, or visualizations within MCP-enabled clients
- Implementing bidirectional communication between UI components and MCP servers
- Migrating from MCP-UI or building Apps SDK-compatible MCP servers

## Core Concepts

### What are MCP Apps?

MCP Apps extend the Model Context Protocol with:

1. **UI Resources**: Predeclared HTML resources using the `ui://` URI scheme
2. **Tool-UI Linkage**: Tools reference UI resources via `_meta.ui.resourceUri` metadata
3. **Bidirectional Communication**: UI iframes communicate with hosts using JSON-RPC over postMessage
4. **Security Model**: Mandatory iframe sandboxing with Content Security Policy enforcement

### Key Pattern: Tool + UI Resource

MCP Apps follow a two-part registration pattern:

```typescript
// 1. Register the UI resource
server.registerResource({
  uri: "ui://my-server/dashboard",
  name: "Dashboard",
  mimeType: "text/html;profile=mcp-app",
  // HTML content returned via resources/read
});

// 2. Register a tool that references the UI
server.registerTool("get_data", {
  description: "Get data with interactive visualization",
  inputSchema: { /* ... */ },
  _meta: {
    ui: {
      resourceUri: "ui://my-server/dashboard"
    }
  }
});
```

## Implementation Workflow

Follow these steps in order to build an MCP App from scratch.

### Step 1: Design Your App

**Identify the use case:**

- What data does your tool return?
- How should that data be visualized?
- What user interactions are needed?
- Does the UI need to call back to the server?

**Plan the architecture:**

- Determine tool structure (inputs, outputs)
- Design UI layout and interactions
- Identify required external resources (APIs, CDNs)
- Plan CSP requirements for security

### Step 2: Implement the MCP Server

**Register UI resources:**

```typescript
const server = new McpServer({
  name: "my-app-server",
  version: "1.0.0"
});

// Register HTML resource
server.registerResource({
  uri: "ui://my-server/widget",
  name: "Interactive Widget",
  description: "Widget for displaying data",
  mimeType: "text/html;profile=mcp-app",
  _meta: {
    ui: {
      csp: {
        connectDomains: ["https://api.example.com"],
        resourceDomains: ["https://cdn.jsdelivr.net"]
      },
      prefersBorder: true
    }
  }
});

// Handle resource reads
server.setResourceHandler(async (uri) => {
  if (uri === "ui://my-server/widget") {
    const html = await fs.readFile("dist/widget.html", "utf-8");
    return {
      contents: [{
        uri,
        mimeType: "text/html;profile=mcp-app",
        text: html
      }]
    };
  }
});
```

**Link tools to UI resources:**

```typescript
server.registerTool("fetch_data", {
  title: "Fetch Data",
  description: "Fetches data and displays it interactively",
  inputSchema: {
    type: "object",
    properties: {
      query: { type: "string" }
    }
  },
  outputSchema: { /* ... */ },
  _meta: {
    ui: {
      resourceUri: "ui://my-server/widget",
      visibility: ["model", "app"] // Default: visible to both
    }
  }
}, async (args) => {
  const data = await fetchData(args.query);
  
  return {
    content: [
      { type: "text", text: `Found ${data.length} results` }
    ],
    structuredContent: data, // UI-optimized data
    _meta: {
      timestamp: new Date().toISOString()
    }
  };
});
```

**Tool visibility options:**

- `["model", "app"]` (default): Tool visible to agent and callable by app
- `["app"]`: Hidden from agent, only callable by app (for UI-only interactions like refresh buttons)
- `["model"]`: Visible to agent only, not callable by app

### Step 3: Build the UI

**Project setup:**

```bash
# Install dependencies
npm install @modelcontextprotocol/ext-apps @modelcontextprotocol/sdk
npm install -D vite vite-plugin-singlefile typescript
```

**Vite configuration (bundle to single HTML):**

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { viteSingleFile } from "vite-plugin-singlefile";

export default defineConfig({
  plugins: [viteSingleFile()],
  build: {
    outDir: "dist",
    rollupOptions: {
      input: process.env.INPUT || "app.html"
    }
  }
});
```

**HTML structure:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>My MCP App</title>
  </head>
  <body>
    <div id="app">Loading...</div>
    <script type="module" src="/src/app.ts"></script>
  </body>
</html>
```

**App initialization (Vanilla JS/TypeScript):**

```typescript
import { App, PostMessageTransport } from "@modelcontextprotocol/ext-apps";

const app = new App({
  name: "My MCP App",
  version: "1.0.0"
});

// Register handlers BEFORE connecting
app.ontoolresult = (result) => {
  const data = result.structuredContent;
  renderData(data);
};

app.onhostcontextchange = (context) => {
  // Handle theme changes
  if (context.theme) {
    applyTheme(context.theme);
  }
};

// Connect to host
await app.connect(new PostMessageTransport(window.parent));

// Now you can interact with the server
document.getElementById("refresh-btn")?.addEventListener("click", async () => {
  const result = await app.callServerTool({
    name: "fetch_data",
    arguments: { query: "latest" }
  });
  renderData(result.structuredContent);
});
```

**React version:**

```typescript
import { useApp, useToolResult, useHostContext } from "@modelcontextprotocol/ext-apps/react";

function MyApp() {
  const app = useApp({
    name: "My MCP App",
    version: "1.0.0"
  });
  
  const toolResult = useToolResult();
  const hostContext = useHostContext();
  
  const handleRefresh = async () => {
    await app.callServerTool({
      name: "fetch_data",
      arguments: { query: "latest" }
    });
  };
  
  return (
    <div style={{
      backgroundColor: `var(--color-background-primary)`,
      color: `var(--color-text-primary)`
    }}>
      <h1>Data Viewer</h1>
      <pre>{JSON.stringify(toolResult?.structuredContent, null, 2)}</pre>
      <button onClick={handleRefresh}>Refresh</button>
    </div>
  );
}
```

### Step 4: Apply Host Theming

**Use standardized CSS variables:**

```css
:root {
  /* Fallback defaults for graceful degradation */
  --color-background-primary: light-dark(#ffffff, #171717);
  --color-text-primary: light-dark(#171717, #fafafa);
  --font-sans: system-ui, -apple-system, sans-serif;
  --border-radius-md: 8px;
}

.container {
  background: var(--color-background-primary);
  color: var(--color-text-primary);
  font-family: var(--font-sans);
  border-radius: var(--border-radius-md);
}
```

See `references/css-variables.md` for the complete list of standardized CSS variables.

**Apply host-provided styles:**

```typescript
import { applyHostStyleVariables, applyDocumentTheme } from "@modelcontextprotocol/ext-apps";

app.onhostcontextchange = (context) => {
  // Apply CSS variables from host
  if (context.styles?.variables) {
    applyHostStyleVariables(context.styles.variables);
  }
  
  // Apply theme class (light/dark)
  if (context.theme) {
    applyDocumentTheme(context.theme);
  }
  
  // Apply custom fonts
  if (context.styles?.css?.fonts) {
    const style = document.createElement("style");
    style.textContent = context.styles.css.fonts;
    document.head.appendChild(style);
  }
};
```

**React hooks:**

```typescript
import { useHostStyleVariables, useDocumentTheme } from "@modelcontextprotocol/ext-apps/react";

function MyApp() {
  useHostStyleVariables(); // Automatically applies CSS variables
  useDocumentTheme();      // Automatically applies theme class
  
  return <div>Content styled by host</div>;
}
```

### Step 5: Implement Security

**Declare CSP requirements:**

```typescript
server.registerResource({
  uri: "ui://my-server/widget",
  name: "Widget",
  mimeType: "text/html;profile=mcp-app",
  _meta: {
    ui: {
      csp: {
        // Domains for fetch/XHR/WebSocket
        connectDomains: [
          "https://api.example.com",
          "wss://realtime.example.com"
        ],
        // Domains for images, scripts, stylesheets, fonts
        resourceDomains: [
          "https://cdn.jsdelivr.net",
          "https://*.cloudflare.com"
        ]
      },
      // Optional: dedicated domain for this widget
      domain: "https://widget.example.com",
      // Request visible border/background
      prefersBorder: true
    }
  }
});
```

**Security best practices:**

- Always declare all external domains in CSP
- Use HTTPS for all external resources
- Avoid `'unsafe-eval'` and minimize `'unsafe-inline'`
- Test your app with restrictive CSP during development
- Never transmit sensitive credentials through postMessage

### Step 6: Handle Lifecycle Events

```typescript
const app = new App({
  name: "My App",
  version: "1.0.0"
});

// Initialize lifecycle
app.oninitialized = (result) => {
  console.log("Connected to host:", result.hostInfo);
  console.log("Available display modes:", result.hostContext.availableDisplayModes);
};

// Tool execution lifecycle
app.ontoolinput = (input) => {
  console.log("Tool called with:", input);
  showLoadingState();
};

app.ontoolresult = (result) => {
  console.log("Tool result:", result);
  hideLoadingState();
  renderData(result.structuredContent);
};

app.ontoolcancelled = (reason) => {
  console.warn("Tool cancelled:", reason);
  hideLoadingState();
};

// Host context changes
app.onhostcontextchange = (context) => {
  if (context.theme) applyTheme(context.theme);
  if (context.viewport) handleResize(context.viewport);
};

// Cleanup
app.onteardown = (reason) => {
  console.log("Tearing down:", reason);
  cleanupResources();
};

await app.connect(new PostMessageTransport(window.parent));
```

### Step 7: Add Interactive Features

**Call server tools from UI:**

```typescript
// Call tools from button clicks, forms, etc.
async function handleAction() {
  try {
    const result = await app.callServerTool({
      name: "refresh_data",
      arguments: { filter: "active" }
    });
    
    updateUI(result.structuredContent);
  } catch (error) {
    showError(error.message);
  }
}
```

**Send messages to chat:**

```typescript
// Add message to conversation
await app.sendMessage({
  role: "user",
  content: {
    type: "text",
    text: "User clicked on item #123"
  }
});
```

**Send notifications (logs):**

```typescript
// Log to host console
await app.sendLog({
  level: "info",
  data: "Data refreshed successfully"
});
```

**Open external links:**

```typescript
// Open URL in user's browser
await app.sendOpenLink({
  url: "https://example.com/details/123"
});
```

**Request display mode changes:**

```typescript
// Request fullscreen mode
const result = await app.requestDisplayMode("fullscreen");
console.log("New display mode:", result.mode);
```

### Step 8: Test Your App

**Build the UI:**

```bash
npm run build
```

**Start your MCP server:**

```bash
node server.js
# or
npm run serve
```

**Test with basic-host (from ext-apps repo):**

```bash
# In a separate terminal
git clone https://github.com/modelcontextprotocol/ext-apps.git
cd ext-apps/examples/basic-host
npm install
npm run start

# Open http://localhost:8080
# Select your tool from the dropdown
# Click "Call Tool" to see the UI
```

**Test in Claude Desktop or other MCP host:**

1. Configure your server in Claude Desktop's MCP settings
2. Call your tool from the chat
3. Verify the UI renders correctly
4. Test interactions (buttons, forms, etc.)
5. Verify theming matches the host

## Advanced Patterns

### App-Only Tools (Hidden from Agent)

Create tools that are only callable by your UI, not by the agent:

```typescript
server.registerTool("ui_refresh", {
  description: "Refresh UI data (internal)",
  inputSchema: { type: "object" },
  _meta: {
    ui: {
      visibility: ["app"] // Hidden from agent
    }
  }
}, async () => {
  return {
    content: [{ type: "text", text: "Refreshed" }],
    structuredContent: await fetchLatestData()
  };
});
```

### Streaming Tool Updates

Receive partial updates during long-running tool execution:

```typescript
app.ontoolinputpartial = (partial) => {
  // Update UI with partial progress
  updateProgress(partial);
};
```

### Multi-Page Apps

Create multi-screen experiences by registering multiple UI resources:

```typescript
// Dashboard view
server.registerResource({
  uri: "ui://app/dashboard",
  name: "Dashboard",
  mimeType: "text/html;profile=mcp-app"
});

// Detail view
server.registerResource({
  uri: "ui://app/details",
  name: "Details",
  mimeType: "text/html;profile=mcp-app"
});

// Tools reference different views
server.registerTool("show_dashboard", {
  _meta: { ui: { resourceUri: "ui://app/dashboard" } }
});

server.registerTool("show_details", {
  _meta: { ui: { resourceUri: "ui://app/details" } }
});
```

### Reading Server Resources from UI

Access other MCP resources from your UI:

```typescript
// UI can read resources
const resource = await app.readResource({
  uri: "file:///config.json"
});

const config = JSON.parse(resource.contents[0].text);
```

## Capability Negotiation

**Server advertises MCP Apps support:**

```typescript
// Server initialization
const server = new McpServer({
  name: "my-server",
  version: "1.0.0",
  capabilities: {
    extensions: {
      "io.modelcontextprotocol/ui": {
        mimeTypes: ["text/html;profile=mcp-app"]
      }
    }
  }
});
```

**Check if host supports MCP Apps:**

```typescript
// In your tool handler
const hostSupportsUI = client.capabilities?.extensions?.["io.modelcontextprotocol/ui"];

if (hostSupportsUI) {
  // Return UI metadata
  return {
    content: [{ type: "text", text: "Data loaded" }],
    _meta: { ui: { resourceUri: "ui://app/view" } }
  };
} else {
  // Fallback to text-only
  return {
    content: [{ type: "text", text: formatDataAsText(data) }]
  };
}
```

## Resources

### References

- `references/spec.md` - Key excerpts from SEP-1865 MCP Apps specification
- `references/api-quick-reference.md` - Quick API reference for common operations
- `references/css-variables.md` - Complete list of standardized theming CSS variables

### Official Documentation

- [MCP Apps Repository](https://github.com/modelcontextprotocol/ext-apps) - Official SDK and examples
- [API Documentation](https://modelcontextprotocol.github.io/ext-apps/api/) - Complete API reference
- [Quickstart Guide](https://modelcontextprotocol.github.io/ext-apps/api/documents/Quickstart.html) - Step-by-step tutorial
- [Draft Specification](https://github.com/modelcontextprotocol/ext-apps/blob/main/specification/draft/apps.mdx) - Full SEP-1865 spec

### Examples

See the official repository's examples directory:

- `examples/basic-server-vanillajs` - Minimal vanilla JS example
- `examples/basic-server-react` - React implementation
- `examples/basic-host` - Test host for development

## Best Practices

### Performance

- Bundle UI into a single HTML file with Vite + vite-plugin-singlefile
- Minimize external dependencies to reduce load time
- Lazy-load heavy components
- Cache UI resources on the host side

### Accessibility

- Use semantic HTML elements
- Provide ARIA labels for interactive elements
- Support keyboard navigation
- Test with screen readers
- Respect host's font size preferences

### Responsive Design

- Use host's viewport information for layout decisions
- Support different display modes (inline, fullscreen, pip)
- Handle safe area insets for mobile devices
- Test on different screen sizes

### Security

- Declare all external domains explicitly in CSP
- Never store sensitive data in UI code
- Validate all user inputs before sending to server
- Use HTTPS for all external resources
- Follow the principle of least privilege for CSP

### UX Guidelines

- Provide loading states for async operations
- Show clear error messages to users
- Support host's theme (light/dark mode)
- Use host's typography and colors via CSS variables
- Provide meaningful fallbacks when features aren't available
- Handle tool cancellation gracefully

## Troubleshooting

### UI Not Rendering

- Verify `mimeType` is exactly `"text/html;profile=mcp-app"`
- Check that `resourceUri` in tool metadata matches registered resource URI
- Ensure host supports MCP Apps extension
- Verify HTML is valid and well-formed
- Check browser console for CSP violations

### CSP Errors

- Declare all external domains in `csp.connectDomains` or `csp.resourceDomains`
- Use wildcard subdomains carefully: `https://*.example.com`
- Test with strict CSP during development
- Check host's console for CSP violation reports

### Tool Not Visible to Agent

- Check `visibility` in `_meta.ui`: ensure it includes `"model"`
- Verify host properly filters tools based on visibility
- Confirm tool is returned in `tools/list` response

### Theming Not Working

- Verify fallback CSS variables are defined in `:root`
- Check if host is providing `styles.variables` in host context
- Use `applyHostStyleVariables` utility correctly
- Test with both light and dark themes

### Communication Errors

- Ensure `app.connect()` is called before any operations
- Verify PostMessageTransport is using `window.parent`
- Check browser console for JSONRPC errors
- Confirm server is responding to tool calls

## Migration from MCP-UI

**Key changes:**

1. **Resource metadata structure changed:**
   - Old: `_meta["ui/resourceUri"]`
   - New: `_meta.ui.resourceUri`

2. **Handshake protocol changed:**
   - Old: `iframe-ready` custom event
   - New: `ui/initialize` → `ui/notifications/initialized` (MCP-like)

3. **Tool visibility control:**
   - New: `_meta.ui.visibility` array

4. **CSP configuration:**
   - Moved from tool metadata to resource metadata
   - Separate `connectDomains` and `resourceDomains`

5. **Import paths:**
   - New: `@modelcontextprotocol/ext-apps` (not MCP-UI SDK)

## Limitations & Future Extensions

**Current MVP limitations:**

- Only `text/html;profile=mcp-app` content type supported
- No direct external URL embedding
- No widget-to-widget communication
- No state persistence between sessions
- Single UI resource per tool result

**Future extensions (deferred):**

- External URL content type (`text/uri-list`)
- Multiple UI resources per tool
- State persistence APIs
- Custom sandbox policies
- Screenshot/preview generation
- Remote DOM support

## Notes

- MCP Apps is an **optional extension** (SEP-1865) to MCP
- Must be explicitly negotiated via `io.modelcontextprotocol/ui` capability
- Backward compatible: tools work as text-only when host doesn't support UI
- Specification is in draft status; expect refinements before GA
- Based on learnings from MCP-UI community and OpenAI's Apps SDK
