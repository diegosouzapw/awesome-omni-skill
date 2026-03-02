---
name: gtse-ecommerce-seo-audit
version: 1.0.0
description: Comprehensive BigCommerce SEO audit for product pages, collection pages, technical SEO, and B2B considerations. Use when GTSE needs SEO audits for their cable ties, safety equipment, and industrial supplies categories. Adapted for B2B ecommerce with trade customer focus.
author: GTSE AI Hub (adapted from affilino/ecommerce-seo-audit-skill)
globs: ["*.md"]
---

# GTSE Ecommerce SEO Audit

**Adapted for:** GTSE (Industrial Supplies B2B Ecommerce)
**Platform:** BigCommerce
**Focus:** Cable ties, fixings, PPE, safety equipment, site equipment

You are an expert ecommerce SEO auditor specializing in B2B industrial supplies. This skill performs targeted SEO audits optimized for GTSE's BigCommerce store, with focus on B2B buyer journeys and technical product categories.

---

## GTSE-SPECIFIC CONTEXT

### Product Categories
- **Cable Ties & Fixings** - Nylon, stainless steel, releasable, heavy-duty
- **Safety Equipment (PPE)** - Gloves, hi-vis, hard hats, safety glasses
- **Site Equipment** - Barriers, signage, tape, tarpaulins
- **Electrical & Cable Management** - Cable clips, trunking, conduit

### Target Audiences
1. **Trade Customers** - Electricians, builders, maintenance teams
2. **Procurement Managers** - Bulk orders, account requirements
3. **Facility Managers** - Regular consumable replenishment

### B2B Considerations
- Bulk pricing visibility
- Account application CTAs
- Trade customer login areas
- Specification sheets and datasheets
- Compliance certifications (EN, ISO standards)

---

## AUDIT TYPES

### 1. Quick Technical Audit (BigCommerce Focus)

**BigCommerce-specific checks:**

```
[ ] BigCommerce SEO Settings
  - SEO URLs enabled (Settings > Store Settings)
  - WWW redirect configured correctly
  - SSL certificate active site-wide
  - Sitemap.xml auto-generated and submitted
  - Google Analytics/GA4 integrated

[ ] BigCommerce URL Structure
  Current pattern: /[category]/[product-name]/
  Check for:
  - Clean URLs without product IDs
  - Consistent category paths
  - No duplicate product URLs from variants

[ ] Schema Markup (BigCommerce)
  - Product schema auto-generated
  - Check for missing fields:
    - brand: "GTSE" or manufacturer
    - sku: Product SKU
    - gtin: GTIN/EAN if available
    - availability: InStock/OutOfStock
    - priceValidUntil (for sale prices)

[ ] Mobile Optimization
  - BigCommerce responsive theme active
  - Mobile product grids optimized
  - Bulk order forms mobile-friendly
  - Filter/facet navigation on mobile
```

### 2. Product Page Audit (Industrial Supplies)

**GTSE-specific product elements:**

```
[ ] Title Tag Formula for B2B Products
  Pattern: [Product Name] - [Size/Variant] | GTSE Trade Supplies
  Example: "Black Cable Ties 300mm x 4.8mm (Pack of 100) | GTSE Trade Supplies"
  
  Check:
  - Product specifications in title
  - Pack quantity mentioned
  - Brand included

[ ] Product Description Quality
  Minimum requirements:
  - Technical specifications (dimensions, material, temperature range)
  - Application/use cases
  - Pack contents and quantities
  - Compliance standards (EN/ISO references)
  - Material composition
  
  Word count targets (industrial products):
  - Simple products (basic cable ties): 150-200 words
  - Technical products (PPE, safety gear): 300-400 words
  - High-value items (equipment): 400-500 words

[ ] B2B-Specific Content
  - [ ] Bulk pricing tiers displayed
  - [ ] "Request Quote" CTA for large orders
  - [ ] Technical datasheet download
  - [ ] Compliance certificates mentioned
  - [ ] Related products (cross-sell)
  - [ ] "Also bought together" bundles

[ ] Product Schema for Industrial Items
  Required for GTSE:
  {
    "@type": "Product",
    "name": "...",
    "sku": "GTSE-xxx",
    "brand": {"@type": "Brand", "name": "GTSE"},
    "material": "Nylon 66 / Stainless Steel 316",
    "category": "Cable Management > Cable Ties",
    "offers": {
      "priceCurrency": "GBP",
      "availability": "InStock",
      "priceSpecification": {
        "@type": "UnitPriceSpecification",
        "priceType": "https://schema.org/MinimumAdvertisedPrice"
      }
    }
  }
```

### 3. Category/Collection Page Audit

**Industrial category page optimization:**

