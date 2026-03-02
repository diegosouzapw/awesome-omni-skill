---
name: benchmark-report-creator
version: 0.1.1
description: Use PROACTIVELY when creating research reports, experiment writeups, technical whitepapers, or empirical study documentation. Orchestrates the complete benchmark report pipeline with structure, diagrams, hi-res PNG capture, and PDF export. Provides working scripts, CSS templates, and complete command sequences for publication-quality AI/ML benchmark reports. Not for slides, blog posts, or simple README files.
author: "Connor"
category: benchmarking
---

# Benchmark Report Creator

## Overview

**End-to-end orchestrator** for creating publication-quality benchmark reports. This skill coordinates the full pipeline rather than duplicating individual component skills.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    BENCHMARK REPORT PIPELINE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. STRUCTURE         2. DIAGRAMS           3. EXPORT                   │
│  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐            │
│  │ Markdown     │    │ HTML diagram │      │ pandoc +     │            │
│  │ template     │───►│ with academic│─────►│ weasyprint   │            │
│  │ (sections)   │    │ styling      │      │ (final PDF)  │            │
│  └──────────────┘    └──────┬───────┘      └──────────────┘            │
│                             │                                           │
│                             ▼                                           │
│                      ┌──────────────┐                                   │
│                      │ Playwright   │  ◄── High-res PNG capture         │
│                      │ script       │      (2x deviceScaleFactor)       │
│                      │ (hi-res PNG) │                                   │
│                      └──────────────┘                                   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Capabilities**:
- Complete pipeline orchestration (not component duplication)
- Working hi-res PNG capture script (fixes Playwright CLI limitations)
- Exact CSS templates from paralleLLM empathy v1.0/v2.0
- Full command sequences with copy-paste examples

## When to Use This Skill

**Trigger Phrases**:
- "create a benchmark report"
- "write up my AI experiment"
- "publication-quality benchmark documentation"
- "full report with diagrams and PDF"
- "empathy experiment style report"

**Use Cases**:
- AI/ML benchmark evaluation reports
- Model comparison studies
- Empirical experiment writeups
- Research documentation with figures

**NOT for**:
- Single diagram creation (use html-diagram-creator)
- Simple markdown to PDF (use markdown-to-pdf-converter)
- ASCII diagrams (use ascii-diagram-creator)

## Quick Start

### Prerequisites

```bash
# Install all dependencies
brew install pandoc
pip install weasyprint
npm install playwright
npx playwright install chromium
```

### Complete Pipeline (3 commands)

```bash
# 1. Create HTML diagram
#    [Use html-diagram-creator skill or copy template]

# 2. Capture diagram to high-res PNG
node scripts/capture-diagram.js diagram.html figures/figure-1.png

# 3. Convert markdown report to PDF
pandoc report.md --standalone --css=templates/pdf-style.css -t html | weasyprint - report.pdf
```

## Pipeline Phases

### Phase 1: Document Structure

Use the **report-creator** pattern or copy from `reference/report-template.md`:

| Section | Content |
|---------|---------|
| Abstract | 150-250 word summary |
| Executive Summary | Key finding + metrics table |
| 1. Background | Research context, hypotheses |
| 2. Methodology | Design, variables, protocol |
| 3. Results | Statistics, observations |
| 4. Discussion | Hypothesis evaluation |
| 5. Limitations | Methodological constraints |
| 6. Future Work | Research directions |
| 7. Conclusion | Synthesis |
| Appendices | Supporting materials |

### Phase 2: Create Diagrams

Use the **html-diagram-creator** pattern for publication-quality HTML diagrams.

**Color Palette (Academic Standard)**:

| Stage | Fill | Border | Usage |
|-------|------|--------|-------|
| Data Preparation | `#E3F2FD` | `#1976D2` | Input processing |
| Execution | `#E8F5E9` | `#388E3C` | API calls, inference |
| Analysis | `#FFF3E0` | `#F57C00` | Evaluation, scoring |

