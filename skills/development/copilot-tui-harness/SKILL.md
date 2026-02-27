---
name: copilot-tui-harness
description: Expert in the Copilot SDK TUI Harness project. Use for development tasks including architecture, event system, plugins, OpenTUI components, and Copilot SDK integration. Triggers on TUI development, harness events, streaming UI, plugin system, event-driven architecture.
---

# Copilot SDK TUI Harness Development Skill

Expert skill for developing the Copilot SDK TUI Harness—a terminal user interface that wraps the GitHub Copilot SDK with an event-driven, plugin-based architecture.

## Project Identity

**Name**: Copilot SDK TUI Harness (copilot-anvil)  
**Stack**: TypeScript + OpenTUI/React + @github/copilot-sdk  
**Runtime**: Bun 1.0+  
**Architecture**: Event-driven with strict UI ↔ SDK decoupling

## Core Principles

### 1. Strict Layer Separation

```
UI Layer (OpenTUI/React)
    ↕ HarnessEvent / UIAction
Harness (Orchestrator)
    ↕ Internal callbacks
CopilotSessionAdapter
    ↕
@github/copilot-sdk
```

**CRITICAL**: The UI layer never imports `@github/copilot-sdk` directly. All SDK interaction flows through the adapter and harness layers.

### 2. Event-Driven Communication

- **Harness → UI**: `HarnessEvent` types (run.started, assistant.delta, log, etc.)
- **UI → Harness**: `UIAction` types (submit.prompt, cancel)
- No direct function calls between layers—everything flows through events

### 3. Plugin-Ready Architecture

The system is designed for extensibility via plugins that can register:
- Tools (callable by the assistant)
- Commands (user-invokable actions)
- Panes (UI components)
- State slices (additional state management)

## Architecture Deep Dive

### Directory Structure

```
src/
├── index.tsx                    # Entry point, bootstraps app
├── copilot/
│   └── CopilotSessionAdapter.ts # ONLY place that imports SDK
├── harness/
│   ├── Harness.ts              # Orchestrator, state manager
│   ├── events.ts               # Event type definitions
│   └── plugins.ts              # Plugin system interfaces
├── ui/
│   ├── App.tsx                 # Main OpenTUI app component
│   ├── theme.ts                # Theme configuration
│   ├── syntaxTheme.ts          # Syntax highlighting theme
│   └── panes/
│       ├── ChatPane.tsx        # Conversation transcript
│       ├── InputBar.tsx        # Prompt input with image support
│       ├── Sidebar.tsx         # Sidebar container
│       ├── TasksPane.tsx       # Task tracking display
│       ├── ContextPane.tsx     # Context information
│       ├── SubagentsPane.tsx   # Subagent monitoring
│       ├── FilesModifiedPane.tsx # Git modified files
│       ├── PlanPane.tsx        # Plan viewer
│       ├── LogsPane.tsx        # System logs
│       ├── StartScreen.tsx     # Welcome screen
│       ├── ModelSelector.tsx   # Model selection modal
│       ├── SessionSwitcher.tsx # Session management
│       ├── SkillsPane.tsx      # Skills browser
│       ├── QuestionModal.tsx   # Interactive questions
│       ├── ConfirmModal.tsx    # Confirmation dialogs
│       ├── CommandModal.tsx    # Ephemeral run display
│       └── DebugOverlay.tsx    # Debug overlay
├── commands/                   # Slash command system
│   └── CommandLoader.ts        # Command registry
└── utils/                      # Shared utilities
    ├── git.ts                  # Git operations
    ├── gitDiff.ts              # Diff utilities
    └── stderrCapture.ts        # Error capture
```

### Key Files and Responsibilities

#### `src/copilot/CopilotSessionAdapter.ts`

**Purpose**: Wraps Copilot SDK, translates SDK events to HarnessEvents