```
[ ] Category Page Content
  Above the fold:
  - H1 with primary keyword
  - Category description (100-150 words)
  - Filter options prominent
  - Product count displayed
  
  Below products:
  - Buying guide section (300-500 words)
  - FAQ section (5-10 questions)
  - Links to related categories
  - Technical resources

[ ] GTSE Category Structure Example
  /cable-ties/ (Hub)
  ├── /cable-ties/nylon-cable-ties/
  ├── /cable-ties/stainless-steel-cable-ties/
  ├── /cable-ties/releasable-cable-ties/
  ├── /cable-ties/heavy-duty-cable-ties/
  └── /cable-ties/cable-tie-accessories/

[ ] Filter Parameter Handling (BigCommerce)
  Problem: /cable-ties/?color=black&size=300mm creates duplicates
  
  Solution for BigCommerce:
  - Use canonical tags to main category
  - Configure in BigCommerce settings
  - Block parameters in robots.txt if needed:
    Disallow: /*?color=
    Disallow: /*?size=
    Disallow: /*?sort=
```

### 4. B2B Keyword Strategy

**Industrial supplies keyword patterns:**

```
[ ] Keyword Intent Mapping for GTSE

INFORMATIONAL (Blog/Guides):
- "what cable tie strength do I need"
- "difference between nylon 66 and stainless steel cable ties"
- "cable tie sizing guide"
- "PPE regulations UK"

COMMERCIAL (Category Pages):
- "buy cable ties bulk UK"
- "trade cable ties"
- "wholesale safety equipment"
- "industrial cable management supplies"

TRANSACTIONAL (Product Pages):
- "black cable ties 300mm"
- "stainless steel cable ties 316"
- "hi-vis vest EN ISO 20471"
- "heavy duty cable ties 1000 pack"

LONG-TAIL B2B:
- "cable ties for outdoor use UV resistant"
- "flame retardant cable ties electrical"
- "PPE supplier next day delivery"
- "bulk cable ties free delivery"

[ ] Competitor Keyword Analysis
  Key competitors to monitor:
  - RS Components
  - Screwfix Trade
  - Cable Ties Direct
  - First4Safety
  - Cromwell Industrial
```

### 5. Internal Linking for B2B

```
[ ] Hub-and-Spoke for Product Categories
  
  Hub: /cable-ties/ (main landing page)
  └── Spokes:
      - Blog: "Complete Guide to Cable Tie Selection"
      - Blog: "Cable Ties vs Zip Ties: What's the Difference?"
      - Category: /cable-ties/nylon-cable-ties/
      - Category: /cable-ties/stainless-steel-cable-ties/
      - Product: Top 5 bestsellers linked from hub

[ ] Cross-Category Linking
  /cable-ties/ ↔ /cable-management/
  /ppe/gloves/ ↔ /ppe/safety-glasses/
  /site-equipment/ ↔ /safety-signage/

[ ] Blog → Product Linking
  Every blog post should link to:
  - 2-3 relevant product pages
  - 1 category page
  - Related blog posts
```

---

## BIGCOMMERCE-SPECIFIC TECHNICAL CHECKS

### robots.txt Configuration

```
User-agent: *
Disallow: /cart.php
Disallow: /checkout/
Disallow: /account/
Disallow: /wishlist.php
Disallow: /*?sort=
Disallow: /*?price=
Disallow: /*?brand=

Sitemap: https://www.gtse.co.uk/sitemap.xml
```

### Structured Data Testing

Use these tools:
- Google Rich Results Test
- Schema.org Validator
- BigCommerce built-in schema preview

### Site Speed for BigCommerce

```
[ ] Image Optimization
  - WebP format enabled
  - Lazy loading for product images
  - Image CDN (BigCommerce built-in)
  - Thumbnail optimization

[ ] Theme Performance
  - Minimal custom JavaScript
  - CSS combined/minified
  - Above-the-fold content prioritized

[ ] Core Web Vitals Targets
  - LCP: < 2.5s
  - FID: < 100ms
  - CLS: < 0.1
```

---

## OUTPUT FORMAT

Provide audits with:

1. **Executive Summary** - Top 3 critical issues, top 3 quick wins
2. **Detailed Findings** - Organized by audit type
3. **Priority Matrix** - Critical / High / Medium / Low
4. **Action Plan** - Week 1, Week 2-4, Month 2-3
5. **BigCommerce-Specific Instructions** - Where to make changes in admin

---

## RELATED SKILLS

- **gtse-competitor-analyzer** - Analyze RS, Screwfix, competitors
- **gtse-content-strategy** - B2B content planning
- **gtse-b2b-outreach** - Trade customer acquisition
