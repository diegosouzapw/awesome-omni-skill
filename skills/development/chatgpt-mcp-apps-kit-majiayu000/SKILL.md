---
name: chatgpt-mcp-apps-kit
description: Guide for implementing ChatGPT Apps using OpenAI Apps SDK. Use when building MCP servers with interactive UI components that integrate with ChatGPT, including widget runtime, authentication, state management, and deployment to the ChatGPT platform.
---

# ChatGPT Apps Builder

## Overview

This skill provides comprehensive guidance for implementing **ChatGPT Apps** using the OpenAI Apps SDK - a framework that enables MCP servers to deliver rich, interactive UI experiences directly within ChatGPT conversations.

**Use this skill when:**

- Building MCP servers that integrate with ChatGPT's widget runtime
- Creating interactive UI components that render inside ChatGPT conversations
- Implementing OAuth 2.1 authentication for user-specific data access
- Managing widget state and bidirectional communication with MCP servers
- Deploying and submitting apps to the ChatGPT platform
- Migrating from other approaches to the OpenAI Apps SDK framework

## Core Concepts

### What are ChatGPT Apps?

ChatGPT Apps are MCP-based applications with three key components:

1. **MCP Server**: Defines tools, enforces auth, returns data, and links tools to UI templates
2. **Widget/UI Bundle**: Renders inside ChatGPT's iframe using the `window.openai` runtime
3. **ChatGPT Model**: Decides when to call tools and narrates the experience using structured data

### Key Pattern: Tool → Data → Widget

```
User prompt
   ↓
ChatGPT model ──► MCP tool call ──► Your server ──► Tool response
   │                                     │
   │                                     ├─ structuredContent (model reads)
   │                                     ├─ content (narration)
   │                                     └─ _meta (widget-only data)
   │                                        │
   └───── renders narration ◄──── widget iframe ◄──────┘
                              (HTML template + window.openai)
```

### Architecture Components

**MCP Server:**
- Registers UI templates as resources with MIME type `text/html+skybridge`
- Defines tools with `_meta["openai/outputTemplate"]` pointing to UI resources
- Returns three data payloads: `structuredContent`, `content`, and `_meta`
- Handles authentication and authorization

**Widget Runtime (`window.openai`):**
- Provides `toolInput`, `toolOutput`, `toolResponseMetadata`, `widgetState`
- Enables bidirectional communication: `callTool`, `sendFollowUpMessage`
- File handling: `uploadFile`, `getFileDownloadUrl`
- Layout controls: `requestDisplayMode`, `requestModal`, `notifyIntrinsicHeight`
- Context signals: `theme`, `displayMode`, `locale`, `userAgent`

**Security Model:**
- Mandatory iframe sandboxing with Content Security Policy
- `openai/widgetCSP` configuration for allowed domains
- OAuth 2.1 with PKCE for user authentication
- Token verification on every MCP request

## Implementation Workflow

### Step 1: Set Up Your Project

**Project structure:**

```
your-chatgpt-app/
├─ server/
│  └─ src/index.ts          # MCP server + tool handlers
├─ web/
│  ├─ src/component.tsx     # React widget
│  └─ dist/app.{js,css}     # Bundled assets
└─ package.json
```

**Install dependencies:**

```bash
# MCP SDK
npm install @modelcontextprotocol/sdk zod

# React widget
cd web
npm install react@^18 react-dom@^18
npm install -D vite esbuild typescript
```

### Step 2: Register UI Templates

UI templates are MCP resources with `text/html+skybridge` MIME type:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { readFileSync } from "node:fs";

const server = new McpServer({ name: "my-app", version: "1.0.0" });

// Read bundled widget assets
const JS = readFileSync("web/dist/app.js", "utf8");
const CSS = readFileSync("web/dist/app.css", "utf8");

