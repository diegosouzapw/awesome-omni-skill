---
name: Content Performance Explainer
description: Diagnose and explain why e-commerce content is or isn't performing against KPIs, using causal analysis frameworks, funnel decomposition, and competitive benchmarking to generate actionable improvement recommendations.

metadata:
  display_name: "Content Performance Explainer"
  short_description: "Explain e-commerce content performance drivers and KPIs"
  default_prompt: "Explain my content performance in simple words and next steps"
  version: "1.0.0"
  tags:
    - cpg-retail
  icon_path: "assets/icon.png"
---

# Content Performance Explainer

## Overview

This skill transforms raw content performance data into clear, actionable explanations of *why* content is succeeding or failing. It moves beyond descriptive analytics ("CVR dropped 12%") to diagnostic and prescriptive analysis ("CVR dropped because the new title lost the primary keyword, reducing search-driven traffic quality by 18% — here's the fix").

Most teams drown in dashboards but starve for insight. This skill bridges the gap between data and decision-making for content teams, brand managers, and e-commerce leaders.

## When to Use

- Explaining a sudden change (positive or negative) in content performance metrics.
- Conducting periodic content performance reviews (weekly, monthly, quarterly).
- Justifying content investment or optimization spend with clear ROI narratives.
- Diagnosing why a content update didn't produce expected results.
- Comparing performance across SKUs, categories, or time periods.
- Preparing executive-facing content performance reports.
- Post-mortem analysis after A/B test results.

## Required Inputs

| Input | Description | Example |
|---|---|---|
| `performance_data` | Time-series metrics for the content being analyzed | Impressions, clicks, CTR, CVR, sessions, orders, revenue, ACoS |
| `content_versions` | Current and historical versions of the content | Title, bullets, images, A+ content with timestamps |
| `time_period` | Analysis window and comparison period | "Last 30 days vs. prior 30 days" |
| `channel` | Platform (affects available metrics) | "Amazon", "Walmart", "DTC Shopify" |
| `category_benchmarks` | Category average performance metrics | `{ "avg_ctr": 0.045, "avg_cvr": 0.12, "avg_aov": 24.50 }` |
| `competitive_data` | Competitor rankings, content changes, pricing | ASIN tracking data |
| `external_factors` | Known events that may impact performance | "Prime Day", "competitor launched new SKU", "seasonal shift" |
| `search_data` | Search term reports, keyword rankings | Keyword rank changes, search volume trends |
| `advertising_data` | Paid media spend and performance (if applicable) | Sponsored Products/Brands data |

## Methodology

### Step 1 — Funnel Decomposition

Break content performance into a sequential funnel, isolating where changes occur:

```
Search Impressions → Clicks (CTR)
    → Detail Page Views → Add-to-Cart (ATC Rate)
        → Purchase (CVR) → Revenue
            → Repeat Purchase → LTV
```

**Stage-by-Stage Diagnostic**:

| Stage | Metrics | Content Levers | Non-Content Factors |
|---|---|---|---|
| **Impressions** | Search impressions, browse impressions | Title keywords, backend terms, category placement | Bid changes, search volume trends, seasonality |
| **CTR** | Click-through rate from search | Title copy, main image, price display, rating/review count | Competitor pricing, ad placements, search result position |
| **Detail Page Views** | Sessions, glance views | (Transition metric — influenced by CTR) | External traffic sources, social referrals |
| **ATC Rate** | Add-to-cart percentage | Bullet points, A+ content, images, pricing, reviews | Stock availability, shipping speed, Subscribe & Save |
| **CVR** | Unit session percentage | Full PDP experience, trust signals, social proof | Checkout friction, payment options, competitor offers |
| **Revenue** | Total sales, revenue per session | Upsell content, bundle presentation, variant selection | Pricing strategy, promotions, AOV |

### Step 2 — Change Attribution Analysis

When performance shifts, identify the most likely cause using the **Attribution Hierarchy**:

1. **Content Changes** (highest attribution certainty): Did any content element change during the period? Compare versions side-by-side.
2. **Competitive Changes**: Did key competitors change pricing, launch new products, or update content?
3. **Algorithmic Changes**: Did search rankings shift without content changes? Possible algorithm update.
4. **Market/Seasonal**: Is there a known seasonal pattern, category trend, or macroeconomic factor?
5. **Advertising Changes**: Did ad spend, bid strategy, or campaign structure change?
6. **External Events**: PR events, social media virality, influencer mentions, recalls, news coverage.

Apply the **Counterfactual Test**: "If this factor had NOT changed, would performance have remained stable?" The factor with the strongest counterfactual is the primary driver.

### Step 3 — Content Element Impact Scoring

Score each content element's contribution to overall performance:

| Content Element | Impact on CTR | Impact on CVR | Diagnostic Questions |
|---|---|---|---|
| **Product Title** | Very High | Medium | Are primary keywords present? Is the benefit clear in first 60 chars? |
| **Main Image** | Very High | Medium | Does it stand out in search results? Is the product clearly visible? |
| **Price/Deal Badge** | High | High | Is pricing competitive? Are promotions visible? |
| **Rating & Review Count** | High | High | Is rating ≥ 4.0? Is review count ≥ 50? |
| **Bullet Points** | Low | High | Do bullets answer top customer questions? Are benefits front-loaded? |
| **A+ / Enhanced Content** | None | Medium-High | Is enhanced content present? Does it reduce bounce and build confidence? |
| **Secondary Images** | None | Medium | Do images demonstrate use cases, ingredients, and size context? |
| **Product Description** | None | Low-Medium | Is it readable and keyword-rich? (Less impactful on Amazon) |

### Step 4 — Performance Narrative Construction

Build a clear, stakeholder-ready explanation using the **Situation → Analysis → Recommendation (SAR)** framework:

**Situation**: State the performance change in business terms.
- "SKU X revenue declined 22% month-over-month ($45K → $35K), driven primarily by a 15% CVR drop."

**Analysis**: Explain the root cause with supporting evidence.
- "The CVR decline coincides with a title change on March 5 that removed the primary keyword 'organic protein powder.' Search impression share dropped 30%, and remaining traffic was less purchase-intent aligned. Competitor Y also launched a new SKU at $2 lower price point, capturing 8% of our branded search impressions."

**Recommendation**: Provide specific, prioritized actions.
- "Priority 1: Restore primary keyword to title (expected +20% impression recovery in 7-14 days). Priority 2: Add competitive comparison in A+ content to defend against competitor Y's price positioning."

### Step 5 — Benchmark Contextualization

Frame performance within appropriate context:

1. **Category Benchmarks**: Compare against category averages — an 8% CVR might be excellent in electronics but poor in grocery.
2. **Historical Trend**: Is this a new decline or continuation of a long-term trend?
3. **Seasonality Adjustment**: Remove seasonal effects to see underlying performance.
4. **Portfolio Context**: How does this SKU perform relative to the brand's other SKUs?
5. **Market Growth/Decline**: Is the entire category growing or contracting?

### Step 6 — Predictive Outlook & Action Prioritization

Project future performance under different scenarios:

| Scenario | Assumptions | Projected Impact |
|---|---|---|
| **Do Nothing** | Current trends continue | -X% revenue over next 30 days |
| **Quick Fix** | Implement Priority 1 recommendation | +Y% recovery within 2-3 weeks |
| **Full Optimization** | Implement all recommendations | +Z% improvement over 60 days |

Prioritize recommendations using the **Impact × Speed Matrix**:

| | Fast (< 1 week) | Medium (1-4 weeks) | Slow (> 4 weeks) |
|---|---|---|---|
| **High Impact** | Do immediately | Schedule this sprint | Plan for next quarter |
| **Medium Impact** | Do immediately | Backlog — prioritize by ICE | Evaluate ROI first |
| **Low Impact** | Do if easy | Deprioritize | Skip |

## Output Specification