**Responsibilities**:
- Initialize CopilotClient
- Create/manage CopilotSession (with `streaming: true`)
- Subscribe to SDK events and emit corresponding HarnessEvents
- Handle prompt submission via session.sendAndWait()
- Must NOT contain UI code or import UI components

**Key Pattern**:
```typescript
// SDK event → Harness event translation
session.on('assistant.message_delta', (event) => {
  this.emit({
    type: 'assistant.delta',
    runId: this.currentRunId,
    text: event.content
  });
});
```

#### `src/harness/Harness.ts`

**Purpose**: Central orchestrator and state manager

**Responsibilities**:
- Maintain UI transcript (for display only—SDK has true memory)
- Own event bus and lifecycle state
- Handle UIAction dispatch from UI
- Support plugin registration and management
- Coordinate between adapter and UI

**Key Patterns**:
- State is immutable; updates create new state objects
- Events are emitted synchronously to all listeners
- Plugins hook into lifecycle via `onEvent()`

#### `src/harness/events.ts`

**Purpose**: Define all event and action types

**Core HarnessEvent Types**:

| Event | Payload | When Emitted |
|-------|---------|--------------|
| `run.started` | `{ runId, createdAt }` | Prompt submitted, SDK begins processing |
| `assistant.delta` | `{ runId, text }` | Streaming token received from SDK |
| `assistant.message` | `{ runId, message }` | Complete assistant response ready |
| `reasoning.delta` | `{ runId, reasoningId, text }` | Reasoning token received (o1 models) |
| `reasoning.message` | `{ runId, reasoningId, content }` | Complete reasoning content |
| `run.finished` | `{ runId, createdAt }` | Run completed successfully |
| `run.cancelled` | `{ runId, createdAt }` | User cancelled with Ctrl+C |
| `tool.started` | `{ runId, toolCallId, toolName, arguments }` | Tool execution begins |
| `tool.progress` | `{ runId, toolCallId, message }` | Tool progress update |
| `tool.completed` | `{ runId, toolCallId, success, output, error }` | Tool execution completes |
| `subagent.started` | `{ runId, toolCallId, agentName, ... }` | Subagent invoked |
| `subagent.completed` | `{ runId, toolCallId, agentName }` | Subagent finished |
| `subagent.failed` | `{ runId, toolCallId, agentName, error }` | Subagent failed |
| `skill.invoked` | `{ runId, name, path }` | Skill invoked |
| `intent.updated` | `{ runId, intent }` | Agent intent updated |
| `todo.updated` | `{ runId, todos }` | Task list updated |
| `plan.updated` | `{ content }` | Plan content updated |
| `turn.started` | `{ runId, turnId }` | New turn begins |
| `turn.ended` | `{ runId, turnId }` | Turn completes |
| `question.requested` | `{ requestId, question, choices, allowFreeform }` | Agent asks question |
| `question.answered` | `{ requestId, answer, wasFreeform }` | User answers question |
| `session.switched` | `{ sessionId, sessionName, transcript }` | Session changed |
| `session.created` | `{ sessionId, sessionName }` | New session created |
| `session.list.updated` | `{ sessions }` | Session list refreshed |
| `model.changed` | `{ model }` | AI model changed |
| `usage.info` | `{ tokenLimit, currentTokens, messagesLength }` | Token usage info |
| `quota.info` | `{ remainingPremiumRequests, consumedRequests }` | Quota info |
| `log` | `{ runId?, level, message, data? }` | System or plugin log message |

**UIAction Types**:

| Action | Payload | Effect |
|--------|---------|--------|
| `submit.prompt` | `{ text, images? }` | Send prompt to Copilot SDK (with optional images) |
| `cancel` | `{}` | Abort current run |
| `change.model` | `{ modelId }` | Switch AI model |
| `answer.question` | `{ requestId, answer, wasFreeform }` | Answer agent question |
| `session.new` | `{}` | Create new session |
| `session.switch` | `{ sessionId }` | Switch to session |
| `session.refresh` | `{}` | Refresh session list |
| `ephemeral.close` | `{}` | Close ephemeral run modal |

