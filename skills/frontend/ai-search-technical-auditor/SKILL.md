---
name: ai-search-technical-auditor
description: Audit front-end code for AI search readiness. Use when reviewing HTML structure, meta tags, schema markup, and technical elements that affect how AI crawlers understand and index web pages.
---

# AI Search Technical Auditor

Audit HTML, meta tags, schema markup, and technical implementation to ensure pages are optimized for AI search engine crawlers.

## Understanding AI Crawlers

AI search engines use different crawlers than traditional search engines. Key AI crawlers include:

| Crawler | Platform | User Agent Contains |
|---------|----------|---------------------|
| GPTBot | OpenAI/ChatGPT | `GPTBot` |
| Google-Extended | Google AI/Gemini | `Google-Extended` |
| ClaudeBot | Anthropic/Claude | `ClaudeBot` |
| PerplexityBot | Perplexity | `PerplexityBot` |
| Bytespider | ByteDance | `Bytespider` |

## Technical Audit Checklist

### 1. robots.txt Configuration

Check that AI crawlers are allowed:

```txt
# Good - Allow AI crawlers
User-agent: GPTBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /
```

```txt
# Bad - Blocking AI crawlers (unless intentional)
User-agent: GPTBot
Disallow: /
```

**Audit questions:**
- [ ] Are AI crawlers explicitly allowed or blocked?
- [ ] Is the intent to block deliberate and documented?
- [ ] Are important content paths accessible?

### 2. HTML Structure

#### Semantic HTML

Check for proper semantic structure:

```html
<!-- Good -->
<article>
  <header>
    <h1>Page Title</h1>
    <p class="summary">TL;DR content here</p>
  </header>
  <section>
    <h2>Section Heading</h2>
    <p>Content...</p>
  </section>
</article>

<!-- Bad -->
<div class="article">
  <div class="title">Page Title</div>
  <div class="content">
    <div class="heading">Section Heading</div>
    <div>Content...</div>
  </div>
</div>
```

**Audit questions:**
- [ ] Uses semantic elements (`<article>`, `<section>`, `<nav>`, `<aside>`)?
- [ ] Heading hierarchy is logical (H1 → H2 → H3)?
- [ ] Only one `<h1>` per page?
- [ ] Main content is in `<main>` element?

#### Content Accessibility

```html
<!-- Good - Content in HTML -->
<h2>Product Features</h2>
<ul>
  <li>Feature one description</li>
  <li>Feature two description</li>
</ul>

<!-- Bad - Content in JavaScript only -->
<div id="features"></div>
<script>
  renderFeatures(); // Content not in initial HTML
</script>
```

**Audit questions:**
- [ ] Key content is in initial HTML (not JavaScript-rendered)?
- [ ] Images have descriptive alt text?
- [ ] Tables have proper headers (`<th>`)?
- [ ] Lists use `<ul>`/`<ol>` elements?

### 3. Meta Tags

#### Essential Meta Tags

```html
<head>
  <!-- Page title - Clear and descriptive -->
  <title>What is GEO? Generative Engine Optimization Guide | Brand</title>
  
  <!-- Meta description - Summarizes page content -->
  <meta name="description" content="GEO (Generative Engine Optimization) 
    is the practice of optimizing content for AI search engines. Learn 
    how to improve visibility in ChatGPT, Perplexity, and Google AI Mode.">
  
  <!-- Canonical URL -->
  <link rel="canonical" href="https://example.com/geo-guide">
  
  <!-- Language -->
  <html lang="en">
  
  <!-- Viewport for mobile -->
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
```

**Audit questions:**
- [ ] Title is descriptive and under 60 characters?
- [ ] Meta description summarizes content (150-160 chars)?
- [ ] Canonical URL is set correctly?
- [ ] Language is declared?
- [ ] Viewport meta tag is present?

#### Open Graph Tags

```html
<meta property="og:title" content="What is GEO?">
<meta property="og:description" content="Guide to Generative Engine Optimization">
<meta property="og:type" content="article">
<meta property="og:url" content="https://example.com/geo-guide">
<meta property="og:image" content="https://example.com/images/geo-guide.jpg">
```

**Audit questions:**
- [ ] OG title and description are set?
- [ ] OG image is specified and valid?
- [ ] OG URL matches canonical?

### 4. Schema Markup (JSON-LD)

