---
name: llm-optimization
description: Optimize websites for AI assistant recommendations. ChatGPT, Gemini, Perplexity, Claude. Get cited in AI answers.
version: 1.0
---

# LLM Optimization Skill

## Purpose

Make websites appear in AI assistant recommendations and citations. Different from traditional SEO - optimized for how LLMs parse and recommend content.

## Core Rules

1. **Structured > Prose** — LLMs extract facts from clear structure
2. **Schema.org is Critical** — Speakable, FAQPage, HowTo schemas
3. **Answer the Question** — First paragraph must directly answer intent
4. **Cite Sources** — Links to authoritative sources build trust
5. **Entity Clarity** — Clear business name, location, service definitions
6. **Freshness Signals** — Last updated dates, recent content
7. **No Walls** — Content must be crawlable, no JS-only rendering
8. **Never Override Truth** — LLM optimization NEVER overrides factual accuracy or legal compliance

## LLM Crawlers to Support

| LLM | Crawlers | Notes |
|-----|----------|-------|
| OpenAI/ChatGPT | GPTBot, OAI-SearchBot, ChatGPT-User | GPTBot = training, others = real-time |
| Google Gemini | Google-Extended | robots.txt control token, not a distinct UA |
| Perplexity | PerplexityBot, Perplexity-User | Bot = indexing, User = real-time fetch |
| Claude | ClaudeBot, Claude-User, Claude-SearchBot | Official Anthropic crawlers |
| Microsoft Copilot | Bingbot | Uses Bing's crawler |

## robots.txt Configuration

```txt
# OpenAI crawlers
User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

# Google AI (control token)
User-agent: Google-Extended
Allow: /

# Perplexity crawlers
User-agent: PerplexityBot
Allow: /

User-agent: Perplexity-User
Allow: /

# Anthropic/Claude crawlers
User-agent: ClaudeBot
Allow: /

User-agent: Claude-User
Allow: /

User-agent: Claude-SearchBot
Allow: /
```

## Content Structure for LLM Extraction

```html
<!-- 1. Direct Answer (first 150 chars) -->
<p class="lead">
  [Business Name] provides [service] in [location].
  [Key differentiator]. [Call to action].
</p>

<!-- 2. Quick Facts Box -->
<aside class="quick-facts" itemscope itemtype="https://schema.org/LocalBusiness">
  <h2>Quick Facts</h2>
  <dl>
    <dt>Service Area</dt><dd itemprop="areaServed">[Areas]</dd>
    <dt>Price Range</dt><dd itemprop="priceRange">[Range]</dd>
  </dl>
</aside>

<!-- 3. FAQ Section (critical for LLM) -->
<section itemscope itemtype="https://schema.org/FAQPage">
  <!-- Each Q&A as schema -->
</section>
```

## Forbidden

- ❌ Content behind JavaScript-only rendering
- ❌ Blocking LLM crawlers in robots.txt
- ❌ Missing Speakable schema
- ❌ Vague, marketing-speak first paragraphs
- ❌ No FAQ section on service pages
- ❌ Missing lastModified dates
- ❌ No structured data

## Definition of Done

- [ ] robots.txt allows all LLM crawlers
- [ ] Speakable schema on all key pages
- [ ] FAQPage schema on service pages
- [ ] First paragraph directly answers search intent
- [ ] Quick Facts box with structured data
- [ ] lastModified meta tag present
- [ ] Content renders without JavaScript
- [ ] Entity names consistent across site

## References

- [ChatGPT Optimization](references/chatgpt-optimization.md)
- [Gemini Optimization](references/gemini-optimization.md)
- [Perplexity Optimization](references/perplexity-optimization.md)
- [Schema Speakable](references/schema-speakable.md)