// Register widget template
server.registerResource(
  "app-widget",
  "ui://widget/app.html",
  {},
  async () => ({
    contents: [
      {
        uri: "ui://widget/app.html",
        mimeType: "text/html+skybridge",
        text: `
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>My App</title>
    <style>${CSS}</style>
  </head>
  <body>
    <div id="root"></div>
    <script type="module">${JS}</script>
  </body>
</html>
        `.trim(),
        _meta: {
          "openai/widgetPrefersBorder": true,
          "openai/widgetDomain": "https://chatgpt.com",
          "openai/widgetCSP": {
            connect_domains: ["https://api.yourservice.com"],
            resource_domains: ["https://*.oaistatic.com"],
            // Optional: redirect_domains, frame_domains
          },
        },
      },
    ],
  })
);
```

**Template metadata:**

- `openai/widgetPrefersBorder`: Show visual border around widget
- `openai/widgetDomain`: Dedicated origin for widget (enables fullscreen button)
- `openai/widgetCSP`: Content Security Policy configuration
  - `connect_domains`: APIs the widget can fetch from
  - `resource_domains`: CDNs for static assets (images, fonts, scripts)
  - `redirect_domains`: Hosts for `openExternal` without safe-link modal
  - `frame_domains`: Allowed iframe origins (discouraged, requires review)

**Cache busting:** When changing widget HTML/JS/CSS in breaking ways, use new URI or filename so ChatGPT loads fresh bundle.

### Step 3: Define Tools

Tools are the contract the ChatGPT model reasons about:

```typescript
import { z } from "zod";

server.registerTool(
  "search_restaurants",
  {
    title: "Search Restaurants",
    description: "Find restaurants matching criteria",
    inputSchema: {
      type: "object",
      properties: {
        location: { type: "string" },
        cuisine: { type: "string" },
      },
      required: ["location"],
    },
    _meta: {
      "openai/outputTemplate": "ui://widget/app.html",
      "openai/toolInvocation/invoking": "Searching restaurants…",
      "openai/toolInvocation/invoked": "Found restaurants",
      "openai/widgetAccessible": true, // Allow widget to call this tool
      "openai/visibility": "public", // "public" or "private"
    },
  },
  async ({ location, cuisine }) => {
    const restaurants = await searchRestaurants(location, cuisine);
    
    return {
      // Concise JSON for the model to read
      structuredContent: {
        count: restaurants.length,
        topResults: restaurants.slice(0, 3).map(r => ({
          name: r.name,
          rating: r.rating,
        })),
      },
      
      // Optional narration for the model's response
      content: [
        { 
          type: "text", 
          text: `Found ${restaurants.length} restaurants in ${location}` 
        }
      ],
      
      // Large/sensitive data exclusively for the widget
      _meta: {
        allRestaurants: restaurants,
        searchTimestamp: new Date().toISOString(),
      },
    };
  }
);
```

**Tool metadata:**

- `openai/outputTemplate`: URI of the widget to render
- `openai/toolInvocation/invoking`: Status message while tool executes
- `openai/toolInvocation/invoked`: Status message when complete
- `openai/widgetAccessible`: Enable `window.openai.callTool` for this tool
- `openai/visibility`: `"public"` (model + widget) or `"private"` (widget only)

**Data payloads:**

- `structuredContent`: Concise JSON the model reads (keep under 4k tokens)
- `content`: Optional Markdown/plaintext narration
- `_meta`: Widget-exclusive data (never sent to model)

**Design for idempotency:** Model may retry calls, so handlers should be safe to execute multiple times.

### Step 4: Build the Widget

**React widget with `window.openai`:**

```typescript
// web/src/app.tsx
import { createRoot } from "react-dom/client";
import { useEffect, useState, useSyncExternalStore } from "react";

// Helper hook to read window.openai globals
function useOpenAiGlobal<K extends keyof typeof window.openai>(
  key: K
): typeof window.openai[K] {
  return useSyncExternalStore(
    (onChange) => {
      const handler = () => onChange();
      window.addEventListener("openai:set_globals", handler);
      return () => window.removeEventListener("openai:set_globals", handler);
    },
    () => window.openai[key]
  );
}

// Helper hook for widget state persistence
function useWidgetState<T>(defaultValue: T) {
  const savedState = useOpenAiGlobal("widgetState") as T;
  const [state, setState] = useState<T>(savedState ?? defaultValue);
  
  useEffect(() => {
    if (savedState) setState(savedState);
  }, [savedState]);
  
  const setWidgetState = (newState: T | ((prev: T) => T)) => {
    setState((prev) => {
      const updated = typeof newState === "function" 
        ? (newState as (prev: T) => T)(prev)
        : newState;
      window.openai.setWidgetState(updated);
      return updated;
    });
  };
  
  return [state, setWidgetState] as const;
}

