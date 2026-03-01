---
name: seo-geo-aeo
description: "Comprehensive search optimization skill covering SEO (Search Engine Optimization), GEO (Generative Engine Optimization for AI search), and AEO (Answer Engine Optimization for featured snippets/voice). Use when optimizing: (1) Website pages and blog posts, (2) Content for AI-powered search engines, (3) Featured snippets and voice search, (4) Local search visibility, (5) Technical SEO audits. Triggers on 'optimize for search', 'SEO audit', 'featured snippet', 'AI search optimization', or any search visibility request."
license: Proprietary
---

# SEO / GEO / AEO Optimization Skill

Comprehensive search optimization covering traditional SEO, AI-powered search (GEO), and answer engines (AEO).

## Quick Reference

| Optimization Type | Target | Key Focus |
|-------------------|--------|-----------|
| SEO | Google, Bing traditional | Rankings, clicks, organic traffic |
| GEO | ChatGPT, Perplexity, SGE, Copilot | AI citations, source attribution |
| AEO | Featured snippets, voice assistants | Direct answers, position zero |
| Local SEO | Maps, local pack | Proximity, relevance, prominence |

## The Modern Search Landscape (2025)

```
User Query
    │
    ├── Traditional Search (Google/Bing)
    │   ├── Organic results (SEO)
    │   ├── Featured snippets (AEO)
    │   ├── Local pack (Local SEO)
    │   └── AI Overview/SGE (GEO)
    │
    ├── AI Search Engines
    │   ├── Perplexity AI
    │   ├── ChatGPT with browsing
    │   ├── Microsoft Copilot
    │   └── Google Gemini
    │
    └── Voice Assistants
        ├── Google Assistant
        ├── Alexa
        └── Siri
```

---

# PART 1: SEO (Search Engine Optimization)

## On-Page SEO

### Title Tag Optimization

```
Format: Primary Keyword | Secondary Keyword | Brand
Length: 50-60 characters
```

**Rules**:
- Primary keyword within first 60 characters
- Front-load important keywords
- Include brand name (unless space-constrained)
- Make it compelling for CTR
- Unique for every page

**Examples**:
```
Good: "ADHD Assessment Sydney | Testing for Children & Adults | ClinicName"
Bad:  "Welcome to Our Website - We Offer Many Services"
```

### Meta Description

```
Length: 150-160 characters
Format: [Hook] + [Value proposition] + [CTA]
```

**Rules**:
- Include primary keyword naturally
- Compelling call-to-action
- Accurate preview of content
- Unique for every page
- Include structured elements where relevant (price, dates)

**Template**:
```
[What you'll learn/get] about [topic]. [Key benefit]. [CTA with keyword].
```

### Heading Structure

```
H1: One per page, contains primary keyword
H2: Major sections, include secondary keywords
H3: Subsections, support H2 topics
H4-H6: Detailed breakdowns as needed
```

**Rules**:
- Logical hierarchy (never skip levels)
- Keywords distributed naturally
- Descriptive and scannable
- Answer user questions in headings

### Content Optimization

**Keyword Placement**:
| Location | Importance | Notes |
|----------|------------|-------|
| Title tag | Critical | Front-load keyword |
| H1 | Critical | Natural inclusion |
| First 100 words | High | Establish relevance |
| H2/H3 headings | High | Throughout content |
| Body text | Medium | Natural density 1-2% |
| Image alt text | Medium | Descriptive with keywords |
| URL slug | Medium | Hyphenated, concise |
| Last paragraph | Low | Reinforce topic |

**Content Quality Signals**:
- Comprehensive coverage (topical depth)
- Original research/data
- Expert authorship (E-E-A-T)
- Fresh, updated content
- Multimedia (images, videos, infographics)
- Internal/external linking

### URL Structure

```
Format: domain.com/category/keyword-slug
Length: Under 75 characters
```

**Rules**:
- Lowercase only
- Hyphens between words (not underscores)
- No special characters
- Include primary keyword
- Remove stop words (the, and, or)
- Logical hierarchy

### Image Optimization

| Element | Optimization |
|---------|-------------|
| File name | descriptive-keyword.jpg |
| Alt text | Descriptive, 125 chars max, keyword if natural |
| File size | Under 200KB for web images |
| Format | WebP preferred, JPEG for photos, PNG for graphics |
| Dimensions | Exact display size (avoid scaling) |
| Lazy loading | Implement for below-fold images |

