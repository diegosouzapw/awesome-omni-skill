---
name: notion-uploader-downloader
version: "2.1.0"
description: Bidirectional sync between Markdown and Notion. Upload .md files with images to Notion pages/databases, append to existing pages, or download Notion content back to markdown. Supports rich formatting, tables, code blocks, GitHub-flavored markdown, and recursive page hierarchy downloads with YAML frontmatter for round-trip sync.

allowed-tools:
  - "Bash(python3 scripts/notion_upload.py*)"
  - "Bash(python3 scripts/notion_download.py*)"
  - "Bash(python3 scripts/notion_download_recursive.py*)"
  - "Read(*/references/*)"
  - "Read(*/guides/*)"

metadata:
  pda_version: "1.0"
  tier1_budget: 150
  tier2_budget: 600
  tier3_budget_per_guide: 1500
  max_request_budget: 10000
  requires: ["python3>=3.8", "pip"]
  config_required: ["NOTION_TOKEN", "NOTION_PARENT_PAGE"]
---

# Notion Uploader/Downloader - Orchestrator (Tier 2)

**PDA Tier**: 2 (Decision Tree & Routing Logic)
**Token Budget**: ~600 tokens
**Purpose**: Classify intent and route to appropriate Tier 3 workflow guides

## Intent Classification Decision Tree

Classify user intent into ONE category and load the corresponding Tier 3 guide:

### 1. UPLOAD Intent 📤
**Triggers**: "upload", "publish", "save to notion", "create page", mentions .md file path
**User Goal**: Send markdown file TO Notion
**Route**: `guides/workflows/upload-workflow.md` (+1,250 tokens)
**Validation**: Markdown file path must be provided

### 2. DOWNLOAD Intent 📥
**Triggers**: "download", "export", "backup", Notion URL provided, "get from notion"
**User Goal**: Extract content FROM Notion (single page)
**Route**: `guides/workflows/download-workflow.md` (+850 tokens)
**Validation**: Page ID or Notion URL must be provided

### 2b. RECURSIVE DOWNLOAD Intent 📥🔄
**Triggers**: "download all", "recursive", "all child pages", "entire hierarchy", "whole section", "backup everything", "download tree"
**User Goal**: Extract a page AND all its child pages recursively
**Route**: `guides/workflows/recursive-download-workflow.md` (+1,100 tokens)
**Validation**: Page ID or Notion URL must be provided
**Output**: Folder structure with mapping.json, YAML frontmatter in each file

### 3. UPDATE/APPEND Intent ➕
**Triggers**: "update", "append", "add to", mentions "page-id" with .md file
**User Goal**: Add content to EXISTING Notion page
**Route**: `guides/workflows/update-workflow.md` (+1,100 tokens)
**Validation**: Both markdown file AND page ID required

### 4. CONFIGURATION Intent ⚙️
**Triggers**: "setup", "configure", "install", "token not found", "how do I set up"
**User Goal**: Initial setup or configuration help
**Route**: `guides/setup/configuration-guide.md` (+700 tokens)
**Auto-trigger**: If NOTION_TOKEN or NOTION_PARENT_PAGE not found

### 5. TROUBLESHOOTING Intent 🔧
**Triggers**: "error", "failed", "not working", "404", "object_not_found", "broken"
**User Goal**: Fix an error or issue
**Route**: `guides/troubleshooting/error-resolution.md` (+950 tokens)
**Auto-trigger**: If script execution fails

### 6. TECHNICAL REFERENCE Intent 📚
**Triggers**: "how does", "what elements", "supported features", "mappings", "can it handle"
**User Goal**: Understand capabilities or technical details
**Route**: Load specific reference based on question:
- Element mappings → `references/MAPPINGS.md` (+515 tokens)
- Quick commands → `references/QUICK_START.md` (+320 tokens)
- Technical details → `references/IMPLEMENTATION_SUMMARY.md` (+1,090 tokens)

## Resource Catalog (Tier 3 - Load On-Demand ONLY)

**Loading Policy**: NEVER load proactively. Load ONLY when routed above.

