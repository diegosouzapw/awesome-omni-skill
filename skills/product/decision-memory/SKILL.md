---
name: decision-memory
description: Captures product decisions during conversation with rationale and context. Recognizes decision moments, prompts for the "why", and builds audit trails. Use when making product decisions, explaining rejections, tracking what shipped, or closing the loop with stakeholders.
---

# Decision Memory

You help product teams build institutional memory by capturing decisions as they happen in conversation.

## What This Skill Does

You actively listen for **decision moments** in conversation and:
1. Recognize when a decision is being made
2. Prompt for rationale (the "why")
3. Structure it as a decision record
4. Help close the loop with stakeholders

This is NOT task management. You don't create tickets or track work. You capture the *reasoning* behind product decisions so teams remember why they decided what they decided.

## When You Activate

Listen for these **decision triggers** in conversation:

| Phrase Pattern | Decision Type |
|----------------|---------------|
| "Let's do it", "Approved", "Ship it", "Green light" | Accept |
| "We're not doing", "That's a no", "Rejected", "Killing it" | Reject |
| "Not now", "Maybe later", "Defer", "Backlog it", "Park it" | Defer |
| "Same as X", "Combine with", "Duplicate of", "Merge into" | Merge |
| "It's live", "We shipped", "Released in", "Launched" | Ship |

When you detect a decision, prompt the user:

> "Sounds like a decision! Want me to capture the rationale? I'll ask a few quick questions."

## The Capture Flow

### 1. Detect
Listen for trigger phrases. Example:
> User: "We're going to reject the dark mode request for now."

### 2. Prompt
Ask focused questions to capture context:

**For Rejections:**
- Why are we saying no? (1 sentence)
- Is this permanent or "not now"?
- Should we notify the requester?

**For Acceptances:**
- What's the core reason we're doing this?
- Any constraints or conditions?
- Target timeline?

**For Deferrals:**
- Why not now?
- What would need to change to revisit?
- When should we reconsider?

**For Ships:**
- What was the original request?
- Who should we notify?
- Any follow-up needed?

### 3. Structure
Create a decision record. See [TEMPLATES/decision-record.md](TEMPLATES/decision-record.md) for format.

Example output:

```markdown
## Decision: Reject Dark Mode Request

**Date**: 2025-01-27
**Type**: Rejected (Deferred)
**Decider**: Product Team

### Context
Multiple users requested dark mode. We evaluated during Q1 planning.

### Decision
Not building dark mode now. Will revisit in Q3.

### Rationale
Usage data shows <5% of users work in low-light environments.
Dev effort (~3 weeks) doesn't justify the reach.

### Revisit
Q3 2025 - reassess if request volume increases significantly.

### Notify
Yes - draft closure message for requesters.
```

### 4. Persist
- **With IdeaLift MCP**: Log via `log_decision` tool automatically
- **Standalone**: Output markdown for user to save

## Decision Types

### Accept
Moving forward with an idea or feature.

Key prompts:
- What's driving this decision?
- Any conditions or constraints?
- Who needs to know?

See [PATTERNS/accept.md](PATTERNS/accept.md)

### Reject
Saying no to a request.

Key prompts:
- Why are we saying no?
- Is this permanent or circumstantial?
- How do we communicate this constructively?

See [PATTERNS/reject.md](PATTERNS/reject.md)

### Defer
Not now, but not never.

Key prompts:
- What's blocking this now?
- What conditions would change our mind?
- When should we revisit?

See [PATTERNS/defer.md](PATTERNS/defer.md)

### Merge
Combining similar ideas or marking duplicates.

Key prompts:
- Which idea is the primary?
- What's shared vs. unique?
- Should requesters be notified?

See [PATTERNS/merge.md](PATTERNS/merge.md)

### Ship
Feature launched - time to close the loop.

Key prompts:
- Who originally requested this?
- What's the announcement?
- Any follow-up needed?

See [PATTERNS/ship.md](PATTERNS/ship.md)

## Closing the Loop

One of the most powerful things you help with is **closure communication**:

When a feature ships:
> "Hey [Name], great news! That feature you requested is now live. [Link]"

When something is rejected:
> "Thanks for the suggestion. We've decided not to build this because [reason]. Here's what we're doing instead..."

See [TEMPLATES/closure-message.md](TEMPLATES/closure-message.md) and [TEMPLATES/rejection-rationale.md](TEMPLATES/rejection-rationale.md)

## Rationale Quality

Good rationale is:
- **Specific**: "Usage data shows <5% need this" not "Not enough demand"
- **Honest**: Real reasons, not corporate speak
- **Future-useful**: Someone in 6 months can understand why

Bad rationale:
- "We decided not to do it" (no reasoning)
- "Not a priority" (what does that mean?)
- "Maybe later" (when? why?)

## MCP Integration

This skill works standalone but is enhanced with **IdeaLift MCP**.

### Standalone Mode
- Outputs decision records as markdown
- User saves to their system
- No persistence or history

### With IdeaLift MCP
- Decisions auto-log via `log_decision` tool
- Links decisions to ideas
- Full audit trail and history
- Searchable decision memory

Setup: See [GUIDES/mcp-integration.md](GUIDES/mcp-integration.md)

## Example Conversations

### Rejecting a Feature

**User**: We need to tell the customer we're not building the Excel export feature.

**You**: Got it - rejecting Excel export. Quick questions:
1. Why are we saying no?
2. Is this permanent or "not now"?
3. Should I draft a message for the customer?

**User**: Low demand, only 2 customers asked. It's a "not now" - revisit if more ask. Yes, draft a message.

**You**: Here's the decision record:

```
## Decision: Defer Excel Export

**Type**: Deferred
**Rationale**: Low demand (2 customers). Revisit if 5+ request.
**Revisit**: When request count hits 5

### Closure Message (Draft)
Thanks for suggesting Excel export. We're not building this right now because demand is low, but we're tracking interest. If this changes, we'll prioritize it.
```

### Shipping a Feature

**User**: The API rate limiting feature just went live!

**You**: Nice! Let's close the loop. Who originally requested this, and should we notify them?

**User**: It was from Acme Corp and two other enterprise customers. Yes, send notifications.

**You**: Here's the ship record and draft notifications:

```
## Decision: Ship API Rate Limiting

**Type**: Shipped
**Date**: 2025-01-27

### Closure Messages

**To Acme Corp:**
Great news! The API rate limiting feature you requested is now live.
Check it out at [Settings > API > Rate Limits].
Thanks for the feedback - it made our product better.
```

## Best Practices

1. **Capture in the moment**: Decisions captured later lose context
2. **One decision, one record**: Don't bundle multiple decisions
3. **Include the "why"**: Future you will thank present you
4. **Set revisit dates**: For deferrals, when to reconsider
5. **Close the loop**: Notify stakeholders when things ship or get rejected

## Quick Reference

| Decision | Key Question | Output |
|----------|--------------|--------|
| Accept | Why yes? | Decision record + next steps |
| Reject | Why no? | Decision record + closure message |
| Defer | Why not now? | Decision record + revisit trigger |
| Merge | Which is primary? | Decision record + consolidation |
| Ship | Who to notify? | Decision record + announcements |

## Resources

- [QUICKSTART.md](QUICKSTART.md) - Get started in 2 minutes
- [EXAMPLES.md](EXAMPLES.md) - More conversation examples
- [TEMPLATES/](TEMPLATES/) - Decision record templates
- [GUIDES/](GUIDES/) - Detailed guides
- [PATTERNS/](PATTERNS/) - Decision type patterns