### Internal Linking

**Strategy**:
- Link to related content contextually
- Use descriptive anchor text (not "click here")
- Distribute link equity to important pages
- Create content clusters/silos
- Update old content with new links
- Aim for 3-5 internal links per 1000 words

## Technical SEO

### Core Web Vitals

| Metric | Target | What It Measures |
|--------|--------|------------------|
| LCP (Largest Contentful Paint) | < 2.5s | Loading performance |
| INP (Interaction to Next Paint) | < 200ms | Interactivity |
| CLS (Cumulative Layout Shift) | < 0.1 | Visual stability |

### Site Architecture

```
Homepage
├── Category 1
│   ├── Subcategory 1.1
│   │   ├── Page A
│   │   └── Page B
│   └── Subcategory 1.2
├── Category 2
└── Category 3
```

**Rules**:
- Maximum 3 clicks to any page
- Flat architecture preferred
- Clear navigation paths
- XML sitemap updated
- Breadcrumb navigation

### Mobile Optimization

- Mobile-first indexing (Google default)
- Responsive design required
- Touch-friendly elements (min 48px tap targets)
- No intrusive interstitials
- Same content as desktop

### Crawlability & Indexation

**robots.txt**:
```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /private/
Sitemap: https://domain.com/sitemap.xml
```

**Meta Robots**:
```html
<meta name="robots" content="index, follow">
<meta name="robots" content="noindex, nofollow"> <!-- For private pages -->
```

### Structured Data (Schema.org)

**Priority Schemas**:
| Content Type | Schema | Benefits |
|-------------|--------|----------|
| Articles | Article, BlogPosting | Rich results, AI understanding |
| Products | Product | Price, availability, reviews |
| Local Business | LocalBusiness | Knowledge panel, maps |
| FAQs | FAQPage | FAQ rich results |
| How-tos | HowTo | Step-by-step rich results |
| Events | Event | Event listings |
| Reviews | Review | Star ratings |
| Breadcrumbs | BreadcrumbList | Navigation path display |

### Site Speed Optimization

**Priorities**:
1. Optimize images (WebP, compression, lazy load)
2. Minimize CSS/JS (minify, defer, async)
3. Enable compression (Gzip/Brotli)
4. Use CDN for static assets
5. Browser caching headers
6. Reduce server response time (< 200ms TTFB)
7. Preload critical resources
8. Remove render-blocking resources

### HTTPS & Security

