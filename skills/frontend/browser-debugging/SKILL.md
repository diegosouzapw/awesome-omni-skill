---
name: browser-debugging
description: "Use when debugging frontend issues in the browser. Covers DevTools usage, network debugging, performance profiling, and console patterns."
---

# Browser Debugging

## Overview

Browser debugging techniques using DevTools. Covers DOM inspection, network debugging, performance profiling, and console patterns for effective frontend debugging.

## When to Use

- Debugging UI rendering issues
- Investigating network requests
- Profiling performance
- Debugging JavaScript errors
- Inspecting application state

## Quick Reference

| Tab | Use For |
|-----|---------|
| **Elements** | DOM structure, CSS, layout |
| **Console** | Logs, errors, JS execution |
| **Network** | API calls, assets, timing |
| **Sources** | Breakpoints, code debugging |
| **Performance** | Render timing, bottlenecks |
| **Application** | Storage, cookies, SW |

---

## Console Patterns

### Enhanced Logging

```javascript
// Group related logs
console.group('User Authentication');
console.log('User:', user);
console.log('Token:', token);
console.groupEnd();

// Collapsed by default
console.groupCollapsed('API Response Details');
console.log(response);
console.groupEnd();

// Table for arrays/objects
console.table(users);

// Styled output
console.log(
  '%cWarning!%c Something unexpected',
  'color: orange; font-weight: bold; font-size: 14px;',
  'color: inherit;'
);

// Time operations
console.time('fetchUsers');
await fetchUsers();
console.timeEnd('fetchUsers'); // "fetchUsers: 234.56ms"

// Count calls
console.count('render'); // "render: 1", "render: 2", etc.
```

### Debugging Helpers

```javascript
// Copy to clipboard
copy(JSON.stringify(data, null, 2));

// Get selected element
$0 // Currently selected element in Elements tab
$1 // Previously selected

// Query selector shortcuts
$('selector')   // document.querySelector
$$('selector')  // document.querySelectorAll

// Monitor function calls
monitor(functionName);  // Logs when called
unmonitor(functionName);

// Debug function (break on call)
debug(functionName);
undebug(functionName);

// Monitor events on element
monitorEvents($0, 'click');
unmonitorEvents($0);
```

### Error Investigation

```javascript
// Stack trace
console.trace('How did we get here?');

// Assert (logs only if false)
console.assert(user.id, 'User ID is missing');

// Error with object
console.error('Failed to fetch:', { endpoint, status, error });
```

---

## Network Debugging

### Filter Requests

```
# In Network tab filter box
method:POST              # Only POST requests
status-code:400          # Only 400 errors
domain:api.example.com   # Only specific domain
larger-than:100k         # Large responses
-status-code:200         # Exclude 200s
```

### Analyze Requests

```
1. Click request in Network tab
2. Check tabs:
   - Headers: Request/response headers
   - Payload: Request body (POST data)
   - Preview: Formatted response
   - Response: Raw response
   - Timing: Breakdown of request phases
```

### Timing Breakdown

| Phase | Meaning |
|-------|---------|
| **Queueing** | Waiting for connection |
| **DNS Lookup** | Resolving domain |
| **Initial connection** | TCP handshake |
| **SSL** | TLS negotiation |
| **Request sent** | Uploading request |
| **Waiting (TTFB)** | Time to first byte |
| **Content Download** | Downloading response |

### Copy as cURL

```
1. Right-click request
2. Copy > Copy as cURL
3. Paste in terminal to reproduce
```

---

## DOM Debugging

### Elements Tab

```javascript
// Break on DOM changes
// Right-click element > Break on > subtree modifications

// Force element state
// Right-click element > Force state > :hover, :focus, etc.

// Edit as HTML
// Right-click element > Edit as HTML

// Scroll into view
// Right-click element > Scroll into view
```

### Layout Debugging

```css
/* Temporary debug outline */
* {
  outline: 1px solid red;
}

/* Or specific elements */
.container * {
  outline: 1px solid blue;
}
```

### Find Event Listeners

