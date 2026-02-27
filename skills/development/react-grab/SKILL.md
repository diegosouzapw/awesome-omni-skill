---
name: react-grab
description: Installs and configures React Grab for visual UI element selection in React/Electron apps. Use when user wants to edit UI visually, select components by hovering, or capture element context.
---

# React Grab Integration

## Step 1: Detect Project Type

**IMPORTANT: Before installing, detect the project type by checking these files:**

| Check For | Project Type |
|-----------|--------------|
| `electron-vite.config.*` | electron-vite |
| `forge.config.*` + `vite.*.config.*` | Electron Forge + Vite |
| `forge.config.*` + `webpack.*.config.*` | Electron Forge + Webpack |
| `electron` in package.json + `webpack.config.*` | Electron + Webpack |
| `electron` in package.json + `vite.config.*` | Electron + Vite |
| `electron` in package.json (no bundler) | Plain Electron |
| `next.config.*` | Next.js |
| `vite.config.*` (no electron) | Vite |
| `webpack.config.*` (no electron) | Webpack |

**Detection Commands:**
```bash
# Check package.json for framework
cat package.json | grep -E "(electron|next|vite)"

# Check for config files
ls -la | grep -E "(electron-vite|forge|next|vite|webpack)\.config"
```

---

## Step 2: Install

```bash
# Use the project's package manager (check lockfile)
pnpm add -D react-grab   # if pnpm-lock.yaml exists
npm install -D react-grab  # if package-lock.json exists
yarn add -D react-grab   # if yarn.lock exists
```

---

## Step 3: Configure Based on Project Type

### A. Next.js

**File to edit:** `app/layout.tsx` (App Router) or `pages/_app.tsx` (Pages Router)

**App Router - `app/layout.tsx`:**
```typescript
import Script from "next/script"

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <head>
        {process.env.NODE_ENV === "development" && (
          <Script src="//unpkg.com/react-grab/dist/index.global.js" strategy="beforeInteractive" />
        )}
      </head>
      <body>{children}</body>
    </html>
  )
}
```

**Pages Router - `pages/_app.tsx`:**
```typescript
import Script from "next/script"
import type { AppProps } from "next/app"

export default function App({ Component, pageProps }: AppProps) {
  return (
    <>
      {process.env.NODE_ENV === "development" && (
        <Script src="//unpkg.com/react-grab/dist/index.global.js" strategy="beforeInteractive" />
      )}
      <Component {...pageProps} />
    </>
  )
}
```

---

### B. Vite (Web Only)

**File to edit:** `src/main.tsx` or `src/index.tsx`

```typescript
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"

// Add this block at the top, after imports
if (import.meta.env.DEV) {
  import("react-grab").then(({ init }) => init())
}

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

---

### C. electron-vite

**Files to edit:**
1. `src/renderer/src/main.tsx` - Initialize react-grab
2. `src/main/index.ts` - Configure CSP

**Step C1 - `src/renderer/src/main.tsx`:**
```typescript
// Add at the top, after imports
if (import.meta.env.DEV) {
  import("react-grab").then(({ init }) => init())
}
```

**Step C2 - `src/main/index.ts`:** Find the `BrowserWindow` creation and add CSP:
```typescript
import { app, BrowserWindow } from "electron"
import { join } from "path"