- SSL certificate required (free via Let's Encrypt)
- Redirect HTTP to HTTPS
- HSTS headers enabled
- Mixed content warnings resolved
- Security headers (CSP, X-Frame-Options)

---

# PART 2: GEO (Generative Engine Optimization)

## Understanding AI Search

### How AI Search Works

```
User Query → AI Model → Web Search → Source Analysis → Synthesized Answer
                                           │
                                           └── Your content cited here
```

### Key AI Search Engines

| Platform | Model | How It Uses Sources |
|----------|-------|---------------------|
| Perplexity | Multiple | Cites sources inline with links |
| ChatGPT + Browse | GPT-4 | References in response |
| Google SGE/AI Overview | Gemini | Expandable source cards |
| Microsoft Copilot | GPT-4 | Footnote citations |
| Claude (with search) | Claude | Inline source attribution |

## GEO Optimization Strategies

### 1. Quotability & Citability

**Make content easy to quote**:
- Clear, standalone statements
- Definitive facts and statistics
- Expert opinions with attribution
- Concise definitions
- Numbered lists and steps

**Example (Quotable)**:
```
"Cognitive Behavioral Therapy (CBT) is a structured, goal-oriented
psychotherapy that helps individuals identify and change negative
thought patterns. Research shows 60-80% of people with anxiety
disorders show significant improvement with CBT treatment."
```

### 2. Authoritative Content Signals

**E-E-A-T for AI**:
- **Experience**: First-hand knowledge demonstrated
- **Expertise**: Author credentials visible
- **Authoritativeness**: Cited by other sources
- **Trustworthiness**: Accurate, up-to-date information

**Implementation**:
```markdown
## About the Author
Dr. Jane Smith is a clinical psychologist with 15 years of experience
specializing in anxiety disorders. She is a registered member of the
Australian Psychological Society and has published research in the
Journal of Clinical Psychology.
```

### 3. Structured, Parseable Content

**Format for AI Understanding**:

```markdown
## What is [Topic]?

[Clear 2-3 sentence definition]

## Key Facts About [Topic]

- Fact 1: [Specific, verifiable statement]
- Fact 2: [Data point with source]
- Fact 3: [Expert consensus]

## How [Topic] Works

1. Step one explanation
2. Step two explanation
3. Step three explanation

## Statistics

| Metric | Value | Source |
|--------|-------|--------|
| [Metric 1] | [Value] | [Source, Year] |
| [Metric 2] | [Value] | [Source, Year] |
```

### 4. Source Attribution

**Cite authoritative sources**:
- Academic journals (with DOI)
- Government statistics
- Industry reports
- Recognized experts
- Primary research

**Format**:
```markdown
According to the World Health Organization (2024), depression affects
approximately 280 million people globally. Research published in The
Lancet (Smith et al., 2023) indicates that early intervention
significantly improves outcomes.
```

### 5. Freshness & Updates

**AI models prefer current content**:
- Include publication dates
- Add "Last updated" timestamps
- Reference recent data (within 2 years)
- Update statistics regularly
- Note when information may change

```markdown
*Last updated: January 2025*
*Statistics from Australian Bureau of Statistics 2024 census data*
```

### 6. Comprehensive Topic Coverage

**Semantic completeness**:
- Cover all aspects of a topic
- Answer related questions
- Include definitions of key terms
- Provide context and background
- Address common misconceptions

### 7. Unique Value Proposition

**Stand out for citation**:
- Original research or data
- Expert interviews
- Case studies
- Proprietary methodologies
- First-hand experience

---

# PART 3: AEO (Answer Engine Optimization)

## Featured Snippet Optimization

### Types of Featured Snippets

| Type | Format | Best For |
|------|--------|----------|
| Paragraph | 40-60 words | Definitions, explanations |
| List | Numbered/bulleted | Steps, rankings, tips |
| Table | Rows and columns | Comparisons, data |
| Video | YouTube embed | How-tos, tutorials |

### Paragraph Snippet Optimization

**Structure**:
```markdown
## What is [Keyword]?

[Keyword] is [clear, concise definition in 40-50 words].
[One additional sentence of context or key benefit].
```

**Example**:
```markdown
## What is Cognitive Behavioral Therapy?

Cognitive Behavioral Therapy (CBT) is a type of psychotherapy that
helps people identify and change negative thought patterns and
behaviors. It is an evidence-based treatment commonly used for
anxiety, depression, and other mental health conditions.
```

### List Snippet Optimization

**Structure**:
```markdown
## How to [Action] [Topic]

Follow these steps to [achieve outcome]:

1. **Step one**: Brief explanation
2. **Step two**: Brief explanation
3. **Step three**: Brief explanation
4. **Step four**: Brief explanation
5. **Step five**: Brief explanation
```

### Table Snippet Optimization

**Structure**:
```markdown
## [Topic] Comparison

| Feature | Option A | Option B | Option C |
|---------|----------|----------|----------|
| Feature 1 | Value | Value | Value |
| Feature 2 | Value | Value | Value |
| Feature 3 | Value | Value | Value |
```

## Voice Search Optimization

### Voice Query Characteristics

- Conversational language
- Question format ("How do I...", "What is...")
- Longer queries (7+ words average)
- Local intent common ("near me")
- Action-oriented

### Optimization Strategies

**1. Question-Based Content**:
```markdown
## Frequently Asked Questions

### How long does therapy take to work?

Most people begin to notice improvements within 6-12 sessions of
cognitive behavioral therapy. However, the duration depends on
individual circumstances and the complexity of the issue being
addressed.
```

**2. Conversational Tone**:
```markdown
Instead of: "CBT duration varies based on presenting concerns."
Use: "How long does CBT take? Most people start feeling better
after about 6 to 12 sessions."
```

**3. Local Voice Optimization**:
```markdown
Include phrases like:
- "near me" alternatives: "in [suburb]", "in [city]"
- "Open now" information: operating hours
- "Best [service] nearby": local qualifiers
```

## People Also Ask (PAA) Optimization

### Identifying PAA Opportunities

1. Search your target keyword
2. Note all PAA questions
3. Create content answering each
4. Structure with clear Q&A format

### PAA Content Format

```markdown
## People Also Ask About [Topic]

### [Question 1 from PAA]?

[Direct answer in 2-3 sentences]. [Supporting detail].
[Link to more comprehensive information if applicable].

### [Question 2 from PAA]?

[Direct answer]. [Supporting information].
```

## FAQ Schema Implementation

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is the question?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The complete answer to the question."
      }
    },
    {
      "@type": "Question",
      "name": "Second question?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The answer to the second question."
      }
    }
  ]
}
```

---

# PART 4: Local SEO

## Google Business Profile Optimization

### Essential Information

| Field | Optimization |
|-------|--------------|
| Business name | Exact legal name (no keyword stuffing) |
| Category | Most specific primary + relevant secondary |
| Address | Exact match with website |
| Phone | Local number, trackable |
| Website | Homepage or relevant landing page |
| Hours | Accurate, including holidays |
| Description | 750 chars, keywords natural, compelling |
| Attributes | All relevant attributes selected |

### NAP Consistency

**NAP = Name, Address, Phone**

Must be identical across:
- Website (footer, contact page)
- Google Business Profile
- Social media profiles
- Directory listings
- Citations

### Local Content Strategy

**Location-Specific Pages**:
```markdown
# [Service] in [Location]

