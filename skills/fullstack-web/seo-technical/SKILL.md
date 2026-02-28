---
name: seo-technical
description: Technical SEO audit across 9 categories including crawlability, indexability, security headers, URL structure, mobile optimization, Core Web Vitals, structured data, JavaScript rendering, and IndexNow protocol with AI crawler management.
---

# SEO Technical Skill

## Purpose

Audit the technical SEO foundation of a website across 9 critical categories. Technical SEO determines whether search engines can discover, crawl, render, and index content correctly. This skill identifies infrastructure-level issues that block or degrade organic visibility.

---

## Category 1: Crawlability

### Standard Crawler Management

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| robots.txt exists | Returns 200, valid syntax | Critical |
| robots.txt not blocking key pages | Sitemap, homepage, key landing pages accessible | Critical |
| XML sitemap exists | Referenced in robots.txt or at /sitemap.xml | High |
| Sitemap is valid | Well-formed XML, all URLs return 200 | High |
| Sitemap size | Under 50MB uncompressed; under 50,000 URLs per file | Medium |
| Sitemap index | Used if more than 50,000 URLs | Medium |
| Crawl depth | Key pages within 3 clicks of homepage | High |
| Internal linking | No orphan pages (pages with 0 internal links pointing to them) | High |
| Redirect chains | No chains longer than 2 hops | Medium |
| Redirect loops | Zero redirect loops | Critical |

### AI Crawler Management

Modern SEO must account for AI crawlers that scrape content for training and retrieval-augmented generation. Managing these crawlers is both a content protection and an AI search visibility decision.

#### AI Crawler Tokens

| Crawler | Owner | User-Agent Token | robots.txt Directive |
|---------|-------|------------------|----------------------|
| GPTBot | OpenAI | `GPTBot` | `User-agent: GPTBot` |
| ChatGPT-User | OpenAI | `ChatGPT-User` | `User-agent: ChatGPT-User` |
| OAI-SearchBot | OpenAI | `OAI-SearchBot` | `User-agent: OAI-SearchBot` |
| ClaudeBot | Anthropic | `ClaudeBot` | `User-agent: ClaudeBot` |
| anthropic-ai | Anthropic | `anthropic-ai` | `User-agent: anthropic-ai` |
| PerplexityBot | Perplexity | `PerplexityBot` | `User-agent: PerplexityBot` |
| Bytespider | ByteDance | `Bytespider` | `User-agent: Bytespider` |
| Applebot-Extended | Apple | `Applebot-Extended` | `User-agent: Applebot-Extended` |
| Google-Extended | Google | `Google-Extended` | `User-agent: Google-Extended` |
| FacebookBot | Meta | `FacebookExternalHit` | `User-agent: FacebookExternalHit` |
| cohere-ai | Cohere | `cohere-ai` | `User-agent: cohere-ai` |
| Diffbot | Diffbot | `Diffbot` | `User-agent: Diffbot` |
| Amazonbot | Amazon | `Amazonbot` | `User-agent: Amazonbot` |
| YouBot | You.com | `YouBot` | `User-agent: YouBot` |

#### AI Crawler Strategy Recommendations

| Strategy | When to Use | Implementation |
|----------|-------------|----------------|
| **Allow All** | Want maximum AI search visibility (citations, AI Overviews) | No blocks in robots.txt |
| **Allow Search, Block Training** | Want AI search citations but protect training data | Allow `OAI-SearchBot`, `ChatGPT-User`; Block `GPTBot`, `Google-Extended` |
| **Block All AI** | Protect proprietary content entirely | Block all AI tokens in robots.txt |
| **Selective Allow** | Allow specific AI platforms aligned with business | Whitelist approach: block all, then allow specific tokens |

#### Audit Checks

| Check | What to Verify | Severity |
|-------|----------------|----------|
| AI crawler directives present | robots.txt mentions at least some AI crawlers | Medium |
| Strategy is intentional | Not accidentally blocking AI search crawlers | High |
| Consistency | If blocking GPTBot, consider whether OAI-SearchBot should also be blocked | Medium |
| Content protection aligned | Blocking matches content licensing strategy | Medium |

---

## Category 2: Indexability

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| No unintended noindex | Key pages do not have `<meta name="robots" content="noindex">` | Critical |
| Canonical tags present | All indexable pages have self-referencing canonical | High |
| Canonical consistency | Canonical URL matches actual URL (protocol, www/non-www, trailing slash) | High |
| No canonical conflicts | JavaScript-rendered canonical matches server-rendered canonical | Critical |
| Duplicate content | Content hash comparison; no substantial duplicates without canonical | High |
| Pagination handling | `rel="next"` / `rel="prev"` or load-more pattern (note: Google ignores these but Bing uses them) | Low |
| Thin content pages | Pages with <100 words flagged for review | Medium |
| Parameter handling | URL parameters not creating duplicate indexable pages | High |
| Index coverage | Compare indexed pages (via sitemap) vs crawled pages; identify gaps | High |

