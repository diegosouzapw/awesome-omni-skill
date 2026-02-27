---
name: seo-geo
description: SEO & GEO (Generative Engine Optimization) for websites. Analyze keywords, generate schema markup, optimize for AI search engines (ChatGPT, Perplexity, Gemini, Copilot, Claude) and traditional search (Google, Bing). Use when user wants to improve search visibility, search optimization, search ranking, AI visibility, ChatGPT ranking, Google AI Overview, indexing, JSON-LD, meta tags, or keyword research.
---

# SEO/GEO Optimization Skill

Comprehensive SEO and GEO (Generative Engine Optimization) for websites. Optimize for both traditional search engines (Google, Bing) and AI search engines (ChatGPT, Perplexity, Gemini, Copilot, Claude).

## Quick Reference

**GEO = Generative Engine Optimization** — Optimizing content to be cited by AI-driven search and answer engines.

**Key Insight:** AI search engines don't rank pages — they **cite sources**. Being cited is the new "ranking #1". Traditional SEO = rank on Google. GEO = get cited in AI answers. Completely different game.

## Workflow

### Step 1: Website Audit

Get the target URL and analyze current SEO/GEO status.

**Basic SEO Audit (Free):**
```bash
python3 scripts/seo_audit.py "https://example.com"
```
**Use this for**: Quick technical SEO check (title, meta, H1, robots, sitemap, load time). No API needed.

---

**Check Meta Tags:**
```bash
curl -sL "https://example.com" | grep -E "<title>|<meta name=\"description\"|<meta property=\"og:|application/ld\+json" | head -20
```

**Check robots.txt:**
```bash
curl -s "https://example.com/robots.txt"
```

**Check sitemap:**
```bash
curl -s "https://example.com/sitemap.xml" | head -50
```

**Verify AI Bot Access:**
```
# These bots MUST be allowed in robots.txt:
- Googlebot (Google + Google AI Overview)
- Bingbot (Bing + Microsoft Copilot)
- PerplexityBot (Perplexity)
- ChatGPT-User (ChatGPT with browsing)
- ClaudeBot / anthropic-ai (Claude)
- GPTBot (OpenAI)
```

**Check for llms.txt (optional, experimental):**
```bash
curl -s "https://example.com/llms.txt"
```

### Step 2: Keyword Research

Use **WebSearch** to research target keywords:

```
WebSearch: "{keyword} keyword difficulty site:ahrefs.com OR site:semrush.com"
WebSearch: "{keyword} search volume 2026"
WebSearch: "site:{competitor.com} {keyword}"
```

**Analyze:**
- Search volume and difficulty
- Competitor keyword strategies
- Long-tail keyword opportunities
- **Conversational queries** — people query AI differently than Google: longer, natural-language, comparison/decision questions ("Best X for Y", "Should I use X or Z?", "How does X compare to Y?")

### Step 3: GEO Optimization (AI Search Engines)

#### The 8 GEO Principles

These are the core practices for getting content cited by generative AI engines:

**1. Optimize for Answers, Not Just Keywords**
- Structure content to directly answer specific, high-intent questions
- Use clear headings phrased as questions
- Provide concise, well-structured summaries at the top ("answer-first" format)
- Include definitions, comparisons, and step-by-step explanations

**2. Build Topical Authority, Not Isolated Pages**
- Create tightly connected content clusters around a core theme
- Show depth through subtopics, FAQs, edge cases, and examples
- Avoid thin, single-keyword pages
- Generative engines reward contextual authority over keyword density

**3. Make Content Extractable**
LLMs quote and summarize. Make it easy:
- Use clean structure with H2 and H3 headings
- Bullet points and tables where relevant
- Short, self-contained paragraphs
- Clear definitions and frameworks
- Avoid fluff. Extraction favors clarity.

**4. Embed Entity Clarity**
- Clearly define companies, products, people, and technical terms
- Use consistent naming across your site
- Provide context around what category you belong to and who you are for
- AI systems rely heavily on entity relationships, not just keywords

**5. Optimize for Citability**
Generative engines tend to cite:
- Original data and proprietary research
- Clear frameworks and methodologies
- Strong opinions backed by reasoning
- Well-structured statistics and specific numbers
- If your content looks like generic rephrasing, it will not be surfaced

**6. Strengthen Off-Site Signals**
Mentions matter more than links alone:
- Appear in authoritative publications
- Get cited in niche blogs and industry media
- Be referenced in forums, GitHub, Reddit, technical communities
- LLMs learn from broad web consensus

**7. Improve Technical Accessibility**
- Fast loading, clean HTML
- Avoid content hidden behind JS-heavy rendering
- Clear schema markup (JSON-LD)
- Accessible robots.txt — do not block AI crawlers if you want visibility
- Content behind gates (forms, logins) is invisible to AI engines

**8. Optimize for Conversational Retrieval**
People query AI differently than search:
- Longer, natural-language queries
- Comparison and decision-making questions
- "Best for X" or "Should I use Y or Z?"
- Create content that answers these conversational patterns directly

#### Princeton GEO Methods (Tactical)

Apply these evidence-based methods to boost AI citation rates:

