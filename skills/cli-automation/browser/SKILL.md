---
name: Browser
description: Code-first Playwright automation via TypeScript scripts. USE WHEN writing reusable automation scripts, VERIFY phase (confirming a web change actually works), headless programmatic testing, or need token-efficient browser automation in code. NOT for quick one-off CLI tasks (use AgentBrowser), NOT for authenticated sites with saved logins (use ChromeMCP+WebExplore), NOT for documenting a UI into a spec (use WebExplore).
---

# Browser - Code-First Browser Automation

**Browser automation and web verification using code-first Playwright.**

---

## 🔌 File-Based MCP

This skill is a **file-based MCP** - a code-first API wrapper that replaces token-heavy MCP protocol calls.

**Why file-based?** Filter data in code BEFORE returning to model context = 99%+ token savings.

**Architecture:** See `$PAI_DIR/skills/CORE/SYSTEM/DOCUMENTATION/FileBasedMCPs.md`

---

## Quick Start

```typescript
import { PlaywrightBrowser } from '$PAI_DIR/skills/Browser/index.ts'

const browser = new PlaywrightBrowser()
await browser.launch()
await browser.navigate('https://example.com')
await browser.screenshot({ path: 'screenshot.png' })
await browser.close()
```

**Why This Approach:**
- MCP loads ~13,700 tokens at startup
- Code-first loads ~50-200 tokens per operation
- Full Playwright API access, not limited to 21 MCP tools

---

## Voice Notification

**When executing a Browser workflow, do BOTH:**

1. **Send voice notification**:
   ```bash
   curl -s -X POST http://localhost:8888/notify \
     -H "Content-Type: application/json" \
     -d '{"message": "Running the Browser workflow"}' \
     > /dev/null 2>&1 &
   ```

2. **Output text notification**:
   ```
   Running the **Browser** workflow...
   ```

---

## Workflow Routing

| Trigger | Workflow |
|---------|----------|
| Navigate to URL, take screenshot | `Workflows/Screenshot.md` |
| Verify page loads correctly | `Workflows/VerifyPage.md` |
| Fill forms, interact with page | `Workflows/Interact.md` |
| Extract page content | `Workflows/Extract.md` |

---

## API Reference

### Navigation
```typescript
await browser.launch(options?)      // Start browser
await browser.navigate(url)         // Go to URL
await browser.goBack()              // History back
await browser.goForward()           // History forward
await browser.reload()              // Refresh
browser.getUrl()                    // Current URL
await browser.getTitle()            // Page title
await browser.close()               // Shut down browser
```

### Capture
```typescript
await browser.screenshot({ path, fullPage, selector })
await browser.getVisibleText(selector?)
await browser.getVisibleHtml({ removeScripts, minify })
await browser.savePdf(path, { format })
await browser.getAccessibilityTree()
```

### Network Monitoring
```typescript
browser.getNetworkLogs(options?)    // Get all network requests/responses
browser.getNetworkStats()           // Get summary statistics
browser.clearNetworkLogs()          // Clear captured logs
```

### Dialog Handling
```typescript
browser.setDialogHandler(auto, response?)   // Configure auto-handling
browser.getPendingDialog()                   // Get current dialog info
await browser.handleDialog(action, promptText?)  // Handle dialog manually
```

### Tab Management
```typescript
browser.getTabs()                   // List all open tabs
await browser.newTab(url?)          // Open new tab
await browser.switchTab(index)      // Switch to tab by index
await browser.closeTab()            // Close current tab
```

### Interaction
```typescript
await browser.click(selector)
await browser.hover(selector)
await browser.fill(selector, value)
await browser.type(selector, text, delay?)
await browser.select(selector, value)
await browser.pressKey(key, selector?)
await browser.drag(source, target)
await browser.uploadFile(selector, path)
```

### Waiting
```typescript
await browser.waitForSelector(selector, { state, timeout })
await browser.waitForText(text, { state, timeout })
await browser.waitForNavigation({ url, timeout })
await browser.waitForNetworkIdle(timeout?)
await browser.wait(ms)
await browser.waitForResponse(urlPattern)
```

