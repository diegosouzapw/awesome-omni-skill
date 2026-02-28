---
name: add-content
description: Create new files in knowledge base with directory governance. Use when user wants to save, create, or add content. Reads RULE.md to understand directory purpose and follows specified rules. Recommends target directory based on content analysis.
---

# Add Content Skill

Generic file creation Skill that works with any directory structure by reading RULE.md to understand purpose and rules.

## When to use this Skill

- User says "save", "create", "add", "note"
- User provides content to store
- User provides URLs to archive
- User mentions "readlater", "bookmark", "save article"
- User requests content preservation

## Workflow

### 1. Content Analysis

**Determine content type**:
- URL: Web content to archive
- Text: Note or article
- Code: Code snippet
- Structured data: JSON, YAML, etc.
- Mixed: Multiple content types

**Extract key information**:
- Main topics and themes
- Keywords and technical terms
- Content category (technical, personal, work, etc.)
- Content length and complexity

### 2. Directory Discovery

**Scan knowledge base**:
1. Identify root directory (current working directory or user-specified)
2. List main directories using `ls` or Glob
3. Read RULE.md from each main directory
4. Build directory catalog with purposes

**Example scan result**:
```
ReadLater/ - Purpose: "Archive web content for later reading"
Research/ - Purpose: "Technical and academic research notes"
Work/ - Purpose: "Professional work-related content"
Personal/ - Purpose: "Personal life documentation"
Miscellaneous/ - Purpose: "General notes and articles"
```

### 3. Smart Classification

**Match content to directories**:
1. Analyze content topics vs directory purposes
2. Check if content type matches directory specialization
3. Consider user's past behavior (if known)
4. Rank directories by suitability (0-100 score)

**Classification logic**:
- URL + directory purpose mentions "web archival" → high score
- Technical content + directory purpose mentions "research" → high score
- Work keywords + directory purpose mentions "professional" → high score
- Default to "Miscellaneous" or similar general-purpose directory

### 4. User Confirmation

**Present recommendation**:
```
I analyzed your content about [topic].

Recommended location: [Directory]/
Reason: [Why this directory suits the content]

Is this correct?
- Yes (proceed with this directory)
- No, use different directory (please specify path)
- Show me other options
```

**If user requests alternatives**:
- Show top 3 ranked directories with scores and reasons
- Allow user to choose or specify custom path

**If user specifies custom path**:
- Validate path exists or create if requested
- Continue with specified path

### 5. Read Target RULE.md

**Locate and read RULE.md**:
1. Check target directory for RULE.md
2. If not found, check parent directories (inheritance)
3. If no RULE.md found, ask user if should create one

**Parse RULE.md for**:
- File naming conventions (e.g., `YYYY-MM-DD-title.md`)
- Required file structure (e.g., directories, metadata files)
- Frontmatter requirements (if any)
- Special instructions (e.g., "fetch from URL", "generate summary")
- Business logic workflows (e.g., web archival process)
- Format examples

### 6. Execute Operation Per RULE.md

**Follow RULE.md instructions exactly**:

**Example 1 - Simple Note**:
```
RULE.md says: "Files named YYYY-MM-DD-title.md with markdown format"
→ Generate filename: 2025-10-28-transformer-architecture.md
→ Create file with content
→ Save in target directory
```

**Example 2 - Web Archival**:
```
RULE.md says:
"When user provides URL:
1. Fetch content using WebFetch
2. Convert to markdown
3. Create directory: Articles/YYYY/MM/YYYY-MM-DD_title-slug/
4. Save article.md and metadata.yaml"

→ Execute WebFetch on URL
→ Extract title, content, date
→ Convert to clean markdown
→ Create directory structure
→ Save article.md with content
→ Create metadata.yaml with:
  - url: [original URL]
  - title: [article title]
  - date: [fetch date]
  - summary: [brief summary]
→ Optionally save images if referenced
```

