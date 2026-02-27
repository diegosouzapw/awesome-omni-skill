---
name: weekly-email-team-instructions
description: "Generates The Edmund Bogen Team's weekly market intelligence package: email, article page, dashboard, and community listings pages. Guides team through data collection, validates consistency, and produces all HTML assets ready for deployment to Constant Contact and GitHub."
---

# Weekly Email Team Instructions Skill

This skill generates The Edmund Bogen Team's complete weekly market intelligence package across 4+ synchronized assets:

1. **Email HTML** (Constant Contact)
2. **Article Page HTML** (GitHub Pages)
3. **Dashboard HTML** (GitHub Pages)  
4. **Community Listings Pages HTML** (GitHub Pages - templated for multiple communities)

## When to Use This Skill

Use this skill every week when it's time to generate the market intelligence package. Team members (Edmund, Nicole, Dina, Samantha) will:

1. Fill out the team workflow template
2. Paste it into Claude
3. Claude validates, asks clarifying questions
4. Claude generates all assets with perfect consistency

## Critical Consistency Requirements

**ABSOLUTE RULES - NEVER VIOLATE:**

- Same article title everywhere (email, article page, dashboard)
- Same article image everywhere (email, article page, dashboard)
- Same date everywhere (email, article page, dashboard)
- Same community statistics everywhere (email, dashboard)
- Same author attribution everywhere
- Same brand colors/fonts everywhere (see brand guidelines)

If there are ANY inconsistencies in the input data, Claude MUST flag them and ask for correction before generating assets.

## Input Process (Option C: Hybrid Validation)

### Step 1: User Pastes Template
Team member pastes the filled-out workflow template (see team-workflow-template.md)

### Step 2: Claude Validates
Claude reviews the pasted data and checks for:
- Missing required fields
- Date consistency (article date = market data date = dashboard date)
- Title consistency
- Image filename format
- All 21 community stats present
- Featured listings have all required fields

### Step 3: Claude Asks Follow-up Questions
For any missing or inconsistent data:
```
"I notice the article date is January 7 but the market data date is January 6. Which date should be used consistently across all assets?"

"You've listed 19 communities but we need data for all 21. Which communities are missing?"

"The featured listing at 7202 Ayrshire Lane is missing the MLS number. What is it?"
```

### Step 4: Claude Generates All Assets
Once validation is complete, Claude produces:
- email.html
- [article-slug].html (e.g., nyc-tenant-czar-article.html)
- index.html (dashboard update)
- [community-name]-listings.html (for each requested community)

### Step 5: Deployment Checklist
Claude provides step-by-step deployment instructions with file paths.

## Required Input Data Structure

### Article Details
```
Title: [Full article title]
Author: [Edmund Bogen | Samantha Gornstein | Dina Ulrich | Nicole Hudson]
Date: [Month DD, YYYY format - e.g., January 7, 2026]
Hero Image: [filename.png or filename.jpg]
Article Slug: [url-friendly-version - e.g., nyc-tenant-czar-article]
Summary: [2-3 sentence summary for email]
Full Article Content: [Complete article text with headings]
```

### Market Data
```
Market Data As Of: [Month DD, YYYY - must match Article Date]

Executive Summary Stats:
- St. Andrews Range: $[LOW]M-$[HIGH]M
- St. Andrews Active: [NUMBER]
- Total Active (all 21 communities): [NUMBER]
- Total Sold YTD: [NUMBER]
- Status Label: [e.g., "New Year", "Fresh Start", "Q1 Strong"]

All 21 Communities (Nicole provides this):
[Paste community stats - format below]
```

### Community Stats Format
```
Community Name | Active | Sold YTD | Absorption Rate
St. Andrews Country Club | 13 | 0 | N/A
Delaire Country Club | 4 | 0 | N/A
Boca West Country Club | 39 | 1 | 3%
[etc. for all 21]
```

### Featured St. Andrews Listings (Top 3)
```
1. Address: [Street Address]
   Price: $[PRICE]
   Beds: [#] | Baths: [#] | SqFt: [#,###]
   MLS#: [MLS NUMBER]
   Days on Market: [#]
   Badge: [New Construction 2026 | Waterfront | Golf & Lake Views | Price Reduced | blank]
   Link: [Full URL from bogenhomes.com]

2. [Same structure]

3. [Same structure]
```

### Communities Needing Full Listings Pages
```
Generate listings pages for:
- [ ] St. Andrews Country Club
- [ ] Polo Club  
- [ ] Seven Bridges
- [ ] [Other communities as needed]
```

For each checked community, user must provide:
- Price range
- Active count
- Sold YTD
- Absorption rate (or "N/A")
- Full listing data for ALL active properties (copy/pasted from MLS via bogenhomes.com)

## Brand Guidelines Application

All generated HTML must follow The Edmund Bogen Team brand guidelines:

### Colors
- Primary Navy: `#1a3e5c` (60% usage)
- Bright Blue/Cyan: `#00a8e1` (10-15% for CTAs and accents)
- White: `#ffffff` (25-30% for balance)
- Black: `#000000` (text on white, dramatic headers)
- Light Gray: `#f4f4f4` to `#e8e8e8` (backgrounds)

