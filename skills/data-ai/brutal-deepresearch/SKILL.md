---
name: brutal-deepresearch
description: Structured deep research pipeline with confirmation gates and resume support. Generates outline, launches parallel research agents, produces validated JSON results and markdown report.
user-invocable: true
allowed-tools: Bash(ls:*), Bash(date:*), Bash(mkdir:*), Bash(rm:*), Bash(python:*), Bash(sleep:*), Bash(wc:*), Read, Write, Edit, Glob, Grep, WebSearch, Task, AskUserQuestion
---

Structured deep research pipeline with confirmation gates and resume support. Generates research outline from model knowledge + web search, launches parallel research agents, produces validated JSON results per item, and generates a markdown report. Supports resuming interrupted sessions.

Agent assumptions (applies to all agents and subagents):
- All tools are functional and will work without error. Do not test tools or make exploratory calls.
- Only call a tool if it is required to complete the task. Every tool call should have a clear purpose.

---

# Brutal Deep Research Process

## Step 0: Load Context & Session Setup

### 0.1 Load Project Target Context

Check for `TARGET.md` in the project root directory.

- If `TARGET.md` exists, read it in full and treat it as required context.
- Do not proceed to Step 1 until this check/read has been completed.

### 0.2 Get Current Date

```bash
date +%Y-%m-%d
```

### 0.3 Determine Next DR Number

Find the highest existing DR number:

```bash
ls workspace/research/ 2>/dev/null | grep -oE '[0-9]{4}' | sort -rn | head -1
```

New DR number = highest + 1. If no research sessions exist, start at `0001`.

### 0.4 Generate Session Slug

Generate a slug from the research topic:
- Lowercase
- Replace spaces and special characters with hyphens
- Max 40 characters
- Remove trailing hyphens

Session directory: `workspace/research/DR-<NNNN>-<slug>/`

**Do not create the directory yet.** Wait until Gate 2 confirmation.

---

## Step 0.5: Resume Mode (Conditional)

**Trigger**: User args contain the word "resume" and a path to an existing session directory.

Example invocation: `/brutal-deepresearch resume research workspace/research/DR-0001-ai-coding`

If resume mode is NOT detected, skip to Step 1.

### 0.5.1 Read Existing Session

1. Read `outline.yaml` from the given session path to get items list
2. Read `fields.yaml` from the given session path to get field definitions
3. Read `progress.yaml` (if it exists) for previous execution context

### 0.5.2 Check Completed Results

```bash
ls <session_path>/results/*.json 2>/dev/null
ls <session_path>/results/*.started 2>/dev/null
```

Determine item status:
- `.json` exists → **completed** (skip this item)
- `.started` exists but no `.json` → **interrupted** (re-research this item)
- Neither exists → **never started** (research this item)

### 0.5.3 Calculate Remaining Items

Compare items in `outline.yaml` against completed results:
- Log which items are completed and will be skipped
- Log which items were interrupted and will be re-researched
- Log which items never started and will be researched

### 0.5.4 Branch to Execution or Report

- **If remaining items exist**: Skip directly to Step 6 (Execute Deep Research), launching agents only for remaining items
- **If all items are complete**: Report "All items already completed" and skip directly to Step 7 (Report Configuration)

**Resume mode skips Steps 1-4 entirely** — the outline and fields are already confirmed from the previous session.

---

## Step 1: Generate Initial Framework

Based on the user's research topic, use model knowledge to generate:

1. **Items List**: The main research objects/items in this domain. Each item should have:
   - `name`: Item name
   - `category`: Classification (if applicable)
   - `description`: Brief description of why this item is relevant

2. **Field Framework**: Suggested research field categories and fields per category. Each field should have:
   - `name`: Field name (snake_case)
   - `description`: What this field captures
   - `detail_level`: One of `brief`, `moderate`, or `detailed`

Present the framework to the user in a readable format.

---

## Step 2 - GATE 1: Confirm Initial Framework

**This is a hard gate. Do not proceed past this step without explicit user confirmation.**

Present:
- The items list with names, categories, and descriptions
- The field framework organized by category

Use AskUserQuestion to ask:
- "Are these items and fields correct? Add/remove anything?"

**Hard gate**: Do not proceed until user confirms. User can request additions or removals here.