**Example 3 - Structured Content (WorkLog)**:
```
RULE.md says:
"Daily logs in YYYY/QX/MM-Month/ structure.
MUST verify datetime first.
Format: [specific template shown in RULE.md]"

→ Execute `date` command to verify current date
→ Parse date: 2025-10-28 → 2025/Q4/10-October/
→ Check if directory structure exists, create if needed
→ Read RULE.md template section
→ Create/update 2025-10-28.md with template format
→ Fill in sections per RULE.md specifications
```

**Key principles**:
- Execute special instructions literally (e.g., "fetch from URL")
- Follow naming conventions exactly
- Create directories as specified
- Include required metadata/frontmatter
- Apply format examples shown in RULE.md

### 7. Governance Update

**Update README.md**:
1. Read current README.md in target directory
2. Find appropriate section (Files, Subdirectories, etc.)
3. Add new entry with:
   - Filename with link
   - Brief description (from content analysis or user input)
   - Timestamp: current date
4. Update "Recent Changes" section
5. Update "Last updated" timestamp
6. Save README.md

**Update parent README.md if needed**:
- If created new subdirectory, update parent's README.md
- Add entry for new subdirectory with description

**Verify updates**:
- Check README.md is valid markdown
- Ensure no duplicate entries
- Confirm timestamps are current

### 8. Report to User

**Confirm operation complete**:
```
✅ Content created successfully

Location: [full path to file]
Format: [file format used]
Governance: [which RULE.md applied]

README.md updated:
- [target directory]/README.md
- [parent directory]/README.md (if applicable)

Summary:
[Brief summary of what was created and where]
```

## Special Cases

### URL Content Archival

When content is URL and RULE.md specifies archival:

1. **Validate URL**:
   - Check URL format is valid
   - Optionally test if URL is accessible

2. **Check for Duplicates**:
   - Search existing README.md for this URL
   - If found: warn user, ask if should re-archive

3. **Fetch Content**:
   - Use WebFetch tool with prompt: "Extract the main article content, title, and publication date"
   - Handle fetch errors gracefully

4. **Process Content**:
   - Convert HTML to clean markdown
   - Extract images if RULE.md requests
   - Generate title slug for directory name
   - Create brief summary (2-3 sentences)

5. **Save Per RULE.md**:
   - Follow directory structure exactly
   - Create all specified files
   - Include all required metadata

### Structured Content with Frontmatter

If RULE.md requires YAML frontmatter:

```yaml
---
title: [Title]
date: [Date]
tags: [tag1, tag2]
category: [Category]
---

[Content here]
```

Generate frontmatter with:
- Required fields per RULE.md
- Inferred values from content analysis
- User-provided metadata if available

### Multiple Files

If RULE.md specifies creating multiple files:

1. Create all files in specified order
2. Ensure cross-references are correct
3. Update README.md to list all created files
4. Report all files created

### Custom Business Logic in RULE.md

Parse RULE.md for custom instructions:

**Pattern detection**:
- "When user provides URL..." → URL handling logic
- "When user mentions [keyword]..." → Keyword-triggered workflow
- "MUST verify [condition]" → Mandatory pre-check
- "After creation, [action]" → Post-creation hook

**Execute custom logic**:
- Follow instructions step-by-step
- Use specified tools (WebFetch, Bash, etc.)
- Maintain specified format
- Report custom workflow execution

## Error Handling

### No Suitable Directory Found
- Ask user: "I couldn't find a suitable directory. Please specify where to save this content, or I can help you create a new directory."
- Offer to create new directory with RULE.md

### RULE.md Missing
- Check parent directories for inheritance
- If no RULE.md in tree: ask user if should create one
- Fallback: use basic file creation with sensible defaults

### Ambiguous Classification
- Present multiple options with scores
- Explain reasoning for each
- Let user choose or specify custom path

### RULE.md Instructions Unclear
- Ask user for clarification
- Offer to interpret instructions and ask for confirmation
- Document unclear instructions for future improvement

