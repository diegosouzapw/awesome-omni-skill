---
name: arxiv-analysis
description: Analyze arXiv research papers and explain them in accessible terms. Use when the user mentions arXiv, research paper, academic paper, scientific paper, preprint, or provides an arxiv.org URL.
---

# arXiv Research Paper Analysis

Analyze arXiv research papers and explain them in plain English with practical implications.

## When to Use

Activate this skill when the user:
- Mentions "arXiv", "research paper", "academic paper", "scientific paper", "preprint"
- Provides an arxiv.org URL
- Asks to explain or summarize a research paper
- Wants to understand a technical paper in accessible terms

## Instructions

1. **Get the arXiv URL** - Ask the user for the URL if not provided
2. **Fetch the paper content** from arXiv using WebFetch
   - If URL is abstract page (arxiv.org/abs/...), fetch that
   - Extract title, authors, abstract, and available content
3. If fetch fails:
   - Inform user: "Could not fetch content from arXiv"
   - Suggest: "Try copying the abstract manually to inbox/ and use /analyze"
   - Stop here
4. **Read the analysis prompt** from `prompts/paper.md`
5. **Extract the paper title** from the fetched content
6. **Generate analysis** following the prompt structure exactly
   - Focus on making complex research accessible
   - Use plain English explanations
7. **Save the report** to `reports/papers/YYYY-MM-DD_sanitized-title.md` where:
   - YYYY-MM-DD is today's date
   - sanitized-title is the title in lowercase, spaces replaced with hyphens, special chars removed
8. **Update the activity log** at `logs/YYYY-MM-DD.md`:
   - Create file if it doesn't exist
   - Add entry under "## Papers Reviewed" section
   - Format: `- [Title](../reports/papers/filename.md) - HH:MM`
9. **Confirm to user** what was saved and where

## Report Format

Include this header in the report:
```markdown
# [Paper Title]

**Source**: [arXiv URL]
**Authors**: [Author names if available]
**Date**: YYYY-MM-DD
**Type**: Research Paper

---

[Analysis content following prompts/paper.md structure]

---

## My Notes

[Empty space for user notes]
```

## Error Handling

- If URL is not an arXiv URL: Inform user and suggest correct format
- If paper not found: Tell user to verify the paper ID
- If prompts/paper.md missing: Use a basic summary structure

## Related

- Slash command equivalent: `/arxiv <url>`
- Prompt file: `prompts/paper.md`
- Output location: `reports/papers/`
