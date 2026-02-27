---
name: geo-visibility-audit
description: Perform manual AI search visibility audits without requiring API tools. Use when auditing brand visibility across AI engines, analyzing competitor presence, or building GEO strategies based on direct AI search testing.
---

# GEO Visibility Audit

Perform comprehensive AI search visibility analysis by directly testing AI search engines. This skill provides a systematic approach to auditing without requiring external API tools.

## Why Visibility Audits Matter

Only 16% of brands systematically track AI search performance, creating a significant opportunity gap. Understanding your current visibility baseline is essential for improvement.

**Key statistics:**

- AI-sourced visitors convert at 27% vs 2.1% from traditional search (12x improvement)
- Web mentions correlate 3x more strongly with AI visibility than backlinks
- 40-60% of domains cited in AI answers change within one month
- 26% of brands have zero mentions in AI Overviews

## Visibility Audit Framework

### Step 1: Define Audit Scope

Before testing, define these parameters:

**Brand Information:**

- Primary brand name
- Alternative names/spellings
- Key products/services
- Target markets/regions

**Competitor Set:**

- 3-5 direct competitors
- 2-3 indirect competitors (optional)

**Topic Clusters to Test:**

- Primary category queries ("best [your category]")
- Product-specific queries ("[your product type] features")
- Comparison queries ("[competitor A] vs [competitor B]")
- How-to/educational queries ("how to [your use case]")

### Step 2: Test Across AI Platforms

Test your defined queries across these platforms:

| Platform       | Access               | Citation Style      | Notes                      |
| -------------- | -------------------- | ------------------- | -------------------------- |
| ChatGPT        | chat.openai.com      | 3-4 citations avg   | Highest search volume      |
| Perplexity     | perplexity.ai        | 13 citations avg    | Shows sources prominently  |
| Google AI Mode | google.com (AI Mode) | Integrated results  | Affects traditional search |
| Claude         | claude.ai            | Conversational      | Growing user base          |
| Bing Copilot   | bing.com             | Microsoft ecosystem | Enterprise users           |

**For each query, document:**

1. Is your brand mentioned?
2. How is your brand described (accurate/inaccurate)?
3. Which competitors are mentioned?
4. What sources are cited?
5. What position does your brand appear (if mentioned)?

### Step 3: Query Testing Template

Use these query patterns for comprehensive coverage:

**Category Queries:**

```
"What are the best [your category]?"
"Top [your category] in 2026"
"[Your category] comparison"
```

**Product Queries:**

```
"What is [your product]?"
"How does [your product] work?"
"[Your product] features"
"[Your product] pricing"
```

**Comparison Queries:**

```
"[Your brand] vs [competitor]"
"[Competitor A] vs [Competitor B] vs [Competitor C]"
"Best [your category] comparison"
```

**How-To Queries:**

```
"How to [your use case]"
"[Your category] tutorial"
"[Your category] getting started"
```

### Step 4: Record Visibility Data

Create a tracking spreadsheet with these columns:

| Query   | Platform | Brand Mentioned | Position    | Description Accurate | Competitors Mentioned | Sources Cited |
| ------- | -------- | --------------- | ----------- | -------------------- | --------------------- | ------------- |
| [query] | ChatGPT  | Yes/No          | 1st/2nd/etc | Yes/No/Partial       | [list]                | [list]        |

### Step 5: Analyze Key Metrics

Calculate these metrics from your testing:

#### Brand Visibility Rate

```
Visibility % = (Queries where brand mentioned / Total queries tested) × 100
```

**Benchmarks:**
| Visibility | Rating | Interpretation |
|------------|--------|----------------|
| > 30% | Excellent | Strong AI presence |
| 20-30% | Good | Solid foundation |
| 10-20% | Fair | Room for improvement |
| < 10% | Poor | Significant gap |

#### Context Accuracy Rate

```
Accuracy % = (Accurate descriptions / Total mentions) × 100
```

Target: 95%+ accuracy

**Check for:**

- Correct product descriptions
- Accurate pricing information
- Current feature lists
- Proper brand positioning
- No outdated information

#### Competitive Share of Voice

```
SOV % = (Your mentions / Total competitor mentions) × 100
```

**Benchmarks:**
| Share of Voice | Position |
|----------------|----------|
| > 25% | Category leader |
| 15-25% | Major player |
| 5-15% | Competitive |
| < 5% | Visibility gap |

### Step 6: Gap Analysis

Identify where competitors appear but you don't:

**Create a gap matrix:**

| Query Topic | Your Visibility | Competitor A | Competitor B | Gap Severity |
| ----------- | --------------- | ------------ | ------------ | ------------ |
| [topic]     | 0%              | 40%          | 30%          | Critical     |
| [topic]     | 20%             | 50%          | 40%          | High         |
| [topic]     | 40%             | 45%          | 35%          | Moderate     |

**Gap categories:**

