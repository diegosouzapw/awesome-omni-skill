---
name: Enhance Meeting Minutes
description: This skill should be used when enhancing FHIR meeting minutes by synthesizing transcript discussion into Confluence pages, capturing reasoning and trade-offs with XML DOM manipulation
version: 1.0.0
dependencies: python>=3.8, lxml>=4.9.0
license: MIT
---

# Enhance Meeting Minutes

## Overview

This skill enhances FHIR Infrastructure meeting minutes by adding detailed discussion context from transcripts to Confluence pages. The process emphasizes content synthesis over transcription and XML DOM manipulation over string operations.

## When to Use This Skill

This skill should be used when:
- Enhancing meeting minutes with transcript discussion
- Adding discussion details to Confluence pages
- Updating minutes with what was actually discussed
- Creating detailed minutes from meeting transcript

## Required Inputs

This skill requires:

1. **Transcript file**: Plain text transcript of the meeting
2. **Confluence page URL**: URL to the meeting minutes page to enhance
3. **`.env` file**: Contains API credentials for Confluence (and optionally JIRA)

## Core Principles

### Source of Truth Hierarchy
1. **Transcript**: What was actually discussed today (absolute source of truth)
2. **Original Minutes**: Structure and metadata only (attendees, JIRA widgets)
3. **JIRA Issues**: Background context ONLY (never claim these were discussed today)

### Content Philosophy

To create effective enhanced minutes:
- Create high-level narratives capturing WHY decisions were made
- Explain trade-offs and alternatives considered
- Show areas of agreement, disagreement, or things left open
- Focus on reasoning and substance

To avoid poor quality output:
- Avoid transcribing conversations verbatim or play-by-play
- Avoid quoting extensively (synthesize instead)
- Avoid using JIRA content as if discussed today
- Avoid writing shallow summaries without reasoning

### Technical Philosophy

To build HTML correctly:
- Use XML DOM manipulation (lxml) for ALL HTML construction
- Avoid regex or string operations to build/modify HTML

**ONE EXCEPTION**: Post-process lxml output for XHTML self-closing tags (br/hr/col). This is the only acceptable string manipulation.

## Complete Workflow

### Phase 1: Gather Context (30-60 minutes)

#### Download Confluence Page

To download the current Confluence page, use `scripts/confluence_api.py`:

```bash
# From URL (automatically extracts page ID)
python scripts/confluence_api.py "https://confluence.hl7.org/display/FHIR/Minutes"

# From page ID directly
python scripts/confluence_api.py 123456
```

**Output**: `current_page.json` with HTML and version number

**API endpoint**: `GET /rest/api/content/{pageId}?expand=body.storage,version`

#### Download JIRA Issues for Background

To gather background context on JIRA issues, use `scripts/jira_api.py`:

```bash
# Download issue JSON
python scripts/jira_api.py FHIR-12345

# Create markdown background summary
python scripts/jira_api.py FHIR-12345 --background
```

**Output**:
- `FHIR-12345.json` - Full issue data
- `FHIR-12345-background.md` - Background summary

**API endpoint**: `GET /rest/api/2/issue/{issueKey}` (works unauthenticated for HL7)

**CRITICAL**: This provides background context ONLY. Never claim JIRA content was discussed today.

#### Read Transcript Carefully

To understand what actually happened in the meeting:

1. Spend 20-40 minutes reading the transcript carefully
2. Note which issues were discussed
3. Identify where substantive discussion occurred
4. Mark key moments of decision or debate
5. Record timestamps for reference

**DO NOT SKIP THIS STEP**. This is where understanding of actual meeting discussion occurs.

### Phase 2: Extract JIRA Widgets (5 minutes)

To preserve JIRA widgets from the original page, use `scripts/extract_jira_widgets.py`:

```bash
python scripts/extract_jira_widgets.py current_page.json
```

**Output**: `jira_widgets.json` - Mapping of issue keys to widget HTML

**What this does**:
- Parses original HTML with lxml
- Finds `<ac:structured-macro ac:name="jira">` elements
- Serializes widgets as HTML strings for re-insertion

**In Python**:
```python
from extract_jira_widgets import extract_jira_widgets, get_widget_element

widgets = extract_jira_widgets(html_content)
widget_elem = get_widget_element(widgets['FHIR-12345'])
```