[Introduction mentioning location and service]

## Our [Service] in [Location]

[Description with local context, landmarks, community]

## Why Choose Us for [Service] in [Location]

[Local credentials, community involvement]

## Serving [Location] and Surrounding Areas

[List of nearby suburbs/areas served]
```

### Review Strategy

**Generating Reviews**:
- Ask satisfied customers
- Send follow-up emails with review link
- QR codes in physical location
- Respond to all reviews (positive and negative)

**Review Response Template**:
```
Positive: "Thank you [Name] for your kind words about your experience
with [service]. We're glad we could help with [specific mention].
We look forward to seeing you again!"

Negative: "Thank you [Name] for your feedback. We're sorry your
experience didn't meet expectations. Please contact us at [email]
so we can make this right."
```

### Local Link Building

- Local business directories
- Chamber of Commerce
- Industry associations
- Local news/media mentions
- Community sponsorships
- Local partnerships

---

# Optimization Checklists

## Pre-Publish SEO Checklist

### Content
- [ ] Target keyword identified and researched
- [ ] Title tag optimized (50-60 chars, keyword front-loaded)
- [ ] Meta description compelling (150-160 chars, CTA)
- [ ] H1 includes primary keyword
- [ ] H2/H3 structure logical with secondary keywords
- [ ] Keyword in first 100 words
- [ ] Content comprehensive (covers topic thoroughly)
- [ ] Internal links included (3-5 per 1000 words)
- [ ] External links to authoritative sources
- [ ] Images optimized (alt text, file size, WebP)

### Technical
- [ ] URL slug clean and contains keyword
- [ ] Mobile-friendly
- [ ] Page speed acceptable (< 3s load)
- [ ] No broken links
- [ ] Canonical tag correct
- [ ] Schema markup implemented

### GEO
- [ ] Content quotable (clear statements)
- [ ] Statistics with sources
- [ ] Author credentials included
- [ ] Last updated date visible
- [ ] Comprehensive topic coverage

### AEO
- [ ] Question-answer format for key queries
- [ ] Featured snippet opportunity targeted
- [ ] FAQ section with schema
- [ ] Voice-friendly language

## Monthly SEO Audit Checklist

- [ ] Core Web Vitals passing
- [ ] No new crawl errors in Search Console
- [ ] Sitemap up to date
- [ ] No broken links (internal/external)
- [ ] Thin content identified and improved
- [ ] Keyword rankings tracked
- [ ] Competitor analysis updated
- [ ] Content freshness (update old posts)
- [ ] Backlink profile healthy
- [ ] Local listings consistent

---

# PART 5: Bing Optimization

## Understanding Bing

### Bing Market Position

| Market | Bing Share | Notes |
|--------|------------|-------|
| USA | ~7% | Second largest, powers Yahoo |
| UK | ~4% | Growing with Edge browser |
| Australia | ~5% | Default in Windows/Edge |
| Enterprise | 15-20% | Windows/Office integration |

### Key Differences from Google

| Factor | Google | Bing |
|--------|--------|------|
| Exact match keywords | Less important | More important |
| Social signals | Minimal impact | Stronger correlation |
| Backlink quality | Emphasis on authority | Quantity also matters |
| Domain age | Minor factor | More significant |
| Multimedia | Important | Very important |
| Local | Google Maps dominant | Bing Places growing |

## Bing Webmaster Tools Setup

### Essential Configuration
1. Verify site ownership (DNS, file, meta tag)
2. Submit XML sitemap
3. Configure geo-targeting
4. Set crawl control preferences
5. Connect to Microsoft Clarity (analytics)

### Bing-Specific Features
- **URL Submission API**: Faster indexing
- **Content Submission API**: Submit content directly
- **Adaptive URL Sampling**: Intelligent crawling
- **Site Scan**: Technical SEO audit

## Bing Ranking Factors

### High Priority Factors

**1. Page Quality**
- Clear, well-written content
- Correct grammar and spelling (Bing penalizes errors)
- Authoritative sources
- Fresh content

**2. Exact Match Keywords**
```
Bing: More literal keyword matching than Google
- Include exact keyword phrases
- Use keywords in title, H1, URL
- Natural but precise placement
```

**3. Social Signals**
```
Bing considers:
- Facebook shares/likes
- Twitter engagement
- LinkedIn shares
- Pinterest pins
```

**4. Multimedia Content**
```
Bing heavily indexes:
- Images (with good alt text)
- Videos (especially from Vimeo)
- Infographics
- Audio content
```

**5. Domain Factors**
- Domain age carries more weight
- Exact match domains still help
- .gov and .edu get trust boost

### Bing Technical SEO

**Crawling Preferences**:
```xml
<!-- Bing-specific robots.txt -->
User-agent: bingbot
Crawl-delay: 1
Allow: /
Sitemap: https://domain.com/sitemap.xml
```

**Meta Tags Bing Respects**:
```html
<!-- Content language -->
<meta http-equiv="content-language" content="en-AU">

