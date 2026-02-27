---
name: SEO GEO (Generative Engine Optimization)
description: Optimization for AI-powered search engines and generative answer systems as of February 2026.
---

# GEO (Generative Engine Optimization) Skill

## Purpose

Optimize content for visibility in AI-powered search experiences including Google AI Overviews, ChatGPT Search, Perplexity, Bing Copilot, and other generative answer engines. GEO represents the evolution of SEO for the AI search era, where content must be structured for both traditional crawlers and large language model citation.

## Key Statistics (February 2026)

| Metric | Value | Source |
|--------|-------|--------|
| Google AI Overviews monthly users | 1.5 billion | Google I/O 2025 |
| ChatGPT weekly active users | 900 million | OpenAI Q4 2025 |
| Brand mentions correlation with AI visibility | 3x stronger than backlinks | Ahrefs December 2025 study |
| AI Overview appearance rate (US) | ~35% of informational queries | BrightEdge Q1 2026 |
| Users who click through from AI answers | 42% (when citation is present) | Rand Fishkin / SparkToro 2025 |
| Content with structured data cited by AI | 2.1x more often | Semrush AI Search Study 2025 |

## Core GEO Optimization Pillars

### 1. Citability Scoring

AI models select passages to cite based on information density, clarity, and standalone comprehensibility.

**Optimal Passage Characteristics:**

| Factor | Target | Rationale |
|--------|--------|-----------|
| **Passage length** | 134 to 167 words | Research shows AI models preferentially cite passages in this range. Too short lacks context; too long gets truncated. |
| **Self-contained meaning** | Each paragraph should make sense without surrounding context | AI models extract individual passages, not full pages. |
| **Factual density** | Include specific numbers, dates, percentages, or named entities | Concrete data points increase citation probability. |
| **First-sentence clarity** | Lead with the key claim or answer | Models often use the first sentence of a paragraph as the citation anchor. |
| **Definitional structure** | Use "[Term] is [definition]" patterns for key concepts | Direct definitions are highly citable. |

**Citability Checklist:**
- Does each key paragraph answer a specific question?
- Can each paragraph be extracted and still make complete sense?
- Does the passage contain at least one specific data point, statistic, or named reference?
- Is the key claim in the first sentence, not buried in the middle?
- Is the language clear and unambiguous (no pronoun-heavy references to other paragraphs)?

### 2. Structural Readability

AI models parse document structure to identify relevant sections. Optimize structure for machine comprehension:

**Heading Hierarchy:**
- Use a single H1 that clearly states the page topic
- H2 headings should map to distinct subtopics (treat them like chapter titles)
- H3 headings should address specific questions within each subtopic
- Never skip heading levels (H1 to H3 without H2)

**Content Patterns That Increase AI Selection:**
- **Definition blocks**: "What is X? X is..." at the start of a section
- **Numbered lists**: Step-by-step processes (AI models love ordered procedures)
- **Comparison tables**: Feature matrices, pros/cons, "X vs Y" layouts
- **FAQ sections**: Question-and-answer pairs using `<details>` or heading + paragraph format
- **Summary blocks**: TL;DR sections at the top or bottom of long articles

**Semantic HTML:**
- Use `<article>`, `<section>`, `<aside>`, `<nav>` elements appropriately
- Use `<table>` for tabular data (not CSS grid layouts pretending to be tables)
- Use `<blockquote>` for expert quotes and citations
- Use `<cite>` elements for source attribution

### 3. Multi-Modal Content

Content that includes multiple formats is selected 156% more often by generative engines compared to text-only content.

| Content Type | AI Selection Boost | Implementation |
|-------------|-------------------|----------------|
| **Text + Image** | +67% | Descriptive alt text, contextual captions, images that illustrate data points |
| **Text + Table** | +89% | Structured data tables with clear headers and labeled values |
| **Text + Video** | +112% | Video transcripts, timestamp summaries, key takeaway lists |
| **Text + Image + Table** | +156% | Combined formats on the same page for maximum selection probability |

**Image Optimization for AI:**
- Alt text must describe what the image shows AND why it matters (not just "chart")
- Use descriptive filenames: `startup-failure-rates-by-industry-2026.jpg` not `img_001.jpg`
- Include image captions that add context not present in alt text
- Create original charts and infographics (AI models can reference unique visual data)

### 4. Authority and Brand Signals

Brand mentions now correlate 3x more strongly than backlinks with AI visibility (Ahrefs, December 2025). This represents a fundamental shift in how authority is measured.

**Brand Signal Optimization:**
- Maintain consistent brand mentions across authoritative third-party sites
- Pursue expert quotes and thought leadership in industry publications
- Build presence on platforms AI models train on (Wikipedia, Reddit, Stack Overflow, industry forums)
- Create original research and data that others cite
- Ensure consistent NAP (Name, Address, Phone) across all business listings
- Claim and optimize knowledge panels and entity associations

**E-E-A-T for AI:**
- **Experience**: Include first-person case studies and real examples
- **Expertise**: Author bios with credentials, linked author pages
- **Authoritativeness**: Brand mentions in trusted publications, awards, partnerships
- **Trustworthiness**: Transparent sourcing, correction policies, "last updated" dates