### Phase 3: Analyze Transcript (60-120 minutes) ⭐ MOST IMPORTANT

To extract substance from the transcript, analyze each issue discussed by identifying:

1. **What sparked the discussion?** - Context and trigger
2. **Key perspectives expressed** - Different viewpoints
3. **Trade-offs and alternatives** - Options considered with pros/cons
4. **Reasoning behind decision** - WHY was it decided this way?
5. **Areas of agreement** - What was uncontroversial
6. **Areas of disagreement** - Concerns or debate
7. **Things left open** - Deferred decisions or follow-ups
8. **Decision or outcome** - What was actually decided

**Time investment**: Allocate 15-30 minutes per issue. Going faster indicates insufficient depth.

**Create working notes** for each issue capturing these elements. Avoid rushing this phase.

### Phase 4: Draft Enhanced Content (30-60 minutes)

To create final content, draft in Markdown first, then convert to HTML. For each issue:

#### Content Structure

```markdown
## FHIR-12345

### Discussion

[Opening paragraph: What was discussed? What sparked it?]

[Body paragraphs: Synthesize key points, perspectives, and reasoning]

The discussion centered on [main topic]. [Participants/The team]
emphasized [key perspective], noting that [reasoning]. However,
[alternative perspective] was also raised regarding [aspect].

Several approaches were considered:
- Approach A: [benefit] but [drawback]
- Approach B: [benefit] but [drawback]

The team decided to [decision], based primarily on [reasoning].
This approach was favored because [why it addresses concerns].
[Note any remaining concerns or open issues].
```

#### Style Guidelines

**Voice and Tense**:
- Use past tense: "The team discussed" not "The team discusses"
- Use third person: "Participants noted" not "We noted"
- Maintain professional but conversational tone

**Focus on WHY**:
- Insufficient: "Decided to use approach B."
- Effective: "Decided to use approach B primarily due to production stability concerns, though this may result in slower recovery from transient failures."

**Synthesis, Not Transcription**:
- Insufficient: "John said X. Then Mary said Y. Bob agreed."
- Effective: "The team weighed two approaches. Approach X offered [benefit] but raised concerns about [issue]. Approach Y provided [different benefit]."

**Length**: Aim for 2-4 substantive paragraphs per issue. One-paragraph summaries lack sufficient depth.

### Phase 5: Build HTML with XML DOM (15-30 minutes)

To convert markdown drafts to HTML and assemble the complete document, use `scripts/markdown_to_html.py` and `scripts/assemble_minutes.py`:

```python
#!/usr/bin/env python3
import sys
sys.path.insert(0, 'scripts')

from confluence_api import download_page
from extract_jira_widgets import extract_jira_widgets, get_widget_element
from markdown_to_html import parse_simple_markdown, create_issue_section
from assemble_minutes import build_complete_minutes
import json

# Load data
with open('current_page.json') as f:
    page_data = json.load(f)

original_html = page_data['body']['storage']['value']
widgets = extract_jira_widgets(original_html)

# Build enhanced sections
enhanced_sections = []

for issue_key in ['FHIR-12345', 'FHIR-12346']:  # Issue keys
    # Read markdown draft
    with open(f'{issue_key}-draft.md') as f:
        draft_md = f.read()

    # Parse to lxml elements
    discussion_elems = parse_simple_markdown(draft_md)

    # Get JIRA widget
    widget = get_widget_element(widgets[issue_key])

    # Create section
    section = create_issue_section(issue_key, widget, discussion_elems)
    enhanced_sections.extend(section)

# Assemble complete document
final_html = build_complete_minutes('current_page.json', enhanced_sections)

# Save
with open('enhanced_minutes.html', 'w') as f:
    f.write(final_html)
```

**Key functions available in scripts**:

- `parse_simple_markdown(text)` - Convert markdown to lxml elements
- `create_heading(level, text)` - Create h1-h6 element
- `create_paragraph(text)` - Create p element
- `create_list(items, ordered=False)` - Create ul/ol element
- `create_issue_section(key, widget, discussion_elems)` - Complete issue section
- `assemble_enhanced_minutes(original_html, enhanced_sections)` - Preserve structure
- `post_process_for_xhtml(html)` - Fix br/hr/col tags (the ONE exception)

