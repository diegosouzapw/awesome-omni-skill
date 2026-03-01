---
name: jit-gui-expert
description: Expert reference for Just-in-Time GUI, generative UI, and ephemeral interfaces for AI-native products. Use when designing UI for voice-first assistants, implementing generative UI, working with A2UI/AG-UI/MCP-UI protocols, or building ephemeral/contextual interfaces.
---

# Just-in-Time GUI Expert

Use this skill when designing interfaces for AI-native products, voice-first assistants, or any system where UI should be generated dynamically rather than pre-designed statically.

---

## 1. What Is Just-in-Time GUI?

**Definition**: A UX paradigm where interfaces are generated dynamically, on-demand, and ephemerally — existing only as long as needed. AI/LLMs assemble UI in real-time based on user context and intent, rather than using pre-designed static screens.

**Coined by**: Andrew Sims (Signal Path) — "emerging UX patterns for AI-native products that trade static screens and flows for UI assembled on the fly."

**Core philosophy**: The interface is **invisible by default** and surfaces only when context demands it — then disappears when its purpose is served.

### Traditional GUI vs JIT GUI

| Aspect | Traditional GUI | Just-in-Time GUI |
|--------|----------------|-------------------|
| Design | Static wireframes designed upfront | Dynamic, assembled per-interaction |
| Persistence | Persists across sessions | Ephemeral — exists for immediate context only |
| Adaptation | User navigates fixed menus | Interface adapts to user intent |
| Component selection | Designer pre-selects | AI agent dynamically decides |
| Content | Hardcoded text/layouts | LLM generates content AND layout |
| Time to value | Multiple clicks through menus | Direct, contextual interface appears immediately |

---

## 2. Core Principles

1. **Context-Dependency** — UI depends on accurate context (what user is doing, history, priorities)
2. **Ambiguity & Adaptation** — Design for uncertainty, not fixed flows
3. **Character-Based Design** — AI agents as "characters with personalities and logic," not just backends
4. **Outcome-Oriented** — Define user goals/constraints for AI to operate within (not discrete design elements)
5. **Stateless Paradigm** — All interface state derives from external context (user intent, app state, environment)
6. **Calm Technology** — Interface moves between periphery and center of attention naturally (Weiser & Brown, 1995)
7. **Zero Inventory** — No unused features sitting in the codebase; generate what's needed, discard the rest

---

## 3. Related Paradigms

### Ambient Computing / Zero UI
- Eliminates traditional screens/buttons; relies on voice, gesture, predictive automation
- Technology complements the human environment seamlessly, not the other way around

### Calm Technology (Weiser & Brown, 1995)
- Technology should serve, not demand attention
- Interfaces fade to background while remaining available
- Move between periphery and center of attention naturally

### Ephemeral Interfaces
- Generated on-the-fly, exist temporarily, auto-dismiss when purpose is served
- Voice interfaces are naturally ephemeral (audio is fleeting)
- "Ephemeral apps": AI field-codes a tiny temporary tool for the task, then it disappears

### AX (Agentic Experience) — Replacing UX
- Jakob Nielsen 2026: Agentic design becomes default expectation
- Interface disappears when not needed; reappears contextually
- AI acts as silent co-designer adjusting based on user behavior
- Success metric: "tasks completed autonomously" not "tokens generated"

---

## 4. Three Implementation Patterns

### Pattern 1: Static Generative UI (High Control, Low Freedom)
```
Pre-built component library → Agent selects which + passes data → Frontend renders
```
- Safest (security), most consistent
- Agent only controls **when/which**, not **how**
- Best for: production approval flows, dashboards, forms with fixed layouts

### Pattern 2: Declarative Generative UI (Shared Control) — RECOMMENDED
```
Agent returns structured spec (JSON/JSONL) → Frontend interprets → Renders consistently
```
- Pre-approved components only — security by design
- Cross-platform: SwiftUI, Flutter, React from same spec
- LLM-friendly: flat, incrementally streamable
- Best for: most production use cases
- Protocols: A2UI, Open-JSON-UI

### Pattern 3: Open-Ended Generative UI (Low Control, High Freedom)
```
Agent returns complete HTML/iframe → Frontend acts as container
```
- Maximum flexibility, highest risk (XSS, performance, inconsistency)
- Best for: experimental features, sandboxed tools only

### Data Flow (All Patterns)
```
User Input → LLM → JSON/Spec Generation → Frontend Rendering → User Interaction
                     ↓
                  Tool Calls → Data Context
```

---

## 5. 2026 Protocol Stack

| Protocol | Layer | Purpose | Origin |
|----------|-------|---------|--------|
| **A2A** | Agent ↔ Agent | Agent coordination | Industry standard |
| **MCP** | Agent ↔ Tools | Backend actions, resources | Anthropic |
| **AG-UI** | Agent ↔ Frontend | Real-time bidirectional communication | CopilotKit/community |
| **A2UI** | UI Payload | Declarative UI spec (JSONL) | Google |
| **Open-JSON-UI** | UI Payload | JSON-based UI spec | Community/OpenAI |
| **MCP-UI** | UI Payload | HTML content via MCP tools | Anthropic extension |

