---
name: generate-note-ideas
description: "Scan published YouTube videos, Substack newsletter issues, and Substack Notes to generate high-quality note ideas. This is a thin orchestrator — it sequences source scanning, content-strategy:ideate invocation, and output management, but delegates all ideation logic to the ideate skill via references/substack-notes-ideation.md."
---

# Generate Substack Note Ideas

## Overview

This orchestrator scans published content across YouTube and Substack, then delegates ideation to `content-strategy:ideate` with the ideation reference file. It handles source fetching, processed-log management, and output persistence -- all actual idea generation logic is delegated to the foundation skill.

This skill sits upstream of `substack:create-note` -- it produces a curated list of note ideas, while `create-note` writes the full note from a selected idea.

**Core Principle**: This is a thin orchestrator. Never generate ideas manually. Always delegate to `content-strategy:ideate` via `references/substack-notes-ideation.md`. This skill manages the source-scanning workflow, duplicate prevention, and output files only.

## When to Use

Use this skill when:
- Generating a batch of Substack Note ideas from published content
- Finding new note topics between newsletter issues
- Maintaining a consistent Notes posting cadence
- Mining existing content for repurposable note angles

## Prerequisites

**Optional**: YouTube MCP tools (`search_videos`, `get_video_details`, `get_video_transcript`, `get_video_comments`) for scanning the user's YouTube channel. If unavailable, the workflow falls back to web search.

**Optional**: An existing `./substack/notes/processed-log.md` file. If it does not exist, the workflow creates it in Step 0.

## Workflow

Execute all steps below in order.

### Step 0: Load Processed Log and Content Bank

Read `./substack/notes/processed-log.md` to determine which sources have already been scanned.

**If file exists:**
- Parse the YouTube Videos table to get a list of already-processed video IDs
- Parse the Newsletter Issues table to get a list of already-processed issue URLs
- Parse the Substack Notes Scanned table for the last scan date

**If file does not exist:**
- Create the `./substack/notes/` directory if it does not exist
- Create the file with the empty template (see Output Structure below -- headers only, no data rows)

Read `./substack/notes/ideas.md` (the content bank) if it exists:
- Tally pending ideas per type to identify which types need new ideas
- Tally high-converter vs high-engagement pending ideas to identify conversion gaps
- Feed all existing ideas (any status) to Step 4 for duplicate prevention
- Report the current bank inventory to the user: "Content bank has X pending ideas across Y types. Types needing ideas: [list]."

**If content bank does not exist:**
- Create it with the empty template (see Output Structure below -- type headers only, no ideas)

### Step 1: Fetch Published YouTube Content

Use YouTube MCP tools to find recent videos from the user's channel.

1. Use `search_videos` to find the user's recent videos
2. Compare video IDs against the processed log -- identify new/unprocessed videos
3. For each new video:
   - Use `get_video_details` for title, description, and stats
   - Use `get_video_transcript` for the full transcript
   - Use `get_video_comments` for top comments (source of audience questions and misconceptions)
4. Collect all new video material for Step 4

**If YouTube MCP tools are unavailable:**
- Fall back to web search for the user's YouTube channel
- Gather what video titles, descriptions, and topics are available from search results

**If no new videos are found:**
- Note this and continue -- Steps 2 and 3 may still produce source material

### Step 2: Fetch Published Substack Content

Use web fetch to scan the user's Substack archive and Notes feed.

1. Web fetch the user's Substack archive page (e.g., `https://{subdomain}.substack.com/archive`)
2. Compare issue URLs against the processed log -- identify new/unprocessed issues
3. For each new issue: web fetch the full issue content
4. Web fetch the user's Substack Notes feed for gap analysis and posted-notes history
5. Collect all new newsletter and notes material for Step 4

**If no new issues are found:**
- Note this and continue -- the Notes feed, web trends, and YouTube content may still produce ideas

### Step 3: Fetch Web Trends

Use web search to find trending topics in the user's niche.

1. Search for trending topics, discussions, and recent developments relevant to the user's content area
2. Identify timely angles that could inform note ideas
3. Collect trend findings for Step 4

**This step always runs** regardless of whether new sources were found in Steps 1-2. Trending topics provide timely note ideas even when no new content has been published.

### Step 4: Invoke `content-strategy:ideate` with Ideation Reference

**MANDATORY**: Invoke `content-strategy:ideate` with `references/substack-notes-ideation.md` to generate structured note ideas.

Provide all source material gathered in Steps 0-3:
- New YouTube video transcripts, titles, descriptions, and top comments (from Step 1)
- New newsletter issue content (from Step 2)
- Past Substack Notes history (from Step 2, for gap analysis)
- Web trend findings (from Step 3)
- Previously generated ideas from `ideas.md` (from Step 0, for duplicate prevention)

The ideate skill applies the ideation framework and returns structured ideas with topic, note type, source, pitch, and strategic rationale.

NOTE: All ideation logic -- source-to-idea extraction, note-type matching, angle generation, duplicate filtering -- lives in the reference file. Do not implement any of this in the orchestrator.