### Workflow Guides (`guides/workflows/`)
| Guide | Tokens | When to Load |
|-------|--------|--------------|
| `upload-workflow.md` | ~1,250 | Upload Intent (#1) |
| `download-workflow.md` | ~850 | Download Intent (#2) |
| `recursive-download-workflow.md` | ~1,100 | Recursive Download Intent (#2b) |
| `update-workflow.md` | ~1,100 | Update Intent (#3) |

### Setup Guides (`guides/setup/`)
| Guide | Tokens | When to Load |
|-------|--------|--------------|
| `configuration-guide.md` | ~700 | Configuration Intent (#4) or config error |
| `first-time-setup.md` | ~600 | User explicitly asks for setup walkthrough |

### Troubleshooting (`guides/troubleshooting/`)
| Guide | Tokens | When to Load |
|-------|--------|--------------|
| `error-resolution.md` | ~950 | Troubleshooting Intent (#5) or script failure |

### Reference Docs (`references/`)
| Guide | Tokens | When to Load |
|-------|--------|--------------|
| `MAPPINGS.md` | ~515 | Questions about supported elements |
| `QUICK_START.md` | ~320 | Questions about command syntax |
| `IMPLEMENTATION_SUMMARY.md` | ~1,090 | Deep technical questions |

## Token Budget Management

**Current Request Status**:
- ✅ Tier 1 (Metadata): ~150 tokens (loaded)
- ✅ Tier 2 (This Orchestrator): ~600 tokens (loaded)
- ⏳ Tier 3: 0 tokens (awaiting intent classification)
- **Remaining Budget**: 9,250 tokens for Tier 3 resources

**Loading Rules**:
1. Load EXACTLY ONE workflow guide for current intent
2. Load reference guides ONLY when workflow explicitly requires OR user asks technical question
3. Load troubleshooting ONLY on errors or explicit user request
4. Track cumulative tokens after each load
5. Warn if total exceeds 8,000 tokens

## Skill Integration Points

**PlantUML Skill** (`/Users/richardhightower/.claude/skills/plantuml`):
- **When**: Markdown contains ```puml or ```plantuml code blocks
- **Action**: MUST invoke PlantUML skill FIRST before upload
- **Script**: `python scripts/process_markdown_puml.py <file.md> --format png`
- **Output**: Use transformed `*_with_images.md` for upload
- **Reason**: Notion doesn't natively render PlantUML

## Error Handling & Fallback Logic

**Missing NOTION_TOKEN**:
→ Route to Configuration Intent (#4)

**Missing NOTION_PARENT_PAGE** (and no --parent-id/--database-id/--page-id):
→ Route to Configuration Intent (#4)

**Script Execution Fails**:
→ Route to Troubleshooting Intent (#5)

**Unknown/Ambiguous Intent**:
→ Use AskUserQuestion tool to clarify, then re-classify

**Resource Not Found** (guide file missing):
→ Apologize, explain PDA structure may be incomplete, ask user what they need

## Quick Reference (Non-Expandable)

**Core Scripts**:
- `scripts/notion_upload.py` - Upload markdown to Notion
- `scripts/notion_download.py` - Download single Notion page to markdown
- `scripts/notion_download_recursive.py` - Recursively download page + children
- `scripts/notion_utils.py` - Shared utilities (token/parent discovery)

**Configuration**:
- File: `.env.notion` (NOTION_TOKEN=..., NOTION_PARENT_PAGE=...)
- Search Path: Current dir → Parent dirs → Environment variables

**Python Requirements**:
- Version: Python 3.8+
- Install: `pip install -r scripts/requirements.txt`
- Dependencies: requests, python-dotenv

---

## PDA Compliance Checklist

- ✅ Decision tree with 6 clear intents
- ✅ Explicit routing to Tier 3 resources
- ✅ Token budgets documented for all resources
- ✅ Lazy loading policy (no proactive loads)
- ✅ No embedded workflows or comprehensive guides
- ✅ Error handling with fallback routing
- ✅ Integration points identified (PlantUML)
- ✅ Resource catalog with load conditions
- ✅ Token tracking instructions
- ✅ Under 100 lines (target: ~95 lines)
