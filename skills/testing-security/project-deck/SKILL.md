---
name: project-deck
description: Generate "future self" Beamer presentation decks as progress logs for research projects. Use when users ask to create a project deck, document project status, make slides summarizing their research, log what they've done on a project, or prepare update materials for coauthors. This skill creates dated LaTeX Beamer presentations (written to ./deck/project-deck-YYYYMMDD.tex) that preserve project context across work sessions - not for public speaking, but for communicating with your future self and collaborators.
---

# Project Deck Generator

Generate Beamer decks as progress logs - visual, structured snapshots of project state.

## Workflow

### Phase 1: Context Discovery

1. Check for previous decks in `./deck/` to understand project history
2. Read CLAUDE.md, README.md if they exist
3. Scan project folder structure (identify data files, code, results, figures)
4. Read any markdown files for context
5. Auto-detect: Is this the first deck (no previous in ./deck/) or subsequent?

### Phase 2: Gap Filling

If context is insufficient, ask targeted questions using AskUserQuestion:

**For first deck (no previous exists):**
- What is the research question?
- What data are you using?
- What is the identification strategy / methodology?
- Who are the authors?

**For subsequent decks (previous exists):**
- What was accomplished since the last deck?
- Any new results or figures?
- What's next?

### Phase 3: Deck Generation

Generate complete Beamer .tex file following the rhetoric principles in [references/deck.md](references/deck.md).

**Output location:** `./deck/project-deck-YYYYMMDD.tex` (create ./deck/ if needed)

**Theme:** Copy [assets/beamerthemeProjectDeck.sty](assets/beamerthemeProjectDeck.sty) to `./deck/` on first use.

#### Deck Structure

```latex
\documentclass[aspectratio=169]{beamer}
\usetheme{ProjectDeck}

\title{Project Title}
\author{Authors}
\date{\today}
```

**Slides to include:**

1. **Title slide** - Project name, authors, date, status indicator (e.g., "Work in Progress")

2. **Research Question** - What we're trying to answer and why it matters
   - Title as assertion: "We ask whether X affects Y"
   - Brief motivation (2-3 bullet points max)

3. **Context/Background** - Key institutional details, prior work gap
   - Title as assertion: "Prior work ignores the Z margin"

4. **Data** - What data, where from, sample size, key numbers
   - Title as assertion: "We use N observations from Source"

5. **Identification Strategy** - How we establish causality / methodology
   - Title as assertion: "We exploit variation in X"

6. **Results** (if available) - Tables/figures with highlighted coefficients
   - Title as assertion: "Treatment increases Y by Z%"
   - Use \alert{} or color boxes to highlight key numbers

7. **Directory Structure** - What files exist and what they do
   - Use verbatim or listing environment
   - Annotate key files

8. **What's Done** - Completed work
   - Bullet list of accomplishments

9. **What's Next** - Open questions, next steps, blockers
   - Prioritized list

10. **Key Decisions** - Rationale for choices made (for future reference)
    - "We chose X because Y"

### Phase 4: Output & Validation

1. Create `./deck/` folder if it doesn't exist
2. Write `./deck/project-deck-YYYYMMDD.tex` with today's date
3. Copy theme file to `./deck/` if not already present
4. **Attempt compilation:**
   - Run `pdflatex project-deck-YYYYMMDD.tex` in `./deck/`
   - If pdflatex is not available or fails completely → skip to step 6
5. **If compilation succeeds, validate and fix:**
   - Parse the .log file for overfull/underfull hbox/vbox warnings
   - For each warning, fix the corresponding issue in the .tex file:
     - Overfull hbox: reduce content, add line breaks, adjust column widths
     - Underfull hbox: adjust spacing or content
     - Overfull vbox: split content across slides, reduce vertical spacing
   - Recompile and repeat until no warnings remain
   - Confirm clean compile to user
6. **If pdflatex unavailable:**
   - Inform user that .tex file is ready
   - Provide manual compilation instructions:
     ```
     cd deck
     pdflatex project-deck-YYYYMMDD.tex
     ```
     Or: "Upload .tex and .sty files to Overleaf"

## Key Principles

### Decks as Logs
Each deck is a standalone snapshot of project state on that date. The `./deck/` folder accumulates a history of these snapshots.

### "Future Self" Rhetoric
From the rhetoric principles - when making a deck for your future self:
- Be more explicit than for live presentation
- Include the "why" behind choices, not just the "what"
- Write in complete thoughts where necessary
- Date everything
- Assume you'll have forgotten the context

### Slide Design
- **One idea per slide** - if you're saying "and also," you need another slide
- **Titles are assertions, not labels** - "Treatment increased distance by 61 miles" not "Results"
- **Lead with conclusions** - state the finding, then support it
- **Visual hierarchy** - what's big is important
- **Highlight key numbers** - use \alert{} or color boxes in tables

### Beauty as Functional Rhetoric

Decks must be beautiful. Beauty captures attention, and attention enables learning. "Humans want to stare at beautiful things" - the goal is stare AND learn.

**Unique design:**
- Must not look like a recognizable Beamer theme (Metropolis, Madrid, etc.)
- If using a base theme, it should be unrecognizable
- Custom colors, fonts, and styling

**Cognitive density:**
- Too much density is not beautiful
- Prefer two slides over one crowded slide
- White space is valuable - let content breathe

**Quantification:**
- Brain needs numbers, tables, figures
- Visualize key results with beautiful charts
- Tables should be clean, highlighted, readable

**TikZ for narrative:**
- Use TikZ for functional, beautiful visualizations
- Examples: timelines, process flows, geographic maps, causal diagrams
- Must service the narrative, not just decorate

**Directory structure:**
- Use tree-style visualization:
  ```
  project/
  ├── data/
  │   └── raw.csv
  ├── code/
  │   └── analysis.do
  └── output/
  ```
- Not balloon/arrow diagrams

**Clean compile (when pdflatex available):**
- Compile the generated .tex file automatically
- Parse log file for ALL overfull/underfull hbox/vbox warnings
- Fix each issue in the source .tex file
- Recompile until log shows no warnings
- Only present final deck to user after clean compile

**Fallback (when pdflatex unavailable):**
- Output .tex file without compilation
- Provide manual compilation instructions
