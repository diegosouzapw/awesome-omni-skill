---
name: openai-apps-sdk-builder
description: Build OpenAI Apps SDK applications - interactive ChatGPT apps with MCP servers, React widgets, and rich UI components for conversational experiences
---

# OpenAI Apps SDK Builder Skill

## Overview

This skill guides Claude in creating OpenAI Apps SDK applications - interactive apps that run inside ChatGPT using the Model Context Protocol (MCP). These apps combine conversational AI with rich UI components (widgets) that appear inline in chat.

## When to Use This Skill

Use this skill when users request:
- "Create an OpenAI app for [use case]"
- "Build a ChatGPT app that [does something]"
- "Make an MCP server for [functionality]"
- Interactive tools/widgets for ChatGPT (maps, galleries, players, forms, etc.)

## Core Concepts

### What is an OpenAI Apps SDK App?

An OpenAI Apps SDK app consists of three integrated components:

1. **MCP Server**: Backend that exposes tools (functions) ChatGPT can call
2. **UI Components/Widgets**: React-based interactive interfaces rendered in ChatGPT
3. **Metadata Bridge**: The `_meta.openai/outputTemplate` that connects tools to UI

### How It Works

```
User Query → ChatGPT decides to use tool → MCP Server executes tool 
→ Returns structured data + metadata → ChatGPT renders widget with data
```

## Architecture Patterns

### MCP Server (Backend)

The server implements three key capabilities:
1. **List Tools** - Advertises available tools with JSON schemas
2. **Call Tools** - Executes tool logic and returns structured content
3. **Return Widgets** - Includes metadata pointing to UI components

### Widget (Frontend)

React components that:
- Run in an iframe within ChatGPT
- Communicate via `window.openai` API
- Receive data through tool outputs
- Can call back to MCP server tools
- Support multiple display modes (inline, fullscreen, PiP)

## Implementation Guide

### Language/Framework Choices

**Python (Recommended for rapid prototyping)**
- FastMCP or official Python MCP SDK
- FastAPI for HTTP transport
- Best for data-heavy apps, ML integration, rapid development

**TypeScript/Node (Recommended for React-heavy apps)**
- Official `@modelcontextprotocol/sdk`
- Express or native Node HTTP
- Best for complex UI, existing Node infrastructure

### Project Structure

```
my-app/
├── src/                          # Widget source code (React)
│   ├── pizza-map/
│   │   ├── index.tsx
│   │   ├── styles.css
│   │   └── types.ts
│   ├── use-openai-global.ts     # Hooks for window.openai
│   └── use-max-height.ts        # Layout utilities
├── assets/                       # Built widget bundles
│   ├── pizza-map-[hash].html
│   ├── pizza-map-[hash].js
│   └── pizza-map-[hash].css
├── server_python/                # Python MCP server
│   ├── main.py
│   └── requirements.txt
├── server_node/                  # TypeScript MCP server
│   ├── src/
│   │   └── index.ts
│   └── package.json
├── build-all.mts                 # Vite build orchestrator
├── vite.config.ts
└── package.json
```

## Python MCP Server Implementation

### Setup (FastAPI + FastMCP)

```python
# requirements.txt
fastapi
uvicorn
fastmcp
pydantic

# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import FileResponse, Response
from fastmcp import FastMCP
import json

app = FastAPI()
mcp = FastMCP(name="My App Server")

# CORS for ChatGPT
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://chatgpt.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Serve widget HTML
@app.get("/components/{component_name}.html")
async def serve_component(component_name: str):
    return FileResponse(
        f"assets/{component_name}.html",
        media_type="text/html+skybridge"  # CRITICAL: Use this MIME type
    )

# MCP endpoint (Streamable HTTP)
@app.post("/mcp")
async def handle_mcp(request: Request):
    # Handle MCP protocol - use FastMCP or manual implementation
    return mcp.handle_request(await request.json())

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Registering Resources (UI Templates)

```python
# Register a UI resource that the server can serve
@mcp.resource("pizza-map")
async def get_pizza_map_resource():
    """Returns the HTML template for the pizza map widget"""
    return {
        "contents": [{
            "uri": "ui://widget/pizza-map.html",
            "mimeType": "text/html+skybridge",  # CRITICAL MIME type
            "text": """
                <div id="pizzaz-root"></div>
                <link rel="stylesheet" href="https://your-cdn.com/pizza-map-abc123.css">
                <script type="module" src="https://your-cdn.com/pizza-map-abc123.js"></script>
            """.strip(),
            "_meta": {
                "openai/widgetDescription": "Interactive map showing pizza locations with ratings",
                "openai/widgetPrefersBorder": True,
                "openai/widgetCSP": {
                    "connect_domains": [],
                    "resource_domains": ["https://your-cdn.com", "https://api.mapbox.com"]
                },
                "openai/widgetDomain": "https://chatgpt.com"
            }
        }]
    }
