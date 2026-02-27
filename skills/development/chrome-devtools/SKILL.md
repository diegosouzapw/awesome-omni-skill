---
name: chrome-devtools
description: Chrome DevTools automation for browser testing, debugging, and performance analysis
license: MIT
compatibility: opencode
metadata:
  audience: qa-engineers frontend-developers
  tooling: chrome-devtools
---

## What I do
- Automate browser interactions through Chrome DevTools Protocol
- Navigate pages, click elements, fill forms, and handle user interactions
- Debug issues by capturing console logs and network requests
- Take screenshots and snapshots for visual regression testing
- Record performance traces and analyze Core Web Vitals
- Emulate different devices, network conditions, and geolocations
- Execute JavaScript in the browser context for advanced debugging

## When to use me
Use me when you need to:
- Automate browser testing and interaction flows
- Debug frontend issues by examining console output and network activity
- Capture screenshots for documentation or regression testing
- Analyze page performance with performance traces
- Test responsive designs with device emulation
- Inspect and interact with page elements programmatically
- Debug WebSocket or network request failures
- Verify page behavior across different network conditions

## Page management
```javascript
// List all open pages
chrome-devtools_list_pages()

// Select a page by ID
chrome-devtools_select_page({ pageId: 0 })

// Navigate to URL
chrome-devtools_navigate_page({ type: "url", url: "https://example.com" })

// Navigate back/forward/reload
chrome-devtools_navigate_page({ type: "back" })
chrome-devtools_navigate_page({ type: "forward" })
chrome-devtools_navigate_page({ type: "reload", ignoreCache: true })

// Create new page
chrome-devtools_new_page({ url: "https://example.com" })

// Close a page (except the last one)
chrome-devtools_close_page({ pageId: 1 })
```

## Page interaction
```javascript
// Take snapshot of page elements
const snapshot = chrome-devtools_take_snapshot()
// Returns elements with unique UIDs for interaction

// Click an element
chrome-devtools_click({ uid: "element-123" })

// Double-click an element
chrome-devtools_click({ uid: "element-123", dblClick: true })

// Hover over element
chrome-devtools_hover({ uid: "element-123" })

// Drag and drop
chrome-devtools_drag({
  from_uid: "source-element",
  to_uid: "target-element"
})

// Type text
chrome-devtools_fill({ uid: "input-456", value: "Hello, World!" })

// Fill multiple form elements at once
chrome-devtools_fill_form({
  elements: [
    { uid: "input-name", value: "John Doe" },
    { uid: "input-email", value: "john@example.com" },
    { uid: "select-country", value: "USA" }
  ]
})

// Press keyboard shortcuts
chrome-devtools_press_key({ key: "Enter" })
chrome-devtools_press_key({ key: "Control+A" })
chrome-devtools_press_key({ key: "Control+Shift+R" })
```

## Screenshots and snapshots
```javascript
// Take viewport screenshot
chrome-devtools_take_screenshot({ format: "png" })

// Take full page screenshot
chrome-devtools_take_screenshot({ fullPage: true, format: "png" })

// Take screenshot of specific element
chrome-devtools_take_screenshot({
  uid: "element-123",
  format: "jpeg",
  quality: 90
})

// Save screenshot to file
chrome-devtools_take_screenshot({
  filePath: "./screenshots/homepage.png"
})

// Take text snapshot (a11y tree)
const snapshot = chrome-devtools_take_snapshot()
// Use UIDs from snapshot for subsequent interactions
```

## Console debugging
```javascript
// List all console messages
chrome-devtools_list_console_messages()

// Filter by message types
chrome-devtools_list_console_messages({
  types: ["error", "warn"]
})

// Get specific message by ID
chrome-devtools_get_console_message({ msgid: 1 })

// Execute JavaScript in page context
chrome-devtools_evaluate_script({
  function: "() => { return document.title; }"
})

// Execute with element argument
chrome-devtools_evaluate_script({
  function: "(el) => { return el.innerText; }",
  args: [{ uid: "element-123" }]
})
```