---

## Step 3: Web Search Supplement

### 3.1 Get Time Range

Use AskUserQuestion to ask for time range:
- Last 6 months
- Since 2024
- Since 2025
- Unlimited

### 3.2 Launch Web Search Agent

Launch 1 web-search-agent (background) using the Task tool with `model: sonnet` and `max_turns: 20`.

**Parameter Retrieval**:
- `{topic}`: User's research topic
- `{YYYY-MM-DD}`: Current date from Step 0.2
- `{step1_output}`: Complete output from Step 1 (items list + field framework)
- `{time_range}`: User-specified time range

**Hard Constraint**: The following prompt must be strictly reproduced, only replacing variables in `{xxx}`. Do not modify structure or wording.

**Prompt Template**:

```
You are an elite internet researcher. Your task is to supplement an existing research framework with missing items and recommended fields.

## Research Methodology

Before searching, determine which search strategies apply to this topic. Use the appropriate strategies from the Search Strategy Reference below.

Get today's date first:
date +%Y-%m-%d

Generate 5-10 different search query variations to maximize coverage:
- Include technical terms, product names, and common variations
- Think of how different people might describe the same topic
- Use exact phrases in quotes for specific names
- Include version numbers and dates when relevant

## Information Gathering Standards
- Read beyond the first few results - valuable information is often buried
- Look for patterns across different sources
- Pay attention to dates to ensure relevance
- Note different approaches and their trade-offs
- Identify authoritative sources and experienced contributors
- Check for updated information or superseded approaches
- Verify across multiple sources when possible

## Task
Research topic: {topic}
Current date: {YYYY-MM-DD}

Based on the following initial framework, supplement latest items and recommended research fields.

## Existing Framework
{step1_output}

## Goals
1. Verify if existing items are missing important objects
2. Supplement items based on missing objects
3. Continue searching for {topic} related items within {time_range} and supplement
4. Supplement new fields

## Output Requirements
Return structured results directly (do not write files):

### Supplementary Items
- item_name: Brief explanation (why it should be added)
...

### Recommended Supplementary Fields
- field_name: Field description (why this dimension is needed)
...

### Sources
- [Source1](url1)
- [Source2](url2)
```

**One-shot Example** (assuming researching AI Coding History):
```
## Task
Research topic: AI Coding History
Current date: 2025-12-30

Based on the following initial framework, supplement latest items and recommended research fields.

## Existing Framework
### Items List
1. GitHub Copilot: Developed by Microsoft/GitHub, first mainstream AI coding assistant
2. Cursor: AI-first IDE, based on VSCode
...

### Field Framework
- Basic Info: name, release_date, company
- Technical Features: underlying_model, context_window
...

## Goals
1. Verify if existing items are missing important objects
2. Supplement items based on missing objects
3. Continue searching for AI Coding History related items within since 2024 and supplement
4. Supplement new fields

## Output Requirements
Return structured results directly (do not write files):

### Supplementary Items
- item_name: Brief explanation (why it should be added)
...

### Recommended Supplementary Fields
- field_name: Field description (why this dimension is needed)
...

### Sources
- [Source1](url1)
- [Source2](url2)
```

### 3.3 Merge Findings

After the web search agent completes, merge its findings with the initial framework:
- Add supplementary items to the items list (avoid duplicates)
- Add recommended fields to the field framework
- Note sources for traceability

---

## Step 4 - GATE 2: Confirm Final Outline

**This is a hard gate. Do not proceed past this step without explicit user confirmation.**

Present the merged outline:
- Complete items list (original + web search additions, clearly marked)
- Complete field framework (original + web search additions, clearly marked)

Use AskUserQuestion to confirm the outline is correct.

### Add-Items/Add-Fields Loop

User can say "add X item" or "add Y field" at this gate. If they do:
1. Add the requested item/field to the framework
2. Re-present the updated framework
3. Ask for confirmation again

**Repeat until user explicitly confirms.** Do not generate files until confirmed.

### 4.1 Create Session Directory and Write Files

After confirmation:

```bash
mkdir -p workspace/research/DR-<NNNN>-<slug>/results
```

Write `outline.yaml`:

```yaml
topic: "<research topic>"
session: "DR-<NNNN>-<slug>"
created: "<YYYY-MM-DD>"
items:
  - name: "<item name>"
    category: "<category>"
    description: "<description>"
  # ... more items
output_dir: "./results"
```

Write `fields.yaml`:

```yaml
categories:
  <category_name>:
    fields:
      - name: "<field_name>"
        description: "<field description>"
        detail_level: "<brief|moderate|detailed>"
      # ... more fields
  # ... more categories
```

---

## Step 5: Deep Research - Preparation

### 5.1 Read Outline

Read `workspace/research/DR-<NNNN>-<slug>/outline.yaml` to get items list.

### 5.2 Resume Check

Check for completed and in-progress results in `results/`:

```bash
ls workspace/research/DR-<NNNN>-<slug>/results/*.json 2>/dev/null
ls workspace/research/DR-<NNNN>-<slug>/results/*.started 2>/dev/null
```

Determine item status:
- `.json` exists → **completed** (skip this item)
- `.started` exists but no `.json` → **interrupted** (re-research this item)
- Neither exists → **never started** (research this item)

Log which items are being resumed vs skipped. Update `progress.yaml` (if it exists) with the current state before proceeding.

### 5.3 Prepare Execution Plan

Calculate:
- Total remaining items (after subtracting completed and in-progress items)
- Display which items will be researched and which are being skipped

---

## Step 6: Execute Deep Research

### 6.1 Launch Research Agents

Launch remaining agents using the Task tool with `model: sonnet`, `run_in_background: true`, and `max_turns: 25`.

**Batching strategy** based on remaining item count:
- **10 or fewer items**: Launch ALL agents in a single parallel batch.
- **More than 10 items**: Split into batches of 10. Launch each batch in parallel, wait for the batch to complete (using filesystem polling per 6.4), then launch the next batch. No inter-batch user approval needed — batching is automatic.

Each agent researches one item and outputs JSON for that item.

**Agent Prompt Template** (per item):

**Hard Constraint**: The following prompt must be strictly reproduced, only replacing variables in `{xxx}`. Do not modify structure or wording.

```
You are an elite internet researcher specializing in finding relevant information across diverse online sources. Your expertise lies in creative search strategies, thorough investigation, and comprehensive compilation of findings.

## Progress Tracking

Before starting research, write a marker file to signal that this agent has started:
Write an empty file to {started_path}

After self-validation passes and the JSON result is confirmed correct, delete the marker file:
rm {started_path}

## Research Methodology

Get today's date first:
date +%Y-%m-%d

Generate 5-10 different search query variations to maximize coverage:
- Include technical terms, product names, and common variations
- Think of how different people might describe the same topic
- Use exact phrases in quotes for specific names
- Include version numbers and dates when relevant

### Search Strategy Reference

Use the following search strategies based on what is relevant to the research topic:

**GitHub/Debug Strategy** (for software, tools, technical projects):
- Search GitHub Issues (open and closed) for known bugs and workarounds
- Search for exact error messages in quotes
- Look for issue templates that match the problem pattern
- Check closed issues for resolution patterns
- Identify version-specific issues

**General Web Strategy** (for broad information gathering):
Sources: Reddit, official documentation, blog posts, Hacker News, Dev.to, Medium, Discord, X/Twitter
- Look for official recommendations first
- Cross-reference with community consensus
- Find examples from production use
- Identify anti-patterns and common pitfalls
- Note evolving best practices
- Create structured comparisons with clear criteria
- Find real-world usage examples and case studies
- Look for performance benchmarks and user experiences

**Academic Papers Strategy** (for research, algorithms, scientific topics):
Sources: Google Scholar, arXiv, Hugging Face Papers, bioRxiv, ResearchGate, Semantic Scholar, ACM Digital Library, IEEE Xplore
- Use Google Scholar as primary source with advanced search operators
- Search by author names, paper titles, DOI numbers
- Include year ranges to find seminal works and recent publications
- Look for related papers and citation patterns
- Search for preprints on arXiv and bioRxiv
- Track citation networks to understand research evolution

**Stack Overflow Strategy** (for programming, APIs, implementation):
Sources: Stack Overflow, Stack Exchange, technical forums
- Search for exact error messages and API names
- Look for accepted answers and highly-voted alternatives
- Check for version-specific solutions

## Information Gathering Standards
- Read beyond the first few results - valuable information is often buried
- Look for patterns in solutions across different sources
- Pay attention to dates to ensure relevance (note if information is outdated)
- Note different approaches and their trade-offs
- Identify authoritative sources and experienced contributors
- Verify information across multiple sources when possible
- Clearly indicate when information is speculative or unverified

## Task
Research {item_related_info}, output structured JSON to {output_path}

## Field Definitions
Read {fields_path} to get all field definitions

## Output Requirements
1. Output JSON according to fields defined in fields.yaml
2. Mark uncertain field values with [uncertain]
3. Add uncertain array at the end of JSON, listing all uncertain field names
4. All field values must be in English

## Self-Validation
After writing the JSON file, read it back and verify:
1. Every field defined in fields.yaml has a corresponding entry in the JSON
2. The JSON is valid (properly formatted)
3. All uncertain fields are listed in the uncertain array
If validation fails, fix the JSON and re-write it.

## Output Path
{output_path}
```