<!-- Geo targeting -->
<meta name="geo.region" content="AU-NSW">
<meta name="geo.placename" content="Sydney">
<meta name="geo.position" content="-33.8688;151.2093">
<meta name="ICBM" content="-33.8688, 151.2093">
```

## Bing Places for Business

### Setup Checklist
- [ ] Claim or create Bing Places listing
- [ ] Import from Google Business Profile (if available)
- [ ] Verify ownership
- [ ] Complete all business information
- [ ] Add photos and logo
- [ ] Encourage reviews on Bing

### Optimization
- Exact NAP match with Google/website
- Complete all available fields
- Regular updates and posts
- Respond to reviews

## Bing + Microsoft Ecosystem

### Copilot Integration
- Bing powers Microsoft Copilot
- Content cited in Copilot responses
- Optimize for GEO principles

### Edge Browser
- Default search in Edge
- Growing market share
- Windows 11 integration

### LinkedIn Connection
- Bing indexes LinkedIn profiles
- Business profiles appear in results
- Company pages show in brand searches

---

# PART 6: International Search Engines

## Baidu (China) - 76% Market Share

### Understanding Baidu

**Key Differences**:
- Chinese government regulated
- Requires ICP license for hosting
- Simplified Chinese content required
- Different algorithm priorities
- Heavy censorship compliance

### Baidu Technical Requirements

**Hosting & Domain**:
```
Requirements:
- Chinese hosting (ICP license mandatory)
- .cn or .com.cn domain preferred
- Fast server response in China
- No VPN-required resources
```

**Language & Encoding**:
```html
<html lang="zh-CN">
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### Baidu Ranking Factors

| Factor | Importance | Notes |
|--------|------------|-------|
| Meta keywords | High | Still uses meta keywords tag |
| Title tag | Critical | Include keywords early |
| Content quality | High | Original, comprehensive |
| Page speed | High | Chinese server location critical |
| Mobile optimization | Critical | Mobile-first market |
| Backlinks | Medium | Quality Chinese sites |
| Domain age | High | Older domains trusted |

