---
name: chatgpt-history
description: Search and extract data from ChatGPT conversation history. Use when the user asks to find, search, or extract information from their ChatGPT history, conversations, or past chats.
---

# ChatGPT History

Search and extract data from ChatGPT conversation history. The full history is in `references/conversations.json`.

## When to Use

- User asks to find something from their ChatGPT history
- User wants to extract data, decisions, or information from past conversations
- User mentions "my ChatGPT chats", "conversation history", or "what did we discuss about X"

## Searching the Conversation History

**Quick search (grep):** From the skill directory, run:

```bash
rg 'search-term' references/conversations.json
```

**Structured search:** For parsed excerpts with conversation title and context:

```bash
pnpm query "your query"
```

Or: `npx tsx scripts/search.ts "your query"`

## File Location

The conversation data is at `references/conversations.json`. Ensure the user has copied their file there (from OpenAI export or any ChatGPT export).

## Output

When extracting data, present results with:
- Conversation title
- Role (user/assistant)
- Relevant excerpt
- Source conversation if multiple matches