**One-shot Example** (assuming researching GitHub Copilot):
```
## Progress Tracking

Before starting research, write a marker file to signal that this agent has started:
Write an empty file to /home/user/workspace/research/DR-0001-ai-coding/results/GitHub_Copilot.started

After self-validation passes and the JSON result is confirmed correct, delete the marker file:
rm /home/user/workspace/research/DR-0001-ai-coding/results/GitHub_Copilot.started

## Task
Research name: GitHub Copilot
category: International Product
description: Developed by Microsoft/GitHub, first mainstream AI coding assistant, ~40% market share, output structured JSON to /home/user/workspace/research/DR-0001-ai-coding/results/GitHub_Copilot.json

## Field Definitions
Read /home/user/workspace/research/DR-0001-ai-coding/fields.yaml to get all field definitions

## Output Requirements
1. Output JSON according to fields defined in fields.yaml
2. Mark uncertain field values with [uncertain]
3. Add uncertain array at the end of JSON, listing all uncertain field names
4. All field values must be in English

## Self-Validation
After writing the JSON file, read it back and verify:
1. Every field defined in fields.yaml has a corresponding entry in the JSON
2. The JSON is valid (properly formatted)
3. All uncertain fields are listed in the uncertain array
If validation fails, fix the JSON and re-write it.

## Output Path
/home/user/workspace/research/DR-0001-ai-coding/results/GitHub_Copilot.json
```

### 6.2 Parameter Construction

For each item being researched:
- `{item_related_info}`: The item's complete YAML content (name + category + description)
- `{output_path}`: Absolute path to `workspace/research/DR-<NNNN>-<slug>/results/<item_name_slug>.json`
  - Slugify: replace spaces with `_`, remove special characters
- `{fields_path}`: Absolute path to `workspace/research/DR-<NNNN>-<slug>/fields.yaml`
- `{started_path}`: Absolute path to `workspace/research/DR-<NNNN>-<slug>/results/<item_name_slug>.started`

### 6.3 Write progress.yaml

Immediately after launching all agents, write `progress.yaml` in the session directory:

```yaml
status: in_progress
started: "<YYYY-MM-DD HH:MM>"
total_items: <N>
items:
  - name: "<Item Name>"
    slug: "<Item_Name>"
    status: pending
  # ... all items being researched
```

Items that were already completed (skipped) should not be listed — only items that agents were launched for.

### 6.4 Monitor Progress (Filesystem-Based)

**CRITICAL**: Do NOT use TaskOutput to read agent results. Agent outputs are large (extensive web search transcripts) and reading them into the orchestrator context will cause context window exhaustion. All research results are already persisted to disk as JSON files — the orchestrator only needs to check file existence.

**Polling loop** — repeat until all items are resolved:

1. Check completion status via filesystem:
   ```bash
   ls <session_path>/results/*.json 2>/dev/null | wc -l
   ls <session_path>/results/*.started 2>/dev/null | wc -l
   ```
2. Calculate: `completed` = count of `.json` files, `in_progress` = count of `.started` files without matching `.json`, `remaining` = total - completed
3. Display progress: "Progress: X/Y items completed, Z still running."
4. If `in_progress > 0`, wait ~30 seconds (use `sleep 30` via Bash) then poll again
5. If `in_progress == 0` (all agents have finished — either produced `.json` or exited), exit the loop