### 5. AI Crawler Detection and Access

Modern AI search requires granting crawler access. Here are the key AI crawlers to manage:

| Crawler | Organization | Purpose | User-Agent String |
|---------|-------------|---------|-------------------|
| **GPTBot** | OpenAI | ChatGPT training and search | `GPTBot` |
| **OAI-SearchBot** | OpenAI | ChatGPT Search real-time results | `OAI-SearchBot` |
| **ClaudeBot** | Anthropic | Claude training data | `ClaudeBot` |
| **anthropic-ai** | Anthropic | Claude web retrieval | `anthropic-ai` |
| **PerplexityBot** | Perplexity | Perplexity search and answers | `PerplexityBot` |
| **AppleBot-Extended** | Apple | Apple Intelligence, Siri | `AppleBot-Extended` |
| **Bytespider** | ByteDance | TikTok AI features | `Bytespider` |
| **Meta-ExternalAgent** | Meta | Meta AI assistant | `Meta-ExternalAgent` |
| **cohere-ai** | Cohere | Cohere model training | `cohere-ai` |
| **Googlebot** | Google | Google Search and AI Overviews | `Googlebot` |

**Robots.txt Strategy:**

```
# Allow all AI search crawlers
User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: AppleBot-Extended
Allow: /

User-agent: Meta-ExternalAgent
Allow: /

# Block training-only crawlers if desired (optional)
# User-agent: Bytespider
# Disallow: /

# User-agent: cohere-ai
# Disallow: /
```

### 6. llms.txt Standard

The `llms.txt` file (placed at site root) provides AI models with structured context about your site:

```markdown
# Site Name

> Brief description of what this site/product does.

## Docs

- [Getting Started](https://example.com/docs/getting-started): How to set up the product
- [API Reference](https://example.com/docs/api): Complete API documentation
- [Pricing](https://example.com/pricing): Plans and pricing information

## Blog

- [Latest Post Title](https://example.com/blog/latest-post): Description of the post

## Optional

- [About Us](https://example.com/about): Company background and mission
```

Place at: `https://yourdomain.com/llms.txt`

An extended version (`llms-full.txt`) can include complete content for AI models that support it.

### 7. RSL 1.0 (Responsible Source Licensing)

RSL 1.0 is an emerging framework for declaring how AI models may use your content:

| License Level | Meaning |
|---------------|---------|
| **RSL-Open** | Content may be used for training and cited in AI outputs |
| **RSL-Cite** | Content may be used if proper attribution/citation is provided |
| **RSL-NoTrain** | Content may be cited in search results but not used for model training |
| **RSL-Restrict** | Content may not be used for training or direct citation |

Implement via meta tag: `<meta name="rsl" content="RSL-Cite">`

Or via HTTP header: `X-RSL: RSL-Cite`

## Platform Specific Optimization

### Google AI Overviews (AIO)

- Triggered primarily by informational and "how to" queries
- Sources from top-ranking pages (traditional SEO still matters)
- Prefers content with clear, factual, well-structured answers
- FAQ sections and definition blocks are frequently pulled
- Structured data (FAQ schema, HowTo schema) increases selection probability

### ChatGPT Search

- Uses Bing index as primary data source
- Real-time web browsing via OAI-SearchBot
- Cites specific passages with direct links
- Prefers authoritative, recently updated content
- Strong preference for content with dates and specificity

### Perplexity

- Crawls the web independently via PerplexityBot
- Provides inline citations with source numbers
- Prefers concise, factual, well-sourced content
- Strong preference for content that includes statistics and data
- Multiple sources are cited per answer (aim to be one of 3 to 5 sources)

### Bing Copilot

- Integrated into Bing search results
- Uses Bing's existing index and ranking signals
- Bing Webmaster Tools optimization carries over
- IndexNow protocol for faster content discovery
- Prefers structured, authoritative content with clear entity relationships

## Rules

1. ALWAYS optimize for citability first. If a passage cannot stand alone, it will not be cited.
2. ALWAYS include specific data points (numbers, percentages, dates) in key paragraphs.
3. ALWAYS maintain proper heading hierarchy for machine parsing.
4. ALWAYS use multi-modal content (text + tables + images minimum) on important pages.
5. ALWAYS allow AI search crawlers (GPTBot, OAI-SearchBot, PerplexityBot, ClaudeBot) in robots.txt unless there is a specific legal or business reason to block them.
6. ALWAYS create and maintain an `llms.txt` file at the site root.
7. NEVER sacrifice traditional SEO for GEO. They are complementary, not competing.
8. NEVER use AI-generated content without human review and factual verification. AI models deprioritize content that appears auto-generated.
9. ALWAYS include author attribution and "last updated" dates on content pages.
10. ALWAYS pursue brand mentions on authoritative platforms as a primary authority signal.
11. ALWAYS structure FAQ sections as explicit question-and-answer pairs.
12. ALWAYS write passages in the 134 to 167 word sweet spot for maximum citability.
