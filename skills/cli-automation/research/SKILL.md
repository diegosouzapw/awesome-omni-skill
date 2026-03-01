---
name: research
description: Research a topic across gopher-cli sources, read relevant documents, and save a structured summary to the vault
argument-hint: [topic]
---

Research the topic: **$ARGUMENTS**

Follow this workflow using the gopher-cli CLI via Bash tool calls. Output is auto-JSON when piped.

## 1. Discover sources

List available namespaces and browse each root to see what's available:

```bash
gopher-cli browse
gopher-cli browse feed.hackernews/
gopher-cli browse vault/
```

## 2. Search for relevant content

Search across every namespace with keywords from the topic. Cast a wide net — try synonyms and related terms:

```bash
gopher-cli search feed.hackernews/ "topic keywords"
gopher-cli search vault/ "topic keywords"
gopher-cli search docs/ "related term"
```

## 3. Read the most relevant documents

Fetch the top results. Extract key facts, quotes, and insights. Note which source each piece came from:

```bash
gopher-cli fetch feed.hackernews/entry/5
gopher-cli fetch vault/research/related-note.md
```

## 4. Synthesize findings

Produce a structured research note in markdown:

```
# Research: [topic]
Date: [today]

## Summary
[2-3 sentence overview]

## Key Findings
- [finding 1 — with source]
- [finding 2 — with source]
- ...

## Details
[longer analysis organized by theme]

## Sources
- [namespace/path] — [what it contained]
```

## 5. Save to vault

Publish the note to the vault:

```bash
gopher-cli publish vault/research/[slugified-topic].md --content "[markdown content]"
```

Confirm what you saved and where.
