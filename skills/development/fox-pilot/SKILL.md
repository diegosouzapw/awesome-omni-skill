---
name: fox-pilot
description: Firefox browser automation CLI for AI agents. Use when users ask to automate Firefox, navigate websites, fill forms, take screenshots, extract web data, or test web apps in Firefox. Trigger phrases include "in Firefox", "fox-pilot", "go to [url]", "click on", "fill out the form", "take a screenshot", "scrape", "automate", or any browser interaction request mentioning Firefox.
---

# Fox Pilot Skill

CLI-based Firefox browser automation optimized for AI agents. Simple commands, persistent sessions, accessibility snapshots with refs.

**Difference from agent-browser:** Fox Pilot controls a real Firefox browser (not Chromium), preserving your existing session, cookies, and extensions.

## Setup

```bash
# Install the CLI globally
npm install -g @fox-pilot/cli

# Install native messaging host (run once)
fox-pilot install

# Install Firefox extension from:
# https://addons.mozilla.org/firefox/addon/fox-pilot/
```

## Quick Start

```bash
fox-pilot open example.com
fox-pilot snapshot                    # Get accessibility tree with refs
fox-pilot click @e2                   # Click by ref from snapshot
fox-pilot fill @e3 "test@example.com" # Fill by ref
fox-pilot screenshot /tmp/page.png
```

## Workflow Pattern

1. **Navigate** to URL
2. **Snapshot** to discover elements (use `-i` for interactive only)
3. **Interact** using refs (`@e1`, `@e2`, etc.)
4. **Verify** state with `get url`, `get title`, or screenshot
5. **Repeat** as needed

## Core Commands

### Navigation

```bash
fox-pilot open <url>               # Navigate to URL
fox-pilot back                     # Go back
fox-pilot forward                  # Go forward
fox-pilot reload                   # Reload page
```

### Snapshot (Element Discovery)

The snapshot command returns an accessibility tree with refs for each element:

```bash
fox-pilot snapshot
# Output:
# - heading "Example Domain" [ref=@e1] [level=1]
# - button "Submit" [ref=@e2]
# - textbox "Email" [ref=@e3]
# - link "Learn more" [ref=@e4]
```

#### Snapshot Options

```bash
fox-pilot snapshot -i              # Interactive elements only (buttons, inputs, links)
fox-pilot snapshot -c              # Compact (remove empty structural elements)
fox-pilot snapshot -d 3            # Limit depth to 3 levels
fox-pilot snapshot -s "#main"      # Scope to CSS selector
fox-pilot snapshot -i -c -d 5      # Combine options
```

**Recommended:** Use `snapshot -i -c` for most cases - shows only actionable elements.

### Interactions

```bash
fox-pilot click <sel>              # Click element
fox-pilot dblclick <sel>           # Double-click
fox-pilot fill <sel> <text>        # Clear and fill input
fox-pilot type <sel> <text>        # Type into element (append)
fox-pilot press <key> [sel]        # Press key (Enter, Tab, Control+a)
fox-pilot select <sel> <val>       # Select dropdown option
fox-pilot check <sel>              # Check checkbox
fox-pilot uncheck <sel>            # Uncheck checkbox
fox-pilot scroll <dir> [px]        # Scroll (up/down/left/right)
fox-pilot hover <sel>              # Hover element
```

### Get Information

```bash
fox-pilot get text <sel>           # Get text content
fox-pilot get html <sel>           # Get innerHTML
fox-pilot get value <sel>          # Get input value
fox-pilot get attr <sel> <attr>    # Get attribute
fox-pilot get title                # Get page title
fox-pilot get url                  # Get current URL
fox-pilot get count <sel>          # Count matching elements
```

### Check State

```bash
fox-pilot is visible <sel>         # Check if visible
fox-pilot is enabled <sel>         # Check if enabled
fox-pilot is checked <sel>         # Check if checked
```

### Screenshots

```bash
fox-pilot screenshot [path]        # Take screenshot
fox-pilot screenshot -f [path]     # Full page screenshot
```

### Waiting

```bash
fox-pilot wait <selector>          # Wait for element visible
fox-pilot wait 2000                # Wait 2 seconds
fox-pilot wait --text "Welcome"    # Wait for text
fox-pilot wait --url "**/success"  # Wait for URL pattern
```

### Semantic Locators (find command)

```bash
fox-pilot find role button click --name "Submit"
fox-pilot find text "Sign In" click
fox-pilot find label "Email" fill "test@test.com"
fox-pilot find placeholder "Search" fill "query"
```

### Tabs

```bash
fox-pilot tab                      # List tabs
fox-pilot tab new [url]            # New tab
fox-pilot tab 2                    # Switch to tab 2
fox-pilot tab close                # Close current tab
```

### JavaScript Execution

```bash
fox-pilot eval "return document.title"
fox-pilot eval "localStorage.getItem('token')"
```

## Selectors

### Refs (Recommended)

Use refs from snapshot for reliable element selection:

```bash
fox-pilot click @e2
fox-pilot fill @e3 "value"
fox-pilot get text @e1
```

### CSS Selectors

```bash
fox-pilot click "#id"
fox-pilot click ".class"
fox-pilot click "button[type=submit]"
```

## Common Patterns

### Login Flow

```bash
fox-pilot open https://example.com/login
fox-pilot snapshot -i
# Shows: textbox "Email" [ref=@e1], textbox "Password" [ref=@e2], button "Sign in" [ref=@e3]
fox-pilot fill @e1 "user@example.com"
fox-pilot fill @e2 "password123"
fox-pilot click @e3
fox-pilot wait --url "**/dashboard"
fox-pilot get url
```

### Form Submission

```bash
fox-pilot open https://example.com/contact
fox-pilot snapshot -i -c
fox-pilot fill @e1 "John Doe"
fox-pilot fill @e2 "john@example.com"
fox-pilot fill @e3 "Hello, this is my message"
fox-pilot click @e4  # Submit button
fox-pilot wait --text "Thank you"
fox-pilot screenshot /tmp/confirmation.png
```

### Scraping Data

```bash
fox-pilot open https://example.com/products
fox-pilot snapshot -s ".product-list"
fox-pilot get text ".product-title"
fox-pilot get attr ".product-link" "href"
```

### Debug Failed Interaction

```bash
fox-pilot screenshot /tmp/debug.png
fox-pilot get url
fox-pilot get title
fox-pilot snapshot -i
```

## Options

| Option             | Description               |
| ------------------ | ------------------------- |
| `--json`           | JSON output (for parsing) |
| `-f, --full`       | Full page screenshot      |
| `-i, --interactive`| Interactive elements only |
| `-c, --compact`    | Compact snapshot          |
| `-d, --depth <n>`  | Limit snapshot depth      |
| `-s, --scope <sel>`| Scope snapshot to selector|

## Tips

- **Always snapshot first** before interacting with unknown pages
- **Use `-i -c` flags** on snapshot to reduce noise
- **Prefer refs over CSS** for reliability
- **Check `--json` output** when you need to parse results programmatically
- **Firefox session is preserved** - your cookies, logins, and extensions work

## Troubleshooting

```bash
# Check if native host is running
lsof -i :9222

# View native host logs
tail -f /tmp/fox-pilot.log

# Reinstall native host
fox-pilot install

# Reload extension
# Go to about:debugging#/runtime/this-firefox and click "Reload"
```