function RestaurantList() {
  const toolOutput = useOpenAiGlobal("toolOutput");
  const theme = useOpenAiGlobal("theme");
  const [selectedId, setSelectedId] = useWidgetState<string | null>(null);
  
  const restaurants = toolOutput?._meta?.allRestaurants ?? [];
  
  const handleRefresh = async () => {
    await window.openai.callTool("search_restaurants", {
      location: toolOutput?.location,
    });
  };
  
  const handleFollowUp = async (restaurantName: string) => {
    await window.openai.sendFollowUpMessage({
      prompt: `Tell me more about ${restaurantName}`,
    });
  };
  
  return (
    <div className={`app ${theme}`}>
      <h2>Restaurants</h2>
      <button onClick={handleRefresh}>Refresh</button>
      
      {restaurants.map((r) => (
        <div 
          key={r.id}
          className={selectedId === r.id ? "selected" : ""}
          onClick={() => setSelectedId(r.id)}
        >
          <h3>{r.name}</h3>
          <p>Rating: {r.rating}⭐</p>
          <button onClick={() => handleFollowUp(r.name)}>
            Learn More
          </button>
        </div>
      ))}
    </div>
  );
}

// Mount the app
const root = document.getElementById("root");
if (root) {
  createRoot(root).render(<RestaurantList />);
}
```

**Vanilla JavaScript version:**

```typescript
// web/src/app.ts
const toolOutput = window.openai.toolOutput;
const restaurants = toolOutput?._meta?.allRestaurants ?? [];

const root = document.getElementById("root");
root.innerHTML = `
  <div class="restaurants">
    <h2>Restaurants</h2>
    ${restaurants.map(r => `
      <div class="card">
        <h3>${r.name}</h3>
        <p>Rating: ${r.rating}⭐</p>
      </div>
    `).join("")}
  </div>
`;