| Method | Visibility Boost | How to Apply |
|--------|-----------------|--------------|
| **Cite Sources** | +40% | Add authoritative citations and references |
| **Statistics Addition** | +37% | Include specific numbers and data points |
| **Quotation Addition** | +30% | Add expert quotes with attribution |
| **Authoritative Tone** | +25% | Use confident, expert language — avoid hedging ("may", "could") |
| **Easy-to-understand** | +20% | Simplify complex concepts for broad accessibility |
| **Technical Terms** | +18% | Include domain-specific terminology |
| **Unique Words** | +15% | Increase vocabulary diversity |
| **Fluency Optimization** | +15-30% | Improve readability and flow |
| ~~Keyword Stuffing~~ | **-10%** | **AVOID — hurts visibility** |

**Best Combination:** Fluency + Statistics = Maximum boost

### Step 4: Traditional SEO Optimization

**Meta Tags, JSON-LD Schema, Content Structure** — see references for templates.

**Content checklist:**
- [ ] H1 contains primary keyword
- [ ] Images have descriptive alt text (not generic filenames)
- [ ] Internal links to related content (content clusters)
- [ ] External links to authoritative sources
- [ ] Content is mobile-friendly
- [ ] Page loads in < 3 seconds

### Step 5: Validate & Monitor

**Schema Validation, Indexing Status, Reporting** — see references for tools.

## Technical Files: What Matters and What Doesn't

### robots.txt — Required (but not for ranking)
- Controls crawler access. Signals whether AI crawlers can see your site.
- Make sure you are NOT blocking OpenAI, Anthropic, Perplexity, etc. unless intentional.
- Many sites accidentally block everything except Googlebot.
- **This does not improve visibility directly. It just prevents you from disappearing.**

### sitemap.xml — Recommended
- Helps search engines and some AI systems discover pages.
- Improves crawl efficiency.
- For content-heavy sites, this is essential.

### llms.txt — Optional, Experimental
There is a trend of adding `/llms.txt` to:
- Explain your site to AI systems
- Define your product clearly
- Provide canonical descriptions
- Give structured summaries

**Reality:** There is no official standard that major LLM providers use systematically. It can help with clarity, but it is not magic. If you do it, treat it like a clean, structured overview of: who you are, what you do, who it's for, and key pages. But do not expect immediate impact.

### Text Files vs. HTML
- Generative engines primarily ingest **web HTML**, not standalone .txt downloads
- Focus on clean HTML, clear entity definition, and structured extractable content
- **That matters far more than any .txt file**

**The real question is not "Do I need txt files?" — it's:**
- Are you clearly defining what category you are in?
- Are you publishing content that answers high-intent questions?
- Are you being cited elsewhere?
- Are you structuring content for extraction?
If not, .txt files won't save you.

## Content Gating and GEO

**Critical:** Gated content (behind forms, logins, paywalls) is invisible to AI search engines. ChatGPT, Perplexity, and Claude cannot read content behind a form.

**Best practice (80/20 rule):**
- **80% ungated**: Blog posts, educational content, case study summaries, comparison pages, pillar pages. These need to rank, get cited, and be shareable.
- **20% gated**: Original research, ROI calculators, detailed technical whitepapers. Gate for intent signal, not lead volume.

**For case studies specifically:** Create an ungated, SEO-optimized web page with the key story and metrics. Keep a detailed PDF version behind a light gate (email only) for people who want the full technical detail.

## Platform-Specific Optimization

### ChatGPT
- Focus on **branded domain authority** (cited 11% more than third-party)
- Update content within **30 days** (3.2x more citations)
- Build **backlinks** (>350K referring domains = 8.4 avg citations)
- Match content style to ChatGPT's response format

### Perplexity
- Allow **PerplexityBot** in robots.txt
- Use **FAQ Schema** (higher citation rate)
- Host **PDF documents** (prioritized for citation)
- Focus on **semantic relevance** over keywords

### Google AI Overview (SGE)
- Optimize for **E-E-A-T** (Experience, Expertise, Authority, Trust)
- Use **structured data** (Schema markup)
- Build **topical authority** (content clusters + internal linking)
- Include **authoritative citations** (+132% visibility)

### Microsoft Copilot / Bing
- Ensure **Bing indexing** (required for citation)
- Optimize for **Microsoft ecosystem** (LinkedIn, GitHub mentions help)
- Page speed **< 2 seconds**
- Clear **entity definitions**

### Claude AI
- Ensure **Brave Search indexing** (Claude uses Brave, not Google)
- High **factual density** (data-rich content preferred)
- Clear **structural clarity** (easy to extract)

## What Actually Moves GEO

When auditing a site, always assess against these core questions:

1. **Category clarity**: Is it immediately obvious what category this company/product is in?
2. **Answer-first content**: Does the content directly answer the questions your ICP is asking?
3. **Citability**: Does the content contain original data, frameworks, or statistics that an AI would want to cite?
4. **Extractability**: Is the content structured so an LLM can easily quote specific sentences or paragraphs?
5. **Off-site signals**: Is the brand being mentioned and cited on authoritative external sites?
6. **Technical access**: Can AI crawlers actually access and read the content?

If the answer to any of these is "no," that's where to focus first.

## References

- [references/platform-algorithms.md](./references/platform-algorithms.md) - Detailed ranking factors for each platform
- [references/geo-research.md](./references/geo-research.md) - Princeton GEO research (9 methods)
- [references/schema-templates.md](./references/schema-templates.md) - JSON-LD templates
- [references/seo-checklist.md](./references/seo-checklist.md) - Complete SEO audit checklist
- [references/tools-and-apis.md](./references/tools-and-apis.md) - Tools and API reference
- [examples/opc-skills-case-study.md](./examples/opc-skills-case-study.md) - Real-world optimization example