1. **Complete absence** - You never appear (critical priority)
2. **Weak presence** - You appear < 10% vs competitor > 30% (high priority)
3. **Competitive** - Similar visibility (maintain/optimize)
4. **Leadership** - You dominate (protect and expand)

### Step 7: Citation Source Analysis

Document which sources AI systems cite:

**Common citation sources:**

- Wikipedia/Wikidata
- Industry publications
- Review sites (G2, Capterra, Trustpilot)
- Reddit discussions
- YouTube videos
- LinkedIn articles
- Official documentation

**Identify gaps:**

- Sources citing competitors but not you
- Sources you could target for mentions
- Missing presence on key platforms

### Step 8: Engine-by-Engine Comparison

Compare your performance across platforms:

| Engine     | Your Visibility | Top Competitor | Gap | Priority     |
| ---------- | --------------- | -------------- | --- | ------------ |
| ChatGPT    | \_%             | \_%            | \_% | High/Med/Low |
| Perplexity | \_%             | \_%            | \_% | High/Med/Low |
| Google AI  | \_%             | \_%            | \_% | High/Med/Low |
| Claude     | \_%             | \_%            | \_% | High/Med/Low |

## Audit Report Template

```markdown
# AI Search Visibility Audit Report

**Brand:** [Brand Name]
**Date:** [Date]
**Queries Tested:** [Number]
**Platforms Tested:** [List]

## Executive Summary

[2-3 paragraph summary of key findings]

## Key Metrics

| Metric           | Current | Target | Status |
| ---------------- | ------- | ------ | ------ |
| Brand Visibility | X%      | 30%+   | ✓/✗    |
| Context Accuracy | X%      | 95%+   | ✓/✗    |
| Share of Voice   | X%      | 15%+   | ✓/✗    |

## Visibility by Platform

| Platform   | Visibility | Accuracy | Notes   |
| ---------- | ---------- | -------- | ------- |
| ChatGPT    | X%         | X%       | [notes] |
| Perplexity | X%         | X%       | [notes] |
| Google AI  | X%         | X%       | [notes] |

## Competitor Comparison

| Competitor     | Visibility | SOV    | Key Advantages  |
| -------------- | ---------- | ------ | --------------- |
| Competitor A   | X%         | X%     | [advantages]    |
| Competitor B   | X%         | X%     | [advantages]    |
| **Your Brand** | **X%**     | **X%** | [current state] |

## Top Visibility Gaps

1. **[Query/Topic]**

   - Your visibility: X%
   - Top competitor: X% (Competitor A)
   - Opportunity: [Description]
   - Priority: Critical/High/Medium

2. **[Query/Topic]**
   - Your visibility: X%
   - Top competitor: X%
   - Opportunity: [Description]
   - Priority: Critical/High/Medium

## Context Accuracy Issues

- [ ] [Issue 1: e.g., "Outdated pricing mentioned"]
- [ ] [Issue 2: e.g., "Missing key product feature"]
- [ ] [Issue 3: e.g., "Incorrect company description"]

## Citation Source Gaps

**Currently citing your brand:**

- [Source 1]
- [Source 2]

**Citing competitors but not you:**

- [Source 1] - Cites: [Competitor A, B]
- [Source 2] - Cites: [Competitor A]

## Recommendations

### Immediate Actions (This Week)

1. [Specific action with expected impact]
2. [Specific action with expected impact]

### Short-Term (This Month)

1. [Specific action with expected impact]
2. [Specific action with expected impact]

### Medium-Term (This Quarter)

1. [Specific action with expected impact]
2. [Specific action with expected impact]

## Next Audit Date

[Schedule: Monthly recommended]
```

## Action Planning Based on Findings

### Quick Wins (1-2 weeks)

- Update outdated content on high-visibility pages
- Fix context accuracy issues
- Add missing schema markup
- Refresh dates on cornerstone content

### Medium Priority (1-3 months)

- Create content for visibility gap topics
- Build presence on citation source sites
- Develop comparison content
- Expand FAQ coverage

### Strategic Initiatives (3-6 months)

- Build third-party authority (PR, partnerships)
- Develop original research/data
- Create comprehensive resource hubs
- Establish thought leadership content

## Ongoing Monitoring Cadence

| Check                  | Frequency | Purpose                    |
| ---------------------- | --------- | -------------------------- |
| Quick visibility check | Weekly    | Spot major changes         |
| Full query testing     | Bi-weekly | Track trends               |
| Competitor review      | Monthly   | Monitor competitive shifts |
| Full audit             | Quarterly | Comprehensive assessment   |

## Tips for Effective Testing

1. **Use incognito/private browsing** to avoid personalized results
2. **Test at different times** to account for variation
3. **Document exact query phrasing** for consistent retesting
4. **Screenshot responses** for evidence and comparison
5. **Note response dates** to track freshness of AI answers
6. **Test on mobile and desktop** for different experiences