---

## Category 3: Security Headers

| Header | Recommended Value | Severity |
|--------|-------------------|----------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | High |
| `X-Content-Type-Options` | `nosniff` | Medium |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Medium |
| `Content-Security-Policy` | Appropriate policy (not overly permissive) | Medium |
| `Referrer-Policy` | `strict-origin-when-cross-origin` or stricter | Low |
| `Permissions-Policy` | Restrict unnecessary APIs (camera, microphone, geolocation) | Low |
| HTTPS enforcement | All HTTP requests redirect to HTTPS | Critical |
| Mixed content | No HTTP resources loaded on HTTPS pages | High |
| SSL certificate | Valid, not expired, covers all subdomains | Critical |

---

## Category 4: URL Structure

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| Clean URLs | Human-readable, descriptive slugs | Medium |
| Consistent format | Consistent trailing slash policy (all with or all without) | High |
| Lowercase | All URLs are lowercase; uppercase variants redirect | Medium |
| No special characters | No spaces, underscores (use hyphens), or encoded characters | Medium |
| Depth | Key pages within 3 levels of hierarchy | Medium |
| Consistent protocol | www vs non-www resolved; one redirects to the other | High |
| No session IDs | URLs do not contain session parameters | High |
| Logical hierarchy | URL path reflects site architecture (`/category/subcategory/page`) | Low |

---

## Category 5: Mobile Optimization

### Mobile-First Indexing (Fully Enforced July 2024)

Google uses the mobile version of content for indexing and ranking for all sites. Desktop-only content that is not present on mobile will not be indexed.

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| Responsive design | Uses viewport meta tag, CSS media queries | Critical |
| Mobile content parity | Same content on mobile and desktop versions | Critical |
| Mobile-friendly fonts | Minimum 16px base font size | Medium |
| Tap targets | Minimum 48x48px with 8px spacing | Medium |
| No horizontal scroll | Content fits within viewport width | High |
| Mobile page speed | LCP < 2.5s on 4G connection | High |
| No intrusive interstitials | No full-screen popups blocking content on mobile | High |
| Structured data parity | Same schema markup on mobile and desktop | High |
| Image parity | Same images (or appropriate mobile alternatives) on both versions | Medium |
| Lazy loading works | Images load correctly when scrolled into view on mobile | Medium |

---

## Category 6: Core Web Vitals

### Current Metrics (Updated March 2024: INP Replaced FID)

| Metric | Good | Needs Improvement | Poor | What It Measures |
|--------|------|--------------------|------|------------------|
| **LCP** (Largest Contentful Paint) | < 2.5s | 2.5s - 4.0s | > 4.0s | Loading performance |
| **INP** (Interaction to Next Paint) | < 200ms | 200ms - 500ms | > 500ms | Responsiveness (replaced FID March 2024) |
| **CLS** (Cumulative Layout Shift) | < 0.1 | 0.1 - 0.25 | > 0.25 | Visual stability |

#### LCP Optimization Checks

| Check | Impact | Severity |
|-------|--------|----------|
| LCP element identified | Hero image, large text block, or video | High |
| LCP image has `fetchpriority="high"` | Prioritizes download | High |
| LCP image is preloaded | `<link rel="preload">` for LCP resource | High |
| No render-blocking CSS/JS above fold | Critical CSS inlined or preloaded | High |
| Server response time (TTFB) < 800ms | Fast server response | High |
| CDN for static assets | Assets served from edge locations | Medium |

#### INP Optimization Checks

| Check | Impact | Severity |
|-------|--------|----------|
| No long tasks > 50ms | Main thread not blocked | High |
| Event handlers are efficient | No heavy computation in click/input handlers | High |
| Code splitting implemented | Only necessary JS loaded per page | Medium |
| Third-party scripts deferred | Non-critical scripts use `defer` or `async` | Medium |
| Web Workers for heavy computation | Offload work from main thread | Low |
| Input debouncing | Search inputs and scroll handlers debounced | Medium |

#### CLS Optimization Checks

| Check | Impact | Severity |
|-------|--------|----------|
| Images have width/height | Prevents layout shift during load | High |
| Fonts use `font-display: swap` or `optional` | Prevents FOIT layout shift | High |
| No dynamically injected content above fold | Late-loading banners, ads, embeds | High |
| Ad slots have reserved dimensions | Placeholder sizing for ad containers | Medium |
| Web fonts preloaded | `<link rel="preload" as="font">` | Medium |
| Animations use `transform`/`opacity` | Not animating layout properties | Medium |

---

## Category 7: Structured Data

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| JSON-LD format used | Preferred over microdata or RDFa | Medium |
| Valid syntax | No JSON parse errors | Critical |
| Correct @type | Matches page content purpose | High |
| Required properties present | Per Google's documentation for each type | High |
| No deprecated types | Not using HowTo (deprecated Sept 2023), SpecialAnnouncement (deprecated July 2025), ClaimReview (deprecated June 2025) | High |
| Rich Results eligible | Passes Google Rich Results Test | High |
| Organization schema on homepage | Name, URL, logo, sameAs | Medium |
| BreadcrumbList | Present on all non-homepage pages | Medium |