function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: join(__dirname, "../preload/index.js"),
      sandbox: false  // Required for react-grab source mapping
    }
  })

  // Add this block for development CSP
  if (process.env.NODE_ENV === "development") {
    mainWindow.webContents.session.webRequest.onHeadersReceived((details, callback) => {
      callback({
        responseHeaders: {
          ...details.responseHeaders,
          "Content-Security-Policy": [
            "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'"
          ]
        }
      })
    })
  }

  // ... rest of window setup
}
```

---

### D. Electron Forge + Vite

**Files to edit:**
1. `src/renderer.ts` or `src/renderer/index.tsx` - Initialize react-grab
2. `src/main.ts` - Configure CSP

**Step D1 - Renderer entry:**
```typescript
// Add at the top, after imports
if (import.meta.env.DEV) {
  import("react-grab").then(({ init }) => init())
}
```

**Step D2 - `src/main.ts`:** Add CSP configuration (same as electron-vite Step C2)

---

### E. Electron + Webpack

**Files to edit:**
1. `src/renderer/index.tsx` - Initialize react-grab
2. `src/main/main.ts` - Configure CSP

**Step E1 - `src/renderer/index.tsx`:**
```typescript
// Add at the top, after imports
if (process.env.NODE_ENV === "development") {
  import("react-grab").then(({ init }) => init())
}
```

**Step E2 - Main process:** Add CSP configuration (same as electron-vite Step C2)

---

### F. Plain Electron (No Bundler)

**Files to edit:**
1. `index.html` - Add script tag
2. `main.js` - Configure BrowserWindow
3. `renderer.js` - Initialize react-grab

**Step F1 - `index.html`:**
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="Content-Security-Policy"
          content="default-src 'self'; script-src 'self' 'unsafe-inline' unpkg.com; style-src 'self' 'unsafe-inline'" />
    <script src="https://unpkg.com/react-grab/dist/index.global.js"></script>
  </head>
  <body>
    <div id="root"></div>
    <script src="./renderer.js"></script>
  </body>
</html>
```

**Step F2 - `main.js`:**
```javascript
const { app, BrowserWindow } = require("electron")
const path = require("path")

function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, "preload.js"),
      contextIsolation: true,
      nodeIntegration: false,
      sandbox: false  // Required for react-grab source mapping
    }
  })

  mainWindow.loadFile("index.html")

  if (process.env.NODE_ENV === "development") {
    mainWindow.webContents.openDevTools()
  }
}

app.whenReady().then(createWindow)
```

**Step F3 - `renderer.js`:**
```javascript
// Initialize react-grab (loaded globally via script tag)
if (window.ReactGrab) {
  window.ReactGrab.init()
}
```

---

## Step 4: Verify Installation

**Run the dev server and check:**
1. Open browser DevTools console
2. Look for: `[react-grab] initialized` message
3. Hover over any element and press **Cmd+C** (Mac) or **Ctrl+C** (Windows)
4. Paste - you should see component info

**Expected output:**
```
<button class="btn">Submit</button>
in SubmitButton
components/button.tsx:42:19
```

---

## Troubleshooting

### All Projects

| Issue | Solution |
|-------|----------|
| "react-grab not defined" | Ensure script loads before React renders |
| Nothing copied on Cmd/Ctrl+C | Check browser console for errors |
| Wrong file paths | Enable source maps in bundler config |

### Electron-Specific

| Issue | Solution |
|-------|----------|
| CSP blocking script | Add `'unsafe-inline' 'unsafe-eval'` to CSP |
| Source maps not working | Set `sandbox: false` in webPreferences |
| Keyboard shortcut not working | Check for conflicting Menu accelerators |
| DevTools eval warning | Expected in dev - CSP needs `'unsafe-eval'` |

### CSP Reference for Electron

**Via Session Headers (Recommended):**
```typescript
// In main process after creating BrowserWindow
if (process.env.NODE_ENV === "development") {
  mainWindow.webContents.session.webRequest.onHeadersReceived((details, callback) => {
    callback({
      responseHeaders: {
        ...details.responseHeaders,
        "Content-Security-Policy": [
          "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'"
        ]
      }
    })
  })
}
```

**Via Meta Tag:**
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'" />
```

---

## Optional: Claude Code Integration

**Add to `package.json` scripts:**

| Project Type | Script |
|--------------|--------|
| Next.js | `"dev:grab": "npx @react-grab/claude-code@latest & next dev"` |
| Vite | `"dev:grab": "npx @react-grab/claude-code@latest & vite"` |
| electron-vite | `"dev:grab": "concurrently \"npx @react-grab/claude-code@latest\" \"electron-vite dev\""` |
| Electron Forge | `"dev:grab": "concurrently \"npx @react-grab/claude-code@latest\" \"electron-forge start\""` |

**Client-side (add after react-grab init):**
```typescript
if (import.meta.env.DEV) {
  import("react-grab").then(() => {
    const script = document.createElement("script")
    script.src = "//unpkg.com/@react-grab/claude-code/dist/client.global.js"
    document.head.appendChild(script)
  })
}
```