```
1. Select element in Elements tab
2. Look at Event Listeners panel (right sidebar)
3. Expand to see handlers and source location
4. Click link to jump to source
```

---

## JavaScript Debugging

### Breakpoints

```javascript
// In code
debugger; // Pauses execution here

// In DevTools Sources tab:
// - Line breakpoints: Click line number
// - Conditional: Right-click line > Add conditional
// - Logpoint: Right-click line > Add logpoint (logs without stopping)
```

### Conditional Breakpoints

```javascript
// Right-click line number > Add conditional breakpoint
// Only breaks when condition is true:

user.id === 'specific-user'
items.length > 10
error !== null
```

### Watch Expressions

```
1. Sources tab > Watch panel
2. Click + to add expression
3. Expression evaluated at each pause

Examples:
- user.profile.settings
- items.filter(x => x.active).length
- localStorage.getItem('token')
```

### Call Stack Navigation

```
When paused at breakpoint:
1. See Call Stack panel
2. Click frames to navigate up
3. See local variables for each frame
4. Identify where problem originated
```

---

## Performance Profiling

### Recording Performance

```
1. Performance tab > Record (or Cmd+Shift+E)
2. Interact with the page
3. Stop recording
4. Analyze timeline
```

### Key Metrics

| Metric | Target | Meaning |
|--------|--------|---------|
| **FCP** | < 1.8s | First Contentful Paint |
| **LCP** | < 2.5s | Largest Contentful Paint |
| **TBT** | < 200ms | Total Blocking Time |
| **CLS** | < 0.1 | Cumulative Layout Shift |

### Find Long Tasks

```
1. Record performance
2. Look for red bars in main thread
3. Long tasks (>50ms) block interactivity
4. Click task to see source
```

### Memory Profiling

```
1. Memory tab
2. Take heap snapshot
3. Perform action
4. Take another snapshot
5. Compare to find leaks
```

---

## React DevTools

### Component Inspection

```
1. Install React DevTools extension
2. Components tab shows component tree
3. Select component to see:
   - Props
   - State
   - Hooks
   - Source location
```

### Profiler

```
1. Profiler tab > Record
2. Interact with app
3. Stop recording
4. See render times per component
5. Identify unnecessary re-renders
```

### Highlight Updates

```
React DevTools > Settings > Highlight updates
Shows visual indicator when components re-render
```

---

## Common Debugging Scenarios

### "Why isn't my style applying?"

```
1. Select element in Elements tab
2. Check Styles panel (right side)
3. Look for:
   - Strikethrough = overridden
   - Gray = invalid/ignored
4. Check Computed tab for final values
5. Look for specificity conflicts
```

### "Why is this API call failing?"

```
1. Network tab > find request
2. Check status code
3. Check Response tab for error message
4. Check Headers for auth/content-type issues
5. Check Payload for request body
6. Copy as cURL to test outside browser
```

### "Why is my component re-rendering?"

```
1. React DevTools Profiler
2. Record during interaction
3. Check what triggered render:
   - Props changed?
   - State changed?
   - Parent rendered?
4. Add React.memo() or useMemo() if needed
```

### "Why is the page slow?"

```
1. Performance tab > Record
2. Look for:
   - Long tasks (red)
   - Layout thrashing
   - Expensive recalculations
3. Lighthouse tab for overall audit
4. Network tab for slow requests
```

---

## Useful Commands

```javascript
// Clear console
clear();

// Get element by selector
document.querySelector('#myId');
$$('.myClass'); // Shorthand in DevTools

// Inspect programmatically
inspect(element);

// Profile code block
profile('myProfile');
// ... code ...
profileEnd('myProfile');

// Get all event listeners
getEventListeners($0);
```

---

## Red Flags - STOP

**Never:**
- Leave debugger statements in production
- Ignore console errors
- Skip network tab for API issues
- Debug in production without source maps

**Always:**
- Check console first for errors
- Use network tab for API issues
- Use breakpoints over console.log for complex bugs
- Clear cache when testing (Cmd+Shift+R)

---

## Integration

**Related skills:** frontend-design, react-patterns
**Tools:** Chrome DevTools, React DevTools, Lighthouse