#### Article Schema

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "What is Generative Engine Optimization (GEO)?",
  "description": "A comprehensive guide to GEO for AI search visibility",
  "author": {
    "@type": "Person",
    "name": "Author Name",
    "url": "https://example.com/authors/author-name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Company Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  },
  "datePublished": "2024-01-15",
  "dateModified": "2024-03-20",
  "mainEntityOfPage": "https://example.com/geo-guide"
}
</script>
```

#### FAQ Schema

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is GEO?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "GEO (Generative Engine Optimization) is the practice of optimizing content to be cited in AI-generated answers."
      }
    },
    {
      "@type": "Question",
      "name": "How does GEO differ from SEO?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "SEO focuses on search rankings while GEO focuses on being cited in AI answers."
      }
    }
  ]
}
</script>
```

#### HowTo Schema

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Create an llms.txt File",
  "description": "Step-by-step guide to creating llms.txt for AI crawlers",
  "step": [
    {
      "@type": "HowToStep",
      "name": "Audit your content",
      "text": "Identify your most important pages for AI discovery"
    },
    {
      "@type": "HowToStep", 
      "name": "Create the file",
      "text": "Create llms.txt at your domain root"
    }
  ]
}
</script>
```

**Audit questions:**
- [ ] Schema type matches content type?
- [ ] JSON-LD is valid (no syntax errors)?
- [ ] Required properties are populated?
- [ ] Dates are in ISO 8601 format?
- [ ] URLs are absolute, not relative?

### 5. Page Speed

AI crawlers, especially agentic ones, have tight time budgets. Target:

| Metric | Target | Critical for |
|--------|--------|--------------|
| Time to First Byte (TTFB) | < 200ms | All crawlers |
| First Contentful Paint (FCP) | < 1.8s | Content extraction |
| Largest Contentful Paint (LCP) | < 2.5s | Full page analysis |
| **AI Crawler Target** | < 0.4s FCP | Real-time agents |

**Audit questions:**
- [ ] TTFB under 200ms?
- [ ] FCP under 1.8s (ideally under 0.4s)?
- [ ] No render-blocking JavaScript?
- [ ] Images are optimized and lazy-loaded?
- [ ] CSS is minified?

### 6. Mobile Friendliness

```html
<!-- Viewport meta tag -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

**Audit questions:**
- [ ] Viewport meta tag is present?
- [ ] Content is readable without zooming?
- [ ] Touch targets are appropriately sized?
- [ ] No horizontal scrolling required?

### 7. URL Structure

```
Good URLs:
/guides/geo-optimization
/products/analytics-platform
/docs/api-reference

Bad URLs:
/page.php?id=123&cat=5
/guides/geo-optimization-best-practices-guide-2024-updated-version-3
/p/12345
```

**Audit questions:**
- [ ] URLs are descriptive and readable?
- [ ] URLs are not excessively long?
- [ ] URLs use hyphens, not underscores?
- [ ] No query parameters for primary content?

## Audit Report Template

```markdown
# AI Search Technical Audit Report

**URL:** [Page URL]
**Date:** [Audit Date]
**Overall Score:** [X/100]

## Summary
[Brief summary of findings]

## Critical Issues
- [ ] Issue 1
- [ ] Issue 2

## Warnings
- [ ] Warning 1
- [ ] Warning 2

## Passed Checks
- [x] Check 1
- [x] Check 2

## Recommendations
1. [Priority 1 recommendation]
2. [Priority 2 recommendation]
3. [Priority 3 recommendation]
```

## Quick Reference: Schema Types

| Content Type | Schema | Required Properties |
|--------------|--------|---------------------|
| Article | Article | headline, author, datePublished |
| FAQ | FAQPage | mainEntity (array of Questions) |
| How-to | HowTo | name, step (array of HowToSteps) |
| Product | Product | name, description, offers |
| Organization | Organization | name, url, logo |
| Person | Person | name |
| BreadcrumbList | BreadcrumbList | itemListElement |

## Validation Tools

After implementing fixes, validate using:

1. **Schema:** Google Rich Results Test, Schema.org Validator
2. **HTML:** W3C Validator
3. **Speed:** Google PageSpeed Insights, WebPageTest
4. **Mobile:** Google Mobile-Friendly Test
5. **robots.txt:** Google Search Console robots.txt Tester