```

### Registering Tools

```python
@mcp.tool()
async def find_pizza_places(location: str, max_results: int = 10) -> dict:
    """
    Find pizza restaurants near a location.
    
    Args:
        location: City or address to search near
        max_results: Maximum number of results to return
    
    Returns:
        Dictionary with pizza places and map data
    """
    # Your business logic here
    places = [
        {
            "id": "1",
            "name": "Pizza Palace",
            "rating": 4.5,
            "coords": [-73.935242, 40.730610],
            "description": "Classic New York style pizza"
        },
        # ... more places
    ]
    
    return {
        # Plain text for conversation transcript
        "content": [{
            "type": "text",
            "text": f"Found {len(places)} pizza places near {location}"
        }],
        
        # Structured data that ChatGPT can reason about
        "structuredContent": {
            "places": places,
            "location": location
        },
        
        # Metadata for widget rendering (NOT shown to model)
        "_meta": {
            "openai/outputTemplate": "ui://widget/pizza-map.html",
            "openai/toolInvocation/invoking": "Searching for pizza...",
            "openai/toolInvocation/invoked": "Found pizza places",
            
            # Additional data for widget only (not shown to model)
            "mapSettings": {
                "center": [-73.935242, 40.730610],
                "zoom": 12
            }
        }
    }
```

### Tool Metadata Annotations

```python
# Read-only hint (for tools that don't modify state)
@mcp.tool(
    _meta={
        "openai/readOnlyHint": True
    }
)
async def list_favorites() -> dict:
    """List user's favorite pizza places without modifying anything"""
    pass

# Custom invocation messages
@mcp.tool(
    _meta={
        "openai/toolInvocation/invoking": "Calculating route...",
        "openai/toolInvocation/invoked": "Route calculated"
    }
)
async def get_directions(from_loc: str, to_loc: str) -> dict:
    """Get directions between two locations"""
    pass

# Locale-aware tools
@mcp.tool()
async def recommend_cafe(location: str, context: dict) -> dict:
    """Recommend a cafe based on user location"""
    locale = context.get("_meta", {}).get("openai/locale", "en")
    user_location = context.get("_meta", {}).get("openai/userLocation", {})
    
    # Use locale and location hints (never for auth!)
    return {...}
```

## TypeScript/Node MCP Server Implementation

### Setup

```typescript
// package.json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "express": "^4.18.0",
    "zod": "^3.22.0"
  }
}

// src/index.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";
import express from "express";

const server = new McpServer({
  name: "My App Server",
  version: "1.0.0"
});

const app = express();

// CORS middleware
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "https://chatgpt.com");
  res.header("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
  res.header("Access-Control-Allow-Headers", "Content-Type");
  next();
});

// Serve widget bundles
app.get("/components/:name.html", (req, res) => {
  res.setHeader("Content-Type", "text/html+skybridge");
  res.sendFile(`assets/${req.params.name}.html`);
});

