---
name: brainstorm
description: "Explore requirements before implementation. Separates WHAT from HOW to ensure the right problem is being solved before any code is written."
---

# /brainstorm

Explore requirements before implementation. Separates WHAT from HOW to ensure the right problem is being solved before any code is written.

## Usage

```
/brainstorm <topic>
```

## Arguments

- `topic`: The feature, problem, or idea to explore (e.g., "user notifications", "improve search performance")

## Constraints

**STRICT RULES — these are non-negotiable:**
- NO code generation
- NO implementation details
- NO technology choices
- NO architecture decisions
- Focus ONLY on requirements: what the user needs, not how to build it

## Instructions

When this skill is invoked:

### Agent Behavior

**Autonomy:**
- Drive the conversation with probing questions
- Synthesize answers into a focused requirements document
- Hand off cleanly to `/feature` when ready

**Discipline:**
- Ask ONE question at a time (max 5 questions total)
- Keep questions specific and grounded
- Stop exploring when requirements are clear enough to act on
- Aim for 200-300 word output document (not a novel)

### Process

#### Phase 0: Assess Need

Before starting, quickly determine if brainstorming is needed:

**Skip brainstorming if:**
- User already has a detailed PRD
- Requirements are crystal clear from the request
- It's a straightforward CRUD feature with no ambiguity

**If skipping:** Say "Requirements are clear — ready for `/feature {topic}`" and stop.

#### Phase 1: Discovery (Max 5 Questions)

Ask probing questions to understand the real need. Pick from these angles based on what's unclear:

**Core Need:**
- What problem does this solve for the user?
- What happens today without this feature?
- Who uses this and in what context?

**Scope & Boundaries:**
- What's the simplest version that delivers value?
- What's explicitly out of scope?
- Are there existing features this overlaps with?

**User Experience:**
- What does the user see/do when this works correctly?
- What should happen when things go wrong?
- Are there accessibility or performance expectations?

**Edge Cases:**
- What are the boundary conditions?
- What happens with no data? Maximum data?
- Are there concurrent usage concerns?

**Success Criteria:**
- How will we know this feature is working correctly?
- What metrics matter?
- What would make this feature a failure?

**Rules:**
- Ask ONE question at a time
- Wait for the answer before asking the next
- Adapt follow-up questions based on answers
- Stop early if requirements become clear (don't force all 5)

#### Phase 2: Synthesize Requirements

After gathering answers, produce a focused requirements document:

```markdown
## Requirements: {Topic}

### Must Have
- {Core requirement 1}
- {Core requirement 2}
- {Core requirement 3}

### Should Have
- {Important but not blocking}
- {Nice-to-have that adds clear value}

### Out of Scope
- {Explicitly excluded}
- {Future consideration, not now}

### Success Criteria
- {Measurable criterion 1}
- {Measurable criterion 2}

### Open Questions
- {Anything still unresolved}
```

**Guidelines:**
- 200-300 words maximum
- Use MoSCoW categories (Must/Should/Out of Scope)
- Be specific — "fast" is not a requirement, "responds in < 200ms" is
- Include success criteria that can be tested

#### Phase 3: Hand Off

Present the requirements document and offer next steps:

```
Requirements ready for: {topic}

Next steps:
1. `/feature {topic}` — Start full implementation lifecycle
2. `/feature {topic} --prd-only` — Create detailed PRD first
3. Refine — Ask me to adjust any requirements above
```

## Example

```
$ /brainstorm user notifications

Let me explore what you need before we start building.

What problem does this solve? Are users currently missing important
events, or is this about engagement/retention?

> Users miss important status changes on their orders

Got it — order status notifications. How should users receive these?
(e.g., in-app only, email, push, SMS?)

> In-app and email for now, push notifications later

What order status changes matter? All of them, or specific transitions?

> Shipped, delivered, and delayed — those are the ones customers ask about

---

## Requirements: User Notifications

### Must Have
- Notify users when order status changes to: shipped, delivered, delayed
- In-app notification center (unread count, mark as read)
- Email notifications for each status change
- User can view notification history

### Should Have
- User preferences to disable specific notification types
- Email templates with order details and tracking link

### Out of Scope
- Push notifications (future phase)
- SMS notifications
- Real-time WebSocket delivery (polling is acceptable for v1)

### Success Criteria
- Users receive in-app notification within 30 seconds of status change
- Email delivered within 5 minutes of status change
- Notification center loads in < 500ms

### Open Questions
- Should delayed notifications include estimated new delivery date?

---

Next steps:
1. `/feature user-notifications` — Start implementation
2. `/feature user-notifications --prd-only` — Create detailed PRD first
3. Refine — Adjust requirements above
```
