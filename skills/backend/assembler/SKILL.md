---
name: assembler
description: Combine all draft sections into final manuscript, apply formatting constraints, and generate reference list. Final step of writer workflow. Requires academic reviewer approval and all drafts/*.md files.
---

# Assembler

Combines all drafted sections into a final manuscript, applies formatting and word limits, and generates the complete reference list.

## Critical Prerequisite: Academic Reviewer Approval

**Do NOT proceed with assembly until the academic reviewer has signed off.**

Before assembling:
1. Check `notes/reviewer-feedback.md` exists
2. Verify all critical issues have been addressed
3. Confirm sign-off status is "Approved" or "Approved with minor revisions"

If reviewer sign-off is missing or issues remain unresolved:
```
## Assembly Blocked

The academic reviewer has not approved this manuscript for assembly.

**Status**: [Not reviewed / Critical issues pending / User input required]

**Outstanding Issues**:
1. [Issue from reviewer-feedback.md]
2. [Issue from reviewer-feedback.md]

**Action Required**:
- Address the issues listed in notes/reviewer-feedback.md
- Re-run the academic reviewer (/writer:review)
- Obtain sign-off before proceeding

Assembly cannot continue until these issues are resolved.
```

## Prerequisites

Required files:
- `scope.md` - Constraints (word limit, journal, citation style)
- `notes/reviewer-feedback.md` - **Academic reviewer sign-off (REQUIRED)**
- `notes/statistical-review.md` - Statistical reviewer sign-off
- `notes/ethics-summary.md` - Ethics document summary (optional, provides ethics statement)
- `drafts/abstract.md` - Abstract and title options
- `drafts/introduction.md` - Introduction section
- `drafts/methods.md` - Methods section
- `drafts/results.md` - Results section
- `drafts/discussion.md` - Discussion section
- `notes/papers-library/*.pdf` - All source PDFs
- `notes/bibliography.md` - Master reference list
- All `notes/papers/*.md` for reference details

## Workflow

```
[Verify Academic Reviewer Sign-Off] ─── REQUIRED
     │
     ▼
[Verify Biostatistician Sign-Off] ─── REQUIRED
     │
     ▼
[Load all drafts]
     │
     ▼
[Compile reference list] ─── From notes/bibliography.md
     │
     ▼
[Assemble manuscript] ─── Combine in order
     │
     ▼
[Apply word limit] ─── Trim if needed
     │
     ▼
[Final formatting] ─── Apply journal style
     │
     ▼
[Output] ─── manuscript.md
```

## Step 0: Verify Sign-Offs

### Academic Reviewer Sign-Off

Read `notes/reviewer-feedback.md` and confirm:

```markdown
## Sign-Off Verification

**Academic Reviewer**:
- Sign-off status: [Approved / Approved with minor revisions / NOT APPROVED]
- Critical issues: [0] remaining
- Major issues: [X] addressed, [Y] acknowledged
- Date: [timestamp]

**Proceed?**: [ ] Yes / [ ] No — [reason if no]
```

### Statistical Reviewer Sign-Off

Read `notes/statistical-review.md` and confirm:

```markdown
**Statistical Reviewer**:
- Sign-off status: [Approved / NOT APPROVED]
- Statistical issues: [0] remaining
- Date: [timestamp]

**Proceed?**: [ ] Yes / [ ] No — [reason if no]
```

**If either sign-off is missing or not approved, STOP and report to user.**

## Step 1: Compile Reference List

### Collect All Citations

From each `notes/papers/*.md`, extract:
- Full citation string
- Citation number used in text

### Order References

**Vancouver/ICMJE style**: Number in order of first appearance in text

Scan through drafts in order:
1. Introduction
2. Methods  
3. Results
4. Discussion

Assign numbers [1], [2], [3]... as each new source appears.

### Format Reference List

Based on `scope.md` citation style:

**AMA/Vancouver:**
```
1. Smith JA, Jones BB, Wilson CC. Title of article. Journal. Year;Vol:Pages. doi:XX
2. Author AA, Author BB. Title. Journal. Year;Vol:Pages.
```

**APA:**
```
Author, A. A., & Author, B. B. (Year). Title. Journal, Vol(Issue), Pages. https://doi.org/XX
```

## Step 2: Assemble Manuscript

Create `manuscript.md` with this structure:

```markdown
# [Title from drafts/abstract.md]

**Authors**: [To be added]

**Affiliations**: [To be added]

**Corresponding Author**: [To be added]

---

## Abstract

[Content from drafts/abstract.md]

**Keywords**: [keywords from abstract]

---

## Introduction

[Content from drafts/introduction.md]

---

## Methods

[Content from drafts/methods.md]

---

## Results

[Content from drafts/results.md]

---

## Discussion

[Content from drafts/discussion.md - includes Conclusion if combined]

---

## Conclusion

[If separate from Discussion - extract from drafts/discussion.md]

---

## Acknowledgments

[Placeholder]

---

## Ethics Statement

[Auto-populated from notes/ethics-summary.md if available, otherwise placeholder]

This study was approved by [Institution/Ethics Board] (approval number: [from ethics-summary.md]). [Informed consent statement as appropriate for study type.]

---

## Conflicts of Interest

[Placeholder]

---

## Funding

[Placeholder]

---

## References

[Compiled reference list]

---

## Figure Legends

[From drafts/results.md]

---

## Tables

[From drafts/results.md]

---

## Supplementary Materials

[If applicable]
```

## Step 2b: Populate Ethics Statement

### If `notes/ethics-summary.md` exists:

Extract from the ethics summary:
- **Approval Number**: Use exact number from "Study Identification" section
- **Approving Body**: IRB, IACUC, Ethics Committee, etc.
- **Institution Name**: If available, otherwise use "[Institution]" placeholder
- **Approval Date**: Include if journal requires it

Generate ethics statement:

```markdown
## Ethics Statement

This study was approved by [Institution Name] [Approving Body]
(Protocol #[Approval Number], approved [Approval Date]).
[Informed consent / waiver of consent statement based on study type].
```

### Consent Statement Templates

Based on study type from `notes/ethics-summary.md`:

**Prospective with consent:**
> Written informed consent was obtained from all participants prior to enrollment.

**Retrospective/waiver:**
> The requirement for informed consent was waived due to the retrospective nature of this study.

**Secondary data analysis:**
> This study used de-identified data and was exempt from consent requirements.

**Animal research:**
> All procedures were approved by [Institution] Institutional Animal Care and Use Committee (IACUC).

**Computational/simulation:**
> This study did not involve human subjects or animals and did not require ethics approval.

### If `notes/ethics-summary.md` does NOT exist:

Leave placeholder for user to complete:

```markdown
## Ethics Statement

[PLACEHOLDER - Please provide:]
- Approval number: ___
- Approving body: ___
- Institution name: ___
- Consent statement: ___
```

Notify user: "No ethics document was provided. Please complete the Ethics Statement section with your approval information, or indicate if ethics approval was not required."

## Step 3: Apply Word Limit

### Count Words

```bash
# Approximate word count (excluding references, figures, tables)
```

Count sections:
- Abstract: [n] words (typically excluded from limit)
- Introduction: [n] words
- Methods: [n] words
- Results: [n] words
- Discussion: [n] words
- **Total body**: [n] words

### Compare to Limit

From `scope.md`:
- Word limit: [n]
- Current count: [n]
- Delta: [over/under by n]

### If Over Limit

Strategies to reduce word count:

1. **Discussion** (usually easiest to trim):
   - Reduce speculative language
   - Consolidate similar points
   - Remove redundant transitions

2. **Methods** (if detailed):
   - Move supplementary details to Supplementary Materials
   - Combine sentences
   - Remove obvious steps

3. **Results** (carefully):
   - Keep all statistics
   - Reduce narrative around clear findings
   - Move secondary findings to Supplementary

4. **Introduction** (last resort):
   - Reduce background breadth
   - Tighten gap statement

**Do NOT cut:**
- Key statistics
- Primary findings
- Essential methods for reproducibility

## Step 4: Final Formatting

### Journal-Specific Adjustments

Based on `scope.md` target journal, apply:

**Radiology / RSNA journals:**
- Structured abstract (Background, Purpose, Materials and Methods, Results, Conclusion)
- "Materials and Methods" not just "Methods"
- Abbreviations defined in abstract and again in text

**JAMA network:**
- Key Points box (Question, Findings, Meaning)
- Structured abstract
- Strict word limits

**Nature/Science:**
- Very concise
- Methods often separate/supplementary
- References in specific format

### Section Headings

Verify correct heading style:
- All caps vs title case
- Numbered vs unnumbered
- Combined "Results and Discussion" vs separate

### Citation Format

Verify all citations match required style:
- Superscript¹ vs brackets [1]
- Author-date (Smith, 2023) if APA
- Correct punctuation around citations

## Step 5: Quality Checks

### Consistency Checks

- [ ] All figures referenced in text
- [ ] All tables referenced in text
- [ ] All references cited in text appear in reference list
- [ ] All references in list are cited in text
- [ ] Abbreviations defined on first use
- [ ] Numbers in abstract match numbers in text
- [ ] No contradictions between sections

### Formatting Checks

- [ ] Heading levels consistent
- [ ] Citation format consistent throughout
- [ ] Statistical reporting format consistent
- [ ] Line spacing/margins per journal (if specified)

### Final Proofread Notes

Flag for user attention:
- [ ] Author names and order
- [ ] Affiliations
- [ ] Corresponding author details
- [ ] Acknowledgments
- [ ] COI disclosures
- [ ] Funding statement
- [ ] Ethics statement / IRB number

## Output

### Primary Output

Save to: `manuscript.md`

### Word Count Summary

```markdown
## Word Count Summary

| Section | Words |
|---------|-------|
| Abstract | XXX |
| Introduction | XXX |
| Methods | XXX |
| Results | XXX |
| Discussion | XXX |
| **Body Total** | **XXX** |
| Target | XXX |
| Status | ✓ Under / ⚠️ Over by X |
```

### Completion Summary

Return to user:

> "Manuscript assembled. Here's the summary:
>
> **Title**: [title]
> **Word Count**: [n] / [limit] words
> **References**: [n] sources cited
> **Figures**: [n]
> **Tables**: [n]
>
> **Needs your input:**
> - Author list and affiliations
> - Acknowledgments
> - Conflicts of interest
> - Funding statement
>
> The manuscript is saved to `manuscript.md`."

## Files Generated

Final project structure:

```
project/
├── papers/
├── data/
├── figures/
├── code/
├── notes/
│   ├── papers/*.md
│   ├── search/*.md
│   ├── code-analysis.md
│   └── data-analysis.md
├── drafts/
│   ├── introduction.md
│   ├── methods.md
│   ├── results.md
│   ├── discussion.md
│   └── abstract.md
├── config.md
├── inventory.md
├── scope.md
└── manuscript.md          ← Final output
```