#### `src/harness/plugins.ts`

**Purpose**: Define plugin system interfaces

**Plugin Interface**:
```typescript
interface HarnessPlugin {
  name: string;
  register(ctx: PluginContext): void;
  onEvent?(event: HarnessEvent): void;
}

interface PluginContext {
  tools: ToolRegistry;
  commands: CommandRegistry;
  panes: PaneRegistry;
  state: StateRegistry;
  emit(event: HarnessEvent): void;
}
```

**Key Concept**: Plugins receive a context object during registration and can subscribe to all harness events.

#### `src/ui/App.tsx`

**Purpose**: Main OpenTUI application component

**Responsibilities**:
- Layout definition (chat + logs + input)
- Subscribe to harness events and update local state
- Dispatch UIActions in response to user input
- Render keybind hints and status bar

**Key Patterns**:
- Use React hooks (useState, useEffect) for local UI state
- Never directly manipulate harness state
- All harness communication via events/actions

#### `src/ui/panes/ChatPane.tsx`

**Purpose**: Display conversation transcript with streaming support

**Responsibilities**:
- Render user and assistant messages
- Show streaming draft during active run
- Display tool calls inline in transcript
- Handle scrolling and layout
- Support syntax highlighting for code blocks

**Key Pattern**: Separate committed messages from streaming buffer:
```typescript
{transcript.map(item => 
  item.kind === "message" ? <Message {...item} /> : <ToolCall {...item} />
)}
{streamingBuffer && <StreamingMessage text={streamingBuffer} />}
```

#### `src/ui/panes/InputBar.tsx`

**Purpose**: Handle prompt input with image attachment support

**Responsibilities**:
- Provide text input field
- Dispatch `submit.prompt` action on Enter
- Disable input during active run
- Handle image attachment (Ctrl+I)
- Show input hints

#### `src/ui/panes/Sidebar.tsx`

**Purpose**: Container for sidebar panels

**Responsibilities**:
- Layout sidebar panes vertically
- Manage space allocation between panes
- Show/hide panes based on content

#### `src/ui/panes/TasksPane.tsx`

**Purpose**: Display active tasks

**Responsibilities**:
- Show running, completed, and failed tasks
- Display task timing and status
- Update in real-time via events

#### `src/ui/panes/ContextPane.tsx`

**Purpose**: Display current context information

**Responsibilities**:
- Show git information (branch, repo, commit)
- Display active tools
- Show token usage and quota

#### `src/ui/panes/SubagentsPane.tsx`

**Purpose**: Monitor subagent execution

**Responsibilities**:
- Display active and completed subagents
- Show subagent status and timing
- Track subagent failures

#### `src/ui/panes/FilesModifiedPane.tsx`

**Purpose**: Show git modified files

**Responsibilities**:
- List unstaged and staged changes
- Display file change types (modified, added, deleted)
- Update periodically from git status

#### `src/ui/panes/PlanPane.tsx`

**Purpose**: Display execution plan

**Responsibilities**:
- Show current plan content
- Support markdown rendering
- Track plan updates

#### `src/ui/panes/ModelSelector.tsx`

**Purpose**: Model selection modal

**Responsibilities**:
- Display available models
- Allow model switching
- Show current model
- Handle keyboard navigation

#### `src/ui/panes/SessionSwitcher.tsx`

**Purpose**: Session management modal

**Responsibilities**:
- List all sessions
- Allow session switching
- Support creating new sessions
- Show session metadata

#### `src/ui/panes/SkillsPane.tsx`

**Purpose**: Skills browser

**Responsibilities**:
- List available skills
- Show skill descriptions
- Allow skill invocation
- Display skill metadata

#### `src/ui/panes/QuestionModal.tsx`

**Purpose**: Interactive user questions

**Responsibilities**:
- Display agent questions
- Show choices (if provided)
- Support freeform input
- Handle keyboard navigation

