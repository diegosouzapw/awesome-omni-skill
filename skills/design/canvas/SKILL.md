---
name: canvas
description: Spawn interactive terminal TUI components (calendars, documents, flight bookings) with real-time IPC communication. Display rich content and collect user selections in tmux split panes.
license: MIT
compatibility: Requires tmux and Bun runtime
metadata:
  author: David Siegel
  version: "0.1.0"
allowed-tools: bash bun tmux
---

# Canvas TUI Toolkit

Interactive terminal displays (TUIs) that spawn in tmux split panes and communicate via IPC. Use for calendars, documents, flight bookings, and other rich interactive scenarios.

## When to Use

- Displaying calendars and picking meeting times
- Showing markdown documents with text selection
- Comparing flights and selecting seats
- Any scenario requiring visual display + user interaction
- When you need to show rich content without blocking the conversation

## Quick Start

```bash
cd ${SKILL_DIR}

# Display calendar in current terminal
bun run src/cli.ts show calendar

# Spawn interactive meeting picker in tmux split
bun run src/cli.ts spawn calendar --scenario meeting-picker --config '{
  "calendars": [
    {"name": "Alice", "color": "blue", "events": [...]},
    {"name": "Bob", "color": "green", "events": [...]}
  ]
}'
```

## Available Canvas Types

| Canvas | Purpose | Scenarios |
|--------|---------|-----------|
| `calendar` | Display calendars, pick meeting times | `display`, `meeting-picker` |
| `document` | View/edit markdown documents | `display`, `edit`, `email-preview` |
| `flight` | Flight comparison and seat selection | `booking` |

## Spawning Canvases

**Always use `spawn` for interactive scenarios** - opens canvas in tmux split pane.

```bash
bun run src/cli.ts spawn [kind] --scenario [name] --config '[json]'
```

**Parameters:**
- `kind`: Canvas type (calendar, document, flight)
- `--scenario`: Interaction mode
- `--config`: JSON configuration for the canvas
- `--id`: Optional canvas instance ID for IPC

## Calendar Canvas

### Display Scenario
View-only calendar display.

```bash
bun run src/cli.ts show calendar --config '{
  "title": "My Week",
  "events": [
    {
      "id": "1",
      "title": "Meeting",
      "startTime": "2026-01-07T09:00:00",
      "endTime": "2026-01-07T10:00:00"
    }
  ]
}'
```

### Meeting Picker Scenario
Interactive time slot selection across multiple calendars.

```bash
bun run src/cli.ts spawn calendar --scenario meeting-picker --config '{
  "calendars": [
    {
      "name": "Alice",
      "color": "blue",
      "events": [
        {"id": "1", "title": "Standup", "startTime": "2026-01-07T09:00:00", "endTime": "2026-01-07T09:30:00"}
      ]
    }
  ],
  "slotGranularity": 30,
  "minDuration": 30,
  "maxDuration": 120
}'
```

**Controls:**
- Mouse click: Select a free time slot
- `←/→`: Navigate weeks
- `t`: Jump to today
- `q` or `Esc`: Cancel

**Returns:**
```json
{
  "startTime": "2026-01-07T14:00:00",
  "endTime": "2026-01-07T15:00:00",
  "duration": 60
}
```

## Document Canvas

### Display Scenario
Read-only markdown document.

```bash
# Spawn in new tmux pane (recommended)
bun run src/cli.ts spawn document --config '{
  "content": "# Hello World\n\nThis is **markdown**.",
  "title": "My Document"
}'

# Or show inline in current terminal
bun run src/cli.ts show document --config '{
  "content": "# Hello World\n\nThis is **markdown**.",
  "title": "My Document"
}'
```

### Edit Scenario
Interactive document with text selection and diff highlighting.

```bash
bun run src/cli.ts spawn document --scenario edit --config '{
  "content": "# Blog Post\n\nSelect some text here.",
  "title": "Edit Mode",
  "diffs": [
    {"startOffset": 20, "endOffset": 30, "type": "add"}
  ]
}'
```

**Controls:**
- Mouse click and drag: Select text
- `↑/↓`: Navigate document
- `q`: Close

**Returns:**
```json
{
  "selectedText": "some text",
  "startOffset": 12,
  "endOffset": 21,
  "startLine": 3,
  "endLine": 3
}
```

## Flight Canvas