app.listen(8000);
```

### Registering Resources

```typescript
server.registerResource(
  "pizza-map",
  "ui://widget/pizza-map.html",
  {},
  async () => ({
    contents: [{
      uri: "ui://widget/pizza-map.html",
      mimeType: "text/html+skybridge",
      text: `
        <div id="pizzaz-root"></div>
        <link rel="stylesheet" href="https://your-cdn.com/pizza-map-abc123.css">
        <script type="module" src="https://your-cdn.com/pizza-map-abc123.js"></script>
      `.trim(),
      _meta: {
        "openai/widgetDescription": "Interactive map showing pizza locations",
        "openai/widgetPrefersBorder": true
      }
    }]
  })
);
```

### Registering Tools

```typescript
server.registerTool(
  "find_pizza_places",
  {
    title: "Find Pizza Places",
    description: "Search for pizza restaurants near a location",
    inputSchema: z.object({
      location: z.string(),
      maxResults: z.number().int().min(1).max(50).optional()
    }),
    _meta: {
      "openai/outputTemplate": "ui://widget/pizza-map.html",
      "openai/readOnlyHint": true
    }
  },
  async ({ location, maxResults = 10 }) => {
    const places = await searchPizzaPlaces(location, maxResults);
    
    return {
      content: [{
        type: "text",
        text: `Found ${places.length} pizza places near ${location}`
      }],
      structuredContent: { places, location },
      _meta: {
        mapSettings: {
          center: places[0]?.coords || [0, 0],
          zoom: 12
        }
      }
    };
  }
);
```

## Widget Development

### React Component Structure

```tsx
// src/pizza-map/index.tsx
import React, { useEffect, useRef } from "react";
import { createRoot } from "react-dom/client";
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { useOpenAiGlobal } from "../use-openai-global";
import { useMaxHeight } from "../use-max-height";

function PizzaMap() {
  // Access tool input/output from window.openai
  const toolInput = useOpenAiGlobal("toolInput");
  const toolOutput = useOpenAiGlobal("toolOutput");
  const metadata = useOpenAiGlobal("toolResponseMetadata");
  
  // Access theme and display mode
  const theme = useOpenAiGlobal("theme");
  const displayMode = useOpenAiGlobal("displayMode");
  
  // Handle responsive height
  const maxHeight = useMaxHeight();
  
  const mapRef = useRef<mapboxgl.Map | null>(null);
  
  useEffect(() => {
    // Initialize map
    const map = new mapboxgl.Map({
      container: "map",
      style: theme === "dark" 
        ? "mapbox://styles/mapbox/dark-v11"
        : "mapbox://styles/mapbox/light-v11",
      center: metadata?.mapSettings?.center || [0, 0],
      zoom: metadata?.mapSettings?.zoom || 2
    });
    
    mapRef.current = map;
    
    // Add markers for each place
    toolOutput?.places?.forEach(place => {
      new mapboxgl.Marker()
        .setLngLat(place.coords)
        .setPopup(
          new mapboxgl.Popup().setHTML(
            `<h3>${place.name}</h3><p>Rating: ${place.rating}</p>`
          )
        )
        .addTo(map);
    });
    
    return () => map.remove();
  }, [toolOutput, theme, metadata]);
  
  // Handle display mode changes
  useEffect(() => {
    if (displayMode === "fullscreen") {
      // Expand to full capabilities
      mapRef.current?.resize();
    }
  }, [displayMode]);
  
  return (
    <div style={{ height: maxHeight, width: "100%" }}>
      <div id="map" style={{ height: "100%", width: "100%" }} />
    </div>
  );
}

// Mount the component
const root = document.getElementById("pizzaz-root");
if (root) {
  createRoot(root).render(<PizzaMap />);
}
```

### Essential Hooks

```typescript
// use-openai-global.ts
import { useEffect, useState } from "react";

export function useOpenAiGlobal<K extends keyof OpenAiGlobals>(
  key: K
): OpenAiGlobals[K] | undefined {
  const [value, setValue] = useState<OpenAiGlobals[K]>();
  
  useEffect(() => {
    // Get initial value
    if (window.openai?.[key]) {
      setValue(window.openai[key]);
    }
    
    // Listen for changes
    const handler = (event: SetGlobalsEvent) => {
      if (key in event.detail.globals) {
        setValue(event.detail.globals[key]);
      }
    };
    
    window.addEventListener("openai:set_globals", handler);
    return () => window.removeEventListener("openai:set_globals", handler);
  }, [key]);
  
  return value;
}

// Specialized hooks
export function useToolInput() {
  return useOpenAiGlobal("toolInput");
}

export function useToolOutput() {
  return useOpenAiGlobal("toolOutput");
}

export function useTheme() {
  return useOpenAiGlobal("theme");
}

export function useDisplayMode() {
  return useOpenAiGlobal("displayMode");
}
```

### Widget State Management

```typescript
// For persisting state across sessions
import { useWidgetState } from "../use-widget-state";

