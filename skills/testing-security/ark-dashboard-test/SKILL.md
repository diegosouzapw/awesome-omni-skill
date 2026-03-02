---
name: Ark Dashboard Test
description: Test the Ark Dashboard UI with Playwright
---

# Ark Dashboard Test

Test Ark Dashboard with Playwright and capture screenshots.

## When to use

- Testing Ark dashboard UI
- Capturing screenshots for PRs
- Validating dashboard changes

## Prerequisites

- Ark deployed and running
- Playwright MCP server available

## Steps

1. **Port forward the dashboard**
   ```bash
   kubectl port-forward svc/ark-dashboard 3000:3000 -n default &
   ```

2. **Test with Playwright MCP tools**
   - `browser_navigate` - Open http://localhost:3000
   - `browser_snapshot` - Check page state
   - `browser_click` - Interact with elements
   - `browser_take_screenshot` - Capture screenshots

3. **Screenshots location**
   Screenshots save to current directory or `.playwright-mcp/screenshots/`.