### File Already Exists
- Check if filename conflicts with existing file
- Offer to: append timestamp, rename, overwrite, or choose different name
- Never overwrite without confirmation

### Operation Fails
- Report error clearly
- Explain what went wrong
- Suggest alternatives or fixes
- Don't leave partial state (clean up if needed)

## Integration with Governance

This Skill automatically invokes the governance protocol:

**Before operation**:
- Locate and read RULE.md
- Validate operation is allowed
- Check README.md for context

**During operation**:
- Follow RULE.md specifications
- Execute any special workflows

**After operation**:
- Update README.md
- Update parent README.md if needed
- Verify updates successful

## Examples

### Example 1: Simple Note

**User**: "Save this note: Today I learned about transformer architecture in deep learning"

**Skill workflow**:
1. Analyzes content → Technical/AI topic
2. Scans directories → Finds Research/AI/, Research/DeepLearning/, Miscellaneous/
3. Reads RULE.md files → Research/AI/ purpose: "AI and machine learning research"
4. Recommends Research/AI/ (score: 95)
5. User confirms
6. Reads Research/AI/RULE.md → Files named `YYYY-MM-DD-topic.md`
7. Creates `2025-10-28-transformer-architecture.md`
8. Updates Research/AI/README.md
9. Reports success with file location

### Example 2: Web Archival

**User**: "readlater https://example.com/interesting-article"

**Skill workflow**:
1. Detects URL
2. Scans directories → Finds ReadLater/
3. Reads ReadLater/RULE.md → Purpose: "Archive web content"
   - Instructions: "Fetch URL, convert to markdown, save in Articles/YYYY/MM/"
4. Recommends ReadLater/ (score: 100, exact match)
5. User confirms (or auto-confirm for obvious match)
6. Executes RULE.md instructions:
   - WebFetch URL
   - Extracts title: "Interesting Article About AI"
   - Converts to markdown
   - Creates `ReadLater/Articles/2025/10/2025-10-28_interesting-article-about-ai/`
   - Saves article.md and metadata.yaml
7. Updates ReadLater/README.md and ReadLater/Articles/2025/10/README.md
8. Reports success with summary

### Example 3: WorkLog Entry

**User**: "Update my worklog"

**Skill workflow**:
1. Detects worklog keyword
2. Scans directories → Finds Work/WorkLog/
3. Reads Work/WorkLog/RULE.md:
   - Purpose: "Daily work logs"
   - Structure: YYYY/QX/MM-Month/
   - MUST verify datetime first
   - Format: [specific template]
4. Recommends Work/WorkLog/ (score: 100)
5. User confirms
6. Executes RULE.md instructions:
   - Runs `date` → 2025-10-28
   - Parses to 2025/Q4/10-October/
   - Checks if directory exists, creates if needed
   - Checks if 2025-10-28.md exists
   - Creates/updates file with template from RULE.md
7. Updates hierarchical README.md files:
   - Work/WorkLog/2025/Q4/10-October/README.md
   - Work/WorkLog/2025/Q4/README.md
   - Work/WorkLog/README.md
8. Reports success with daily log location

## Best Practices

1. **Always read RULE.md completely** - Don't assume, read the full file
2. **Ask for confirmation on ambiguous choices** - User knows their needs
3. **Follow RULE.md exactly** - It's the contract for directory behavior
4. **Update README.md immediately** - Don't batch updates
5. **Report clearly** - User should know exactly what happened
6. **Handle errors gracefully** - Don't leave broken state
7. **Preserve existing structure** - Don't modify unrelated files
8. **Document custom workflows** - If RULE.md has special instructions, note them

## Notes

- This Skill is generic - it works with ANY directory structure
- Business logic comes from RULE.md, not hardcoded in Skill
- Classification is smart but always asks for user confirmation
- RULE.md format is natural language - no strict schema required
- Integration with governance agent is automatic
- Works in parallel with CLAUDE.md subagents independently