#### `src/ui/panes/CommandModal.tsx`

**Purpose**: Ephemeral run display

**Responsibilities**:
- Show ephemeral run output
- Display status and progress
- Allow closing modal

#### `src/ui/panes/StartScreen.tsx`

**Purpose**: Welcome screen

**Responsibilities**:
- Show before first interaction
- Display keybind hints
- Provide getting started info

## Event Flow Patterns

### Typical Prompt Submission Flow

```
1. User types in InputBar, presses Enter
2. InputBar dispatches UIAction({ type: "submit.prompt", text })
3. Harness.dispatch() receives action
4. Harness emits HarnessEvent({ type: "run.started", runId, createdAt })
5. Harness calls CopilotSessionAdapter.sendPrompt(text)
6. Adapter calls session.sendAndWait({ prompt: text })
7. SDK streams response tokens
8. For each token:
   a. Adapter receives SDK event
   b. Adapter emits HarnessEvent({ type: "assistant.delta", runId, text })
   c. Harness updates streaming buffer
   d. UI re-renders with updated buffer
9. SDK completes
10. Adapter emits HarnessEvent({ type: "assistant.message", runId, message })
11. Harness commits buffer to transcript
12. Harness emits HarnessEvent({ type: "run.finished", runId, createdAt })
13. UI shows completed message, re-enables input
```

### Cancellation Flow

```
1. User presses Ctrl+C during run
2. UI dispatches UIAction({ type: "cancel" })
3. Harness.dispatch() receives action
4. Harness tells adapter to cancel
5. Adapter stops processing deltas (best-effort)
6. Adapter emits HarnessEvent({ type: "run.cancelled", runId, createdAt })
7. Harness clears streaming buffer, resets to idle
8. UI re-enables input
```

## Development Guidelines

### When Adding a New Feature

**1. Define Events First**
- Add new event types to `src/harness/events.ts`
- Define clear contracts (what data, when emitted)

**2. Implement in Layers**
- If SDK-related: extend `CopilotSessionAdapter`
- If state/logic: extend `Harness`
- If UI: add/modify components in `src/ui/`

**3. Maintain Separation**
- Never import SDK in UI
- Keep adapter simple—just translation
- Put logic in harness, not adapter or UI

### When Adding a Plugin

**1. Define the plugin object**:
```typescript
const myPlugin: HarnessPlugin = {
  name: "my-feature",
  register(ctx) {
    // Register tools, commands, etc.
    ctx.tools.register("myTool", async (args) => {
      return { result: "..." };
    });
  },
  onEvent(event) {
    // React to lifecycle events
    if (event.type === "run.started") {
      console.log("Run started!");
    }
  }
};
```

**2. Register with harness**:
```typescript
harness.use(myPlugin);
```

**3. Test integration**:
- Verify tools are callable by assistant
- Ensure events are received
- Check for conflicts with existing plugins

### When Adding a New Pane

**1. Create component**: `src/ui/panes/NewPane.tsx`

**2. Design state needs**:
- What events does it subscribe to?
- What actions does it dispatch?
- What data does it display?

**3. Add to layout**: Update `src/ui/App.tsx`
- Import the pane
- Add to the appropriate section (main area, sidebar, or modal)
- Connect to harness state

**4. Handle keyboard shortcuts** (if needed):
- Add keybind in `App.tsx`
- Update state to show/hide pane

**5. (Future) Register via plugin**: Use `ctx.panes.register()`

**Existing panes for reference**:
- Main area: `ChatPane`, `InputBar`
- Sidebar: `TasksPane`, `ContextPane`, `SubagentsPane`, `FilesModifiedPane`, `PlanPane`
- Modals: `ModelSelector`, `SessionSwitcher`, `SkillsPane`, `QuestionModal`, `ConfirmModal`, `CommandModal`
- Overlays: `StartScreen`, `DebugOverlay`

### When Modifying SDK Integration

**ONLY edit `src/copilot/CopilotSessionAdapter.ts`**

