---
name: add-paper
description: Add a research paper to the maxpool research-papers collection. Use when the user provides an ArXiv URL, PDF link, or asks to add/summarize a research paper for the website. Handles paper fetching, insight extraction, HTML generation, and index updates.
allowed-tools: WebFetch, Read, Write, Edit, Glob, Grep, Bash(ls:*), Bash(mkdir:*)
---

# Add Research Paper Skill

This skill automates adding research papers to the `/research-papers/` directory following the established template patterns.

## When to Use This Skill

Trigger this skill when the user:
- Provides an ArXiv URL (html, abs, or pdf format)
- Asks to "add a paper" or "create a research summary"
- Wants to include a paper in the research-papers collection

## Input URL Handling

The user may provide URLs in different formats. Convert them to HTML format for best parsing:

| Input Format | Example | Convert To |
|--------------|---------|------------|
| ArXiv HTML | `arxiv.org/html/2512.04123v1` | Use as-is (preferred) |
| ArXiv Abstract | `arxiv.org/abs/2512.04123v1` | `arxiv.org/html/2512.04123v1` |
| ArXiv PDF | `arxiv.org/pdf/2512.04123v1.pdf` | `arxiv.org/html/2512.04123v1` |
| AlphaXiv | `alphaxiv.org/abs/2512.04123` | Fetch directly |

## Workflow

### Step 1: Fetch and Parse Paper

1. Use `WebFetch` to retrieve the paper content from the HTML URL
2. Extract these key elements:
   - **Title**: Full paper title
   - **Authors**: Author names and affiliations
   - **Date**: Publication/submission date
   - **Abstract**: Paper abstract
   - **Key Sections**: Introduction, methodology, results, conclusion

### Step 2: Analyze and Extract Insights

From the paper content, identify:

1. **3-5 Key Findings** - The most important discoveries or contributions
2. **Performance Metrics** - Specific percentages, improvements, benchmarks
   - Use `<span class="performance-improvement">+24%</span>` for improvements
   - Use `<span class="performance-decline">-15%</span>` for declines
   - Use `<span class="metric">73%</span>` for neutral metrics
3. **Methodology** - How the research was conducted
4. **Benchmarks/Datasets** - What was tested and where
5. **Figure URLs** - Extract image URLs from the HTML for inclusion

### Step 3: Extract Figure URLs and Determine Sizing

For ArXiv HTML pages, figures are typically at:
- Direct `<img>` tags in the HTML content
- ArXiv CDN paths

For AlphaXiv papers, use pattern:
```
https://paper-assets.alphaxiv.org/figures/[paper-id]/img-[N].jpeg
```

#### Image Sizing Guidelines

**IMPORTANT**: Paper figures vary greatly in size. Apply appropriate sizing based on content type:

| Figure Type | Recommended Width | Usage |
|-------------|-------------------|-------|
| Architecture diagrams | `max-width: 100%` | Full width for complex diagrams |
| Most paper figures | `max-width: 800px` | **Default** - bar charts, results, comparisons |
| Charts with legends | `max-width: 750px` | Pie charts, distribution charts |
| Tables as images | `max-width: 800px` | Comparison tables, results |
| Simple diagrams | `max-width: 650px` | Flowcharts, concept illustrations |
| Very small figures | `max-width: 500px` | Icons, simple symbols |

**Apply sizing using inline style on the `<img>` tag:**

**IMPORTANT**: Use `width: 100%; max-width: Xpx;` — not just `max-width` alone!
- `max-width` alone won't scale up small images
- `width: 100%` makes the image fill its container
- `max-width` then caps it at a reasonable size

```html
<!-- Full width for architecture diagrams -->
<div class="figure">
    <img src="..." alt="..." style="width: 100%;">
    <div class="figure-caption">Figure 1: System Architecture</div>
</div>

<!-- Default for most figures (RECOMMENDED) -->
<div class="figure">
    <img src="..." alt="..." style="width: 100%; max-width: 800px;">
    <div class="figure-caption">Figure 2: Performance Results</div>
</div>

<!-- Smaller figures -->
<div class="figure">
    <img src="..." alt="..." style="width: 100%; max-width: 650px;">
    <div class="figure-caption">Figure 3: Concept Diagram</div>
</div>
```

**Rules of thumb:**
1. Academic paper figures are usually high-resolution → use **800px as default**
2. If the figure has lots of small text/labels → use 100% width
3. Only constrain to 500-650px for genuinely small/simple diagrams
4. **When in doubt, use `max-width: 800px`** — too small is worse than too large
5. Always keep `max-width: 100%` in the base CSS to prevent overflow

### Step 4: Generate HTML File

1. Read the template from `TEMPLATE.md` in this skill directory
2. Create file: `/research-papers/[topic]_report.html`
3. Use descriptive filename (e.g., `agent_memory_systems_report.html`)

**Required Sections:**

```
1. Navigation header (copy from template)
2. Title with <br> for subtitle
3. Authors div with date
4. Executive Summary (.abstract box) - 2-3 paragraphs with metrics
5. ELI5 box (.eli5-box) - Simple analogy for non-experts
6. Key figures from the paper (.figure with .figure-caption)
7. Part-based content (Part 1:, Part 2:, etc.)
8. Key finding boxes (.key-finding) for important discoveries
9. Methodology boxes (.methodology-box) for methods
10. Tables for comparisons and benchmarks
11. Conclusion box (.conclusion-box) with summary
12. Source box (.source-box) with original paper link
13. Navigation footer (same as header)
```

### Step 5: Update Index

Edit `/research-papers/index.html` to add a new table row:

```html
<tr>
    <td>
        <a href="[filename].html" target="_blank" class="paper-link">
            <strong>[Paper Title]</strong>
        </a>
    </td>
    <td class="description">[150-200 word description with key metrics, methods, and improvements. Focus on specific numbers and practical implications.]</td>
</tr>
```

Add the new entry after the last `<tr>` in the `<tbody>`.

### Step 6: Validate

Before completing, verify:

- [ ] All required sections present
- [ ] Figure URLs are valid and load
- [ ] Metrics highlighted with appropriate classes
- [ ] Navigation links are correct (`../index.html`, etc.)
- [ ] ELI5 explanation is genuinely simple
- [ ] Source box links to original paper
- [ ] Index.html entry added with description

## Quality Guidelines

### Content Standards

1. **Information Density**: Pack maximum useful information per paragraph
2. **Specific Metrics**: Use "24%" not "significant improvement"
3. **ELI5 Analogies**: Use real-world comparisons anyone can understand
4. **Part Structure**: Organize as "Part 1:", "Part 2:", etc.
5. **Key Findings**: Highlight in dedicated boxes
6. **Honest Limitations**: Include what the paper doesn't solve

### Description for Index (150-200 words)

The index description should include:
- What the paper introduces (method/framework/analysis)
- Key metrics and improvements (specific percentages)
- Benchmarks or environments tested
- Main implications or breakthroughs
- Academic but accessible tone

## File References

- Template: See `TEMPLATE.md` in this directory for complete HTML structure
- Existing papers: Check `/research-papers/*.html` for style reference
- Guidelines: See `/research-papers/CLAUDE.md` for detailed documentation

## Example Usage

User: "Add this paper: https://arxiv.org/html/2512.04123v1"

1. Fetch the ArXiv HTML page
2. Extract title, authors, abstract, key sections
3. Identify 3-5 key findings with metrics
4. Find figure URLs in the HTML
5. Generate `/research-papers/[topic]_report.html` using template
6. Add entry to `/research-papers/index.html`
7. Report completion with file path
