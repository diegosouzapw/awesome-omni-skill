---
name: geo-audit
description: Audit and optimize website for AI search engines like ChatGPT, Perplexity, Google AI Overviews, and Claude. Use when discussing GEO (Generative Engine Optimization), SEO for AI, llms.txt, AI crawlers, structured data for LLMs, or visibility in AI search results.
---

# GEO (Generative Engine Optimization) Audit Skill

Analyzes and optimizes websites for AI search engine visibility across ChatGPT, Perplexity, Google AI Overviews, and Claude.

## When This Skill Activates

Claude automatically uses this skill when you mention:
- GEO or Generative Engine Optimization
- AI search optimization
- llms.txt implementation
- Visibility in ChatGPT/Perplexity/Claude
- Structured data for AI
- AI crawler access

## Audit Process

Run 4 parallel subagents for comprehensive analysis:

### Agent 1: Technical GEO Audit
```
Use Task tool with subagent_type='general-purpose':

Check the following in the codebase:
1. llms.txt file existence at public/.well-known/llms.txt or public/llms.txt
2. robots.txt configuration - must allow these AI crawlers:
   - GPTBot (OpenAI)
   - ChatGPT-User
   - PerplexityBot
   - ClaudeBot / Claude-Web
   - Google-Extended
   - Amazonbot
   - cohere-ai
3. Sitemap completeness - all pages included with fresh lastmod dates
4. Static HTML rendering - verify SSG/SSR vs client-only
5. Canonical URLs present and correct

Report: List each item as PASS/FAIL with specific fix instructions.
```

### Agent 2: Structured Data Audit
```
Use Task tool with subagent_type='general-purpose':

Analyze JSON-LD structured data:
1. Check layout.tsx and page components for existing schema
2. Verify required schemas exist:
   - Organization (company info)
   - WebSite (with SearchAction if applicable)
   - SoftwareApplication (for app pages)
   - Article (for blog posts)
   - FAQPage (for FAQ sections)
   - AggregateRating (for reviews)
3. Validate schema completeness per schema.org spec
4. Check for missing entity connections

Report: List schemas found, missing schemas needed, and specific additions.
```

### Agent 3: Content Quality Audit
```
Use Task tool with subagent_type='general-purpose':

Analyze content for AI-friendliness:
1. Meta descriptions - clear, factual, 150-160 chars
2. Heading hierarchy - one H1, logical H2/H3 structure
3. E-E-A-T signals:
   - Expertise: Technical explanations, methodology details
   - Experience: User testimonials, case studies, results
   - Authority: Credentials, certifications, app store ratings
   - Trust: Privacy info, contact details, security badges
4. FAQ-style content that AI engines prefer
5. Citation opportunities to authoritative sources

Report: Content improvements by page with specific suggestions.
```

### Agent 4: Platform Presence Audit
```
Use Task tool with subagent_type='general-purpose':

Check brand visibility on AI-cited platforms:
1. Search for brand mentions on Reddit, ProductHunt, HackerNews
2. Check if Wikipedia presence is warranted
3. Verify app store listings exist and are optimized
4. Check for reviews structured data
5. Analyze social proof elements

Report: Platform presence status and growth opportunities.
```

## Fix Implementation

After audit, offer to implement fixes using these references:

### Technical Fixes
See: [technical-fixes.md](./technical-fixes.md)
- Create llms.txt
- Update robots.txt for AI crawlers
- Fix sitemap issues

### Schema Fixes
See: [schema-reference.md](./schema-reference.md)
- Add missing JSON-LD schemas
- Enhance existing structured data

### Content Fixes
See: [content-reference.md](./content-reference.md)
- Improve meta descriptions
- Add E-E-A-T signals
- Create FAQ content

## Site Context

- **Domain**: hoppa.fit
- **Stack**: Next.js (Pages Router)
- **Purpose**: AI-powered fitness app with pose detection
- **Key Pages**: Home, Forum, Blog, Login, Privacy, Terms

## Priority Framework

1. **Critical** (blocks AI discovery): Missing llms.txt, blocked crawlers, no structured data
2. **High** (significant impact): Incomplete sitemap, weak meta descriptions, missing schemas
3. **Medium** (incremental gains): E-E-A-T enhancements, FAQ content, citation additions
4. **Quick Wins** (easy fixes): robots.txt updates, heading fixes, lastmod dates
