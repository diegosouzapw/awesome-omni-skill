---
name: using-live-documentation
description: Use BEFORE implementing, writing, configuring, or setting up ANY feature involving libraries, frameworks, or complex APIs - even before reading existing code. Fetches current documentation to ensure correct usage. Triggers on third-party libraries (such as react-query, FastAPI, Django, pytest), complex standard library modules (such as subprocess, streams, pathlib, logging), and "how to" questions about library usage. Do NOT use for trivial built-ins (such as dict.get, Array.map) or pure algorithms. Load this skill first to receive guidance on finding current documentation when implementing features, exploring code, or answering library-related questions.
---

# Using Live Documentation

## Overview

**Your training data is outdated. Current documentation is always more accurate.**

When implementing features, answering questions, or debugging issues involving libraries/frameworks/tools, you MUST fetch current documentation using Context7 before writing code or making recommendations.

## Core Principle

LLM training data becomes stale the moment training ends. Libraries evolve:
- APIs change between versions
- Best practices get updated
- New features get added
- Old patterns get deprecated

**Never implement from memory. Always verify with current docs.**

## Mandatory Workflow

### Step 1: Recognize the Trigger

You MUST use documentation search when you encounter ANY of these:

- Library name mentioned (react-query, fastapi, pydantic, express, etc.)
- Framework name mentioned (Next.js, Django, React, Vue, etc.)
- Version number specified (react-query v5, Python 3.12, etc.)
- Technical concept tied to specific tool (optimistic updates in react-query)
- Implementation questions (how do I X in Y?)
- Best practices questions (what's the right way to X?)
- Debugging library-specific behavior

**Red flags that mean you're about to fail:**
- "Based on my knowledge of..."
- "From what I remember about..."
- "The typical pattern for..."
- Writing code without checking docs first
- Having uncertainty about the correct approach

### Step 2: Dispatch Documentation Search Subagent

**Why subagent instead of direct Context7:**
- Saves 10,000-20,000 tokens of context in main agent
- Subagent filters docs to only what you need
- Main agent stays focused on implementation
- Better token management across the session

**How to dispatch:**

Dispatch the documentation-searcher agent with the following information:

- **Library name**: Exact package/library name (e.g., "react-query", "fastapi", "pydantic")
- **Topic**: Specific concept or feature (e.g., "optimistic updates", "path parameters", "field validators")
- **What you need**: Specific APIs, patterns, or examples you're looking for

The agent will search Context7 documentation and provide a focused synthesis with:
- Exact API signatures
- Recommended patterns and best practices
- Code examples
- Version-specific guidance

### Step 3: Implement Using Verified Patterns

**After receiving subagent synthesis:**

1. Cite what you learned: "According to react-query v5 docs (from subagent search)..."
2. Use exact API signatures provided
3. Follow recommended patterns from synthesis
4. Note any differences from what you expected
5. If gaps exist, dispatch another search or use WebSearch

**Never:**
- Mix training data patterns with doc patterns
- Assume API names/signatures
- Skip documentation check "to save time"
- Implement first, verify later
- Use Context7 MCP tools directly (always dispatch documentation-searcher agent)

## Red Flags - STOP

If you're thinking ANY of these, you're about to violate the skill:

### Context Rationalization Flags
- ❌ "I'm only using X% of budget" - Percentage hides absolute waste
- ❌ "Well within acceptable limits" - Ignores session-wide compounding
- ❌ "I have plenty of budget left" - Context is for ENTIRE session
- ❌ "This is just one search" - "Just one" becomes "just one more"

### Efficiency Framing Flags
- ❌ "Direct access is more efficient" - You're optimizing for wrong metric
- ❌ "Subagent dispatch is overhead" - It's an investment, not overhead
- ❌ "Completed in fewer messages" - Messages don't matter, tokens do
- ❌ "For straightforward lookups, direct is optimal" - Context math doesn't change

### Quality Justification Flags
- ❌ "I got comprehensive examples" - You don't need comprehensive, you need relevant
- ❌ "I can filter the docs myself" - Filtering doesn't remove docs from context
- ❌ "I need detailed information" - Subagent provides exactly what you need

**The context math:**
- Direct Context7: 15,000-25,000 tokens per search
- Subagent: 2,000-5,000 tokens per search
- Difference: 10,000-20,000 tokens SAVED per search
- 3 searches: 48,000 tokens saved
- That's 48,000 tokens for MORE searches, longer conversations, complex implementations

**Never use "I have budget left" to justify waste.**

## When NOT to Use Documentation Search

**Skip documentation search for:**
- Language built-ins (Python dict, JavaScript Array)
- Standard library basics (Python os.path, JavaScript fs)
- Well-known universal concepts (HTTP status codes, REST principles)
- Questions about YOUR codebase (use Read/Grep)

**But DO use documentation search for:**
- Third-party libraries, even familiar ones
- Framework-specific patterns
- Version-specific APIs
- Best practices for tools

**When in doubt: dispatch a subagent.** The cost of a subagent search (2,000-5,000 tokens) is tiny compared to implementing wrong patterns from training data.

## Context Management Strategy

**Why subagents are mandatory:**

**Context savings per search:**
- Direct Context7: 15,000-25,000 tokens per search
- Subagent approach: 2,000-5,000 tokens per search
- Savings: 10,000-20,000 tokens per search

**Across a session:**
- 3 direct searches: ~60,000 tokens
- 3 subagent searches: ~12,000 tokens
- Savings: ~48,000 tokens

**That's 48,000 tokens available for:**
- More codebase files
- Longer conversations
- Additional library searches
- Complex implementations

## Verification Checklist

Before claiming you've implemented something correctly, verify:

- [ ] Dispatched documentation-searcher agent to fetch current documentation
- [ ] Provided clear library name, topic, and what you need
- [ ] Received synthesis with API signatures
- [ ] API signatures match documentation exactly
- [ ] Patterns follow current best practices from synthesis
- [ ] No uncertainties remain about correct approach
- [ ] Can cite documentation source for key decisions
- [ ] Did NOT use Context7 MCP tools directly

**If you have ANY uncertainty after receiving synthesis:**
- Dispatch another documentation-searcher agent with refined topic
- Use WebSearch for supplementary info
- Ask human for clarification

**Never:**
- Use Context7 MCP tools directly
- Ship uncertain implementation
- Skip documentation search to "save time"

## Common Mistakes

### Mistake 1: "I remember this API"

```
❌ "I know react-query uses useQuery, let me write this..."
✅ "Let me dispatch documentation-searcher agent to verify the current useQuery API..."
```

**Why it fails:** APIs change. Your memory is from training cutoff.

### Mistake 2: "Subagent overhead isn't worth it"

```
❌ "This is just one search, I'll use Context7 directly..."
✅ "Even one search saves 15,000 tokens. Always dispatch documentation-searcher agent."
```

**Why it fails:** "Just one" becomes "just one more" throughout the session. Context compounds.

### Mistake 3: "I'll verify after writing"

```
❌ [Writes full implementation] "Let me check if this is right..."
✅ [Dispatches documentation-searcher agent first] "Now I'll implement using verified patterns..."
```

**Why it fails:** Fixing wrong code takes longer than writing correct code once.

## Integration with Other Workflows

**With Test-Driven Development:**
1. Dispatch documentation-searcher agent BEFORE writing test
2. Receive synthesis with API signatures
3. Write test using documented patterns
4. Implement using same synthesis

**With Brainstorming:**
1. During design discussion, dispatch documentation-searcher agent for relevant docs
2. Base design on current capabilities from synthesis
3. Don't propose deprecated patterns
4. Verify feasibility with current API

**With Debugging:**
1. Dispatch documentation-searcher agent when error involves library
2. Check if API usage matches synthesis patterns
3. Verify you're using correct version's API
4. Look for migration guides if version changed

## Summary

**Before implementing ANYTHING involving a library/framework:**

1. Recognize trigger (library name → stop)
2. Dispatch documentation-searcher agent
3. Provide clear library name, topic, and what you need
4. Receive synthesis with API signatures and patterns
5. Implement using verified patterns from synthesis
6. Cite documentation source

**Critical rules:**
- **NEVER use Context7 MCP tools directly**
- **ALWAYS dispatch documentation-searcher agent for documentation**
- **Context savings: 10,000-20,000 tokens per search**
- **Your training data is always outdated**
- **Current documentation is always more accurate**
- **Dispatch agent first, write code second**

**This is not optional. This is mandatory.**