---

## Category 8: JavaScript Rendering

### December 2025 Google Guidance on JS Rendering

Google has provided updated guidance on JavaScript rendering and canonical conflicts:

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| Server-rendered canonical matches JS canonical | No conflicting canonical tags between SSR and CSR | Critical |
| Critical content in initial HTML | Key content visible without JavaScript execution | High |
| Internal links in HTML | Navigation links crawlable without JS | High |
| Meta tags in initial HTML | title, description, canonical present server-side | Critical |
| No JavaScript redirects for indexable pages | Server-side redirects preferred (301/302) | High |
| Dynamic rendering or SSR for key pages | If SPA, ensure prerendering for search engines | High |
| Hydration does not alter SEO elements | React/Vue hydration does not change title, canonical, or meta tags | Critical |
| `<noscript>` fallback | Alternative content or message for no-JS scenarios | Low |

#### SPA-Specific Checks

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| Prerender middleware active | Serves static HTML to bots (Googlebot, social crawlers) | Critical |
| Route-level meta injection | Server injects unique meta tags per route before sending HTML | Critical |
| Client-side routing generates crawlable URLs | History API (not hash-based routing) | High |
| Lazy-loaded content is crawlable | Content within viewport or accessible via scrolling | Medium |
| JavaScript errors do not break rendering | Error boundaries prevent blank pages | High |

---

## Category 9: IndexNow Protocol

IndexNow enables instant notification to search engines when content changes, rather than waiting for crawlers to discover updates.

| Check | Pass Criteria | Severity |
|-------|---------------|----------|
| IndexNow supported | Key file at `/{key}.txt` or in `<meta>` tag | Low |
| API key valid | Key file accessible and matches submitted key | Low |
| Supported engines | Bing, Yandex, Naver, Seznam (Google does not support IndexNow) | Low |
| Submission on publish | New/updated content triggers IndexNow ping | Low |
| Batch submissions | Bulk URL submission for large content updates | Low |

### IndexNow Implementation

```
POST https://api.indexnow.org/indexnow
Content-Type: application/json

{
  "host": "example.com",
  "key": "your-api-key",
  "keyLocation": "https://example.com/your-api-key.txt",
  "urlList": [
    "https://example.com/new-page",
    "https://example.com/updated-page"
  ]
}
```

---

## Output Format

```markdown
# Technical SEO Audit: [Domain]
**Date:** [YYYY-MM-DD]
**Overall Technical Score:** XX/100

## Category Scores
| Category | Score | Max | Status |
|----------|-------|-----|--------|
| Crawlability | XX | 15 | [PASS/WARN/FAIL] |
| Indexability | XX | 15 | [PASS/WARN/FAIL] |
| Security Headers | XX | 10 | [PASS/WARN/FAIL] |
| URL Structure | XX | 10 | [PASS/WARN/FAIL] |
| Mobile Optimization | XX | 15 | [PASS/WARN/FAIL] |
| Core Web Vitals | XX | 15 | [PASS/WARN/FAIL] |
| Structured Data | XX | 10 | [PASS/WARN/FAIL] |
| JavaScript Rendering | XX | 5 | [PASS/WARN/FAIL] |
| IndexNow | XX | 5 | [PASS/WARN/FAIL] |
| **Total** | **XX** | **100** | |

## Issues Found
### Critical
- [issue + specific fix]

### High
- [issue + specific fix]

### Medium
- [issue + specific fix]

### Low
- [issue + specific fix]

## AI Crawler Status
| Crawler | Status | Directive |
|---------|--------|-----------|
| GPTBot | Allowed/Blocked | [directive from robots.txt] |
| ClaudeBot | Allowed/Blocked | [directive] |
| PerplexityBot | Allowed/Blocked | [directive] |
| ... | ... | ... |

**AI Crawler Strategy Assessment:** [Intentional/Unintentional/Missing]
**Recommendation:** [strategy recommendation based on site goals]
```

---

## Usage

```bash
# Full technical audit
/seo-technical https://example.com

# Technical audit focused on specific category
/seo-technical https://example.com --focus crawlability

# Technical audit with CWV data from PageSpeed Insights
/seo-technical https://example.com --include-cwv
```

---

## Notes

- Core Web Vitals thresholds are based on 75th percentile of page loads
- INP officially replaced FID as a Core Web Vital in March 2024
- Mobile-first indexing was fully enforced for all sites as of July 2024
- Google's December 2025 guidance emphasizes canonical consistency between server-rendered and client-rendered HTML
- IndexNow is optional but beneficial for Bing; Google does not participate in IndexNow
- AI crawler management is an emerging discipline; strategies should be reviewed quarterly as the landscape evolves
- Security headers are ranked by SEO impact (HTTPS is critical; CSP is medium) rather than pure security severity
