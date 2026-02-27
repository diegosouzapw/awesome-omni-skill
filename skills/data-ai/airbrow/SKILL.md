---
name: airbrow
description: Ultra-fast browser CLI for LLM agents. Use for web navigation, form filling, screenshots, data extraction, and web testing. 50x less memory than Playwright, 10x smaller context output. Rust/CDP based, zero dependencies.
---

# Browser Automation with airbrow

## Quick start

```bash
open                          # Start browser
goto https://example.com      # Navigate to page
observe                       # Get interactive elements with refs
click r1                      # Click element by ref
fill r2 "text"                # Fill input by ref
quit                          # Close browser
```

## Core workflow

1. Start: `open` (or `open -v` for visible browser)
2. Navigate: `goto <url>`
3. Observe: `observe` (returns elements with refs like `r1`, `r2`)
4. Interact using refs from observe output
5. Re-observe after navigation or significant DOM changes

## Commands

### Session

```bash
open                   # Start headless browser
open -v                # Start visible browser (headed)
reconnect              # Reconnect to browser if disconnected
quit                   # Close browser and exit
```

### Navigation

```bash
goto <url>             # Navigate to URL
back                   # Go back
forward                # Go forward
reload                 # Reload page
```

### Observation (page analysis)

```bash
observe                # Interactive elements in viewport (recommended)
snap                   # URL and title only
snapshot               # Full page with text content
content                # Main HTML content (5KB max)
content full           # Full HTML
```

### Interactions (use refs from observe)

```bash
click r1               # Click element
dblclick r1            # Double-click
fill r2 "text"         # Clear and fill input
type r2 "text"         # Type char-by-char (for autocomplete)
press Enter            # Press key
press Tab              # Tab key
press Escape           # Escape key
hover r1               # Hover element
focus r1               # Focus element
blur                   # Remove focus
check r1               # Check checkbox
uncheck r1             # Uncheck checkbox
select r1 "value"      # Select dropdown option
drag r1 r2             # Drag from r1 to r2
```

### Scroll

```bash
scroll down            # Scroll down 500px
scroll up              # Scroll up 500px
scroll 1000            # Scroll down 1000px
scroll -500            # Scroll up 500px
scrollto r1            # Scroll element into view
top                    # Scroll to top
bottom                 # Scroll to bottom
pageup                 # Scroll up by viewport
pagedown               # Scroll down by viewport
```

### Get information

```bash
get text r1            # Get element text
get value r1           # Get input value
get html r1            # Get element HTML
get title              # Get page title
get url                # Get current URL
get count "selector"   # Count matching elements
```

### Screenshots & PDF

```bash
screenshot             # Save to shot.png
screenshot path.png    # Save to specific path
pdf                    # Save to page.pdf
pdf output.pdf         # Save to specific path
```

### Wait

```bash
wait 2000              # Wait milliseconds
wait r1                # Wait for element
wait r1 5000           # Wait for element with timeout
waitvisible r1         # Wait for element to be visible
waitresponse api/data  # Wait for network response matching pattern
isvisible r1           # Check if element is visible
ischecked r1           # Check if checkbox is checked
```

### Multi-tab

```bash
newtab                 # Open new blank tab
newtab https://...     # Open new tab with URL
tabs                   # List all tabs
tab 0                  # Switch to tab 0
tab 1                  # Switch to tab 1
closetab               # Close current tab
closetab 2             # Close tab 2
```

### Iframe

```bash
frames                 # List all frames
frame 0                # Switch to frame by index
frame "name"           # Switch to frame by name
mainframe              # Return to main frame
```

### Network control

```bash
block "*.jpg"          # Block URLs matching pattern
block "ads.com/*"      # Block ad domain
blockimages            # Block all images
blockmedia             # Block video/audio
blockfonts             # Block fonts
unblock                # Remove all blocks
```

### Cookies

```bash
cookies                # Get all cookies
cookies .example.com   # Get cookies for domain
setcookie name val .example.com  # Set cookie
delcookie name .example.com      # Delete cookie
clearcookies           # Clear all cookies
```

### Storage

```bash
storage get key        # Get localStorage value
storage set key value  # Set localStorage value
storage del key        # Delete key
storage clear          # Clear localStorage
```

### Files

```bash
upload r1 /path/file.pdf           # Upload file to input
upload r1 file1.pdf file2.pdf      # Upload multiple files
downloadpath /path/to/downloads    # Set download directory
```

### Device emulation

```bash
viewport 1920 1080     # Set viewport size
mobile                 # Mobile emulation (375x667)
dark                   # Enable dark mode
light                  # Enable light mode
print                  # Print media type
```

### JavaScript

```bash
eval "document.title"              # Execute JavaScript
eval "window.scrollY"              # Get scroll position
eval "localStorage.clear()"        # Clear storage via JS
```

### Semantic locators (alternative to refs)

```bash
click text:Submit                  # Click by text content
click role:button                  # Click by ARIA role
fill label:Email "user@test.com"   # Fill by label
fill placeholder:Search "query"   # Fill by placeholder
click //button[@type='submit']    # Click by XPath
```

## Example: Google search

```bash
open
goto https://google.com
observe
# Output: [["r1","in","Search"],["r2","btn","Google Search"]]

fill r1 "claude ai"
press Enter
wait 1000
observe
```

## Example: Form submission

```bash
open
goto https://example.com/login
observe
# Output: [["r1","in","Email"],["r2","in","Password"],["r3","btn","Sign In"]]

fill r1 "user@example.com"
fill r2 "password123"
click r3
wait 2000
observe
```

## Example: Multi-tab research

```bash
open
goto https://google.com
observe

newtab https://bing.com
observe

tabs
# Output: {"tabs":[{"i":0,"u":"google.com"},{"i":1,"u":"bing.com"}]}

tab 0
observe
```

## Example: Scraping with network optimization

```bash
open
blockimages
blockmedia
goto https://news.ycombinator.com
observe
content
```

## Example: Dark mode screenshot

```bash
open
goto https://example.com
dark
wait 500
screenshot dark_mode.png
```

## Example: File upload

```bash
open
goto https://example.com/upload
observe
# Find file input: [["r3","in","Choose File"]]

upload r3 /path/to/document.pdf
click text:Submit
waitresponse upload 30000
observe
```

## Output format

### observe output
```json
{"e":[["r1","btn","Submit"],["r2","in","Email"]],"n":2,"t":"Page Title","u":"https://example.com"}
```

- `e`: Elements array `[ref, role, label]`
- `n`: Element count
- `t`: Page title
- `u`: Current URL

### Element roles
| Role | Meaning |
|------|---------|
| `a` | Link |
| `btn` | Button |
| `in` | Input |
| `sel` | Select |
| `txt` | Textarea |
| `combobox` | Combobox |

## Performance

| Metric | airbrow | Playwright |
|--------|---------|------------|
| Memory | 3.5 MB | 150+ MB |
| Context | ~2 KB | ~20 KB |
| Startup | ~200ms | ~1500ms |
| Binary | 2 MB | Node runtime |

## Tips

- Always `observe` before interacting to get current refs
- Use `blockimages` for faster page loads
- Use `type` instead of `fill` for search boxes (triggers autocomplete)
- Refs (`r1`, `r2`) are more reliable than CSS selectors
- `observe` only returns elements visible in viewport - use `pagedown` to see more
- Re-observe after any navigation or DOM change