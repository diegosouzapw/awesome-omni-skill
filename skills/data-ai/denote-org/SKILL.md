---
name: denote-org
description: "Use this skill when working with ~/org/ directory, Denote files (YYYYMMDDTHHMMSS--title__tags.org), or org-mode knowledge bases. Provides scripts for: parsing Denote filenames/metadata, extracting org file TOC, and navigating 3,000+ file PKM systems. Trigger on: ~/org/, ~/org/llmlog/, ~/org/bib/, Denote ID parsing, org heading extraction."
---

# Denote-Org Skills

## Overview

This skill enables Claude to work with Denote-based Personal Knowledge Management (PKM) systems at scale. Denote is a simple yet powerful note-taking system for Emacs that combines file naming conventions, org-mode structure, and knowledge graph links. This skill provides procedural knowledge and executable scripts for handling 3,000+ file knowledge bases efficiently.

## When to Use This Skill

Claude should use this skill when:

1. **Working with Denote files** - Files named `YYYYMMDDTHHMMSS--title__tag1_tag2.org`
2. **Large-scale PKM systems** - Knowledge bases with hundreds or thousands of org files
3. **Knowledge graph navigation** - Following `[[denote:TIMESTAMP]]` links across files
4. **Multi-silo management** - Searching across multiple Denote directories (~/org/, ~/claude-memory/, project docs/)
5. **Literate programming** - Executing org-mode code blocks with `:tangle` and `:results` options
6. **Denote metadata extraction** - Parsing file names and org-mode frontmatter
7. **Org-mode as primary collaboration format** - When both `.org` and `.md` exist for the same content, treat the org file as the primary, authoritative source and use markdown only as an export/share format.

## What is Denote?

Denote is a note-taking system created by Protesilaos Stavrou that uses:

- **File naming**: `YYYYMMDDTHHMMSS--title__tag1_tag2.org`
  - Timestamp = Unique identifier (Denote ID)
  - Title = Hyphen-separated (supports 한글/Korean)
  - Tags = Underscore-separated (English)

- **Org-mode frontmatter**:
  ```org
  #+title:      Title
  #+date:       [2025-10-21 Tue 10:53]
  #+filetags:   :tag1:tag2:tag3:
  #+identifier: 20251021T105353
  ```

- **Knowledge graph**: `[[denote:20251021T105353]]` links between files

- **Silo concept**: Separate directories for different knowledge domains

**Validated with 3,000+ org files in production PKM systems.**

## Knowledge Base Structure

> **Location**: `~/org/` (symlink to `~/sync/org/`)
> **Scale**: 3,100+ org files

### Main Directories

| Directory | Purpose | Scale |
|-----------|---------|-------|
| `bib/` | Bibliography notes (books, people, concepts) | 800+ files |
| `meta/` | Meta topics (programming, tools, workflows) | - |
| `notes/` | Personal notes and collections | - |
| `journal/` | Daily/weekly journals | - |
| `llmlog/` | AI/LLM conversation logs | - |

### Content Areas
Philosophy, Science, Programming, AI, Education, Literature, Spirituality, etc.

### How to Find Files

```bash
# Find by Denote ID
fd "20251217T105827" ~/org/ --type f

# Find by tag
fd "__llmlog" ~/org/ --type f

# Find by title keyword
fd "wsl2" ~/org/ --type f

# Search content
rg "검색어" ~/org/ -t org

# List recent files
ls -lt ~/org/llmlog/*.org | head -10
```

### Reference
For detailed knowledge base configuration, see `~/org/AGENTS.md`.

## Core Capabilities

> **Important**: Scripts are executed via Bash, not Python import. Use the CLI commands shown below.

> **Behavioral rule:** When reading org-mode documents, use structured approach: headings → history → focused sections. Only use grep as secondary tool.

### 1. Denote File Name Parsing (✅ Implemented)

**CLI Usage:**
```bash
# Parse a file (filename + frontmatter)
python3 ~/.claude/skills/denote-org/scripts/denote_parser.py ~/org/path/to/file.org

# Parse filename only
python3 ~/.claude/skills/denote-org/scripts/denote_parser.py --filename "20251021T105353--제목__tag.org"

# JSON output (recommended for parsing)
python3 ~/.claude/skills/denote-org/scripts/denote_parser.py --json ~/org/path/to/file.org
```

**Example output (--json):**
```json
{
  "path": "/home/user/org/llmlog/20241127T161109--제목__tag1_tag2.org",
  "filename_meta": {
    "timestamp": "20241127T161109",
    "title": "제목",
    "tags": ["tag1", "tag2"],
    "ext": "org"
  },
  "frontmatter": {
    "title": "제목",
    "identifier": "20241127T161109",
    "filetags": ["tag1", "tag2"],
    "date": "[2024-11-27 Wed 16:11]"
  }
}
```

**When to use:**
- Extracting Denote ID from filename
- Getting title and tags without manual parsing
- Processing multiple files in batch