### Booking Scenario
Flight comparison and seat selection with cyberpunk theme.

```bash
bun run src/cli.ts spawn flight --scenario booking --config '{
  "flights": [
    {
      "id": "ua123",
      "airline": "United Airlines",
      "flightNumber": "UA 123",
      "origin": {
        "code": "SFO",
        "name": "San Francisco International",
        "city": "San Francisco",
        "timezone": "PST"
      },
      "destination": {
        "code": "DEN",
        "name": "Denver International",
        "city": "Denver",
        "timezone": "MST"
      },
      "departureTime": "2026-01-08T12:55:00-08:00",
      "arrivalTime": "2026-01-08T16:37:00-07:00",
      "duration": 162,
      "price": 34500,
      "currency": "USD",
      "cabinClass": "economy",
      "aircraft": "Boeing 737-800",
      "stops": 0,
      "seatmap": {
        "rows": 30,
        "seatsPerRow": ["A", "B", "C", "D", "E", "F"],
        "aisleAfter": ["C"],
        "unavailable": ["1A", "1B", "1C"],
        "premium": ["2A", "2B"],
        "occupied": ["3A", "4B"]
      }
    }
  ]
}'
```

**Controls:**
- `↑/↓`: Navigate between flights
- `Tab`: Switch to seatmap
- `←/→/↑/↓` (in seatmap): Move cursor
- `Space`: Select seat
- `Enter`: Confirm
- `q`: Cancel

**Returns:**
```json
{
  "selectedFlight": { ... },
  "selectedSeat": "12A"
}
```

## IPC Communication

Canvases communicate via Unix domain sockets.

**Canvas → Controller:**
```typescript
{ type: "ready", scenario }        // Canvas ready
{ type: "selected", data }         // User made selection
{ type: "cancelled", reason? }     // User cancelled
{ type: "error", message }         // Error occurred
```

**Controller → Canvas:**
```typescript
{ type: "update", config }  // Update canvas config
{ type: "close" }           // Close canvas
{ type: "ping" }            // Health check
```

## Programmatic API

```typescript
import { pickMeetingTime, editDocument, bookFlight } from "${SKILL_DIR}/src/api";

// Meeting picker
const meeting = await pickMeetingTime({
  calendars: [...],
  slotGranularity: 30,
});

// Document editor
const doc = await editDocument({
  content: "# My Document",
  title: "Edit Mode",
});

// Flight booking
const flight = await bookFlight({
  flights: [...]
});
```

## Requirements

- **tmux**: Canvas spawning requires active tmux session
- **Bun**: Runtime for executing canvas commands
- **Terminal with mouse support**: For interactive scenarios

## File Structure

```
canvas/
├── SKILL.md           # This file
├── README.md          # Additional documentation
├── package.json       # Dependencies
├── run-canvas.sh      # Wrapper script
├── src/
│   ├── cli.ts         # CLI entry point
│   ├── canvases/      # React/Ink canvas components
│   ├── scenarios/     # Scenario definitions
│   ├── ipc/           # IPC server/client
│   └── api/           # High-level API
└── scripts/           # Helper scripts
```

## Tips

- Always check for tmux session before spawning: `tmux list-sessions`
- Use `show` for quick displays, `spawn` for interactive scenarios
- Canvas IDs are optional but useful for managing multiple canvases
- IPC sockets are created in `/tmp/canvas-*.sock`
- Canvases auto-cleanup on exit or error

## Examples

### Example 1: Pick Meeting Time
```bash
# User: "Find a time for Alice and Bob to meet tomorrow"
# You: Spawn meeting picker

bun run src/cli.ts spawn calendar --scenario meeting-picker --config '{
  "calendars": [
    {"name": "Alice", "color": "blue", "events": [...]},
    {"name": "Bob", "color": "green", "events": [...]}
  ],
  "slotGranularity": 30
}'

# Wait for selection, then confirm with user
```

### Example 2: Review Document
```bash
# User: "Show me the email draft"
# You: Display email in document canvas

bun run src/cli.ts spawn document --config '{
  "content": "Dear Team,\n\nPlease review...",
  "title": "Email Draft"
}'
```

### Example 3: Book Flight
```bash
# User: "Compare these flights and pick a seat"
# You: Spawn flight booking canvas

bun run src/cli.ts spawn flight --scenario booking --config '{
  "flights": [...]
}'

# Wait for user to select flight + seat
```
