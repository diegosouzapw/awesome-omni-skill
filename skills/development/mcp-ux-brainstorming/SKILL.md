---
name: MCP UX Brainstorming
description: This skill should be used when the user asks to "brainstorm app ideas", "design a ChatGPT app", "what kind of app should I build", "MCP app concept", "ideate widget UX", "plan conversational experience", "design for ChatGPT", or needs help generating and evaluating user experience concepts for OpenAI Apps SDK MCP applications.
version: 0.1.0
---

# MCP UX Brainstorming for OpenAI Apps SDK

## Overview

MCP-native design requires a fundamentally different mindset than traditional app design. This skill helps brainstorm, evaluate, and refine UX concepts specifically optimized for ChatGPT's conversational environment.

## The MCP-Native Mindset

### Extract, Don't Port

The most common mistake is trying to recreate an existing app inside ChatGPT. Instead:

| Traditional App Thinking | MCP-Native Thinking |
|--------------------------|---------------------|
| "Let's build our dashboard in ChatGPT" | "What single insight from our dashboard would help users most?" |
| "Users need all these settings" | "What's the one thing users actually want to do?" |
| "We need multi-step forms" | "Can the model collect this through conversation?" |
| "Show all the data" | "What data helps the user take the next action?" |

### The Conversational Advantage

MCP apps excel when they leverage what chat does uniquely well:

1. **Natural language input** - Users describe intent, not navigate menus
2. **Context awareness** - The model knows what came before
3. **Multi-turn guidance** - Complex workflows through conversation
4. **Composability** - Your app works with other apps and ChatGPT features

## Brainstorming Framework

### Step 1: Identify the Core Value

Ask these questions about your product/service:

```
1. What's the ONE thing users want to accomplish?
2. What would they naturally ask ChatGPT to do?
3. What data/action do you have that ChatGPT doesn't?
4. What's tedious in your current UX that conversation simplifies?
```

**Good Core Values:**
- "Book a restaurant" (clear action, natural to ask)
- "Find the best flight" (search + decision)
- "Check my order status" (data lookup)
- "Generate a report" (automation)

**Weak Core Values:**
- "Browse our catalog" (no clear intent)
- "Manage settings" (not conversational)
- "View analytics dashboard" (too broad)

### Step 2: Map the Conversational Flow

Design three entry points:

**1. Open-ended prompt**
```
User: "Help me plan dinner for tonight"
Model: Uses your restaurant app to suggest options
Widget: Shows 3-5 restaurant cards with key info
```

**2. Direct command**
```
User: "Book a table at Chez Pierre for 7pm"
Model: Calls booking tool directly
Widget: Confirmation card with booking details
```

**3. First-run discovery**
```
User: Has your app connected
Model: "I can help you find and book restaurants. What are you in the mood for?"
```

### Step 3: Design Tool Boundaries

Define what the model handles vs. what needs UI:

| Model Handles (No Widget) | Widget Handles (Visual UI) |
|---------------------------|----------------------------|
| Understanding intent | Displaying multiple options |
| Gathering missing info | Showing images/maps |
| Making recommendations | Confirming destructive actions |
| Explaining results | Complex data visualization |
| Follow-up suggestions | Interactive selection |

### Step 4: Evaluate Against Principles

Score your concept (1-5) on each:

| Principle | Question | Score |
|-----------|----------|-------|
| **Conversational Value** | Does it leverage natural language or context? | |
| **Unique Data** | Does it provide something ChatGPT can't? | |
| **Atomic Actions** | Can actions complete in 1-2 tool calls? | |
| **UI Necessity** | Does the widget add value beyond text? | |
| **Task Completion** | Can users finish the task in chat? | |

**Threshold:** Aim for 20+ total. Below 15 suggests reconsidering the concept.

## Ideation Techniques

### The "Ask ChatGPT" Test

Imagine users already have ChatGPT. What would they naturally ask that your service could answer?