### 2. Org Headings TOC (✅ Implemented)

**CLI Usage:**
```bash
# Extract table of contents from org file
python3 ~/.claude/skills/denote-org/scripts/org_headings_toc.py ~/org/path/to/file.org
```

**Example output:**
```
1	히스토리
1	로그 :LLMLOG:
2	[2024-11-27 Wed 16:11]
3	git 리포지토리에서...
```

Format: `LEVEL<TAB>TITLE` - easy to parse.

**When to use:**
- Quick structural overview of large org files
- Decide which sections to read in detail
- Before reading entire file into context

### 3. Finding Denote Files (🚧 Coming Soon)

```bash
# Not yet implemented - use glob/grep for now
fd --type f "20251021T105353" ~/org/
```

### 4. Denote Link Resolution (🚧 Coming Soon)

```bash
# Not yet implemented - use grep for now
grep -r "20251021T105353" ~/org/ --include="*.org" -l
```

**When to use:
- File contains `[[denote:...]]` links
- Need to open linked files
- Validate link integrity

### 5. Multi-Silo Management

#### 5.1 Preferred org-mode reading strategy

When working with org-mode files in this environment, follow this structured process instead of treating them as plain text:

1. **Identify org-context and silos**
   - If a file path matches any of the following, assume it is part of a Denote/org knowledge base and prefer org-mode behavior over ad-hoc grep:
     - `~/org/**`
     - `~/org/meta/**`
     - `~/org/bib/**`
     - `~/org/notes/**`
     - `~/org/llmlog/**`
     - `~/claude-memory/**`
     - `~/repos/gh/*/docs/**`
     - `~/repos/work/*/docs/**`
   - Treat these as **silos** with different roles:
     - `~/org/meta`  : meta-level models, config, and system design
     - `~/org/bib`   : bibliography and reference material
     - `~/org/notes` : long-form notes and thinking
     - `~/org/llmlog`: LLM conversation logs and experiments
     - project `docs/`: per-repo documentation tied to code

2. **Parse headings before content**
   - For large org files, do not read the entire file into context at once.
   - Instead, first extract only:
     - The top-level headings (`*`, `**`, `***`)
     - Their titles and hierarchy
   - Build a lightweight table of contents (TOC) and use that to decide which sections to inspect in detail.

3. **Locate and summarize history sections early**
   - If any heading matches common history patterns, treat it as a version/change log and inspect it early:
     - `* 히스토리`, `* HISTORY`, `* History`, `* 작업 로그`, `* Changelog`, or similar
   - Summarize:
     - How the document has evolved over time
     - Recent significant changes
     - Any explicit version markers or dates

4. **Drill down into relevant sections only**
   - After building the TOC and understanding history, only expand the specific headings that are relevant to the current task or user question.
   - Use grep/rg as a **secondary tool** to locate candidate sections, then return to structured org parsing for interpretation.

5. **Prefer org over markdown when both exist**
   - If both `X.org` and `X.md` exist for the same conceptual document:
     - Treat `X.org` as canonical for structure, metadata, and detailed reasoning.
     - Use `X.md` only as an export/share artifact or when the user explicitly prefers markdown.

This strategy exists to maximize collaboration on org documents as first-class knowledge artifacts, not just as flat text blobs.

**Common silos:**
- `~/org/` - Main knowledge base (3,000+ files)
- `~/claude-memory/` - AI memory system  
- `~/repos/*/docs/` - Project-specific docs

## Quick Start Examples

### Example 1: Parse Denote File

```bash
# Get metadata from a Denote file
python3 ~/.claude/skills/denote-org/scripts/denote_parser.py --json ~/org/llmlog/20241127T161109--제목__tag.org
```

### Example 2: Get TOC Before Reading Large File

```bash
# First, get structure overview
python3 ~/.claude/skills/denote-org/scripts/org_headings_toc.py ~/org/notes/large-file.org

# Output: 
# 1	섹션1
# 2	하위섹션
# Then read only relevant sections
```

### Example 3: Find Denote File by ID (using standard tools)

```bash
# Until denote_finder.py is implemented
fd "20251021T105353" ~/org/ --type f
# or
find ~/org -name "*20251021T105353*"
```

## Common Pitfalls

### Korean Encoding
**Problem:** 한글 filenames may cause encoding issues
**Solution:** All scripts use UTF-8

### Large Files
**Problem:** Reading entire large org file exceeds context
**Solution:** Use `org_headings_toc.py` first, then read specific sections

## Available Scripts

```
scripts/
├── denote_parser.py       ✅ Parse filenames and frontmatter (--json)
├── org_headings_toc.py    ✅ Extract TOC from org file
├── denote_finder.py       🚧 Coming soon
├── denote_linker.py       🚧 Coming soon
└── denote_rename.py       🚧 Coming soon
```

## See Also

- [Denote Package](https://protesilaos.com/emacs/denote) by Protesilaos Stavrou
- [Org-mode](https://orgmode.org/) documentation