**After polling loop completes:**
1. Update `progress.yaml`:
   - Set each item's status to `completed` if its `.json` file exists, or `failed` if only `.started` exists or neither exists
   - Set overall `status` to `completed` if all items done, or `partial` if some are missing
2. Report final status: "All agents complete. X/Y items researched successfully."
3. If any items failed, list them and suggest using resume mode to retry

---

## Step 7: Report Configuration

### 7.1 Scan Summary Fields

Read all completed JSON results and identify fields suitable for TOC display:
- Numeric fields (stars, scores, citations)
- Short metric fields (dates, versions, ratings)
- Fields that appear across most/all items

### 7.2 Present Options

Present a dynamic options list based on actual fields found in the JSON results.

---

## Step 7.5 - GATE 3: Confirm Report Config

**This is a hard gate. Do not proceed past this step without explicit user confirmation.**

Use AskUserQuestion to ask:
- Which summary fields to display in the TOC alongside item names?
- Present the available fields as options

**Hard gate**: Do not generate report until user confirms field selection.

---

## Step 8: Generate Report

### 8.1 Generate Report Script

Generate `generate_report.py` in the session directory.

The script must handle:

**1. JSON Structure Compatibility**

Support two JSON structures:
- Flat structure: Fields directly at top level `{"name": "xxx", "release_date": "xxx"}`
- Nested structure: Fields in category sub-dict `{"basic_info": {"name": "xxx"}, "technical_features": {...}}`

Field lookup order: Top level -> category mapping key -> Traverse all nested dicts

**2. Category Mapping**

Map between fields.yaml category names and JSON keys:
```python
CATEGORY_MAPPING = {
    "Basic Info": ["basic_info", "Basic Info"],
    "Technical Features": ["technical_features", "technical_characteristics", "Technical Features"],
    "Performance Metrics": ["performance_metrics", "performance", "Performance Metrics"],
    "Milestone Significance": ["milestone_significance", "milestones", "Milestone Significance"],
    "Business Info": ["business_info", "commercial_info", "Business Info"],
    "Competition & Ecosystem": ["competition_ecosystem", "competition", "Competition & Ecosystem"],
    "History": ["history", "History"],
    "Market Positioning": ["market_positioning", "market", "Market Positioning"],
}
```

**3. Complex Value Formatting**
- List of dicts (e.g., key_events, funding_history): Format each dict as one line, separate kv with ` | `
- Normal list: Short lists joined with comma, long lists displayed with line breaks
- Nested dict: Recursive formatting, display with semicolon or line breaks
- Long text strings (over 100 chars): Add line breaks `<br>` or use blockquote format for readability

**4. Extra Fields Collection**

Collect fields that exist in JSON but not defined in fields.yaml, put in "Other Info" category. Filter out:
- Internal fields: `_source_file`, `uncertain`
- Nested structure top-level keys matching category names
- `uncertain` array: Display each field name on separate line

**5. Uncertain Value Skipping**

Skip conditions:
- Field value contains `[uncertain]` string
- Field name is in `uncertain` array
- Field value is None or empty string

**6. Report Format**

The generated report.md must have:
- **Table of Contents**: Every item with number, name (anchor link), and user-selected summary fields
  - Example: `1. [GitHub Copilot](#github-copilot) - Stars: 10k | Score: 85%`
- **Detailed Sections**: One section per item, organized by field category
  - Each category as a subsection heading
  - Each field as a labeled entry

### 8.2 Execute Script

```bash
python workspace/research/DR-<NNNN>-<slug>/generate_report.py
```

---

## Step 9: Present Summary

Present completion stats:

```
## Research Complete: <topic>

**Session**: workspace/research/DR-<NNNN>-<slug>/
**Items Researched**: <count>

### Output Files
- outline.yaml - Research outline and items list
- fields.yaml - Field definitions
- progress.yaml - Execution progress tracking
- results/ - JSON results per item (<count> files)
- generate_report.py - Report generation script
- report.md - Final markdown report

### Items with High Uncertainty
- <item name>: <count> uncertain fields
- ...
```