### JavaScript
```typescript
await browser.evaluate(script)
browser.getConsoleLogs({ type, search, limit, clear })
await browser.setUserAgent(ua)
```

### Viewport
```typescript
await browser.resize(width, height)
await browser.setDevice('iPhone 14')
```

### iFrame
```typescript
await browser.iframeClick(iframeSelector, elementSelector)
await browser.iframeFill(iframeSelector, elementSelector, value)
```

---

## VERIFY Phase Integration

**The Browser skill is MANDATORY for VERIFY phase of web changes.**

Before claiming ANY web change is "live" or "working":

1. Launch browser
2. Navigate to the EXACT URL
3. Verify the EXACT element that changed
4. Take screenshot as evidence
5. Close browser

```typescript
// VERIFY Phase Pattern
import { PlaywrightBrowser } from '$PAI_DIR/skills/Browser/index.ts'

const browser = new PlaywrightBrowser()
await browser.launch({ headless: true })
await browser.navigate('https://example.com/changed-page')
await browser.waitForSelector('.changed-element')
const text = await browser.getVisibleText('.changed-element')
await browser.screenshot({ path: '/tmp/verify.png' })
await browser.close()

console.log(`Verified: "${text}"`)
```

**If you haven't LOOKED at the rendered page, you CANNOT claim it works.**

---

## CLI Tool

**Location:** `Tools/Browse.ts`

```bash
# Open URL in visible browser
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts open <url>

# Take screenshot
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts screenshot <url> [path]

# Verify element exists
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts verify <url> <selector>
```

**Examples:**
```bash
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts open https://danielmiessler.com
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts screenshot https://example.com /tmp/shot.png
bun run $PAI_DIR/skills/Browser/Tools/Browse.ts verify https://example.com "body"
```

---

## Examples

### Verify Page Loads

```bash
bun $PAI_DIR/skills/Browser/examples/verify-page.ts https://danielmiessler.com
```

### Take Screenshot

```bash
bun $PAI_DIR/skills/Browser/examples/screenshot.ts https://example.com screenshot.png
```

### Fill Form

```typescript
const browser = new PlaywrightBrowser()
await browser.launch()
await browser.navigate('https://example.com/form')
await browser.fill('#email', 'test@example.com')
await browser.fill('#password', 'secret')
await browser.click('button[type="submit"]')
await browser.waitForNavigation()
await browser.close()
```

---

## Alternative Implementations (Reference Only)

### Option A: Playwright MCP (Microsoft Official)
```bash
# npx @playwright/mcp@latest
# 25K GitHub stars, uses accessibility tree
# Pro: Official Microsoft support, well-maintained
# Con: 13,700 tokens at startup
```

### Option B: Chrome DevTools MCP (Google Official)
```bash
# npx @anthropic/chrome-devtools-mcp
# Best debugging capabilities, CDP protocol
# Pro: Deep browser internals access
# Con: Chrome-only, complex setup
```

### Option C: claude --chrome (Native Anthropic)
```bash
# claude --chrome
# Simplest option - built into Claude Code
# Pro: Zero configuration, native integration
# Con: Limited API compared to Playwright
```

### Option D: Stagehand (Browserbase)
```bash
# npx stagehand
# 19.9K stars, won Anthropic hackathon
# Pro: AI-native actions (act, extract, observe)
# Con: Emerging, less mature than Playwright
```

---

## Token Savings Comparison

| Approach | Tokens | Notes |
|----------|--------|-------|
| Playwright MCP | ~13,700 | Loaded at startup, always |
| Code-first | ~50-200 | Only what you use |
| **Savings** | **99%+** | Per operation |

---

## Full Documentation

**CLI Tool:** `Tools/Browse.ts`
**Implementation:** `README.md`
**API Reference:** `index.ts`
**Examples:** `examples/`
