---
name: seo-geo
description: "Audit website for Generative Engine Optimization (GEO) - optimization for AI search engines like Perplexity, ChatGPT Search, Gemini, and Claude."
autoInvoke: false
priority: high
triggers:
  - "seo:geo"
  - "geo audit"
  - "ai discovery"
  - "perplexity optimization"
  - "chatgpt search"
  - "generative engine optimization"
allowed-tools: Read, Grep, Glob, WebFetch, Bash
---

# SEO GEO (AI Discovery) Command

Audit website for Generative Engine Optimization (GEO) - optimization for AI search engines.

---

## Usage

```bash
# GEO audit on URL
/seo:geo https://myapp.com

# Audit specific page
/seo:geo https://myapp.com/docs/guide

# Check robots.txt AI crawler access
/seo:geo --crawlers

# Check content structure only
/seo:geo --content
```

---

## Checks Performed

### 1. AI Crawler Access

**Check robots.txt for AI bot access:**

```toon
ai_crawlers[6]{bot,engine,status}:
  GPTBot,OpenAI/ChatGPT,Check Allow/Disallow
  Google-Extended,Google Gemini,Check Allow/Disallow
  PerplexityBot,Perplexity,Check Allow/Disallow
  ClaudeBot,Anthropic Claude,Check Allow/Disallow
  Applebot-Extended,Apple Intelligence,Check Allow/Disallow
  CCBot,Common Crawl,Check Allow/Disallow
```

**Recommended robots.txt:**

```txt
User-agent: GPTBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Applebot-Extended
Allow: /
```

### 2. Content Structure

```toon
content_checks[6]{check,requirement}:
  Answer-first,Direct answer in first paragraph
  Heading hierarchy,H1 -> H2 -> H3 (no skipping)
  Semantic HTML,article section aside nav used
  Lists and tables,Structured scannable content
  Word count,Minimum 300 words meaningful content
  Readability,Clear concise language
```

### 3. Entity Signals (E-E-A-T)

```toon
entity_checks[5]{check,implementation}:
  Author schema,Person with credentials + sameAs
  Organization schema,Company info + logo + sameAs
  Author page,Dedicated page with bio
  About page,Company/author information
  External citations,Links to authoritative sources
```

### 4. Freshness Signals

```toon
freshness_checks[4]{check,requirement}:
  datePublished,ISO 8601 format in schema
  dateModified,Updated when content changes
  Visible dates,Human-readable dates on page
  Update history,Revision log for major content
```

### 5. FAQ Implementation

```toon
faq_checks[3]{check,requirement}:
  FAQPage schema,JSON-LD with Question/Answer
  HTML structure,Semantic FAQ markup
  Answer quality,Complete actionable answers
```

---

## Output Format

Generate report with:

| Category | Score | Status |
|----------|-------|--------|
| AI Crawler Access | X/100 | PASS/WARNING/FAIL |
| Content Structure | X/100 | PASS/WARNING/FAIL |
| Entity Signals | X/100 | PASS/WARNING/FAIL |
| Freshness Signals | X/100 | PASS/WARNING/FAIL |
| FAQ Implementation | X/100 | PASS/WARNING/FAIL |

**Overall GEO Score:** X/100

Include:
- robots.txt status per AI bot
- Specific E-E-A-T improvements
- AI Citation test instructions

---

## LLM.txt (Optional)

Consider adding `/llm.txt` for AI-specific instructions:

```txt
# /llm.txt
name: Your Site Name
description: What your site is about
author: Author/Company Name
topics: topic1, topic2, topic3
citation-name: Preferred Citation Name
```

---

## AI Citation Test

Test your content visibility in AI search:

1. **Perplexity:** Search "what is [your topic]"
2. **ChatGPT:** Ask about your expertise area
3. **Gemini:** Query your brand name

---

## Related Commands

| Command | Purpose |
|---------|---------|
| `/seo:check` | Full SEO audit |
| `/seo:schema` | Schema validation |

---

**Version:** 1.20.0