```yaml
output:
  executive_summary: string           # 2-3 sentence performance narrative
  performance_change:
    metric: string                     # Primary KPI analyzed
    current_value: float
    previous_value: float
    change_pct: float
    direction: string                  # "improved" | "declined" | "stable"
  funnel_analysis:
    impressions: { value: float, change: float, health: string }
    ctr: { value: float, change: float, health: string }
    cvr: { value: float, change: float, health: string }
    revenue: { value: float, change: float, health: string }
  root_cause_analysis:
    primary_driver: string
    contributing_factors: list[string]
    confidence: float                  # 0-100 confidence in attribution
    evidence: list[string]
  content_element_scores: dict         # Element → impact assessment
  benchmark_comparison:
    vs_category: string                # "above" | "at" | "below"
    vs_historical: string
    percentile: float                  # Category performance percentile
  recommendations:
    - priority: int
      action: string
      expected_impact: string
      timeline: string
      effort: string                   # "low" | "medium" | "high"
  projected_scenarios: dict
```

## Analysis Framework

**Content Performance Health Score**: Aggregate metric combining multiple dimensions:

| Dimension | Weight | Healthy | Warning | Critical |
|---|---|---|---|---|
| Search Visibility | 25% | Impressions stable/growing | -10% to -20% MoM | > -20% MoM |
| Click Efficiency | 20% | CTR ≥ category avg | CTR 70-99% of avg | CTR < 70% of avg |
| Conversion Effectiveness | 25% | CVR ≥ category avg | CVR 70-99% of avg | CVR < 70% of avg |
| Content Completeness | 15% | All fields populated, A+ live | Missing 1-2 elements | Missing title/bullet optimization or A+ |
| Competitive Position | 15% | Top 3 organic rank | Rank 4-10 | Rank > 10 or declining |

**Health Score**: 0-100. Traffic light system: Green (≥ 75), Yellow (50-74), Red (< 50).

## Examples

**Scenario**: Organic granola bar on Amazon. Revenue dropped 35% in 4 weeks.

**Executive Summary**: "Revenue declined 35% ($28K → $18K) over 4 weeks, primarily driven by a 40% drop in search impressions after losing page-1 organic rank for 'organic granola bars.' Root cause: a title edit on Feb 1 replaced the primary keyword with a brand sub-line. Secondary factor: a key competitor launched a Subscribe & Save offer, improving their conversion and organic rank. Restoring the keyword to the title is the highest-priority fix, with expected recovery within 10-14 days."

**Funnel Breakdown**:
- Impressions: -40% (search rank dropped from #3 to #18)
- CTR: +5% (fewer but more brand-aware impressions)
- CVR: -8% (competitor's S&S offer pulled comparison shoppers)
- Net Revenue Impact: -35%

**Recommendation Priority Stack**:
1. Restore "organic granola bars" to title position 1-3. (Impact: High, Speed: Fast, Effort: Low)
2. Enable Subscribe & Save at 5% discount. (Impact: High, Speed: Medium, Effort: Medium)
3. Update A+ comparison chart to address competitor's value proposition. (Impact: Medium, Speed: Medium, Effort: Medium)

## Guidelines

- Always separate correlation from causation — a content change and a performance shift occurring simultaneously doesn't prove causation without controlling for other variables.
- Present findings with appropriate confidence levels — don't overstate certainty when multiple factors coincide.
- Tailor the depth and language of the explanation to the audience (executive = high-level SAR; content team = detailed funnel with element-level recommendations).
- Include "what's working" alongside "what's broken" — reinforce winning content patterns to prevent accidental regression.
- Acknowledge data limitations — Amazon's attribution window, delayed reporting, and aggregated metrics all introduce uncertainty.
- Compare against the right benchmark — a luxury skincare brand should not benchmark against mass-market grocery.

## Validation Checklist

- [ ] Funnel is decomposed stage-by-stage with metrics for each stage.
- [ ] Content changes during the analysis period are identified and version-compared.
- [ ] Non-content factors (competitive, seasonal, advertising) are assessed and accounted for.
- [ ] Root cause attribution uses the counterfactual test and is stated with confidence level.
- [ ] Performance is contextualized against category benchmarks, historical trends, and seasonality.
- [ ] Recommendations are specific, prioritized by impact × speed, and include expected effect sizes.
- [ ] Projected scenarios (do nothing / quick fix / full optimization) are provided.
- [ ] Executive summary follows the SAR framework and is stakeholder-ready.
- [ ] Content element impact scores are calculated for each PDP component.
- [ ] Analysis distinguishes between correlation and causation with appropriate caveats.