### Step 5: Present Ideas to User

Present the batch of ideas (5-10) in the structured format from the ideation reference.

Each idea includes:
- **Topic** -- what the note is about
- **Type** -- suggested note type (from the `create-note` type taxonomy)
- **Source** -- which source content inspired this idea
- **Pitch** -- one-line description of the note angle
- **Rationale** -- why this idea is strategically sound

The user can:
1. **Approve the batch** -- all ideas are saved
2. **Remove specific ideas** -- selected ideas are dropped before saving
3. **Request more ideas** -- return to Step 4 with additional guidance

**CRITICAL**: Do not save any output until the user approves. Present and wait.

### Step 6: Save to Content Bank

After user approval, update two files:

**1. File approved ideas into `./substack/notes/ideas.md` by type:**
- For each approved idea, add it under the matching note type section
- Use the content bank idea format (Topic, Status: pending, Source, Pitch, Rationale, Conversion, Added date)
- Place new ideas at the bottom of each type section (newest last)

**2. Update Quick Stats header:**
- Recalculate total pending/drafted/published counts
- Update "Last ideation run" date
- Recalculate "Types needing ideas" (types with 0 pending ideas)

**3. Update `./substack/notes/processed-log.md`:**
- Add a row for each new YouTube video scanned (video ID, title, scan date, ideas generated count)
- Add a row for each new newsletter issue scanned (URL, title, scan date, ideas generated count)
- Update the Substack Notes Scanned section with the latest scan date and gap analysis summary

### Step 7: Handoff Suggestion

Inform the user they can pick any idea and invoke `substack:create-note` to write the full note.

Do NOT automatically invoke `create-note`. This skill generates ideas only -- the user decides when and which idea to execute.

## Output Structure

### Content Bank (`./substack/notes/ideas.md`)

The content bank organizes ideas by note type for easy retrieval during writing sessions. Ideas flow from ideation runs into type-organized sections with status tracking.

```markdown
# Substack Notes Content Bank

## Quick Stats
- Total pending: 0 | Drafted: 0 | Published: 0
- Last ideation run: YYYY-MM-DD
- Types needing ideas: [list types with 0 pending ideas]

---

## Single-Punch Wisdom

## Income Proof Story

## Pattern Observation

## Contrarian Statement

## Problem → Solution

## Build-in-Public Update

## List-Based Tactical

## Vulnerable Personal Story

## Newsletter Teaser

## Direct Advice
```

Each idea within a type section uses this format:

```markdown
### [Specific topic in 5-10 words]
**Status:** pending
**Source:** [YouTube — "Title" (video ID) | Newsletter — "Title" (URL) | Web trend — description]
**Pitch:** [One sentence describing the note's core message]
**Rationale:** [Why this idea is worth posting — engagement potential, gap it fills, timeliness]
**Conversion:** [High Converter | High Engagement | Both | Moderate]
**Added:** YYYY-MM-DD
```

**Status values:**
- `pending` — idea generated but not yet written
- `drafted` — note has been drafted via `substack:create-note`
- `published` — note has been published to Substack

### Processed Log (`./substack/notes/processed-log.md`)

```markdown
# Processed Content Log

## YouTube Videos
| Video ID | Title | Scanned | Ideas Generated |
|----------|-------|---------|-----------------|

## Newsletter Issues
| URL | Title | Scanned | Ideas Generated |
|-----|-------|---------|-----------------|

## Substack Notes Scanned
| Last Scan Date | Notes Reviewed | Gap Analysis |
|----------------|----------------|--------------|
```

## Quality Checklist

Verify completion before finalizing:
- [ ] Processed log loaded or created (Step 0)
- [ ] Content bank loaded and inventory reported (Step 0)
- [ ] YouTube videos scanned via MCP tools or web search fallback (Step 1)
- [ ] Substack newsletter issues scanned via web fetch (Step 2)
- [ ] Web trends gathered (Step 3)
- [ ] `content-strategy:ideate` invoked with `references/substack-notes-ideation.md` (Step 4)
- [ ] Ideas presented to user for approval (Step 5)
- [ ] Approved ideas filed by type in content bank `./substack/notes/ideas.md` (Step 6)
- [ ] Quick Stats updated in content bank (Step 6)
- [ ] Processed log updated with newly scanned sources (Step 6)
- [ ] Handoff to `substack:create-note` suggested (Step 7)

## Common Pitfalls to Avoid

1. **Generating ideas manually**: All ideation logic lives in the ideate reference file. Delegate to `content-strategy:ideate` -- do not implement idea generation in this orchestrator.
2. **Re-scanning processed sources**: Always check the processed log first. Skip sources already scanned.
3. **Ignoring the notes history**: Past notes feed gap analysis. Always load them in Step 2.
4. **Skipping web trends**: Trending topics provide timely ideas even when no new content exists. Step 3 always runs.
5. **Auto-invoking create-note**: This skill generates ideas only. The user decides when to write.
6. **Saving before user approval**: Always present ideas and get explicit approval before writing to files.