---

# Search Strategy Reference

These strategies are used by research agents to systematically explore information sources.

## GitHub Debug Strategy

**Trigger**: Software bugs, error debugging, issue lookup, version-specific problems

**Sources**: GitHub Issues (open and closed)

**Query Strategy**:
- Search for exact error messages in quotes
- Look for issue templates that match the problem pattern
- Find workarounds, not just explanations
- Check if it's a known bug with existing patches or PRs
- Look for similar issues even if not exact matches
- Identify if the issue is version-specific
- Search for both the library name + error and more general descriptions
- Check closed issues for resolution patterns

## General Web Strategy

**Trigger**: General information, news, product comparisons, best practices

**Sources**:
- Reddit (r/programming, r/webdev, r/javascript, topic-specific subreddits) - real-world experiences
- Official documentation and changelogs - authoritative information
- Blog posts and tutorials - detailed explanations
- Hacker News - high-quality technical discourse
- Dev.to - developer community with technical articles
- Medium - technical blog platform with in-depth articles
- Discord - official discussion channels for open source projects
- X/Twitter - technical announcements and discussions from developers

**Query Strategy**:
- Look for official recommendations first
- Cross-reference with community consensus
- Find examples from production codebases
- Identify anti-patterns and common pitfalls
- Note evolving best practices and deprecated approaches
- Create structured comparisons with clear criteria
- Find real-world usage examples and case studies
- Look for performance benchmarks and user experiences
- Identify trade-offs and decision factors
- Consider scalability, maintenance, and learning curve

## Academic Papers Strategy

**Trigger**: Paper search, academic research, algorithm fundamentals

**Sources**:
- Google Scholar (scholar.google.com) - comprehensive academic search engine
- arXiv (arxiv.org) - preprints in physics, math, CS, and related fields
- Hugging Face Papers (huggingface.co/papers) - trending ML/AI papers with community upvotes
- bioRxiv (biorxiv.org) - preprints in biology and life sciences
- ResearchGate (researchgate.net) - academic social network with papers and author profiles
- Semantic Scholar (semanticscholar.org) - AI-powered academic search
- ACM Digital Library and IEEE Xplore - CS and engineering papers

**Query Strategy**:
- Use Google Scholar as primary source with advanced search operators
- Search by author names, paper titles, DOI numbers, institutions, and publication years
- Use quotation marks for exact titles and author name combinations
- Include year ranges to find seminal works and recent publications
- Look for related papers and citation patterns to identify seminal works
- Search for preprints on arXiv, bioRxiv, and institutional repositories
- Check author profiles and ResearchGate for publications and PDFs
- Identify open-access versions and legal paper download sources
- Track citation networks to understand research evolution
- Note impact factors, h-index, and citation counts for relevance assessment
- Search for conference proceedings, journals, and workshop papers

## Stack Overflow Strategy

**Trigger**: Programming Q&A, code implementation, API usage

**Sources**:
- Stack Overflow and other Stack Exchange sites - technical Q&A
- Technical forums and discussion boards - community wisdom

**Query Strategy**:
- Search for exact error messages and API signatures
- Look for accepted answers and highly-voted alternatives
- Check for version-specific solutions and deprecated approaches
- Cross-reference with official documentation
- Note common pitfalls mentioned in answers

---

# Mindset

You are not here to rubber-stamp research topics. You are here to ensure comprehensive, structured, high-quality research output. Every gap you miss is a blind spot in the final report.

Be direct. Be thorough. Be systematic.

Do not:
- Accept vague topics without clarification
- Skip confirmation gates
- Launch research agents before user confirms the outline
- Generate files before Gate 2 confirmation
- Generate reports before Gate 3 confirmation
- Use TaskOutput to read research agent results (causes context exhaustion — use filesystem polling instead)
- Launch more than 10 agents simultaneously (use automatic batching for larger sets)

Do:
- Generate comprehensive initial frameworks from domain knowledge
- Supplement with web search to catch missing items
- Present clear, structured summaries at each gate
- Support add-items/add-fields within Gate 2
- Track uncertain values across all results
- Generate reproducible report scripts
- Track progress in progress.yaml throughout execution
- Write .started markers before each agent begins research
- Support resume from any interrupted state