### Typography
- Font Family: Montserrat (Google Fonts) for article pages
- System fonts for email: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`
- Headers: Light weight (300), ALL CAPS with wide letter-spacing
- Body: Regular (400), 16-18px minimum, 1.6-1.8 line-height
- CTAs: ALL CAPS, medium weight, bright blue background

### Layout Principles
- Institutional, data-driven aesthetic (investment research + luxury real estate)
- Full-width hero images with navy overlay gradient
- Large vertical spacing between sections (40-60px)
- Clean stat cards with left border accents
- No playful UI patterns
- Authority > friendliness, Precision > decoration

See `/mnt/skills/user/edmund-bogen-brand/SKILL.md` for complete brand guidelines.

## Asset Generation Specifications

### Asset 1: Email HTML (Constant Contact)

**File name:** `email.html`

**Structure:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Article Title]</title>
  <style>
    /* Inline CSS for email compatibility */
    /* Max width: 600px, single column */
  </style>
</head>
<body>
  <!-- Header: The Edmund Bogen Team + Market Intelligence label -->
  
  <!-- Hero Section: Article image with title/summary overlay -->
  <!-- Link to full article on GitHub Pages -->
  
  <!-- St. Andrews Listings Link -->
  
  <!-- Executive Market Summary: 4-stat grid -->
  
  <!-- All 21 Communities Link -->
  
  <!-- Community Stats Grid: 21 communities with Active/Sold/YTD -->
  
  <!-- Featured St. Andrews Listings: 3 property cards -->
  
  <!-- Edmund's Mastermind CTA -->
  
  <!-- Interactive Dashboard Link -->
  
  <!-- About Edmund/Author Bio: Photo, credentials, contact -->
  
  <!-- Footer: Douglas Elliman legal/compliance -->
</body>
</html>
```

**Critical Email Rules:**
- Inline CSS only (no external stylesheets)
- Max width 600px
- Single column layout
- Mobile-first responsive stacking
- All images must have absolute URLs (edmundbogen.github.io)
- Test in multiple email clients

### Asset 2: Article Page HTML (GitHub Pages)

**File name:** `[article-slug].html` (e.g., `nyc-tenant-czar-article.html`)

**Structure:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Article Title] | The Edmund Bogen Team</title>
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  <style>
    /* External CSS allowed */
    /* Montserrat font family */
    /* Full-width hero with overlay */
  </style>
</head>
<body>
  <!-- Navigation: Logo + Properties/Contact links -->
  
  <!-- Hero Image: Full-width with navy gradient overlay -->
  
  <!-- Article Header: Category label + Title + Author + Date -->
  
  <!-- Key Stats Callouts: 3 large metrics -->
  
  <!-- Article Content: Multiple H2 sections -->
  <!-- Pull Quotes: Highlighted quotes with left border -->
  <!-- Data Tables: Cost increases, impact numbers -->
  <!-- Embedded visualizations if needed -->
  
  <!-- CTA Section: "Why South Florida Continues to Win" -->
  
  <!-- Author Bio: Photo, full credentials, contact -->
  
  <!-- Footer: Team branding + disclaimer -->
</body>
</html>
```

**Critical Article Rules:**
- Montserrat font via Google Fonts
- Full-width hero image (1920px+)
- Navy overlay gradient on hero (40-60% opacity)
- Long-form prose (no bullet points in body unless explicitly in source content)
- Pull quotes styled with left cyan border
- Author bio matches author specified in input

### Asset 3: Dashboard HTML (GitHub Pages)

**File name:** `index.html` (updates existing dashboard)

**Structure:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Luxury Communities Market Intelligence Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    /* Chart.js styling */
  </style>
</head>
<body>
  <!-- Header: EBT Logo + Title + Subtitle + Date -->
  
  <!-- St. Andrews CTA Link -->
  
  <!-- Featured Article Hero: Same image/title as email -->
  
  <!-- Executive Market Summary: Same 4-stat grid as email -->
  
  <!-- Chart Tabs: Absorption Rates, Inventory Analysis, Price Trends -->
  <!-- Chart.js interactive visualizations -->
  
  <!-- Edmund's Mastermind CTA -->
  
  <!-- Community Performance Grid: 21 communities -->
  
  <!-- Interactive Analysis Tools -->
  
  <!-- Edmund Bio -->
  
  <!-- Footer -->
</body>
</html>
```

**Critical Dashboard Rules:**
- Chart.js library for interactive charts
- Same featured article section as email (image/title/summary/link)
- Same executive summary stats as email
- All 21 community cards with consistent data
- Links to individual community listings pages

### Asset 4: Community Listings Pages HTML (GitHub Pages - Templated)

**File name:** `[community-slug]-listings.html` (e.g., `st-andrews-listings.html`)

**This is a TEMPLATE that works for any of the 21 communities**

**Structure:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Community Name] - Available Listings | The Edmund Bogen Team</title>
  <style>
    /* Property card styling */
  </style>