### Phase 6: Upload to Confluence (5 minutes)

To upload the enhanced minutes, use `scripts/upload_to_confluence.py`:

```bash
python scripts/upload_to_confluence.py \
  <page_id> \
  enhanced_minutes.html \
  current_page.json
```

**Requires environment**: `CONFLUENCE_URL`, `CONFLUENCE_USER`, `CONFLUENCE_TOKEN`

**In Python**:
```python
from upload_to_confluence import upload_from_file

response = upload_from_file(
    page_id='123456',
    html_file='enhanced_minutes.html',
    current_version_file='current_page.json'
)
```

**API endpoint**: `PUT /rest/api/content/{pageId}`

To verify success, check output for HTTP 200 and new version number.

## Time Investment

Expected time allocation per phase:

- **Phase 1** (Context): 30-60 min
- **Phase 2** (Widgets): 5 min
- **Phase 3** (Analysis): 60-120 min ⭐ **Most critical - avoid rushing**
- **Phase 4** (Drafting): 30-60 min
- **Phase 5** (Building): 15-30 min
- **Phase 6** (Upload): 5 min

**Total**: 2.5-4.5 hours

**Most time should be in Phases 3-4** (content work). Spending more time on technical work than content work indicates incorrect prioritization.

## Common Pitfalls to Avoid

### Content Mistakes (CRITICAL)

1. **Shallow summaries**: "We discussed FHIR-12345 and decided to implement it."
   - **Solution**: Add 2-4 paragraphs explaining WHY, trade-offs, reasoning

2. **Conflating sources**: Using JIRA decisions as if discussed today
   - **Solution**: Use only transcript for today's discussion. JIRA is background only.

3. **Over-quoting**: Verbatim transcript with timestamps
   - **Solution**: Synthesize ideas into narrative form

4. **Missing WHY**: Stating decisions without reasoning
   - **Solution**: Always explain why decisions were made, what factors mattered

### Technical Mistakes

1. **String manipulation for HTML**: Using regex/concat to build HTML
   - **Solution**: Use lxml etree.Element() for all DOM construction

2. **Forgetting deepcopy()**: `append()` moves elements, not copies
   - **Solution**: `from copy import deepcopy` and use `deepcopy(elem)` when preserving

3. **Removing JIRA widgets**: Building content without preserving widgets
   - **Solution**: Extract first, re-insert when building sections

4. **Excessive post-processing**: Using regex to fix structural issues
   - **Solution**: Only post-process for XHTML self-closing tags (br/hr/col)

## Success Criteria

To evaluate completion quality:

1. **Content Quality**: Rich narratives with reasoning and trade-offs (2-4 paragraphs per issue)
2. **Technical Correctness**: Valid XHTML that Confluence accepts
3. **Structure Preservation**: All JIRA widgets and metadata intact
4. **Source Accuracy**: Only transcript content presented as today's discussion

## Scripts Reference

All scripts in `scripts/` directory support:
- Standalone CLI usage
- Python module import
- Environment variable configuration via `.env`

### confluence_api.py
Download Confluence pages, extract page IDs from URLs

### jira_api.py
Download JIRA issues (authenticated or public), create background files

### extract_jira_widgets.py
Parse HTML, extract JIRA widgets for preservation

### markdown_to_html.py
Convert markdown to lxml element trees with proper DOM construction

### assemble_minutes.py
Assemble complete HTML preserving original structure using deepcopy()

### upload_to_confluence.py
Upload to Confluence with error handling

## Key Lessons

This workflow was developed through multiple failed iterations. The critical lessons learned:

1. **Content quality matters more than technical perfection** - Perfect HTML with shallow content represents failure
2. **Transcript analysis cannot be rushed** - 60-120 minutes minimum required for proper depth
3. **Synthesis is a skill** - Practice capturing WHY not WHO
4. **XML DOM manipulation is non-negotiable** - Avoid string operations for HTML
5. **User feedback is ultimate judge** - Expect iteration

The biggest failure mode: Focusing on technical work while producing shallow content that fails to capture reasoning and trade-offs.