Common tasks:
- Change session options (model, streaming, etc.)
- Add new SDK event subscriptions
- Adjust prompt formatting
- Handle new SDK capabilities

Never:
- Import SDK elsewhere
- Put business logic in adapter
- Directly manipulate UI from adapter

## Common Patterns

### Pattern: Buffered Streaming

```typescript
// In Harness
private streamingBuffer = "";

onDelta(event: AssistantDeltaEvent) {
  this.streamingBuffer += event.text;
  this.notifyUI();
}

onComplete(event: AssistantMessageEvent) {
  this.transcript.push({
    role: "assistant",
    content: this.streamingBuffer,
    createdAt: new Date()
  });
  this.streamingBuffer = "";
  this.notifyUI();
}
```

### Pattern: Idempotent Event Handling

```typescript
// In Plugin
onEvent(event: HarnessEvent) {
  if (event.type === "run.started") {
    // Always safe to call, even if called multiple times
    this.resetState();
  }
}
```

### Pattern: Async Tool Registration

```typescript
// In Plugin
register(ctx: PluginContext) {
  ctx.tools.register("fetchData", async (args) => {
    const data = await fetch(args.url);
    return await data.json();
  });
}
```

## Testing Strategies

### Manual Testing

```bash
bun run dev
```

**Test cases**:
1. Send a prompt, verify streaming display
2. Send multiple prompts, verify session continuity
3. Press Ctrl+C during streaming, verify cancellation
4. Trigger error (bad auth), verify error handling
5. Check logs pane shows lifecycle events

### Integration Testing

- Mock CopilotSessionAdapter
- Emit events manually
- Verify harness state updates
- Verify UI renders correctly

### Unit Testing

- Test event type definitions
- Test plugin registration logic
- Test utility functions

## Debugging

### Enable verbose logging

```typescript
// In Harness or Adapter
this.emit({
  type: "log",
  level: "info",
  message: "Debug: streaming delta",
  data: { text, runId }
});
```

### Check event flow

Add logging to:
1. `Harness.dispatch()` — see incoming actions
2. `Adapter` event handlers — see SDK events
3. UI `useEffect` hooks — see state updates

### Inspect state

Use OpenTUI devtools or add temporary UI to display:
- Current harness state
- Active run ID
- Streaming buffer content
- Plugin registry contents

## Performance Considerations

### Streaming Updates

- Debounce UI updates if delta rate is very high
- Use React memoization for message list
- Limit log pane to last N entries

### Event Listeners

- Remove listeners on component unmount
- Use weak references if holding onto event listeners long-term

### Plugin Load Time

- Keep `register()` fast—defer heavy init to first use
- Don't block on network calls during registration

## Common Pitfalls

### ❌ Importing SDK in UI

```typescript
// WRONG - never do this
import { CopilotClient } from "@github/copilot-sdk";
```

**Fix**: Use harness events and actions instead

### ❌ Putting Logic in Adapter

```typescript
// WRONG - adapter should not have business logic
adapter.sendPrompt = (text) => {
  if (text.includes("secret")) {
    // Don't do this here!
    throw new Error("Forbidden");
  }
  session.sendAndWait({ prompt: text });
}
```

**Fix**: Put validation in harness or plugin

### ❌ Mutating Harness State from UI

```typescript
// WRONG - never mutate harness state directly
harness.state.transcript.push(newMessage);
```

**Fix**: Dispatch an action, let harness update its own state

### ❌ Blocking Event Handlers

```typescript
// WRONG - don't block the event loop
onEvent(event) {
  if (event.type === "run.started") {
    // This blocks!
    const result = syncExpensiveOperation();
  }
}
```

**Fix**: Use async/await or defer work to next tick

## Scripts and Commands

| Command | Purpose |
|---------|---------|
| `bun run dev` | Start TUI in development mode |
| `bun run start` | Same as dev (alias) |
| `bun install` | Install dependencies |

