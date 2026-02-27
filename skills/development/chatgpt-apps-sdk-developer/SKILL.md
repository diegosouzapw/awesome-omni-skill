---
name: "chatgpt-apps-sdk-developer"
description: "Assists in developing ChatGPT Apps using the Apps SDK and Model Context Protocol (MCP). Invoke when the user wants to create, debug, or understand ChatGPT Apps, MCP servers, or their UI integration."
---

# ChatGPT Apps SDK Developer

This skill assists in developing applications for ChatGPT using the Apps SDK. ChatGPT Apps combine a UI (running in an iframe) with an MCP (Model Context Protocol) server to provide rich, interactive experiences within ChatGPT.

## Core Concepts

-   **MCP (Model Context Protocol)**: The backend logic. It exposes "tools" and "resources" to ChatGPT.
-   **Apps SDK**: The bridge between ChatGPT and your application. It allows the UI to communicate with ChatGPT and the MCP server.
-   **UI**: A web application (HTML/JS/React, etc.) displayed in an iframe. It communicates with ChatGPT via `postMessage`.

## Capabilities

1.  **Project Scaffolding**: Create starter templates for ChatGPT Apps.
    -   **Simple HTML/JS**: A single-file approach for simple tools (like the To-Do list quickstart).
    -   **Hono + MCP**: A TypeScript-based MCP server using Hono and `@hono/mcp`.
    -   **React/Next.js**: For more complex UIs.

2.  **MCP Server Implementation**:
    -   Define tools using `@modelcontextprotocol/sdk` or `@hono/mcp`.
    -   Connect to external APIs (e.g., microCMS, databases).
    -   Handle tool calls and return structured data.

3.  **UI Implementation**:
    -   Setup the `postMessage` bridge.
    -   Render data returned from MCP tool calls.
    -   Send actions back to ChatGPT/MCP.

## Quickstart Templates

### 1. Simple HTML/JS (No Build Step)

Use this for quick prototypes or simple widgets.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>My App</title>
    <script type="module">
        // Initialize the bridge
        // postMessage format: { type: 'mcp:...', ... }
    </script>
</head>
<body>
    <!-- UI Elements -->
</body>
</html>
```

### 2. Hono MCP Server

Use this for robust backend logic.

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { Hono } from "hono";

const server = new McpServer({
    name: "my-app",
    version: "1.0.0",
});

// Define tools
server.tool("get_data", {}, async () => {
    return { content: [{ type: "text", text: "Hello" }] };
});

// ... serve with Hono
```

## Development Workflow

1.  **Develop MCP Server**: Implement the backend logic and tools.
2.  **Develop UI**: Create the frontend that visualizes the tool outputs.
3.  **Test**: Use the ChatGPT Apps environment (currently in preview/beta) or an MCP client debugger.
4.  **Deploy**: Host the UI and MCP server (they can be on the same domain or different ones, subject to CORS/iframe policies).

## Reference Implementation (Z Coffee Example)

Based on the "Z Coffee" example:
-   **Data Source**: microCMS (headless CMS).
-   **Backend**: Node.js/Hono with MCP SDK.
-   **Frontend**: React (or simple HTML) displaying coffee shop locations and details.

## Troubleshooting

-   **CORS**: Ensure your server allows requests from ChatGPT's origin.
-   **Iframe**: The UI must be embeddable in an iframe (`X-Frame-Options` headers).
-   **Manifest**: Ensure the `openai.json` or equivalent manifest (if applicable for the specific deployment method) is correctly configured.

## Resources

-   [Apps SDK Documentation](https://developers.openai.com/apps-sdk)
-   [Quickstart](https://developers.openai.com/apps-sdk/quickstart)
