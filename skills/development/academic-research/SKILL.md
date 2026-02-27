---
name: academic-research
description: Create comprehensive academic research notes with deep literature coverage. Auto-detects language (EN prompt→EN output, TR prompt→TR output). Supports Obsidian markdown and PDF. Performs 8-15 iterative search cycles with 25-50+ sources for comprehensive coverage. Uses footnote citations and visual overviews.
---

# Academic Research Skill

Deep literature research with 25-50+ sources through iterative search cycles.

## Workflow

### 1. Auto-Detect & Clarify

**Language:** Auto-detected from user prompt (no need to ask)

**Ask User:**

**Output Format:**
- "Obsidian markdown mı PDF mi?" / "Obsidian markdown or PDF?"

**If Obsidian:** "Vault'ta ilgili notları arayayım mı?" / "Search vault for related notes?"
**If yes:** "Vault yolu?" / "Vault path?" → Default: `/home/user/Documents/Obsidian Vault/`

**If PDF:** Skip vault integration

**Save Location:** "Nereye kaydedeyim?" / "Where to save?"

**Topic Analysis & Mode Suggestion:**
- **Narrow/Specific** → Focused: 12-18 sources, 5-7 search cycles
- **Broad/Complex** → Comprehensive: 25-35 sources, 8-12 cycles  
- **Very Broad** → Deep: 40-50+ sources, 12-15 cycles
- **Comparative** → Synthesis: 15-25 sources, 6-9 cycles

State suggestion, proceed unless user objects.

**Scope:** "Odak veya hariç tutulacak alan?" / "Focus or exclusions?"

### 2. Iterative Research Process

**Search Cycles Based on Mode:**
- Focused: 5-7 iterative cycles
- Comprehensive: 8-12 iterative cycles
- Deep: 12-15+ iterative cycles
- Synthesis: 6-9 iterative cycles

**Each Cycle:**
1. Execute search with specific query
2. Evaluate results (quality, relevance, gaps)
3. Identify what's missing (perspectives, time periods, methodologies)
4. Formulate next targeted query
5. Cross-validate findings
6. Document effective vs. ineffective strategies
7. Repeat until comprehensive coverage

**Research Behaviors:**
- **Cycle 1-2:** Broad searches → Key concepts, seminal works, major authors
- **Cycle 3-5:** Targeted searches → High-impact papers, meta-analyses, reviews
- **Cycle 6-8:** Gap filling → Missing perspectives, alternative views, contradictions
- **Cycle 9-11:** Cross-validation → Verify contested claims, find consensus
- **Cycle 12+:** Final sweep → Recent work, emerging trends, practical applications

**Source Diversity Targets:**
- **Types:** 60%+ peer-reviewed articles, 10-15% meta-analyses/reviews, 5-10% books, rest varied
- **Temporal:** Balance seminal works + recent (50%+ within 5 years for active fields)
- **Geographic:** Multiple regions/countries when possible
- **Perspectives:** Supporting + Critical + Alternative views

**Quality Priority:** Peer-reviewed > Meta-analyses > Conference > Preprints

**Search in English** for best global sources (translate findings if Turkish output in a formal, academic language)

### 3. Generate Output

**File Naming:**
- Descriptive, Title Case
- EN: `Cognitive Load Theory.md`
- TR: `Bilişsel Yük Kuramı.md`

**For Obsidian:**
- Use appropriate template (en/tr)
- Footnote citations: `text[^1]` → `[^1]: Full citation 🟢`
- Visual overview (mindmap)
- Wikilinks to related notes (if vault provided)
- MOC integration
- Tag suggestions (no #)
- Dataview metadata
- Support with basic TikZ whenever logical

**For PDF:**
- PDF-optimized template
- Footnote citations
- Visual overview
- Professional formatting
- No wikilinks

**Quality Standards by Mode:**

| Mode | Sources | Cycles | 🟢 High-Impact | Tables | Diagrams |
|------|---------|--------|----------------|--------|----------|
| Focused | 12-18 | 5-7 | 50%+ | 1+ | 1+ |
| Comprehensive | 25-35 | 8-12 | 60%+ | 2+ | 2+ |
| Deep | 40-50+ | 12-15+ | 65%+ | 3+ | 3+ |
| Synthesis | 15-25 | 6-9 | 55%+ | 3+ | 1+ |

### 4. Present Output

1. Brief summary (2-3 sentences of key findings)
2. Share file via present_files
3. Mention: source count, quality distribution, confidence score
4. Highlight: notable gaps, debates, or surprising findings
5. List: suggested next research steps
6. If Obsidian: mention related notes linked

### 5. Continue Research

If user says "continue", "expand", or "more sources":
1. Load existing note
2. Review research log → identify gaps
3. Ask which gap to address
4. Execute 3-5 additional cycles
5. Integrate findings (avoid duplication)
6. Update metadata (source_count, last_updated)
7. Present updated note with summary

## Research Depth Indicators

Track and report in note:
- Total search cycles executed
- Total sources found vs. included
- Quality distribution (🟢🟡🔴 counts)
- Temporal coverage (year range, % recent)
- Geographic coverage (countries/regions)
- Methodological diversity (RCT, meta, qualitative, etc.)
- Perspective balance (supporting/critical/alternative)

## Key Features

**Deep Coverage:**
- 8-15 iterative search cycles
- 25-50+ sources (comprehensive/deep modes)
- Multiple perspectives required
- Gap identification and filling
- Cross-validation of claims

**Smart Integration:**
- Auto language detection
- Format choice (Obsidian/PDF)
- Optional vault search
- Footnote citations
- Visual overviews

**Quality Tracking:**
- Source diversity metrics
- Confidence scoring (⭐1-5)
- Research cycle documentation
- Methodology notes

## References

Templates: `references/template-{en|tr}.md`
Citation: `references/apa7-quick-ref.md`
Diagrams: `references/mermaid-patterns.md`
Search: `references/search-strategies.md`