function MyWidget() {
  const [state, setState] = useWidgetState({
    favorites: [],
    selectedId: null
  });
  
  const addFavorite = (id: string) => {
    setState(prev => ({
      ...prev,
      favorites: [...prev.favorites, id]
    }));
  };
  
  // Note: Widget state is exposed to ChatGPT model
  // Keep it small (<4k tokens) for performance
  
  return <div>...</div>;
}
```

### Calling MCP Tools from Widget

```typescript
function InteractiveWidget() {
  const [loading, setLoading] = useState(false);
  
  const handleAction = async () => {
    setLoading(true);
    
    try {
      const result = await window.openai.callTool("update_preference", {
        preference: "dark_mode",
        value: true
      });
      
      console.log("Tool result:", result);
    } catch (error) {
      console.error("Tool call failed:", error);
    } finally {
      setLoading(false);
    }
  };
  
  return <button onClick={handleAction}>Update Preference</button>;
}
```

### Display Mode Management

```typescript
function ExpandableWidget() {
  const displayMode = useDisplayMode();
  
  const requestFullscreen = () => {
    window.openai.requestDisplayMode({ mode: "fullscreen" });
  };
  
  const requestPiP = () => {
    window.openai.requestDisplayMode({ mode: "pip" });
  };
  
  return (
    <div>
      {displayMode !== "fullscreen" && (
        <button onClick={requestFullscreen}>
          Expand to Fullscreen
        </button>
      )}
      
      {displayMode === "inline" && (
        <button onClick={requestPiP}>
          Pop Out (PiP)
        </button>
      )}
    </div>
  );
}
```

## Building and Bundling Widgets

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { resolve } from "path";
import { readdirSync } from "fs";

// Auto-discover all widget entry points
const widgets = readdirSync("src").filter(dir => 
  !dir.startsWith("use-") && !dir.includes(".")
);

const input = Object.fromEntries(
  widgets.map(widget => [widget, resolve(__dirname, `src/${widget}/index.tsx`)])
);

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: "assets",
    rollupOptions: {
      input,
      output: {
        entryFileNames: "[name]-[hash].js",
        chunkFileNames: "[name]-[hash].js",
        assetFileNames: "[name]-[hash].[ext]"
      }
    }
  }
});
```

### Build Script

```typescript
// build-all.mts
import { build } from "vite";
import { writeFileSync } from "fs";

async function buildAll() {
  // Build all widgets
  await build();
  
  // Generate HTML wrappers with hashed asset references
  const widgets = ["pizza-map", "pizza-carousel", "pizza-video"];
  
  for (const widget of widgets) {
    const hash = "abc123"; // Extract from build output
    const html = `
      <!DOCTYPE html>
      <html>
        <head>
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <link rel="stylesheet" href="https://your-cdn.com/${widget}-${hash}.css">
        </head>
        <body>
          <div id="pizzaz-root"></div>
          <script type="module" src="https://your-cdn.com/${widget}-${hash}.js"></script>
        </body>
      </html>
    `.trim();
    
    writeFileSync(`assets/${widget}-${hash}.html`, html);
  }
}

buildAll();
```

## Best Practices

### 1. Tool Design

**DO:**
- Use clear, descriptive tool names (e.g., `find_pizza_places` not `search`)
- Write detailed docstrings - they become tool descriptions
- Keep tool focused on one clear action
- Mark read-only tools with `openai/readOnlyHint`
- Return both human-readable text AND structured data

**DON'T:**
- Use generic names like `process` or `handle`
- Create tools that do multiple unrelated things
- Rely on `_meta` fields for authorization
- Expose sensitive data in `structuredContent`

### 2. Widget Development

**DO:**
- Use `text/html+skybridge` MIME type for all widget resources
- Handle theme changes (light/dark mode)
- Support all display modes (inline, fullscreen, PiP)
- Keep widget state small (<4k tokens)
- Use semantic HTML and WCAG AA contrast ratios
- Test on mobile, tablet, and desktop