</head>
<body>
  <!-- Back Navigation: ← Back to Market Dashboard -->
  
  <!-- Page Title: [Community Name] - Available Listings -->
  <!-- Updated Date + Active Count -->
  
  <!-- 4-Stat Summary: Price Range, Active, Sold YTD, Absorption -->
  
  <!-- Property Cards (repeated for each listing):
       - Address + Feature Badge (conditional)
       - Price
       - 6-attribute grid: Beds, Baths, Living SqFt, Year Built, Lot SqFt
       - MLS#
       - View Details CTA (link to bogenhomes.com)
       - Days on Market badge
  -->
  
  <!-- Contact CTA: Questions about [Community Name]? -->
  
  <!-- Footer: Team branding + phone -->
</body>
</html>
```

**Critical Listings Page Rules:**
- Community name is variable throughout
- Stats are community-specific
- Property cards are dynamically generated from input data
- Feature badges are conditional (New Construction, Waterfront, Golf & Lake Views, Price Reduced)
- Days on Market is calculated or provided
- View Details links must go to bogenhomes.com URLs (provided in input)

## Validation Checklist

Before generating assets, Claude MUST verify:

- [ ] Article title is identical in all locations
- [ ] Article date matches market data date matches dashboard date
- [ ] Article image filename is consistent everywhere
- [ ] All 21 community stats are present
- [ ] Executive summary stats match across email and dashboard
- [ ] Featured St. Andrews listings have all required fields (address, price, beds, baths, sqft, MLS, days, link)
- [ ] Author bio matches specified author
- [ ] All colors match brand palette (#1a3e5c, #00a8e1, #ffffff)
- [ ] Typography follows brand guidelines
- [ ] All images use absolute URLs (edmundbogen.github.io)
- [ ] Links are functional and point to correct destinations

If ANY validation item fails, Claude must stop and ask for correction before proceeding.

## Output Format

After generation, Claude provides:

1. **All HTML files** as separate code blocks with clear file names
2. **Deployment checklist** with step-by-step instructions
3. **File path map** showing where each file goes

Example output:
```
✅ Generated 6 files:

1. email.html → Upload to Constant Contact
2. nyc-tenant-czar-article.html → Push to GitHub: /nyc-tenant-czar-article/index.html
3. index.html → Push to GitHub: /luxury-community-dashboard/index.html
4. st-andrews-listings.html → Push to GitHub: /luxury-community-dashboard/st-andrews-listings.html
5. polo-club-listings.html → Push to GitHub: /luxury-community-dashboard/polo-club-listings.html
6. seven-bridges-listings.html → Push to GitHub: /luxury-community-dashboard/seven-bridges-listings.html

📧 Email Recipients: edmund@bogenhomes.com, nicole@, dina@, samantha@
```

## Edge Cases & Error Handling

### Missing Community Data
If user provides data for only 19 of 21 communities:
- Flag the missing communities by name
- Ask if they should be excluded or if data needs to be added
- Do not proceed until resolved

### Inconsistent Dates
If article date ≠ market data date:
- Flag the discrepancy
- Ask which date is correct
- Update all instances to match

### Missing Featured Listings
If fewer than 3 St. Andrews featured listings provided:
- Ask if this is intentional (slow market week)
- Offer to reduce featured section to 2 or 1
- Adjust email/dashboard layouts accordingly

### Author Not Specified
If author field is blank:
- Default to "Edmund Bogen"
- Ask for confirmation
- Update bio section with correct author

### Invalid Image Format
If image filename doesn't end in .png, .jpg, or .jpeg:
- Flag the issue
- Ask for correct filename
- Verify file exists or will exist at edmundbogen.github.io path

## Team Usage Instructions

See `team-workflow-template.md` for the fill-in-the-blank template team members use.

**Typical workflow:**
1. Nicole compiles weekly community stats (copy/paste from MLS)
2. Edmund/Samantha/Dina writes article
3. Team member opens Claude with this skill
4. Pastes filled template
5. Answers Claude's validation questions
6. Receives all HTML files
7. Deploys to Constant Contact + GitHub
8. Sends email to team

**Estimated time:** 15-20 minutes from data collection to deployed assets

## Technical Notes

- Email HTML uses inline CSS for maximum compatibility
- Article page and dashboard use external CSS + Google Fonts
- Chart.js loaded via CDN for dashboard
- All images must be hosted on edmundbogen.github.io before generation
- Mobile responsive breakpoints: 768px, 1024px
- Douglas Elliman legal footer required on all assets

## Success Criteria

Generated assets are successful if:
1. ✅ All 4+ HTML files are valid and render correctly
2. ✅ No broken links or missing images
3. ✅ Brand guidelines are followed perfectly
4. ✅ Data is consistent across all assets
5. ✅ Email renders correctly in Gmail, Outlook, Apple Mail
6. ✅ Team can deploy without edits
7. ✅ Mobile responsive on all screen sizes

---

## Quick Start for Team Members

**"I need to generate this week's market intelligence package."**

1. Fill out `team-workflow-template.md`
2. Paste completed template into Claude
3. Answer any clarifying questions
4. Receive all HTML files
5. Deploy following provided checklist

That's it. The skill handles all the complexity, validation, and generation automatically.