// Subscribe to theme changes
window.addEventListener("openai:set_globals", () => {
  document.body.className = window.openai.theme;
});
```

### Step 5: Apply Theming

Use CSS variables that ChatGPT injects:

```css
:root {
  /* Fallback defaults */
  --color-background-primary: light-dark(#ffffff, #171717);
  --color-text-primary: light-dark(#171717, #fafafa);
  --color-border: light-dark(#e5e5e5, #404040);
  --font-sans: system-ui, -apple-system, sans-serif;
  --border-radius-md: 8px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
}

.app {
  background: var(--color-background-primary);
  color: var(--color-text-primary);
  font-family: var(--font-sans);
  padding: var(--spacing-md);
}

.card {
  border: 1px solid var(--color-border);
  border-radius: var(--border-radius-md);
  padding: var(--spacing-md);
  margin-bottom: var(--spacing-sm);
}
```

**Optional: Use the Apps SDK UI kit** at [apps-sdk-ui](https://openai.github.io/apps-sdk-ui) for ready-made components.

### Step 6: Bundle the Widget

Use Vite or esbuild to create single-file bundle:

```javascript
// vite.config.ts
import { defineConfig } from "vite";
import { viteSingleFile } from "vite-plugin-singlefile";

export default defineConfig({
  plugins: [viteSingleFile()],
  build: {
    outDir: "dist",
    rollupOptions: {
      input: "index.html"
    }
  }
});
```

Build command:

```bash
cd web
npm run build  # Outputs dist/app.js and dist/app.css
```

### Step 7: Test Locally

1. Build widget: `npm run build` in `web/`
2. Start MCP server: `node dist/index.js`
3. Use [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector):
   - Point to `http://localhost:<port>/mcp`
   - List resources and verify widget HTML
   - Call tools and verify widget renders
   - Test `window.openai` APIs in browser devtools

### Step 8: Deploy

**Requirements:**
- HTTPS endpoint (ChatGPT requires HTTPS)
- During development: `ngrok http <port>` or similar tunnel
- Production: Deploy to Cloudflare Workers, Fly.io, Vercel, AWS, etc.

**Test deployment:**

```bash
# Tunnel localhost
ngrok http 3000
# Use ngrok URL when creating connector in ChatGPT
```

## Authentication & Authorization

### OAuth 2.1 Setup

**Protected Resource Metadata** (`/.well-known/oauth-protected-resource`):

```json
{
  "resource": "https://your-mcp.example.com",
  "authorization_servers": [
    "https://auth.yourcompany.com"
  ],
  "scopes_supported": ["read", "write"],
  "resource_documentation": "https://yourcompany.com/docs"
}
```

**Authorization Server Metadata** (`.well-known/oauth-authorization-server`):

```json
{
  "issuer": "https://auth.yourcompany.com",
  "authorization_endpoint": "https://auth.yourcompany.com/oauth2/authorize",
  "token_endpoint": "https://auth.yourcompany.com/oauth2/token",
  "registration_endpoint": "https://auth.yourcompany.com/oauth2/register",
  "code_challenge_methods_supported": ["S256"],
  "scopes_supported": ["read", "write"]
}
```

**Tool Security Schemes:**

```typescript
server.registerTool(
  "get_user_data",
  {
    title: "Get User Data",
    inputSchema: { /* ... */ },
    securitySchemes: [
      { type: "oauth2", scopes: ["read"] }
    ],
    _meta: {
      "openai/outputTemplate": "ui://widget/app.html"
    }
  },
  async ({ input }, context) => {
    // Verify token
    if (!context.authorization) {
      return {
        content: [{ type: "text", text: "Authentication required" }],
        _meta: {
          "mcp/www_authenticate": [
            'Bearer resource_metadata="https://your-mcp.example.com/.well-known/oauth-protected-resource", error="insufficient_scope", error_description="Login required"'
          ]
        },
        isError: true
      };
    }
    
    // Verify token, scopes, audience, expiry
    const user = await verifyToken(context.authorization);
    const data = await fetchUserData(user.id);
    
    return {
      structuredContent: { summary: data.summary },
      _meta: { fullData: data }
    };
  }
);
```

**OAuth Flow:**
1. ChatGPT queries protected resource metadata
2. ChatGPT dynamically registers client with authorization server
3. User authenticates and grants scopes
4. ChatGPT exchanges authorization code for access token (with PKCE)
5. Token attached to MCP requests: `Authorization: Bearer <token>`
6. MCP server verifies token on every request

**Redirect URIs to allowlist:**
- Production: `https://chatgpt.com/connector_platform_oauth_redirect`
- Review: `https://platform.openai.com/apps-manage/oauth`

## Advanced Features

### Component-Initiated Tool Calls

Enable widgets to call tools directly:

```typescript
// Tool definition
_meta: {
  "openai/outputTemplate": "ui://widget/app.html",
  "openai/widgetAccessible": true,
  "openai/visibility": "public" // or "private" to hide from model
}

// Widget code
async function refreshData() {
  const result = await window.openai.callTool("search_restaurants", {
    location: "Paris"
  });
  // Widget automatically re-renders with new toolOutput
}
```

### File Handling

**Upload files:**

```typescript
// Widget code
async function handleUpload(event: ChangeEvent<HTMLInputElement>) {
  const file = event.target.files?.[0];
  if (!file) return;
  
  const { fileId } = await window.openai.uploadFile(file);
  console.log("Uploaded:", fileId);
}
```

**Download files:**

```typescript
// Widget code
const { downloadUrl } = await window.openai.getFileDownloadUrl({ fileId });
image.src = downloadUrl;
```

**Tool file parameters:**

```typescript
server.registerTool(
  "process_image",
  {
    title: "Process Image",
    inputSchema: {
      type: "object",
      properties: {
        image: {
          type: "object",
          properties: {
            download_url: { type: "string" },
            file_id: { type: "string" }
          }
        }
      }
    },
    _meta: {
      "openai/outputTemplate": "ui://widget/app.html",
      "openai/fileParams": ["image"]
    }
  },
  async ({ image }) => {
    const response = await fetch(image.download_url);
    const buffer = await response.arrayBuffer();
    // Process image...
    return { content: [], structuredContent: {} };
  }
);
```

### Display Modes

Request alternate layouts:

```typescript
// Widget code
await window.openai.requestDisplayMode({ mode: "fullscreen" });
// Options: "inline", "pip", "fullscreen"
// Note: PiP may coerce to fullscreen on mobile
```

### Modals

Spawn host-controlled overlays:

```typescript
// Widget code
await window.openai.requestModal({
  title: "Checkout",
  component: "checkout-modal"
});
```

### Navigation

Use standard routing (React Router):

```typescript
import { BrowserRouter, Routes, Route, useNavigate } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<ListView />} />
        <Route path="/detail/:id" element={<DetailView />} />
      </Routes>
    </BrowserRouter>
  );
}

function ListView() {
  const navigate = useNavigate();
  return (
    <button onClick={() => navigate("/detail/123")}>
      View Details
    </button>
  );
}
```

ChatGPT mirrors iframe history to UI navigation controls.

### Localization

Read locale and format accordingly:

```typescript
// Widget code
const locale = window.openai.locale ?? "en-US";
const formatter = new Intl.DateTimeFormat(locale);
const price = new Intl.NumberFormat(locale, { 
  style: "currency", 
  currency: "USD" 
});
```

Or use i18n libraries:

```typescript
import { IntlProvider } from "react-intl";
import en from "./locales/en-US.json";
import es from "./locales/es-ES.json";

const messages = { "en-US": en, "es-ES": es };
const locale = window.openai.locale ?? "en-US";

<IntlProvider locale={locale} messages={messages[locale]}>
  <App />
</IntlProvider>
```

### Close Widget

From widget:

```typescript
window.openai.requestClose();
```

From server:

```typescript
return {
  content: [],
  structuredContent: {},
  _meta: {
    "openai/closeWidget": true
  }
};
```

## Security & Privacy

### Principles

- **Least privilege**: Only request needed scopes and permissions
- **Explicit consent**: Users understand what they're granting
- **Defense in depth**: Assume prompt injection and malicious inputs

### Data Handling

- `structuredContent`: Include only data needed for current prompt
- Storage: Publish retention policy, honor deletion requests
- Logging: Redact PII, store correlation IDs only

### Network Security

- Widgets run in sandboxed iframe with strict CSP
- No privileged browser APIs (`alert`, `prompt`, `clipboard`)
- `fetch` allowed only with CSP compliance
- Subframes blocked by default (require `frame_domains`)

### Authentication

- Use OAuth 2.1 with PKCE and dynamic client registration
- Verify token signature, issuer, audience, expiry on every request
- Reject expired/malformed tokens with `401` and `WWW-Authenticate`

### Prompt Injection Mitigation

- Review tool descriptions regularly
- Validate all inputs server-side
- Require human confirmation for destructive operations
- Test with injection prompts during QA

## Testing & Deployment

### Local Testing

1. **MCP Inspector**: Test tools and widget rendering
   - `http://localhost:<port>/mcp`
   - Verify resources list
   - Call tools and inspect widget iframe
   - Use browser devtools for `window.openai` debugging

2. **Dogfooding**: Test with trusted users before broad rollout

3. **OAuth Testing**: Use Inspector's Auth settings to walk through flow

### Production Deployment

**Hosting options:**
- Cloudflare Workers
- Fly.io
- Vercel
- AWS Lambda/ECS
- Your existing infrastructure

**Requirements:**
- HTTPS endpoint
- Low latency (< 500ms tool responses recommended)
- OAuth protected resource metadata at `/.well-known/oauth-protected-resource`
- Authorization server with discovery metadata

**Monitoring:**
- Set up alerts for failed auth attempts
- Track tool execution times
- Monitor CSP violations in widget
- Log correlation IDs for debugging

### Submission

**Pre-submission checklist:**
1. OAuth redirect URIs allowlisted:
   - `https://chatgpt.com/connector_platform_oauth_redirect`
   - `https://platform.openai.com/apps-manage/oauth`
2. Tool descriptions clear and accurate
3. Security review complete
4. CSP correctly configured
5. Widget tested across light/dark themes
6. Mobile/desktop responsive design
7. Localization for supported languages
8. Error messages user-friendly
9. Performance under load verified
10. Documentation complete

**Review process:**
- Apps using `frame_domains` subject to stricter review
- Expect questions about data handling and security
- May be rejected for directory if concerns remain

## Troubleshooting

### Widget Not Rendering

**Symptom**: White screen or no widget appears

**Solutions:**
- Verify `mimeType: "text/html+skybridge"`
- Check bundled JS/CSS URLs resolve correctly
- Inspect browser console for CSP violations
- Confirm `openai/widgetCSP` allows necessary domains

### `window.openai` Undefined

**Symptom**: `Cannot read property 'toolOutput' of undefined`

**Solutions:**
- Only available in `text/html+skybridge` templates
- Check MIME type spelling and format
- Verify widget loaded without errors
- Use browser devtools to inspect iframe

### CSP Violations

**Symptom**: Network requests blocked, inline styles stripped

**Solutions:**
- Add domains to `connect_domains` (APIs)
- Add domains to `resource_domains` (static assets)
- Avoid inline scripts/styles (bundle them)
- Check browser console for specific CSP errors

### Stale Widget Cache

**Symptom**: Old widget version keeps loading after deploy

**Solutions:**
- Change template URI: `ui://widget/app-v2.html`
- Change bundled filename: `app-20250101.js`
- Clear ChatGPT cache (hard refresh)
- Wait ~5 minutes for CDN propagation

### Authentication Loops

**Symptom**: OAuth flow starts but never completes

**Solutions:**
- Verify redirect URI in allowlist
- Check `code_challenge_methods_supported` includes `S256`
- Confirm `resource` parameter echoed in tokens
- Use MCP Inspector Auth tab to debug flow step-by-step
- Check authorization server logs

### Large Payloads

**Symptom**: Slow rendering, model performance degraded

**Solutions:**
- Trim `structuredContent` to essentials (< 4k tokens)
- Move large data to `_meta` (widget-only)
- Paginate or lazy-load data in widget
- Compress images before embedding

## Examples

**Official examples repository:** [openai-apps-sdk-examples](https://github.com/openai/openai-apps-sdk-examples)

### Pizzaz List
Ranked card list with favorites and CTAs
- Tool: Search pizzerias by location
- Widget: Sortable list with star ratings
- Features: Favorites, external links, follow-up prompts

### Pizzaz Carousel
Embla-powered horizontal scroller for media-heavy layouts
- Tool: Fetch restaurant galleries
- Widget: Image carousel with lazy loading
- Features: Swipe navigation, fullscreen viewer

### Pizzaz Map
Mapbox integration with fullscreen inspector
- Tool: Search restaurants near location
- Widget: Interactive map with markers
- Features: PiP mode, geolocation, clustering

### Pizzaz Album
Stacked gallery for deep dives on single place
- Tool: Get restaurant details with photos
- Widget: Pinterest-style photo grid
- Features: Modal lightbox, share actions

### Pizzaz Video
Scripted player with overlays
- Tool: Fetch restaurant promotional videos
- Widget: Custom video player
- Features: Overlay controls, transcript display

## Best Practices

### Server Design

1. **Idempotent handlers**: Model may retry, design for safety
2. **Structured content first**: Model reads this, keep it concise
3. **Metadata for widgets**: Move large/sensitive data to `_meta`
4. **Clear tool descriptions**: Model chooses tools based on these
5. **Security in depth**: Verify tokens, validate inputs, log actions

### Widget Design

1. **Theme-aware styling**: Use CSS variables, test light/dark
2. **Responsive layouts**: Mobile and desktop support
3. **Loading states**: Handle async operations gracefully
4. **Error boundaries**: Catch and display errors user-friendly
5. **Accessibility**: Keyboard navigation, ARIA labels, focus management

### State Management

1. **Persist with setWidgetState**: Keep state under 4k tokens
2. **Scope to widget instance**: State tied to specific message
3. **Clear on new conversations**: Fresh `widgetId` = empty state
4. **Sync with useOpenAiGlobal**: Multiple components stay consistent
5. **Validate rehydrated state**: Handle corrupted or old state

### Performance

1. **Bundle size**: Keep under 500KB for fast load
2. **Code splitting**: Lazy load features not needed immediately
3. **Image optimization**: Compress, resize, use modern formats
4. **API caching**: Cache responses when appropriate
5. **Debounce inputs**: Avoid excessive tool calls from rapid interactions

### Security

1. **Validate inputs**: Never trust data from model or user
2. **Redact secrets**: No tokens/keys in `structuredContent` or `_meta`
3. **CSP allowlist**: Only necessary domains, avoid wildcards
4. **Rate limiting**: Protect backend from abuse
5. **Audit logs**: Track actions for compliance and debugging

## Quick Reference

### `window.openai` API

| Category | Property/Method | Description |
|----------|----------------|-------------|
| **Data** | `toolInput` | Tool arguments from invocation |
| **Data** | `toolOutput` | `structuredContent` from server |
| **Data** | `toolResponseMetadata` | `_meta` payload (widget-only) |
| **Data** | `widgetState` | Persisted UI state snapshot |
| **Data** | `setWidgetState(state)` | Store new state (< 4k tokens) |
| **Tools** | `callTool(name, args)` | Invoke MCP tool from widget |
| **Messaging** | `sendFollowUpMessage({ prompt })` | Post user-authored message |
| **Files** | `uploadFile(file)` | Upload file, receive `fileId` |
| **Files** | `getFileDownloadUrl({ fileId })` | Get temp download URL |
| **Layout** | `requestDisplayMode({ mode })` | Request inline/PiP/fullscreen |
| **Layout** | `requestModal({ ... })` | Spawn host modal overlay |
| **Layout** | `notifyIntrinsicHeight(height)` | Report dynamic height |
| **Layout** | `requestClose()` | Close the widget |
| **Navigation** | `openExternal({ href })` | Open vetted external link |
| **Context** | `theme` | "light" or "dark" |
| **Context** | `displayMode` | "inline", "pip", or "fullscreen" |
| **Context** | `locale` | RFC 4647 locale string |
| **Context** | `userAgent` | Client user agent |
| **Context** | `maxHeight` | Container max height |
| **Context** | `safeArea` | Safe area insets |
| **Context** | `view` | View context info |

### Tool Metadata Reference

| Key | Value | Description |
|-----|-------|-------------|
| `openai/outputTemplate` | `"ui://widget/app.html"` | Widget URI to render |
| `openai/widgetAccessible` | `true/false` | Enable `callTool` from widget |
| `openai/visibility` | `"public"/"private"` | Model visibility |
| `openai/toolInvocation/invoking` | `"Loading…"` | Status while executing |
| `openai/toolInvocation/invoked` | `"Complete"` | Status when done |
| `openai/fileParams` | `["image"]` | Fields treated as file params |

### CSP Configuration

```typescript
"openai/widgetCSP": {
  connect_domains: ["https://api.example.com"],    // Fetch destinations
  resource_domains: ["https://cdn.example.com"],   // Static assets
  redirect_domains: ["https://checkout.example.com"], // openExternal
  frame_domains: ["https://*.embed.example.com"]   // Subframes (discouraged)
}
```

### OAuth Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/.well-known/oauth-protected-resource` | Protected resource metadata |
| `/.well-known/oauth-authorization-server` | OAuth 2.0 discovery |
| `/.well-known/openid-configuration` | OpenID Connect discovery |

## Resources

**Official Documentation:**
- [Apps SDK Docs](https://developers.openai.com/apps-sdk)
- [Apps SDK Quickstart](https://developers.openai.com/apps-sdk/quickstart)
- [MCP Specification](https://modelcontextprotocol.io/specification)
- [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector)

**Examples & Tools:**
- [Apps SDK Examples](https://github.com/openai/openai-apps-sdk-examples)
- [Apps SDK UI Kit](https://openai.github.io/apps-sdk-ui)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)

**Community:**
- [Apps SDK Blog](https://developers.openai.com/blog)
- [ChatGPT App Guidelines](https://developers.openai.com/apps-sdk/app-submission-guidelines)

**Authentication:**
- [Auth0 MCP Guide](https://github.com/openai/openai-mcpkit/blob/main/python-authenticated-mcp-server-scaffold/README.md)
- [Stytch MCP Guide](https://stytch.com/docs/guides/connected-apps/mcp-server-overview)
- [RFC 9728 (OAuth Resource Metadata)](https://datatracker.ietf.org/doc/html/rfc9728)
- [RFC 8414 (OAuth Discovery)](https://datatracker.ietf.org/doc/html/rfc8414)

## Related Skills

- **mcp-builder**: General MCP server implementation patterns
- **mcp-mcp-apps-kit**: MCP Apps (SEP-1865) for Claude Desktop