**DON'T:**
- Use HTML `<form>` elements (blocked in iframe)
- Store sensitive data in widget state (it's visible to the model)
- Assume localStorage/sessionStorage work (they don't in iframe)
- Hard-code aspect ratios that might distort
- Include your logo (ChatGPT adds it automatically)

### 3. Data Flow

```
User Input
    ↓
ChatGPT (decides to use tool)
    ↓
MCP Server Tool Execution
    ↓
Returns:
  - content: Text for conversation (visible)
  - structuredContent: Data for model reasoning (visible)
  - _meta: Widget data + config (NOT visible to model)
    ↓
ChatGPT renders widget using _meta.openai/outputTemplate
    ↓
Widget receives data via window.openai
    ↓
Widget can call tools back via window.openai.callTool()
```

### 4. Security

**DO:**
- Use CORS to restrict origins to `https://chatgpt.com`
- Validate all tool inputs with schemas
- Implement OAuth 2.1 for user authentication
- Use CSP (Content Security Policy) in widget metadata
- Rate limit tool calls
- Validate webhook signatures

**DON'T:**
- Trust `_meta["openai/userLocation"]` for authorization
- Store secrets in widget code or metadata
- Allow unrestricted CORS origins
- Skip input validation

### 5. Performance

**DO:**
- Keep widget bundles small (<500KB ideally)
- Use code splitting for large widgets
- Optimize images (WebP, compression)
- Cache static assets aggressively
- Use CDN for widget bundles
- Minimize widget state updates

**DON'T:**
- Load entire libraries for small features
- Make excessive API calls from widgets
- Store large datasets in widget state
- Trigger unnecessary re-renders

## Testing

### Local Development

```bash
# Terminal 1: Build widgets
npm run build

# Terminal 2: Serve static assets with CORS
npm run serve  # Assets at http://localhost:4444

# Terminal 3: Run MCP server
# Python
uvicorn main:app --reload --port 8000

# Node
npm run dev
```

### Testing with ngrok

```bash
# Expose local server to internet
ngrok http 8000

# Use ngrok URL in ChatGPT Developer Mode
# Settings > Connectors > Add Connector
# MCP URL: https://abc123.ngrok-free.app/mcp
```

### MCP Inspector

```bash
# Test tools without ChatGPT
npx @modelcontextprotocol/inspector http://localhost:8000/mcp

# Validates:
# - Tool schemas
# - Resource responses
# - Metadata structure
# - Widget rendering
```

## Deployment

### Requirements

- HTTPS endpoint (required for production)
- Low cold-start latency (<2s ideal)
- Support for streaming HTTP or SSE transport
- CORS configured for `https://chatgpt.com`

### Hosting Options

**Serverless (AWS Lambda, Google Cloud Functions)**
- Pros: Auto-scaling, pay-per-use
- Cons: Cold starts, 15-30s timeout limits
- Best for: Low-traffic apps, bursty workloads

**Container (Cloud Run, Fargate, Railway)**
- Pros: Consistent performance, longer timeouts
- Cons: More complex setup, always-on costs
- Best for: Production apps, steady traffic

**VPS (DigitalOcean, Linode, Hetzner)**
- Pros: Full control, predictable costs
- Cons: Manual scaling, server management
- Best for: High-performance needs, custom infrastructure

### Environment Variables

```bash
# Production
OPENAI_API_KEY=sk-...              # If calling OpenAI APIs
WIDGET_CDN_URL=https://cdn.example.com
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
OAUTH_CLIENT_ID=...
OAUTH_CLIENT_SECRET=...

# Development
WIDGET_CDN_URL=http://localhost:4444
DEBUG=true
```

### CDN Setup for Widgets

```bash
# Build with production CDN URL
export WIDGET_CDN_URL=https://persistent.oaistatic.com/your-app

npm run build

# Upload assets/ to CDN
aws s3 sync assets/ s3://your-bucket/
# or
gsutil -m rsync -r assets/ gs://your-bucket/
```

## Common Widget Patterns

### 1. Map Widget
- Libraries: Mapbox GL, Leaflet
- Use cases: Restaurant finder, store locator, real estate
- Features: Markers, popups, clustering, directions

### 2. Carousel/Gallery
- Libraries: Embla, Swiper
- Use cases: Product browsing, image galleries, portfolios
- Features: Touch gestures, lazy loading, thumbnails

### 3. Video/Audio Player
- Libraries: Video.js, Plyr
- Use cases: Courses, music, podcasts
- Features: Timeline, chapters, playback controls, captions

### 4. List/Grid
- Libraries: React Virtualized, TanStack Virtual
- Use cases: Search results, catalogs, feeds
- Features: Infinite scroll, filtering, sorting, actions

### 5. Form/Survey
- Libraries: React Hook Form, Formik
- Use cases: Booking, checkout, data collection
- Features: Validation, multi-step, autosave

### 6. Chart/Visualization
- Libraries: Recharts, D3, Chart.js
- Use cases: Analytics, reports, dashboards
- Features: Interactive legends, tooltips, drill-down

## Troubleshooting

### Widget Not Rendering

**Symptom:** ChatGPT shows text response but no widget

**Possible Causes:**
1. Wrong MIME type - MUST be `text/html+skybridge`
2. Missing `_meta["openai/outputTemplate"]` in tool response
3. Resource URI doesn't match template reference
4. CORS not allowing `https://chatgpt.com`
5. Widget bundle has JavaScript errors

**Fix:**
```python
# Ensure correct MIME type
return FileResponse(
    "widget.html",
    media_type="text/html+skybridge"  # ← Critical!
)

# Ensure metadata is correct
return {
    "_meta": {
        "openai/outputTemplate": "ui://widget/my-widget.html"  # ← Must match resource URI
    }
}
```

### Tool Not Being Called

**Symptom:** ChatGPT responds with general knowledge instead of using your tool

**Possible Causes:**
1. Tool name too generic
2. Docstring/description unclear
3. Tool not registered properly
4. MCP server not responding

**Fix:**
```python
# BAD: Generic name and description
@mcp.tool()
async def search(query: str):
    """Search for results"""
    pass

# GOOD: Specific name and clear description
@mcp.tool()
async def find_pizza_restaurants(location: str, max_results: int = 10):
    """
    Search for pizza restaurants near a specific location.
    
    Use this tool when the user wants to find pizza places,
    pizzerias, or Italian restaurants that serve pizza.
    """
    pass
```

### CORS Errors

**Symptom:** Browser console shows CORS error when loading widget

**Fix:**
```python
# Python/FastAPI
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://chatgpt.com"],  # Be specific!
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Widget State Not Persisting

**Symptom:** Widget state resets on refresh or new tool calls

**Cause:** Not using `setWidgetState` properly

**Fix:**
```typescript
// Use the official hook
const [state, setState] = useWidgetState({ count: 0 });

// Updates persist across sessions
setState(prev => ({ count: prev.count + 1 }));
```

## Example Apps to Reference

When building, reference these official examples from the OpenAI repository:

1. **Pizzaz List** - Card list with favorites and CTAs
2. **Pizzaz Carousel** - Horizontal scroller for media
3. **Pizzaz Map** - Mapbox integration with fullscreen
4. **Pizzaz Album** - Stacked gallery view
5. **Pizzaz Video** - Video player with overlays
6. **Solar System** - 3D interactive visualization

## Checklist Before Publishing

- [ ] Tool names are specific and descriptive
- [ ] All tools have clear docstrings
- [ ] Read-only tools marked with `openai/readOnlyHint`
- [ ] CORS restricted to `https://chatgpt.com`
- [ ] Widget uses `text/html+skybridge` MIME type
- [ ] Widget handles light and dark themes
- [ ] Widget supports all display modes (inline, fullscreen, PiP)
- [ ] Widget is accessible (WCAG AA contrast, alt text, keyboard nav)
- [ ] No HTML forms used (use divs + onClick instead)
- [ ] Widget state is minimal (<4k tokens)
- [ ] Tested with MCP Inspector
- [ ] Tested in ChatGPT Developer Mode
- [ ] HTTPS endpoint configured for production
- [ ] OAuth implemented if needed
- [ ] Rate limiting configured
- [ ] Error handling for all tool calls
- [ ] Widget bundles optimized and hosted on CDN

## Resources

- Official Docs: https://developers.openai.com/apps-sdk/
- Examples Repo: https://github.com/openai/openai-apps-sdk-examples
- MCP Spec: https://spec.modelcontextprotocol.io/
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- FastMCP: https://github.com/jlowin/fastmcp

## Summary

When building OpenAI Apps SDK applications:

1. **Start with the MCP server** - Define tools with clear schemas
2. **Build widget components** - React apps using window.openai API
3. **Connect with metadata** - Use `_meta["openai/outputTemplate"]`
4. **Test locally** - Use MCP Inspector and ngrok
5. **Deploy with HTTPS** - Required for ChatGPT integration
6. **Follow design guidelines** - Accessible, responsive, theme-aware

The key insight: Apps SDK combines conversational AI with rich UI by using MCP as the protocol bridge. The model decides when to call tools, tools return data + UI metadata, and widgets render the interactive experience - all within the ChatGPT chat interface.