## Dependencies

### Production

- `@github/copilot-sdk` — Core SDK for agent runtime
- `@opentui/core` — Terminal UI framework (core)
- `@opentui/react` — React reconciler for OpenTUI
- `react` — React library
- `diff` — Diff utility (future use)

### Development

- `typescript` — TypeScript compiler
- `@types/node` — Node.js type definitions
- `@types/react` — React type definitions
- `@types/diff` — Diff type definitions

## Configuration Files

- `package.json` — Project manifest, scripts, dependencies
- `tsconfig.json` — TypeScript compiler options
- `bun.lock` — Bun lockfile

## Resources

- Full requirements: `docs/REQUIREMENTS.md`
- Agent docs: `AGENTS.md`
- Main README: `README.md`

## Quick Reference: File → Responsibility

| File | What to Change |
|------|----------------|
| `CopilotSessionAdapter.ts` | SDK integration, event translation |
| `Harness.ts` | State management, orchestration, sessions |
| `events.ts` | Add new event/action types |
| `plugins.ts` | Extend plugin system |
| `App.tsx` | Layout, top-level UI structure |
| `ChatPane.tsx` | Conversation display with tool calls |
| `LogsPane.tsx` | Log display (optional, not in default layout) |
| `Sidebar.tsx` | Sidebar container and layout |
| `TasksPane.tsx` | Task tracking display |
| `ContextPane.tsx` | Context information display |
| `SubagentsPane.tsx` | Subagent monitoring |
| `FilesModifiedPane.tsx` | Git status display |
| `PlanPane.tsx` | Plan viewer |
| `ModelSelector.tsx` | Model selection modal |
| `SessionSwitcher.tsx` | Session management |
| `SkillsPane.tsx` | Skills browser |
| `QuestionModal.tsx` | Interactive questions |
| `theme.ts` | Theme configuration |
| `syntaxTheme.ts` | Syntax highlighting |
| `InputBar.tsx` | Prompt input with image support |

## Troubleshooting

### "Copilot not authenticated"

```bash
npm install -g @github/copilot-cli
copilot auth login
```

### Streaming not working

- Check `streaming: true` in session options
- Verify adapter subscribes to `assistant.message_delta`
- Check harness emits `assistant.delta` events

### Cancellation not working

- SDK may not support true abort
- Current approach: ignore subsequent deltas, reset state
- Session remains valid for next prompt

### Plugin not receiving events

- Verify plugin registered: `harness.use(plugin)`
- Check `onEvent` method exists and has correct signature
- Add logging to verify events are being emitted

### UI not updating

- Check component subscribes to harness events
- Verify state changes trigger re-render
- Use React DevTools to inspect component state

## Future Enhancements (Planned)

Architecture supports these additions:

1. **Tasks**: ✅ Implemented — Task tracking and status display
2. **Memory**: Memory provider interface, context injection
3. **Git & Diffs**: ✅ Partially implemented — Diff viewing, modified files; approval gating pending
4. **GitHub PR**: PR tools, PR resources, PR pane
5. **File Browser**: File selection pane, context injection
6. **Sessions**: ✅ Implemented — Multi-session support with persistence
7. **Skills**: ✅ Implemented — Skills browser and invocation
8. **Image Support**: ✅ Implemented — Image attachment for vision models
9. **Plans**: ✅ Implemented — Plan tracking and display
10. **Subagents**: ✅ Implemented — Subagent monitoring and status

## Summary

This skill provides comprehensive knowledge for working on the Copilot SDK TUI Harness project. Key takeaways:

1. **Strict separation**: UI never imports SDK
2. **Event-driven**: All communication via events/actions
3. **Plugin-ready**: Extensibility built into core
4. **Three layers**: UI → Harness → Adapter → SDK
5. **TypeScript + OpenTUI**: Strongly typed, terminal-first

When developing, always consider: Which layer does this belong in? What events does it produce/consume? How does it fit into the plugin system?