### Baidu-Specific Meta Tags
```html
<!-- Meta keywords (Baidu still uses these) -->
<meta name="keywords" content="关键词1, 关键词2, 关键词3">

<!-- Baidu site verification -->
<meta name="baidu-site-verification" content="CODE">

<!-- Mobile adaptation -->
<meta name="mobile-agent" content="format=html5; url=https://m.domain.com/">

<!-- No Baidu transcoding -->
<meta http-equiv="Cache-Control" content="no-transform">
```

### Baidu Webmaster Tools
- **Baidu Zhanzhang** (站长平台)
- Submit sitemap
- Monitor indexing
- Check for penalties
- URL submission

### Baidu Content Guidelines
- Simplified Chinese (not Traditional)
- No sensitive political content
- Comply with Chinese regulations
- Local cultural relevance
- WeChat/Weibo integration helps

---

## Yandex (Russia) - 60% Market Share

### Understanding Yandex

**Market Position**:
- Dominant in Russia (60%+)
- Strong in CIS countries
- Growing in Turkey
- Advanced AI/ML algorithms

### Yandex Technical Requirements

**Language & Region**:
```html
<html lang="ru">
<meta charset="UTF-8">
<!-- Regional targeting -->
<meta name="geo.region" content="RU">
```

### Yandex Ranking Factors

| Factor | Importance | Notes |
|--------|------------|-------|
| Content quality | Critical | Yandex AGS filter penalizes thin content |
| User behavior | Very High | Time on site, bounce rate heavily weighted |
| Mobile UX | High | Mobile-first indexing |
| Commercial factors | High | For commercial queries |
| Regional relevance | High | Important for local results |
| ICS (Site Quality Index) | High | Yandex's quality metric |

### Yandex-Specific Optimization

**Yandex.Webmaster Setup**:
1. Add and verify site
2. Submit sitemap
3. Set regional targeting
4. Monitor ICS (Index Citation Score)
5. Check for AGS penalties

**Yandex Meta Tags**:
```html
<!-- Yandex verification -->
<meta name="yandex-verification" content="CODE">

<!-- No Yandex translation -->
<meta name="yandex" content="notranslate">

<!-- Turbo pages -->
<link rel="alternate" type="application/rss+xml"
      href="https://domain.com/turbo-feed.rss">
```

### Yandex Unique Features

**Turbo Pages**:
- AMP-like fast loading pages
- Prioritized in mobile search
- RSS feed implementation

**Yandex.Metrica**:
- Free analytics (like GA)
- Session recordings
- Heat maps
- Form analytics

**Yandex.Business**:
- Local business listings
- Maps integration
- Reviews management

---

## Naver (South Korea) - 70% Market Share

### Understanding Naver

**Unique Characteristics**:
- Portal-style search results
- Heavy emphasis on Naver's own properties
- Blog content highly ranked
- Naver Cafe communities important
- Knowledge iN (Q&A) appears prominently

### Naver SERP Structure

```
Naver Results Page:
├── Ads (paid)
├── Naver Blog results
├── Naver Cafe (community) results
├── Knowledge iN (Q&A)
├── News
├── Images
├── Videos
├── Web results (external sites)
└── Shopping (if commercial)
```

### Naver Optimization Strategy

**1. Naver Blog (Critical)**
```
- Create official Naver Blog
- Post regularly (2-3x per week)
- High-quality, original content
- Engage with community
- Link to main website
```

**2. Naver Cafe Presence**
```
- Join relevant Naver Cafes
- Participate authentically
- Share valuable content
- Build community presence
```

**3. Knowledge iN**
```
- Answer questions in your expertise
- Build authority profile
- Link to detailed resources
```

### Naver Technical SEO

**Naver Webmaster Tools (Search Advisor)**:
- Submit sitemap
- Monitor indexing
- Check robots.txt
- Review crawl errors

**Naver-Specific Tags**:
```html
<!-- Naver site verification -->
<meta name="naver-site-verification" content="CODE">

<!-- Naver no archive -->
<meta name="naverbot" content="noarchive">
```

### Naver Ranking Factors