**Full stack**: A2A → MCP (tools) → AG-UI (runtime) → A2UI/Open-JSON-UI (payload)

### A2UI (Google's Standard) — Deep Dive

**Format**: JSON Lines (JSONL) — each line is a complete message.

**Why A2UI wins for voice-first**:
- Security-first: declarative data, NOT executable code
- Framework-agnostic: SwiftUI, Flutter, React, Angular from same spec
- LLM-friendly: flat list of components with ID references = easy for LLMs to generate incrementally
- Streaming: each JSONL line can be processed as it arrives

**Adopted by**: Google, LangChain, AWS, Microsoft, Mastra, PydanticAI

**Example**:
```jsonl
{"component": "card", "id": "c1", "title": "Meeting in 15 min"}
{"component": "text", "id": "t1", "content": "Team standup with 4 attendees"}
{"component": "button", "id": "b1", "label": "Join", "onclick": "action:join_meeting"}
{"component": "button", "id": "b2", "label": "Snooze 5m", "onclick": "action:snooze"}
```

**Client implements**:
1. Maintains a "catalog" of pre-approved components (Card, Button, TextField, etc.)
2. Agent can only request rendering of trusted, predefined components
3. Client maps abstract descriptions to native widgets (SwiftUI views, React components)
4. Unknown components are safely ignored

---

## 6. Voice-First JIT GUI Patterns

### Architecture for Voice + Ephemeral UI
```
User speaks → Whisper transcribes → Manager LLM reasons
                                         ↓
                              ┌─────────────────────────┐
                              │ LLM decides output mode: │
                              ├─────────────────────────┤
                              │ Voice only → TTS         │
                              │ Voice + Card → TTS + UI  │
                              │ Approval → Button card   │
                              │ Data → Visual widget     │
                              └─────────────────────────┘
                                         ↓
                              SSE → Client renders ephemerally
                              Auto-dismiss after timeout/action
```

### Notification-as-Interface
Instead of opening windows/apps, use cards as the primary visual surface:

| Context | UI Generated |
|---------|-------------|
| Calendar alert | Compact card with snooze/dismiss/join buttons |
| Email summary | Expandable card with sender, subject, actions |
| Approval request | Two-button card (approve/defer) |
| Task list | Checklist widget with checkboxes |
| Calendar query | Compact timeline widget (not text) |

### Ephemeral Lifecycle
1. **Appear** on SSE event from server
2. **Live** for duration of interaction
3. **Auto-dismiss** after: task completion | user swipe | timeout (30s) | voice command ("dismiss")
4. **Never persist** — no saved state, no history

### Multimodal Seamlessness
- **Voice** triggers request (fast, natural)
- **Visual** confirms action (reassurance, data display)
- **Click/tap** for fine control (precision, selection)
- No modality treated as separate — best one naturally emerges per context

---

## 7. Key Design Rules

| DO | DON'T |
|----|-------|
| Voice-first, UI-second | Build elaborate pre-designed screens |
| Declarative specs (A2UI JSONL) | Generate raw HTML from LLM |
| Pre-approved component catalog | Allow arbitrary component creation |
| Auto-dismiss ephemeral UI | Keep UI elements persistent |
| LLM decides what UI to show | Hardcode UI selection logic |
| Start invisible, surface contextually | Show UI by default |
| Single-task focused cards | Multi-purpose complex layouts |
| Stream components as they arrive | Wait for full response before rendering |

### The Calm Technology Test
Before adding any visual UI, ask:
1. Can this be voice-only? If yes, don't add UI.
2. Does the user need to SEE something? (data, options, confirmation)
3. Will this demand attention or complement it?
4. Does it auto-dismiss, or does user need to manage it?

---

## 8. Thought Leaders (Quick Reference)

- **Andrew Sims** (Signal Path) — Coined "Just-in-Time Interfaces"; AI agents as characters, not services
- **Linus Lee** (Thrive Capital) — Moving from "language space" to "meaning space"; pinch-to-summarize, dial-for-formality
- **Amelia Wattenberger** — Chatbots lack flow state & sensory richness; JIT GUI needs multimodal feedback
- **Jakob Nielsen** — 2026: agentic design default; interface disappears when unneeded, reappears contextually
- **Yaniv Leviathan** (Google) — Proved LLMs generate high-quality UIs; released PAGEN dataset

---

## 9. For Extended Reference

See `reference.md` in this skill directory for:
- Full frameworks & libraries table (Vercel AI SDK, CopilotKit, assistant-ui, etc.)
- Academic papers with citations (5 key papers)
- All source URLs organized by category
- Real-world implementation details (Dynamic Island, Google GenUI, MCP Apps)
- Code examples (A2UI JSONL, Vercel streamUI, JSON Schema forms)
