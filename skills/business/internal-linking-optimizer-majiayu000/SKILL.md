---
name: internal-linking-optimizer
description: 'Use when the user asks to "fix internal links", "improve site architecture", "link structure", "distribute page authority", "internal linking strategy", "orphan pages", "site architecture is messy", or "pages have no links pointing to them". Analyzes and optimizes internal link structure to improve site architecture, distribute page authority, and help search engines understand content relationships. Creates strategic internal linking plans. For a broader on-page audit, see on-page-seo-auditor. For external link analysis, see backlink-analyzer.'
license: Apache-2.0
metadata:
  author: aaron-he-zhu
  version: "2.0.0"
  geo-relevance: "low"
  tags:
    - seo
    - internal linking
    - site architecture
    - link structure
    - page authority
    - link equity
    - content silos
    - navigation optimization
  triggers:
    - "fix internal links"
    - "improve site architecture"
    - "link structure"
    - "distribute page authority"
    - "internal linking strategy"
    - "site navigation"
    - "link equity"
    - "orphan pages"
    - "site architecture is messy"
    - "pages have no links pointing to them"
---

# Internal Linking Optimizer


> **[SEO & GEO Skills Library](https://skills.sh/aaron-he-zhu/seo-geo-claude-skills)** · 20 skills for SEO + GEO · Install all: `npx skills add aaron-he-zhu/seo-geo-claude-skills`

<details>
<summary>Browse all 20 skills</summary>

**Research** · [keyword-research](../../research/keyword-research/) · [competitor-analysis](../../research/competitor-analysis/) · [serp-analysis](../../research/serp-analysis/) · [content-gap-analysis](../../research/content-gap-analysis/)

**Build** · [seo-content-writer](../../build/seo-content-writer/) · [geo-content-optimizer](../../build/geo-content-optimizer/) · [meta-tags-optimizer](../../build/meta-tags-optimizer/) · [schema-markup-generator](../../build/schema-markup-generator/)

**Optimize** · [on-page-seo-auditor](../on-page-seo-auditor/) · [technical-seo-checker](../technical-seo-checker/) · **internal-linking-optimizer** · [content-refresher](../content-refresher/)

**Monitor** · [rank-tracker](../../monitor/rank-tracker/) · [backlink-analyzer](../../monitor/backlink-analyzer/) · [performance-reporter](../../monitor/performance-reporter/) · [alert-manager](../../monitor/alert-manager/)

**Cross-cutting** · [content-quality-auditor](../../cross-cutting/content-quality-auditor/) · [domain-authority-auditor](../../cross-cutting/domain-authority-auditor/) · [entity-optimizer](../../cross-cutting/entity-optimizer/) · [memory-management](../../cross-cutting/memory-management/)

</details>

This skill analyzes your site's internal link structure and provides recommendations to improve SEO through strategic internal linking. It helps distribute authority, establish topical relevance, and improve crawlability.

## When to Use This Skill

- Improving site architecture for SEO
- Distributing authority to important pages
- Fixing orphan pages with no internal links
- Creating topic cluster internal link strategies
- Optimizing anchor text for SEO
- Recovering pages that have lost rankings
- Planning internal links for new content

## What This Skill Does

1. **Link Structure Analysis**: Maps current internal linking patterns
2. **Authority Flow Mapping**: Shows how PageRank flows through site
3. **Orphan Page Detection**: Finds pages with no internal links
4. **Anchor Text Optimization**: Improves anchor text diversity
5. **Topic Cluster Linking**: Creates pillar-cluster link strategies
6. **Link Opportunity Finding**: Identifies where to add links
7. **Navigation Optimization**: Improves site-wide link elements

## How to Use

### Analyze Current Structure

```
Analyze internal linking structure for [domain/sitemap]
```

```
Find internal linking opportunities for [URL]
```

### Create Linking Strategy

```
Create internal linking plan for topic cluster about [topic]
```

```
Suggest internal links for this new article: [content/URL]
```

### Fix Issues

```
Find orphan pages on [domain]
```

```
Optimize anchor text across the site
```

## Data Sources

> See [CONNECTORS.md](../../CONNECTORS.md) for tool category placeholders.

**With ~~web crawler + ~~analytics connected:**
Claude can automatically perform a full site crawl via ~~web crawler to map the complete link graph, fetch page performance metrics from ~~analytics to identify high-value pages, and analyze link flow throughout the site. This enables data-driven internal linking strategies.

**With manual data only:**
Ask the user to provide:
1. Sitemap URL or list of important pages
2. Key page URLs that need more internal links
3. Content categories or topic clusters
4. Any existing link structure documentation

Proceed with the analysis using provided data. Note in the output which findings are from automated crawl vs. manual review.

## Instructions

When a user requests internal linking optimization:

1. **Analyze Current Internal Link Structure**

   ```markdown
   ## Internal Link Structure Analysis
   
   ### Overview
   
   **Domain**: [domain]
   **Total Pages Analyzed**: [X]
   **Total Internal Links**: [X]
   **Average Links per Page**: [X]
   
   ### Link Distribution
   
   | Links per Page | Page Count | Percentage |
   |----------------|------------|------------|
   | 0 (Orphan) | [X] | [X]% |
   | 1-5 | [X] | [X]% |
   | 6-10 | [X] | [X]% |
   | 11-20 | [X] | [X]% |
   | 20+ | [X] | [X]% |
   
   ### Top Linked Pages
   
   | Page | Internal Links | Authority | Notes |
   |------|----------------|-----------|-------|
   | [URL 1] | [X] | High | [notes] |
   | [URL 2] | [X] | High | [notes] |
   | [URL 3] | [X] | Medium | [notes] |
   
   ### Under-Linked Important Pages
   
   | Page | Current Links | Traffic | Recommended Links |
   |------|---------------|---------|-------------------|
   | [URL 1] | [X] | [X]/mo | [X]+ |
   | [URL 2] | [X] | [X]/mo | [X]+ |
   
   **Structure Score**: [X]/10
   ```

2. **Identify Orphan Pages**

   ```markdown
   ## Orphan Page Analysis
   
   ### Definition
   Orphan pages have no internal links pointing to them, making them 
   hard for users and search engines to discover.
   
   ### Orphan Pages Found: [X]
   
   | Page | Traffic | Priority | Recommended Action |
   |------|---------|----------|-------------------|
   | [URL 1] | [X]/mo | High | Link from [pages] |
   | [URL 2] | [X]/mo | Medium | Add to navigation |
   | [URL 3] | 0 | Low | Consider deleting/redirecting |
   
   ### Fix Strategy
   
   **High Priority Orphans** (have traffic/rankings):
   1. [URL] - Add links from: [relevant pages]
   2. [URL] - Add links from: [relevant pages]
   
   **Medium Priority Orphans** (potentially valuable):
   1. [URL] - Add to category/tag page
   2. [URL] - Link from related content
   
   **Low Priority Orphans** (consider removing):
   1. [URL] - Redirect to [better page]
   2. [URL] - Delete or noindex
   ```

3. **Analyze Anchor Text Distribution**

   > **CORE-EEAT alignment**: Internal linking quality maps to R08 (Internal Link Graph) in the CORE-EEAT benchmark -- use descriptive anchors, ensure links support topical authority. See [content-quality-auditor](../../cross-cutting/content-quality-auditor/) for full audit.

   ```markdown
   ## Anchor Text Analysis
   
   ### Current Anchor Text Patterns
   
   **Most Used Anchors**:
   
   | Anchor Text | Count | Target Pages | Assessment |
   |-------------|-------|--------------|------------|
   | "click here" | [X] | [X] pages | ❌ Not descriptive |
   | "read more" | [X] | [X] pages | ❌ Not descriptive |
   | "[exact keyword]" | [X] | [page] | ⚠️ May be over-optimized |
   | "[descriptive phrase]" | [X] | [page] | ✅ Good |
   
   ### Anchor Text Distribution by Page
   
   **Page: [Important URL]**
   
   | Anchor Text | Source Page | Status |
   |-------------|-------------|--------|
   | "[anchor 1]" | [source URL] | ✅/⚠️/❌ |
   | "[anchor 2]" | [source URL] | ✅/⚠️/❌ |
   
   **Issues Found**:
   - Over-optimized anchors: [X] instances
   - Generic anchors: [X] instances
   - Same anchor to multiple pages: [X] instances
   
   ### Anchor Text Recommendations
   
   **For Page: [URL]**
   
   Current: "[current anchor]" used [X] times
   
   Recommended variety:
   - "[variation 1]" - Use from [page type]
   - "[variation 2]" - Use from [page type]
   - "[variation 3]" - Use from [page type]
   
   **Anchor Score**: [X]/10
   ```

4. **Create Topic Cluster Link Strategy**

   ```markdown
   ## Topic Cluster Internal Linking
   
   ### Cluster: [Main Topic]
   
   **Pillar Page**: [URL]
   **Cluster Articles**: [X]
   
   ### Current Link Map
   
   ```
   [Pillar Page]
      ├── [Cluster Article 1] ←→ [linked?]
      ├── [Cluster Article 2] ←→ [linked?]
      ├── [Cluster Article 3] ←→ [linked?]
      └── [Cluster Article 4] ←→ [linked?]
   ```
   
   ### Recommended Link Structure
   
   ```
   [Pillar Page]
      ├── Links TO all cluster articles ✅
      │
      ├── [Cluster Article 1]
      │   ├── Link TO pillar ✅
      │   └── Link TO related cluster articles
      │
      ├── [Cluster Article 2]
      │   ├── Link TO pillar ✅
      │   └── Link TO related cluster articles
      │
      └── [etc.]
   ```
   
   ### Links to Add
   
   | From Page | To Page | Anchor Text | Location |
   |-----------|---------|-------------|----------|
   | [URL 1] | [URL 2] | "[anchor]" | [paragraph/section] |
   | [URL 2] | [URL 3] | "[anchor]" | [paragraph/section] |
   | [Pillar] | [Cluster 1] | "[anchor]" | [section] |
   ```

5. **Find Contextual Link Opportunities**

   ```markdown
   ## Contextual Link Opportunities
   
   ### Link Opportunity Analysis
   
   For each page, find relevant pages to link to based on:
   - Topic relevance
   - Keyword overlap
   - User journey logic
   - Authority distribution needs
   
   ### Opportunities Found
   
   **Page: [URL 1]**
   **Topic**: [topic]
   **Current internal links**: [X]
   
   | Opportunity | Target Page | Anchor Text | Why Link |
   |-------------|-------------|-------------|----------|
   | Paragraph 2 mentions "[topic]" | [URL] | "[topic phrase]" | Topic match |
   | Section on "[subject]" | [URL] | "[anchor]" | Related guide |
   | CTA at end | [URL] | "[anchor]" | User journey |
   
   **Page: [URL 2]**
   [Continue for each page...]
   
   ### Priority Link Additions
   
   **High Impact Links** (add these first):
   
   1. **From**: [Source URL]
      **To**: [Target URL]
      **Anchor**: "[anchor text]"
      **Why**: [reason - e.g., "Target page needs authority boost"]
      **Where to add**: [specific location in content]
   
   2. **From**: [Source URL]
      **To**: [Target URL]
      [etc.]
   ```

6. **Optimize Navigation and Footer Links**

   ```markdown
   ## Site-Wide Link Optimization
   
   ### Current Navigation Analysis
   
   **Main Navigation**:
   - Links present: [list]
   - Missing important pages: [list]
   - Too many links: [Yes/No]
   
   **Footer Navigation**:
   - Links present: [list]
   - SEO value: [assessment]
   
   ### Navigation Recommendations
   
   | Element | Current | Recommended | Reason |
   |---------|---------|-------------|--------|
   | Main nav | [X] links | [Y] links | [reason] |
   | Footer | [X] links | [Y] links | [reason] |
   | Sidebar | [status] | [recommendation] | [reason] |
   | Breadcrumbs | [status] | [recommendation] | [reason] |
   
   ### Pages to Add to Navigation
   
   1. [Page] - Add to [location] because [reason]
   2. [Page] - Add to [location] because [reason]
   
   ### Pages to Remove from Navigation
   
   1. [Page] - Move to [footer/remove] because [reason]
   ```

7. **Generate Link Implementation Plan**

   ```markdown
   # Internal Linking Optimization Plan
   
   **Site**: [domain]
   **Analysis Date**: [date]
   
   ## Executive Summary
   
   - Total link opportunities found: [X]
   - Orphan pages to fix: [X]
   - Estimated traffic impact: [+X%]
   - Priority actions: [X]
   
   ## Current State
   
   | Metric | Current | Target | Gap |
   |--------|---------|--------|-----|
   | Avg links per page | [X] | [X] | [X] |
   | Orphan pages | [X] | 0 | [X] |
   | Over-optimized anchors | [X]% | <10% | [X]% |
   | Topic cluster coverage | [X]% | 100% | [X]% |
   
   ## Priority Actions
   
   ### Phase 1: Critical Fixes (Week 1)
   
   **Fix Orphan Pages**:
   - [ ] [URL] - Add links from [X] pages
   - [ ] [URL] - Add links from [X] pages
   
   **High-Value Link Additions**:
   - [ ] Link [Page A] to [Page B] with "[anchor]"
   - [ ] Link [Page A] to [Page C] with "[anchor]"
   
   ### Phase 2: Topic Clusters (Week 2-3)
   
   **Cluster 1: [Topic]**
   - [ ] Ensure pillar links to all [X] cluster articles
   - [ ] Add [X] cross-links between cluster articles
   
   **Cluster 2: [Topic]**
   - [ ] [Tasks]
   
   ### Phase 3: Optimization (Week 4+)
   
   **Anchor Text Diversity**:
   - [ ] Vary anchors for [Page] - currently [X]% exact match
   - [ ] [Additional tasks]
   
   **Navigation Updates**:
   - [ ] Add [Page] to main navigation
   - [ ] Update footer links
   
   ## Implementation Guide
   
   ### Adding Internal Links
   
   Best practices:
   1. Add links contextually within content
   2. Use descriptive anchor text (not "click here")
   3. Link to relevant, helpful pages
   4. Aim for 3-10 internal links per 1,000 words
   5. Vary anchor text for the same target
   
   ### Anchor Text Guidelines
   
   | Type | Example | Usage |
   |------|---------|-------|
   | Exact match | "keyword research" | 10-20% |
   | Partial match | "tips for keyword research" | 30-40% |
   | Branded | "Brand's guide to..." | 10-20% |
   | Natural | "this article", "learn more" | 20-30% |
   
   ## Tracking Success

   Monitor these metrics weekly:
   - [ ] Rankings for target keywords
   - [ ] Traffic to previously orphan pages
   - [ ] Crawl stats in ~~search console
   - [ ] Internal link distribution changes
   ```

## Validation Checkpoints

### Input Validation
- [ ] Site structure or sitemap provided (URL or file)
- [ ] Target pages or topic clusters clearly defined
- [ ] If optimizing specific page, page URL or content provided

### Output Validation
- [ ] Every recommendation cites specific data points (not generic advice)
- [ ] All link suggestions include source page, target page, and recommended anchor text
- [ ] Orphan page lists include URLs and recommended actions
- [ ] Source of each data point clearly stated (~~web crawler data, ~~analytics, user-provided, or manual analysis)

## Example

**User**: "Find internal linking opportunities for my blog post about 'email marketing best practices'"

**Output**:

```markdown
## Internal Linking Opportunities

**Page**: /blog/email-marketing-best-practices/
**Current Internal Links**: 2

### Recommended Links to Add

| Section | Text to Link | Target Page | Anchor |
|---------|--------------|-------------|--------|
| Para 2 | "building your email list" | /blog/grow-email-list/ | "building your email list" |
| Para 5 | "subject lines" | /blog/email-subject-lines/ | "write compelling subject lines" |
| Section on segmentation | "audience segments" | /blog/email-segmentation-guide/ | "segment your audience" |
| CTA section | "marketing automation" | /services/email-automation/ | "email automation services" |
| Conclusion | "email marketing tools" | /blog/best-email-tools/ | "top email marketing tools" |

### Pages That Should Link TO This Article

| Source Page | Location | Anchor Text |
|-------------|----------|-------------|
| /blog/digital-marketing-guide/ | Email section | "email marketing best practices" |
| /services/marketing-services/ | Related content | "email marketing strategies" |
| /blog/lead-generation-tips/ | Email mention | "email marketing techniques" |

### Priority Actions

1. Add 5 outbound internal links (listed above)
2. Request 3 inbound links from related pages
3. Add to "Marketing" category page
```

## Tips for Success

1. **Quality over quantity** - Add relevant links, not random ones
2. **User-first thinking** - Links should help users navigate
3. **Vary anchor text** - Avoid over-optimization
4. **Link to important pages** - Distribute authority strategically
5. **Regular audits** - Internal links need maintenance as content grows

## Link Architecture Patterns

### Common Architecture Models

| Model | Description | Best For | Limitations |
|-------|------------|---------|------------|
| **Hub-and-Spoke** | Central pillar links to/from cluster pages | Topic authority, content hubs | Can isolate topic clusters |
| **Silo Structure** | Strict category hierarchies, vertical linking | Large e-commerce, strict taxonomy | Limits cross-topic discovery |
| **Flat Architecture** | All pages 2-3 clicks from homepage | Small sites (<100 pages) | Doesn't scale to large sites |
| **Pyramid** | Homepage → Categories → Subcategories → Pages | News sites, large blogs | Deep pages get less authority |
| **Mesh/Matrix** | Cross-links between related content freely | Knowledge bases, wikis | Can become chaotic without rules |

### Hub-and-Spoke Implementation

```
Homepage
  └── Topic Hub A (pillar page)
  │     ├── Cluster Article A1 ←→ A2
  │     ├── Cluster Article A2 ←→ A3
  │     └── Cluster Article A3 ←→ A1
  └── Topic Hub B (pillar page)
        ├── Cluster Article B1 ←→ B2
        └── Cluster Article B2 ←→ B1

Cross-links: A2 → B1 (related subtopics)
```

## Anchor Text Diversity Framework

### Anchor Text Types

| Type | Example | Target Distribution | Risk Level |
|------|---------|-------------------|------------|
| Exact match | "keyword research tools" | 10-15% | Over-optimization risk if higher |
| Partial match | "best tools for keyword research" | 20-30% | Safe, natural variation |
| Branded | "Ahrefs keyword explorer" | 15-25% | Always safe |
| Generic | "click here", "learn more", "read this" | 5-10% | Low SEO value |
| Descriptive/natural | "this comprehensive guide covers..." | 20-30% | Most natural, recommended |
| Naked URL | "example.com/page" | 5-10% | Natural for citations |

### Anchor Text Best Practices
- Vary anchor text for the same target page
- Use descriptive text that tells users AND search engines what to expect
- Never use identical anchor text for different target pages
- Monitor for accidental over-optimization of commercial keywords

## Link Equity Flow Model

### Link Equity Distribution

| Page Position | Equity Received | Action to Increase |
|--------------|----------------|-------------------|
| Homepage | Highest (all external links flow here) | Distribute to key pages via prominent links |
| Category pages | High (linked from homepage + child pages) | Link to from blog posts, not just nav |
| Top content | Medium-High (if well-linked internally) | Increase internal links from other strong pages |
| Deep pages | Low (few internal links) | Add contextual links from related pages |
| Orphan pages | Zero (no internal links!) | Critical: add at least 3 internal links |

### Link Equity Optimization Rules
1. **Link from strong pages to weak pages** — pages with high authority should link to priority pages
2. **Reduce click depth** — important pages should be within 3 clicks of homepage
3. **Fix orphan pages** — every page needs at least one internal link
4. **Use contextual links** — links within body content pass more value than navigation/footer links
5. **Limit links per page** — diminishing returns above 100 internal links per page

## Internal Link Audit Checklist

| Check | Tool/Method | Pass Criteria |
|-------|------------|--------------|
| Orphan pages | Crawl report | Zero orphan pages |
| Click depth | Crawl report | All priority pages ≤3 clicks from home |
| Broken internal links | Crawl report | Zero 404 internal links |
| Redirect chains | Crawl report | No chain >2 redirects |
| Anchor text diversity | Manual audit | No anchor text >30% exact match |
| Bidirectional links | Manual audit | Related pages link to each other |
| Navigation consistency | Manual audit | Key pages in main nav |
| Contextual links per page | Manual audit | 3-5 contextual links per 1000 words |

## Reference Materials

- [Link Architecture Patterns](./references/link-architecture-patterns.md) — Architecture models, implementation guides, and link equity optimization strategies

## Related Skills

- [content-gap-analysis](../../research/content-gap-analysis/) — Find content to link to
- [seo-content-writer](../../build/seo-content-writer/) — Create linkable content
- [on-page-seo-auditor](../on-page-seo-auditor/) — Audit overall on-page SEO
- [technical-seo-checker](../technical-seo-checker/) — Check crawlability
- [content-quality-auditor](../../cross-cutting/content-quality-auditor/) — Full 80-item CORE-EEAT audit
- [schema-markup-generator](../../build/schema-markup-generator/) — Breadcrumb and navigation schema
