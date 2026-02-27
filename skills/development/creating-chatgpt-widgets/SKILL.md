---
name: creating-chatgpt-widgets
description: Create production-grade widgets for ChatGPT Apps using the OpenAI Apps SDK. Use when users ask to build widgets, UI components, or visual interfaces for ChatGPT applications. Supports any widget type including progress trackers, quiz interfaces, content viewers, data cards, carousels, forms, charts, dashboards, maps, video players, or custom interactive elements. IMPORTANT - Always clarify requirements before building. Creates complete implementations following official OpenAI UX/UI guidelines with window.openai integration, theme support, and accessibility.
---

# Creating ChatGPT Widgets

Create production-ready widgets for ChatGPT Apps following official OpenAI guidelines.

## What This Skill Does

- Creates widgets for ChatGPT Apps with `window.openai` integration
- Supports all display modes (inline, fullscreen, pip)
- Implements theme support, accessibility, responsive design
- Follows official OpenAI UX/UI guidelines

## What This Skill Does NOT Do

- Create native mobile apps or browser extensions
- Build MCP server backend logic
- Handle user authentication (see `references/authentication.md` for patterns)
- Deploy widgets to production
- Create standalone web applications

## Dependencies

| Dependency | Version/Notes |
|------------|---------------|
| `window.openai` API | Required - injected by ChatGPT runtime |
| Modern browser | ES2020+, CSS Grid/Flexbox |
| CSP compliance | No inline scripts, approved CDNs only |

**No external dependencies required** - widgets run in ChatGPT's sandboxed iframe.

---

## 🛑 STOP: Clarify Before Building

**Never build a widget without understanding requirements.** Ask these questions:

### Required Clarifications

1. **Data Shape**: "What data will this widget display? Can you show the expected `toolOutput` structure?"
   ```
   Example: { items: [{ id: 1, title: "...", status: "..." }], total: 10 }
   ```

2. **Read vs Write**: "Is this display-only or does it allow editing/submission?"
   - Read only → Simple rendering, no callTool
   - Write/Submit → Need form handling, callTool integration, loading states

3. **Single vs Multi-turn**: "Does the task complete in one exchange or persist across interactions?"
   - Single turn → Stateless, all data in toolOutput
   - Multi-turn → Need widgetState persistence, sendFollowUpMessage

4. **Display Mode**: "Which display mode fits your use case?"
   - `inline` (default) - Simple displays, ≤2 actions, in conversation flow
   - `fullscreen` - Complex workflows, editing, maps, detailed views
   - `pip` - Persistent content like video, games, live sessions

5. **MCP Tool Name**: "What tool should be called for actions?" (if interactive)

### Optional Clarifications

6. **Design Preference**: Minimal (default), branded colors, specific theme?
7. **Component Type**: List, Map, Album, Carousel, Shop, or custom?

### Before Asking

Check existing context first:
- Review conversation for prior answers about data shape or requirements
- Infer widget type from user's domain (e.g., "course app" → progress tracker)
- Check if user provided sample data or mockups

### If User Skips Questions

- **Required questions**: Explain why needed, ask again simply
- **Optional questions**: Use sensible defaults (minimal design, custom type)
- **Ambiguous answers**: Confirm interpretation before building

**Proceed only after requirements are clear.**

---

## Official Documentation

Reference these for latest patterns and complex widgets:

| Resource | URL | Use For |
|----------|-----|---------|
| UX Principles | https://developers.openai.com/apps-sdk/concepts/ux-principles | Design decisions |
| UI Guidelines | https://developers.openai.com/apps-sdk/concepts/ui-guidelines | Visual design rules |
| **Component Types** | https://developers.openai.com/apps-sdk/plan/components | List, Map, Album, Carousel, Shop patterns |
| Build ChatGPT UI | https://developers.openai.com/apps-sdk/build/chatgpt-ui | Implementation, CSP, bundling |
| **State Management** | https://developers.openai.com/apps-sdk/build/state-management | Widget state, persistence patterns |
| Authentication | https://developers.openai.com/apps-sdk/build/auth | OAuth 2.1 for user-specific data *(advanced)* |
| UI Components | https://openai.github.io/apps-sdk-ui/ | Pre-built UI library |
| Example Apps | https://github.com/openai/openai-apps-sdk-examples | Reference implementations |

**For complex widgets** (maps, charts, 3D, video players) not covered below, fetch from these docs.

> **Version Note**: OpenAI Apps SDK is actively evolving. When building complex widgets, fetch latest docs to verify API signatures and CSP rules haven't changed.

---

## UX Principles Enforcement (Mandatory)

Before implementation, verify widget concept passes:

### Must Follow
- [ ] **Extract, Don't Port** - Atomic actions only, not full app ports
- [ ] **Inline by Default** - Use fullscreen only when justified
- [ ] **Max 2 Actions** - Limit buttons per inline card
- [ ] **No Widget Navigation** - Conversation handles routing
- [ ] **No Duplicated Functions** - Don't recreate chat input, settings

### Must Avoid
- ❌ Long-form content in widget (let ChatGPT explain)
- ❌ Complex multi-step workflows in single widget
- ❌ Deep navigation trees
- ❌ Nested scrolling
- ❌ Standalone dashboards

**If widget concept violates these, redesign before building.**

See `references/ux-principles.md` for complete guidelines.

