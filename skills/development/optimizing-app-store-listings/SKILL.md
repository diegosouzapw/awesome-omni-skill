---
name: optimizing-app-store-listings
description: Helps optimize mobile app listings for Apple App Store and Google Play Store. Use when working on ASO (App Store Optimization), writing app titles/descriptions/keywords, analyzing competitor apps, improving app visibility, increasing organic downloads, or when the user mentions app store rankings, metadata optimization, or mobile app marketing.
---

# App Store Optimization (ASO)

ASO is the process of improving an app's visibility and conversion rate in app stores by optimizing metadata, creative assets, and performance metrics. 70% of App Store visitors use search to find apps, and 65% of downloads occur immediately after a search.

## Quick Start

**Optimizing an existing app?** Jump to the [ASO Audit Checklist](#aso-audit-checklist).

**Writing new metadata?** Use the [Metadata Templates](#metadata-templates) below.

**Researching keywords?** See [Keyword Research Workflow](#keyword-research-workflow).

## Platform Character Limits

### Apple App Store

| Field | Limit | Indexed? | Notes |
|-------|-------|----------|-------|
| Title | 30 chars | Yes (highest weight) | Include primary keyword |
| Subtitle | 30 chars | Yes (high weight) | Expand on value proposition |
| Keywords | 100 chars | Yes | Comma-separated, no spaces after commas |
| Description | 4,000 chars | No | For users only, not search |
| Promotional Text | 170 chars | No | Can update without new version |

### Google Play Store

| Field | Limit | Indexed? | Notes |
|-------|-------|----------|-------|
| Title | 30 chars | Yes (highest weight) | Include primary keyword |
| Short Description | 80 chars | Yes (high weight) | Elevator pitch |
| Full Description | 4,000 chars | Yes | Use keywords naturally (~1 per 250 chars) |

## Core ASO Principles

### 1. Title is King

The app title carries the strongest indexation weight on both platforms. Structure it as:

```
[Brand Name] - [Primary Keyword/Value Prop]
```

**Examples:**
- `Calm - Sleep & Meditation`
- `Duolingo - Language Lessons`
- `Notion - Notes & Projects`

### 2. Natural Language Optimization

Modern app stores use NLP to understand conversational queries. Optimize for how users speak:

| Old Approach | New Approach |
|--------------|--------------|
| "budget tracker app" | "track my monthly expenses" |
| "meditation app" | "help me relax before bed" |
| "photo editor free" | "edit photos easily" |

### 3. Keyword Strategy by Platform

**Apple App Store:**
- Keywords field is hidden from users (your secret weapon)
- Separate keywords with commas, no spaces after commas
- Don't repeat words already in title/subtitle
- Use singular OR plural, not both (Apple indexes both)

**Google Play:**
- No hidden keyword field—everything is in visible metadata
- Repeat important keywords 3-5x naturally across title + descriptions
- Use long-tail phrases in full description
- Avoid keyword stuffing (hurts rankings)

### 4. Localization Matters

Localization goes beyond translation—it requires cultural adaptation. Keywords that perform well in English may not work in other markets. Always:

- Research local search behavior
- Adapt visuals for cultural context
- Test locally relevant screenshots

## Metadata Templates

### Apple App Store Template

```
TITLE (30 chars max):
[Brand] - [Primary Benefit]

SUBTITLE (30 chars max):
[Secondary Benefit/Feature]

KEYWORDS (100 chars max, comma-separated):
[keyword1],[keyword2],[long tail phrase],[keyword3]

DESCRIPTION STRUCTURE:
Paragraph 1: Hook + primary value proposition
Paragraph 2: Key features (bullet points work well)
Paragraph 3: Social proof (awards, press, user count)
Paragraph 4: Call to action
```

### Google Play Template

```
TITLE (30 chars max):
[Brand] - [Primary Benefit]

SHORT DESCRIPTION (80 chars max):
[Concise value prop with primary keyword]

FULL DESCRIPTION STRUCTURE:
Opening hook with primary keyword (first 80 chars visible)

KEY FEATURES:
• Feature 1 with keyword
• Feature 2 with keyword
• Feature 3 with keyword

WHY USERS LOVE US:
[Social proof, testimonials]

WHAT'S NEW:
[Recent updates showing active development]
```

## Keyword Research Workflow

### Step 1: Seed Keywords

Generate initial keywords from:
- Your app's core functionality
- User problems you solve
- Competitor app names and categories

### Step 2: Expand with Long-Tail

Transform generic keywords into specific phrases:

| Seed | Long-Tail Variations |
|------|---------------------|
| fitness | home workout for beginners, 7 minute workout, abs exercises |
| notes | voice memo recorder, meeting notes, quick notes widget |
| budget | expense tracker, spending analyzer, bill reminder |

### Step 3: Analyze Competition

For each target keyword:
1. Search in the app store
2. Note top 10 results
3. Analyze their titles, subtitles, descriptions
4. Identify gaps and opportunities

### Step 4: Prioritize Keywords

Score keywords by:
- **Search volume**: How many people search this term?
- **Relevance**: How well does it describe your app?
- **Competition**: How hard to rank for this term?
- **Conversion potential**: Will searchers want your app?

Target keywords with high relevance + moderate competition first.

## ASO Audit Checklist

Use this checklist to audit an existing app listing:

```
METADATA AUDIT
- [ ] Title includes primary keyword
- [ ] Title is under 30 characters
- [ ] Subtitle/short description provides clear value
- [ ] Keywords don't repeat words from title (iOS)
- [ ] Description leads with strongest hook
- [ ] Description includes social proof
- [ ] No prohibited terms (best, #1, free, top)

VISUAL ASSETS AUDIT
- [ ] Icon is distinctive and readable at small sizes
- [ ] Screenshots tell a story (not just UI dumps)
- [ ] First 2 screenshots show core value
- [ ] Screenshots have clear, readable captions
- [ ] Preview video (if any) hooks in first 3 seconds

PERFORMANCE AUDIT
- [ ] Rating is 4.0+ stars
- [ ] Recent reviews are positive
- [ ] Developer responds to negative reviews
- [ ] App is updated within last 3 months
- [ ] No major crash reports or bugs mentioned

LOCALIZATION AUDIT
- [ ] Metadata localized for target markets
- [ ] Screenshots localized (not just translated)
- [ ] Keywords researched per locale
```

## Common ASO Mistakes

1. **Keyword stuffing** - Hurts rankings and looks spammy
2. **Ignoring screenshots** - They're your strongest conversion tool
3. **Set and forget** - ASO requires continuous iteration
4. **Copying competitors exactly** - Find gaps, don't duplicate
5. **Using prohibited terms** - "Best," "#1," "Free" can get you rejected
6. **Neglecting ratings** - Low ratings tank conversion and rankings
7. **Poor localization** - Direct translation misses cultural context

## Timeline Expectations

ASO is a marathon, not a sprint:

- **Week 1-2**: Metadata changes indexed
- **Week 3-4**: Initial ranking shifts visible
- **Week 5-8**: Meaningful organic traffic changes
- **Week 8-12**: Significant ranking improvements
- **Ongoing**: Continuous optimization and testing

## Advanced Topics

For detailed guidance on specific areas, see:

- [keyword-research.md](keyword-research.md) - Deep dive on keyword strategy
- [visual-assets.md](visual-assets.md) - Screenshot and icon optimization
- [competitive-analysis.md](competitive-analysis.md) - Analyzing competitor listings
- [localization.md](localization.md) - International ASO strategy

## Guidelines

- Always research before recommending keywords
- Never use trademarked terms without permission
- Focus on relevance over volume for keywords
- Recommend A/B testing for major changes
- Consider both platforms' unique requirements
- Prioritize user experience over gaming algorithms