→ Full templates: `reference/html-templates.md`

### Phase 3: Capture to PNG (CRITICAL)

**Important**: The Playwright CLI `--device-scale-factor` flag does NOT exist. Use the provided Node.js script instead:

```bash
# Working high-res capture (2x retina quality)
node scripts/capture-diagram.js diagram.html output.png

# The script handles:
# - deviceScaleFactor: 2 (retina quality)
# - .diagram-container selector targeting
# - Proper file:// protocol handling
```

**Why not CLI?** Playwright's CLI screenshot command doesn't support `--device-scale-factor`. The bundled script uses the Playwright API directly.

### Phase 4: Embed in Markdown

```html
<figure style="margin: 2em auto; page-break-inside: avoid; text-align: center;">
  <img src="figures/figure-1.png" alt="Description" style="max-width: 100%; height: auto;">
  <figcaption style="text-align: center; font-style: italic; margin-top: 1em;">
    Figure 1: Caption text.
  </figcaption>
</figure>
```

### Phase 5: Export to PDF

```bash
# Two-step (for debugging)
pandoc report.md -o report.html --standalone --css=templates/pdf-style.css
weasyprint report.html report.pdf

# One-liner (production)
pandoc report.md --standalone --css=templates/pdf-style.css -t html | weasyprint - report.pdf
```

## File Structure

```
benchmark-report-creator/
├── SKILL.md                    # This file (orchestrator)
├── templates/
│   └── pdf-style.css          # Academic CSS (empathy v1.0/v2.0)
├── scripts/
│   └── capture-diagram.js     # Working hi-res PNG capture
├── reference/
│   ├── report-template.md     # Full markdown template
│   ├── html-templates.md      # Diagram HTML templates
│   └── command-reference.md   # All commands in one place
└── examples/
    └── sample-report/         # Complete working example
```

## CSS Template Standards

The bundled `templates/pdf-style.css` follows empathy v1.0/v2.0 conventions:

### Tables (Academic Style)
- Header: 2px solid top/bottom borders
- Body: 1px solid bottom borders
- Last row: 2px solid bottom border
- `page-break-inside: avoid`

### Figures
- `max-width: 100%` (NOT min-width)
- `margin: 2em auto` for centering
- `page-break-inside: avoid`
- Italic captions centered below

### Page Control
- 2cm margins
- Headings: `page-break-after: avoid`
- Orphans/widows: 3 lines minimum

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Blurry PNG | Using CLI instead of script | Use `scripts/capture-diagram.js` |
| Image off-center | Missing margin auto | Add `margin: 2em auto` to figure |
| Table split | Missing page-break rule | CSS has this by default |
| Weasyprint warnings | Unsupported CSS props | Safe to ignore (gap, overflow-x) |
| Empty PNG | Wrong selector | Check `.diagram-container` exists |

## Success Checklist

- [ ] All prerequisites installed (pandoc, weasyprint, playwright)
- [ ] Diagram HTML created with academic styling
- [ ] PNG captured at 2x resolution using script (not CLI)
- [ ] Markdown has proper figure tags with captions
- [ ] PDF renders without errors
- [ ] Tables don't split across pages
- [ ] Figures centered with captions

## Related Skills (User Global)

These skills may be installed in user's `~/.claude/skills/`:

| Skill | Purpose | When to Use Directly |
|-------|---------|---------------------|
| report-creator | Document templates | Structure guidance only |
| html-diagram-creator | Diagram HTML | Single diagram creation |
| html-to-png-converter | PNG export reference | CLI-only workflows |
| markdown-to-pdf-converter | PDF pipeline | Non-benchmark documents |

**This orchestrator combines all four** into a single coherent workflow for benchmark reports.

---

**Based on**: paralleLLM empathy-experiment-v1.0.pdf and v2.0.pdf