---

## State Management

> Reference: https://developers.openai.com/apps-sdk/build/state-management

### Three State Categories

| State Type | Owner | Lifetime | Example |
|------------|-------|----------|---------|
| **Business Data** | MCP Server | Long-lived | Tasks, documents, user records |
| **UI State** | Widget | Message-scoped | Selections, expanded panels, sort order |
| **Cross-Session** | Backend Storage | Persistent | Saved filters, preferences |

### Data Flow Pattern

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User action in widget                                    │
│ 2. Widget calls server tool (callTool)                      │
│ 3. Server updates authoritative data                        │
│ 4. Server returns new snapshot (toolOutput)                 │
│ 5. Widget re-renders with snapshot + local UI state         │
└─────────────────────────────────────────────────────────────┘
```

**Server is authoritative** - Never let UI state diverge from server data.

### Critical Constraints

- ❌ **Avoid localStorage** - Widget state doesn't persist there reliably
- ❌ **No auto re-sync** - Widget must re-apply local state on new data
- ✅ **widgetState** - Only persists for that message's widget instance
- ✅ **<4KB tokens** - Keep widgetState small (shown to model)

---

## Quick Implementation Reference

### Data Access
```typescript
// Tool output (structuredContent from MCP)
const data = window.openai?.toolOutput;

// Private metadata (_meta, not sent to model)
const meta = window.openai?.toolResponseMetadata;

// Listen for updates
window.addEventListener('openai:set_globals', () => {
  const newData = window.openai?.toolOutput;
});
```

### Theme Support (Required)
```typescript
const theme = window.openai?.theme ?? 'light';
// Apply system fonts: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif
```

### Tool Invocation (When Interactive)
```typescript
const result = await window.openai?.callTool('tool_name', { param: value });
const responseData = result?.structuredContent;
```

### State Persistence (When Needed)
```typescript
// Save (shown to model, <4KB)
window.openai?.setWidgetState({ selectedId: 5 });

// Read
const saved = window.openai?.widgetState;
```

### Error Handling (Required)
```typescript
// Tool call with error handling
async function safeToolCall(toolName: string, params: object) {
  try {
    const result = await window.openai?.callTool(toolName, params);
    if (result?.isError) {
      showError(result.content?.[0]?.text ?? 'Operation failed');
      return null;
    }
    return result?.structuredContent;
  } catch (err) {
    showError('Connection error. Please try again.');
    return null;
  }
}

// Data validation
function validateToolOutput(data: unknown): data is ExpectedType {
  return data != null && typeof data === 'object' && 'requiredField' in data;
}
```

**Always handle**: Missing data, malformed responses, network failures, timeouts.

See `references/window-openai-api.md` for complete API reference.

---

## Component Types (Official)

> Reference: https://developers.openai.com/apps-sdk/plan/components

### List
- Dynamic collections with empty-state handling
- Best for: search results, task lists, notifications
- Supports filtering via conversation

### Map
- Geographic data with marker clustering
- Detail panels on selection
- Best for: location-based content, store finders
- Requires fullscreen for complex interaction

### Album
- Media grids with fullscreen transitions
- Best for: image galleries, portfolios
- Swipe navigation in fullscreen

### Carousel
- Featured content with swipe interactions
- Best for: product highlights, recommendations
- 3-8 items for scannability

### Shop
- Product browsing with checkout options
- Best for: e-commerce, booking flows
- Supports callTool for transactions

---

## Additional Patterns

### Progress Tracker
- Circular/linear progress visualization
- Module breakdown with percentages
- Expandable sections for details

### Quiz/Assessment
- Question with options display
- Selection state management
- Submit via `callTool`, results visualization

### Form/Input
- Input validation, loading states
- Clear error display
- Submit via `callTool`

For patterns not listed, fetch from official docs.

---

## Output Checklist

Every widget must include:

### Functional
- [ ] `window.openai` data access with null checks
- [ ] Event listener for `openai:set_globals`
- [ ] Loading state (before data arrives)
- [ ] Error state (when data.isError)
- [ ] Empty state (when no data)

### Visual
- [ ] Theme support (light/dark via `window.openai.theme`)
- [ ] System fonts (no custom fonts unless content area)
- [ ] WCAG AA contrast (4.5:1 text, 3:1 UI)
- [ ] Responsive layout (desktop + mobile breakpoints)
- [ ] Focus indicators for interactive elements
- [ ] Keyboard navigation support

### UX Compliance
- [ ] Follows "Extract, Don't Port"
- [ ] Inline mode unless justified
- [ ] ≤2 actions per card
- [ ] No hardcoded data
- [ ] Clear JSON payload structure

---

## Reference Files

Read based on need (progressive disclosure):

| File | When to Read |
|------|--------------|
| `references/window-openai-api.md` | API details, CSP configuration |
| `references/state-management.md` | Widget state, persistence patterns |
| `references/display-modes.md` | Choosing/implementing display modes |
| `references/ux-principles.md` | Validating design decisions |
| `references/react-hooks.md` | Building React widgets |
| `references/authentication.md` | OAuth 2.1 for user-specific data *(advanced)* |

## Asset Templates

Copy and customize:
- `assets/templates/vanilla-widget/` - HTML/CSS/JS, no build step
- `assets/templates/react-widget/` - TypeScript with hooks
