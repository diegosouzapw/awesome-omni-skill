---
name: ask-peer
description: Consult with a peer engineer for plan review, code review, implementation discussions, or problem-solving brainstorming. Use when you need a second opinion, want to validate your approach, or check for overlooked issues.
---

# Peer Engineer Consultation

Get a second opinion from a peer engineer (Claude subagent) for:

- **Planning review**: Validate your implementation approach before starting
- **Code review**: Check completed work for issues or improvements
- **Problem-solving**: Brainstorm solutions when stuck
- **Sanity check**: Confirm you're on the right track

## Usage

Use the `/ask-peer` command followed by your consultation request:

```bash
/ask-peer Review this implementation plan for adding user authentication
```

Or simply describe what you need help with:

```bash
/ask-peer I'm stuck on how to handle error cases in the payment flow
```

## Examples

**Plan review before implementation:**

```bash
/ask-peer Review my plan to refactor the authentication module. Check for security concerns and missing edge cases.
```

**Code review after completion:**

```bash
/ask-peer Review the changes I made to src/auth/. Look for bugs, security issues, and code quality problems.
```

**Problem-solving consultation:**

```bash
/ask-peer I'm getting intermittent test failures in the payment module. Help me think through possible causes.
```

## What the Peer Agent Provides

- Frank, objective feedback as an equal
- Specific concerns with suggested alternatives
- Questions to challenge assumptions
- Practical solutions rather than perfection