## Network debugging
```javascript
// List all network requests
chrome-devtools_list_network_requests()

// Filter by resource types
chrome-devtools_list_network_requests({
  resourceTypes: ["xhr", "fetch", "document"]
})

// Get specific request details
chrome-devtools_get_network_request({ reqid: 42 })

// Get currently selected request in DevTools
chrome-devtools_get_network_request()
```

## Performance analysis
```javascript
// Start performance trace with page reload
chrome-devtools_performance_start_trace({
  reload: true,
  autoStop: true,
  filePath: "./traces/performance.json"
})

// Start trace without reload
chrome-devtools_performance_start_trace({
  reload: false,
  autoStop: false,
  filePath: "./traces/performance.json.gz"
})

// Stop active trace
chrome-devtools_performance_stop_trace({
  filePath: "./traces/performance.json"
})

// Analyze performance insights
chrome-devtools_performance_analyze_insight({
  insightSetId: "lcp-insight-set-123",
  insightName: "LCPBreakdown"
})
```

## Emulation and testing
```javascript
// Resize viewport
chrome-devtools_resize_page({ width: 375, height: 667 })  // iPhone SE
chrome-devtools_resize_page({ width: 1920, height: 1080 })  // Desktop

// Emulate network conditions
chrome-devtools_emulate({
  networkConditions: "Slow 3G"
})

// Emulate CPU throttling (1-20x slowdown)
chrome-devtools_emulate({
  cpuThrottlingRate: 4
})

// Emulate geolocation
chrome-devtools_emulate({
  geolocation: {
    latitude: 37.7749,
    longitude: -122.4194
  }
})

// Clear emulations
chrome-devtools_emulate({
  networkConditions: "No emulation",
  cpuThrottlingRate: 1,
  geolocation: null
})
```

## Waiting and synchronization
```javascript
// Wait for text to appear
chrome-devtools_wait_for({
  text: "Welcome",
  timeout: 10000
})

// Navigate with custom timeout
chrome-devtools_navigate_page({
  type: "url",
  url: "https://slow-site.com",
  timeout: 30000
})
```

## File upload
```javascript
// Upload file through file input
chrome-devtools_upload_file({
  uid: "file-input-123",
  filePath: "/path/to/file.pdf"
})
```

## Dialog handling
```javascript
// Accept dialog (alert, confirm, prompt)
chrome-devtools_handle_dialog({ action: "accept" })

// Dismiss dialog
chrome-devtools_handle_dialog({ action: "dismiss" })

// Accept with prompt text
chrome-devtools_handle_dialog({
  action: "accept",
  promptText: "User entered text"
})
```

## Best practices
- Always take a snapshot before interacting with elements to get stable UIDs
- Wait for elements to appear before clicking (use wait_for or check snapshot)
- Use explicit timeouts for slow-loading pages
- Save screenshots with descriptive names for documentation
- Capture console errors during test runs for debugging
- Use performance traces to identify bottlenecks
- Test responsive designs by emulating different screen sizes
- Filter network requests to focus on XHR/fetch for API testing
- Handle dialogs promptly after they appear

## Common workflows

### E2E test flow
```javascript
1. chrome-devtools_list_pages() - Check available pages
2. chrome-devtools_select_page({ pageId: 0 }) - Select target page
3. chrome-devtools_navigate_page({ type: "url", url: "..." }) - Navigate
4. chrome-devtools_take_snapshot() - Get page structure
5. chrome-devtools_fill({ uid: "...", value: "..." }) - Fill forms
6. chrome-devtools_click({ uid: "..." }) - Submit
7. chrome-devtools_take_screenshot() - Capture result
8. chrome-devtools_list_console_messages() - Check for errors
```

### Performance debugging flow
```javascript
1. chrome-devtools_navigate_page({ type: "url", url: "..." })
2. chrome-devtools_performance_start_trace({ reload: true, autoStop: true })
3. chrome-devtools_list_network_requests() - Analyze resources
4. chrome-devtools_performance_analyze_insight({ ... }) - Review insights
```

### Network debugging flow
```javascript
1. chrome-devtools_list_network_requests({ resourceTypes: ["xhr", "fetch"] })
2. chrome-devtools_get_network_request({ reqid: 123 }) - Inspect payload
3. chrome-devtools_list_console_messages({ types: ["error"] }) - Check errors
```
