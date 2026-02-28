---
name: ai-notes
description: Manage AI notes in ~/Compost/AI-Notes - read existing notes, list recent documents, and write new implementation plans (仕様書, 計画書, 設計書). Auto-triggers when reading from or writing to notes, specifications, or plans.
---

## Notes Management Script

**IMPORTANT**: Use the `notes.ts` script for all AI notes operations.

```bash
# Script location
~/.claude/skills/ai-notes/notes.ts
```

### Generate Output Path

Generate correct file paths for new notes:

```bash
# Single document
deno run -A ~/.claude/skills/ai-notes/notes.ts generate "<title>"
# => ~/Compost/AI-Notes/2025-12/24-0821-<title>.md

# Multiple documents (returns directory path)
deno run -A ~/.claude/skills/ai-notes/notes.ts generate "<title>" --multiple
# => ~/Compost/AI-Notes/2025-12/24-0821-<title>/
```

### List Existing Notes

List notes ordered by file path (newest first) with H1 titles:

```bash
# List all notes
deno run -A ~/.claude/skills/ai-notes/notes.ts list

# List 10 most recent notes
deno run -A ~/.claude/skills/ai-notes/notes.ts list --limit 10

# Pagination: skip first 10, show next 10
deno run -A ~/.claude/skills/ai-notes/notes.ts list --limit 10 --offset 10
```

**Output format**: Each line shows the full path and the first H1 heading from the file:
```
/path/to/note.md: タイトル
```

## Path Convention

- Base directory: `~/Compost/AI-Notes/{year}-{month}/`
- Single document: `{day}-{hour}{minutes}-{title}.md`
- Multiple documents: `{day}-{hour}{minutes}-{title}/{number}-{title}.md`

## Content Requirements

- **Language**: Write documents in Japanese
- **Code Comments**: ALWAYS write code comments in English, even when the document is in Japanese
  - Example code snippets should have English comments
  - Inline code documentation should be in English
  - Only narrative text outside code blocks should be in Japanese
- **Diagrams**: Use Mermaid syntax for all diagrams and flowcharts

## Reading Notes

When user requests reading from notes:

1. **List recent notes**: Use `notes.ts list --limit 20` to get recent documents
2. **Search by pattern**: Use Glob tool with pattern like `~/Compost/AI-Notes/**/*{keyword}*.md`
3. **Search by content**: Use Grep tool to search across note contents
4. **Complex searches**: Use Task tool with Explore agent for multi-step exploration

**Search priority**:
- Start with `notes.ts list` to see recent notes
- For specific topics, use Grep across `~/Compost/AI-Notes`
- For complex queries, delegate to Explore agent
