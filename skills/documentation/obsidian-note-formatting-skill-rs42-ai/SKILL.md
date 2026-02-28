---
name: obsidian-note-formatting-skill
description: Create well-formatted Obsidian notes with proper frontmatter, templates, and organization. Use when users want to create notes for their Obsidian vault, document ideas or technical content, or need help structuring information in Obsidian-compatible markdown format.
---

# Obsidian Note-Taker

This skill helps create properly formatted Obsidian notes with consistent structure, frontmatter, and organization.

## When to Use This Skill

Activate this skill when:
- User asks to "create a note" or "document" something for Obsidian
- User provides information that needs structuring as an Obsidian note
- User mentions their vault, knowledge base, or Zettelkasten
- User wants to capture an idea, meeting notes, or technical documentation

## Core Workflow

### 1. Determine Note Type

Ask the user or infer from context:
- **Technical Documentation**: Architecture, guides, how-tos, system docs
- **Business/Ideas**: Brainstorming, concepts, project ideas
- **Personal Notes**: Meetings, reflections, daily notes
- **Research Notes**: Literature notes, source materials

### 2. Select Folder Placement

Based on note type, suggest location:
- Technical → `1 - Main Notes/`
- Ideas → `Personal/Thoughts & Ideas/`
- Personal → `Personal/`
- Research → `Research/` or topic-specific folder

Ask user to confirm: "Based on this being [type], I'll suggest placing this in [folder]. Is that correct?"

### 3. Generate Frontmatter

Create YAML frontmatter following this template:

```yaml
---
date: YYYY-MM-DD  # Today's date
status: capture    # Options: capture, develop, refine, complete
type: note         # Options: note, guide, architecture, workflow, idea
tags:
  - tag1          # 2-4 relevant tags based on content
  - tag2
---
```

### 4. Apply Template Structure

Use the appropriate template from `references/templates.md` based on note type.

### 5. Format Content

- Use `[[wikilinks]]` for internal references
- Use `[text](url)` for external links
- Maintain proper heading hierarchy (# > ## > ###)
- Include "Related Notes" section at the end

### 6. Find Related Notes in Vault

**Before finalizing the note, search the user's Obsidian vault for related notes:**

Use the Obsidian MCP tools to find relevant existing notes:

```
# Search vault for notes related to the topic
mcp__MCP_DOCKER__obsidian_simple_search(
    query="[key terms from note title/content]",
    context_length=50
)
```

Then add the most relevant notes (3-5 maximum) to the "Related Notes" section as wikilinks:
```markdown
## Related Notes
- [[Actual Note Title 1]]
- [[Actual Note Title 2]]
- [[Actual Note Title 3]]
```

If no related notes are found, you can omit the "Related Notes" section or leave it empty for the user to fill later.

## Quick Templates

For immediate use without loading references:

### Technical Note
```markdown
---
date: YYYY-MM-DD
status: capture
type: note
tags:
  - technical
  - [topic]
---

# [Title]

## Overview
[One paragraph summary]

## Key Concepts
- Concept 1
- Concept 2

## Implementation Details
[Main content]

## Related Notes
- [[Related Note 1]]
- [[Related Note 2]]
```

### Idea Note
```markdown
---
date: YYYY-MM-DD
status: capture
type: idea
tags:
  - idea
  - [domain]
---

# [Idea Title]

## Problem Statement
[What problem does this solve?]

## Proposed Solution
[How to solve it]

## Next Steps
- [ ] Action 1
- [ ] Action 2

## Related Ideas
- [[Related Idea]]
```

## File Naming Convention

- Title case with spaces: `System Architecture for Auth Service.md`
- Remove special characters: `: / \ | * ? " < >`
- Keep under 60 characters
- No underscores or hyphens in filenames

## Tag Suggestions

Suggest tags based on content domain:
- **Technical**: `architecture`, `api`, `database`, `security`, `devops`
- **AI/ML**: `ai`, `llm`, `claude`, `automation`, `agents`
- **Business**: `business`, `startup`, `revenue`, `strategy`
- **Personal**: `meeting`, `reflection`, `planning`, `learning`

Format: lowercase, use hyphens for multi-word tags

## Validation Checklist

Before presenting the note:
- [ ] Frontmatter present and valid YAML
- [ ] Date in YYYY-MM-DD format
- [ ] Status is valid option
- [ ] At least 2 relevant tags
- [ ] Title clear and under 60 chars
- [ ] Content follows template structure
- [ ] Wikilinks properly formatted
- [ ] Folder placement suggested

## Output Format

Present the complete note and instructions:

```
Here's your formatted Obsidian note:

[COMPLETE NOTE CONTENT]

📁 Save to: `[folder]/[filename].md`

Would you like me to adjust anything?
```

## Saving to Obsidian Vault

**After presenting the formatted note, ask the user if they want to save it to their Obsidian vault.**

### Step 1: Present Note and Ask for Confirmation

After showing the formatted note, ask:

```
Would you like me to save this note to your Obsidian vault at: `[suggested-folder]/[filename].md`?
```

### Step 2: If User Confirms, Save Using MCP Tool

Only if the user confirms (says yes, save it, go ahead, etc.), then use the Obsidian MCP integration:

```
mcp__MCP_DOCKER__obsidian_append_content(
    filepath="[suggested-folder]/[filename].md",
    content=[complete_formatted_note_including_frontmatter]
)
```

### Step 3: Confirm Success

After successful save, inform the user:

```
✅ Note saved to Obsidian vault at: `[filepath]`
```

### Step 4: Error Handling

If MCP save fails:
1. Explain: "I couldn't save directly to your vault"
2. Display the formatted note content
3. Suggest manual save: "You can copy this content and save it manually to: `[filepath]`"

### Important Notes

- **Always ask before saving** - Never save without user confirmation
- The filepath should match the suggested location from "Output Format"
- Include the complete note with frontmatter in the `content` parameter
- If user declines to save, that's fine - they have the formatted note to copy manually

### Example Workflow

For a technical note titled "Testing MCP Integration":

1. Format note with frontmatter:
   ```yaml
   ---
   date: 2025-10-20
   status: capture
   type: note
   tags:
     - testing
     - mcp
   ---

   # Testing MCP Integration

   [content here]
   ```

2. Determine folder: `1 - Main Notes/`

3. Save using MCP:
   ```
   mcp__MCP_DOCKER__obsidian_append_content(
       filepath="1 - Main Notes/Testing MCP Integration.md",
       content=[complete note from step 1]
   )
   ```

4. Confirm: "✅ Note saved to Obsidian vault at: `1 - Main Notes/Testing MCP Integration.md`"

## Advanced Features

For complex requirements, see:
- `references/templates.md` - Detailed templates for all note types
- `references/folder-structure.md` - Complete folder organization guide
- `scripts/validate_frontmatter.py` - Validate note structure