| Factor | Importance | Notes |
|--------|------------|-------|
| Naver Blog presence | Critical | Often outranks external sites |
| Content freshness | Very High | Recent content prioritized |
| User engagement | High | Comments, shares, likes |
| Mobile optimization | High | Growing mobile usage |
| Korean language quality | High | Natural Korean writing |
| Naver Cafe activity | High | Community signals |

---

## Yahoo Japan - Significant Market Share

### Understanding Yahoo Japan

**Key Points**:
- Separate from Yahoo (USA)
- Uses Google's search technology (since 2010)
- But has unique features and layout
- Popular with older demographics
- Strong in certain verticals

### Yahoo Japan Optimization

**Since it uses Google's engine**:
- Standard Google SEO applies
- But optimize for Yahoo Japan-specific features
- Register with Yahoo Japan Business
- Focus on Yahoo Japan's content properties

### Yahoo Japan Unique Features
- Yahoo! Shopping integration
- Yahoo! News prominent
- Yahoo! Auctions (like eBay)
- Yahoo! Knowledge Bag (Q&A)

---

## DuckDuckGo & Privacy-Focused Search

### DuckDuckGo Market

**Growing Demographics**:
- Privacy-conscious users
- Tech-savvy audiences
- European markets (GDPR awareness)
- iOS users (Apple default option)

### DuckDuckGo Ranking Factors

**Sources DuckDuckGo Uses**:
- Bing results (primary)
- Wikipedia
- Apple Maps (local)
- Own crawler (DuckDuckBot)
- Various APIs and sources

**Optimization Approach**:
```
Since DDG uses Bing heavily:
1. Optimize for Bing (see Part 5)
2. Ensure Wikipedia accuracy
3. Build diverse backlink profile
4. Focus on content quality
5. Apple Maps listing for local
```

### DuckDuckGo Instant Answers

**Structured Data Sources**:
- Wikipedia
- Wikidata
- Stack Overflow
- GitHub
- CrunchBase
- Various APIs

**To Appear in Instant Answers**:
- Contribute to Wikipedia
- Maintain accurate Wikidata entries
- Build presence on DDG partner sites

---

## Ecosia & Brave Search

### Ecosia (Eco-Conscious Users)

**Characteristics**:
- Plants trees with ad revenue
- Uses Bing results primarily
- Growing eco-conscious market
- European focus

**Optimization**: Same as Bing optimization

### Brave Search

**Characteristics**:
- Independent index (not using Google/Bing)
- Privacy-focused
- Growing with Brave browser users
- Unique ranking algorithm

**Brave Search Optimization**:
```
Focus on:
- Technical SEO excellence
- Fast page speed
- Clean code/markup
- Quality content
- Natural link profile
```

---

# PART 7: Search Engine Comparison Matrix

## Quick Reference

| Search Engine | Region | Share | Key Focus |
|---------------|--------|-------|-----------|
| Google | Global | 92% | Quality content, E-E-A-T, mobile |
| Bing | USA/UK | 3-7% | Exact match, social, multimedia |
| Baidu | China | 76% | ICP license, Chinese content, meta keywords |
| Yandex | Russia | 60% | User behavior, ICS score, regional |
| Naver | Korea | 70% | Naver Blog, Cafe, fresh content |
| Yahoo Japan | Japan | 15% | Google tech, Yahoo properties |
| DuckDuckGo | Global | 2% | Bing-based, Wikipedia, privacy |

## Optimization Priority by Market

### English-Speaking Markets
1. Google (primary)
2. Bing (secondary)
3. DuckDuckGo (growing)

### Chinese Market
1. Baidu (essential)
2. Sogou (secondary)
3. 360 Search (tertiary)

### Russian Market
1. Yandex (primary)
2. Google Russia (secondary)
3. Mail.ru (tertiary)

### Korean Market
1. Naver (essential)
2. Google Korea (secondary)
3. Daum (tertiary)

### Japanese Market
1. Google Japan (primary)
2. Yahoo Japan (significant)
3. Bing Japan (minor)

---

## File References

- `references/schema-templates.md` - Common schema markup templates
- `references/keyword-research.md` - Keyword research methodology
- `checklists/technical-audit.md` - Full technical SEO audit
- `checklists/local-seo.md` - Local SEO optimization checklist
- `checklists/content-optimization.md` - Content optimization checklist