```
"What's my account balance?" → Banking app
"When does my flight leave?" → Travel app
"What should I cook with chicken?" → Recipe app
"Is this product any good?" → Review app
"Schedule a meeting with Sarah" → Calendar app
```

### The Workflow Decomposition

Take a complex workflow and extract atomic actions:

**Traditional: E-commerce checkout**
```
Browse → Add to cart → Enter address → Select shipping → Payment → Confirm
```

**MCP-native extraction:**
- "Reorder my usual" (1 tool call)
- "Track my package" (1 tool call)
- "Find deals on headphones" (1 tool call + carousel)

### The Widget Audit

For each proposed widget, ask:

1. **Could the model say this instead?** If yes, skip the widget.
2. **Is this for the user or for "show"?** Skip decorative widgets.
3. **Does it help the NEXT action?** Include actionable widgets.
4. **Is it glanceable?** Widgets should communicate in 2-3 seconds.

## Concept Patterns That Work

### Pattern: Quick Lookup + Action

```
User: "What's my balance?"
Model: Calls balance tool
Widget: Card showing balance + quick action buttons
Follow-up: "Would you like to transfer funds?"
```

**Why it works:** Answers question immediately, offers next step.

### Pattern: Search + Select

```
User: "Find me a hotel in Paris under $200"
Model: Calls search tool with filters
Widget: Carousel of 5 hotel cards
User: Clicks one or says "Tell me more about the second one"
```

**Why it works:** Visual comparison is faster than text lists.

### Pattern: Generate + Review

```
User: "Write a marketing email for our sale"
Model: Generates draft
Widget: Rich text preview with edit button
User: "Make it shorter" or clicks edit
```

**Why it works:** Visual preview, conversational refinement.

### Pattern: Status + Options

```
User: "Where's my order?"
Model: Calls tracking tool
Widget: Timeline visualization
Follow-up: "It's delayed. Want me to contact support?"
```

**Why it works:** Visual status, proactive help.

## Anti-Patterns to Avoid

### Anti-Pattern: The Portal

❌ Building a mini-app with navigation, tabs, settings
✅ Single-purpose widgets for specific moments

### Anti-Pattern: The Form Dump

❌ Complex multi-field forms in widgets
✅ Model gathers info through conversation, widget confirms

### Anti-Pattern: The Dashboard

❌ Charts, metrics, and data grids
✅ Single insight cards with the "so what?" answer

### Anti-Pattern: The Upsell

❌ Ads, premium prompts, marketing content
✅ Valuable actions that naturally lead to engagement

## Brainstorming Session Template

Use this structure for ideation sessions:

```
## App Concept: [Name]

### Core Value
What single thing does this enable?

### Natural Prompts
What would users say to trigger this?
- "..."
- "..."
- "..."

### Tools Needed
| Tool | Input | Output | Widget? |
|------|-------|--------|---------|
| | | | |

### Conversational Flow
1. User says: ...
2. Model does: ...
3. Widget shows: ...
4. Follow-up: ...

### Evaluation Scores
- Conversational Value: /5
- Unique Data: /5
- Atomic Actions: /5
- UI Necessity: /5
- Task Completion: /5
- **Total: /25**

### Risks/Questions
- ...
```

## Additional Resources

### Reference Files

For detailed patterns and examples:
- **`references/concept-evaluation.md`** - Detailed scoring rubric
- **`references/successful-apps.md`** - Analysis of successful ChatGPT apps

### Example Files

Working concept examples in `examples/`:
- **`examples/brainstorm-restaurant.md`** - Restaurant booking concept
- **`examples/brainstorm-ecommerce.md`** - E-commerce concept

### Official Documentation

- UX Principles: https://developers.openai.com/apps-sdk/concepts/ux-principles/
- UI Guidelines: https://developers.openai.com/apps-sdk/concepts/ui-guidelines/
