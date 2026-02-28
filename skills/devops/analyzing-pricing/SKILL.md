---
name: analyzing-pricing
description: Analyzes pricing strategies, competitive pricing benchmarks, pricing models, value metrics, and willingness-to-pay to optimize pricing and positioning. Use when the user requests pricing analysis, competitive pricing comparison, pricing strategy, pricing model evaluation, or wants to optimize pricing decisions.
---

# Analyzing Pricing

This skill performs comprehensive pricing analysis including competitive benchmarking, pricing model evaluation, value metric identification, and pricing strategy recommendations.

## When to Use This Skill

Invoke this skill when the user:
- Requests pricing analysis or strategy
- Wants competitive pricing benchmarking
- Asks about pricing models (subscription, usage-based, tiered, etc.)
- Needs help identifying value metrics
- Mentions willingness-to-pay or price sensitivity
- Wants to optimize current pricing
- Asks "how should we price this?" or "are we priced correctly?"
- Needs pricing positioning recommendations

## Core Pricing Activities

### Competitive Pricing Benchmark

Systematically compare pricing across competitors:

**Steps:**
1. Identify direct and indirect competitors
2. Research pricing for each competitor (tiers, models, add-ons)
3. Normalize pricing for comparison (per user, per month, per unit)
4. Document what's included at each price point
5. Identify pricing patterns and clusters
6. Assess your pricing relative to competition
7. Determine pricing positioning (value/premium/budget)

**Output Format:**
```markdown
# Competitive Pricing Benchmark: [Category]

## Pricing Overview

| Competitor | Entry Price | Mid-Tier Price | Enterprise | Model | Notes |
|-----------|-------------|----------------|------------|-------|-------|
| Comp A | $10/user/mo | $25/user/mo | Custom | Per-user | Annual discount 20% |
| Comp B | $99/mo flat | $299/mo flat | Custom | Flat-rate | Up to 10 users |
| Comp C | $0.05/unit | $0.03/unit | $0.01/unit | Usage-based | Volume discounts |
| Your Product | $15/user/mo | $30/user/mo | Custom | Per-user | Annual discount 15% |

## Pricing Model Distribution
- Per-user subscription: 60%
- Flat-rate subscription: 20%
- Usage-based: 15%
- Freemium: 5%

## Price Point Analysis

**Entry Tier:**
- Range: $0 - $15/user/mo
- Median: $10/user/mo
- Your position: 75th percentile (premium)

**Mid Tier:**
- Range: $20 - $35/user/mo
- Median: $25/user/mo
- Your position: 60th percentile

**Enterprise:**
- Most use custom pricing
- Typical starting point: $500-1000/mo minimum

## Feature-to-Price Mapping

**Features included at entry tier:**
- [Feature A]: 90% of competitors
- [Feature B]: 70% of competitors
- [Feature C]: 30% of competitors

**Premium features (higher tiers only):**
- [Feature X]: Available at $25+/user
- [Feature Y]: Available at $30+/user

## Packaging Patterns

**Good-Better-Best:**
- Used by: 75% of competitors
- Typical structure: 3 tiers
- Middle tier captures: ~60% of customers

**Volume Discounts:**
- Start at: 10-25 users typically
- Discount range: 10-30%

## Your Pricing Position

**Relative Position:** Premium (top 25%)
**Justification:** [Whether features justify premium]
**Risk:** [Potential issues with current pricing]
**Opportunity:** [Pricing optimization possibilities]

## Key Insights

1. [Insight 1]: [Finding and implication]
2. [Insight 2]: [Finding and implication]
3. [Insight 3]: [Finding and implication]

## Recommendations

1. [Recommendation with rationale]
2. [Recommendation with rationale]
```

### Pricing Model Evaluation

Assess different pricing model options:

**Steps:**
1. Identify possible pricing models for the product
2. Analyze pros and cons of each model
3. Assess fit with customer value perception
4. Evaluate revenue predictability and scalability
5. Consider sales complexity and friction
6. Assess competitive alignment or differentiation
7. Recommend optimal model(s)

**Common Pricing Models:**

```markdown
# Pricing Model Analysis

## Model 1: Per-User (Seat-Based)

**Description:** Price per user/seat per time period

**Pros:**
- Predictable revenue per customer
- Easy to understand
- Scales with customer growth
- Industry standard in many categories

**Cons:**
- Incentivizes seat sharing/workarounds
- May not align with value delivered
- Can limit adoption in price-sensitive segments

**Best for:**
- Collaboration tools
- Team-based software
- When value scales with users

**Competitive prevalence:** High (60% of competitors)

**Revenue model:**
- Predictability: High
- Scalability: Linear with users
- Expansion potential: Moderate

## Model 2: Usage-Based (Consumption)

**Description:** Pay for what you use (API calls, transactions, compute time)

**Pros:**
- Perfect alignment with value delivered
- Low barrier to entry (start small)
- Natural expansion as usage grows
- Fair for sporadic users

**Cons:**
- Revenue unpredictability
- Customer budget uncertainty
- Requires strong usage tracking
- Sales complexity for forecasting

**Best for:**
- Infrastructure/platform services
- Variable usage patterns
- API products
- High-volume transactional use

**Competitive prevalence:** Growing (15% of competitors)

**Revenue model:**
- Predictability: Low to Medium
- Scalability: High with customer success
- Expansion potential: Very high

## Model 3: Tiered (Good-Better-Best)

**Description:** Multiple package tiers with different features/limits

**Pros:**
- Serves different customer segments
- Clear upgrade path
- Maximizes revenue capture
- Easy to compare

**Cons:**
- Requires careful tier design
- Feature allocation complexity
- Can confuse customers if too complex

**Best for:**
- Diverse customer base (SMB to Enterprise)
- Clear feature differentiation
- When bundling multiple capabilities

**Competitive prevalence:** Very High (75% use 3 tiers)

**Revenue model:**
- Predictability: High
- Scalability: Moderate
- Expansion potential: Built-in via upgrades

## Model 4: Freemium

**Description:** Free tier with paid upgrades

**Pros:**
- Low friction adoption
- Viral growth potential
- Large user base for product-led growth
- Built-in try-before-buy

**Cons:**
- Low conversion rates (typically 2-5%)
- High support costs for free users
- Risk of cannibalization
- Requires scale

**Best for:**
- Network-effect products
- High trial-to-paid conversion potential
- Low marginal cost per user
- Viral growth strategies

**Competitive prevalence:** Moderate (25%)

**Revenue model:**
- Predictability: Low (depends on conversion)
- Scalability: Very high
- Expansion potential: High if conversion works

## Model 5: Flat-Rate

**Description:** Single price for unlimited use/users

**Pros:**
- Extremely simple
- Predictable for customer
- Low sales friction
- Easy to communicate

**Cons:**
- Leaves revenue on table (large customers)
- May price out small customers
- Limited expansion revenue

**Best for:**
- Commoditized offerings
- Small customer base
- When simplicity is key differentiator

**Competitive prevalence:** Low (10%)

## Hybrid Models

**Per-user + Usage overage:**
- Base per-user fee + charges for usage above limit
- Example: $20/user + $0.10 per API call over 10,000

**Tiered + Usage:**
- Tiers based on usage bands
- Example: $99/mo for 0-10k events, $299/mo for 10-50k events

## Recommended Model: [Selected Model]

**Rationale:**
- [Reason 1]
- [Reason 2]
- [Reason 3]

**Implementation considerations:**
- [Consideration 1]
- [Consideration 2]
```

### Value Metric Identification

Determine what metric to base pricing on:

**Steps:**
1. Identify how customers perceive value
2. List possible value metrics (users, usage, outcomes, features)
3. Assess each metric's alignment with value delivery
4. Evaluate metric understandability and measurability
5. Consider growth characteristics of each metric
6. Assess competitive norms and customer expectations
7. Recommend primary value metric

**Value Metric Framework:**

```markdown
# Value Metric Analysis

## What is a Value Metric?

The unit you charge for (user, transaction, seat, GB, etc.) that ideally:
- Aligns with value customers receive
- Grows as customer grows
- Is easy to understand and predict
- Feels fair

## Candidate Metrics

### Metric 1: [e.g., Per Active User]

**Alignment with value:** High/Medium/Low
- Reasoning: [Does value scale with this metric?]

**Growth characteristics:**
- Natural growth: [Does this metric grow as customer succeeds?]
- Expansion potential: [Revenue expansion opportunity]

**Customer perception:**
- Understandability: High/Medium/Low
- Predictability: High/Medium/Low
- Fairness: High/Medium/Low

**Operational feasibility:**
- Measurability: Easy/Moderate/Difficult
- Gaming risk: Low/Medium/High (can customers manipulate?)

**Competitive context:**
- Industry standard: Yes/No
- Competitive differentiation: [If different from norm]

**Example pricing:**
- $X per [metric] per month

### Metric 2: [e.g., API Calls per Month]
[Same analysis structure]

### Metric 3: [e.g., Outcome-Based (e.g., $ saved)]
[Same analysis structure]

## Comparison Matrix

| Metric | Value Alignment | Growth | Predictability | Fairness | Ease |
|--------|----------------|--------|----------------|----------|------|
| Active Users | High | Medium | High | High | Easy |
| API Calls | Very High | High | Medium | Medium | Easy |
| Features Used | Low | Low | High | Low | Easy |
| Outcome | Very High | High | Low | Very High | Hard |

## Recommendation: [Selected Metric]

**Primary value metric:** [Chosen metric]

**Rationale:**
1. [Reason 1]
2. [Reason 2]
3. [Reason 3]

**Pricing structure:**
[How to implement, e.g., tiers based on this metric]

**Potential issues:**
- [Issue 1 and mitigation]
- [Issue 2 and mitigation]
```

### Willingness-to-Pay Assessment

Estimate customer willingness to pay:

**Steps:**
1. Analyze competitive pricing as floor/ceiling
2. Assess value delivered vs. alternatives
3. Research customer segment economics
4. Calculate ROI or value created
5. Review feedback/reviews mentioning price
6. Identify price sensitivity by segment
7. Recommend pricing ranges

**WTP Framework:**

```markdown
# Willingness-to-Pay Analysis

## Reference Points

**Competitive Range:**
- Low: $X (budget competitors)
- Mid: $Y (mainstream competitors)
- High: $Z (premium competitors)

**Current Alternative Costs:**
What customers pay today for alternatives:
- [Alternative 1]: $X
- [Alternative 2]: $Y
- [DIY/Manual process]: $Z (time cost)

## Value-Based Pricing

**Value Delivered:**
[Quantified value customer receives]

Examples:
- Time saved: 10 hours/week × $50/hour = $500/week value
- Revenue increase: 5% increase on $100k = $5k/year value
- Cost reduction: $2k/month savings

**Value Capture:**
Typical pricing = 10-30% of value delivered

**Maximum WTP:** $[Value created]
**Suggested price:** $[10-30% of value] (leaving most value with customer)

## Segment-Specific WTP

### Enterprise Segment
- **Budget availability:** High
- **Value perception:** [Productivity, risk reduction, scalability]
- **WTP range:** $X - $Y per [unit]
- **Price sensitivity:** Low
- **Decision criteria:** ROI, features, security > price

### Mid-Market Segment
- **Budget availability:** Medium
- **Value perception:** [Efficiency, cost savings]
- **WTP range:** $X - $Y per [unit]
- **Price sensitivity:** Medium
- **Decision criteria:** Value for money, ease of use

### SMB Segment
- **Budget availability:** Low
- **Value perception:** [Affordability, quick wins]
- **WTP range:** $X - $Y per [unit]
- **Price sensitivity:** High
- **Decision criteria:** Price, simplicity, fast ROI

## Price Sensitivity Signals

**From Reviews:**
- "Too expensive" mentions: [Frequency]
- "Great value" mentions: [Frequency]
- Price-related churn mentions: [Frequency]

**From Competitive Comparison:**
- Lost deals to cheaper alternatives: [%]
- Won deals despite higher price: [%]

## Anchoring Strategy

**Anchor high, discount down:**
- List price: $[Higher price]
- Discount for: annual commit, early adopter, volume
- Creates perception of value

**Anchor medium:**
- List price at market median
- Premium tier for expansion
- Entry tier for acquisition

## Recommended Pricing Ranges

**Entry tier:** $X - $Y
- Target: SMB, individual users
- Rationale: [Why this range]

**Mid tier:** $X - $Y
- Target: Mid-market, teams
- Rationale: [Why this range]

**Enterprise:** $X+ or Custom
- Target: Large organizations
- Rationale: Value-based, custom packaging
```

### Pricing Optimization Analysis

Evaluate current pricing for optimization:

**Steps:**
1. Analyze current pricing structure
2. Review pricing-related metrics (conversion, churn, expansion)
3. Identify pricing friction points
4. Assess packaging clarity and appeal
5. Evaluate discount strategy impact
6. Test pricing change scenarios
7. Recommend specific optimizations

**Optimization Areas:**

```markdown
# Pricing Optimization Recommendations

## Current State Assessment

**Current pricing:**
- [Tier 1]: $X/[unit]
- [Tier 2]: $Y/[unit]
- [Tier 3]: $Z/[unit]

**Performance metrics:**
- Tier distribution: [% in each tier]
- Annual vs. monthly: [Split]
- Discount rate: [Average discount %]
- Price-related churn: [%]

## Identified Issues

### Issue 1: [e.g., Poor tier distribution]
- **Problem:** 80% of customers in lowest tier
- **Impact:** Leaving revenue on table
- **Root cause:** Middle tier not differentiated enough
- **Fix:** Repackage tiers, move key feature to middle tier

### Issue 2: [e.g., High price-related churn]
- **Problem:** XX% cite price in churn reason
- **Impact:** $XXk annual revenue loss
- **Root cause:** Price increase not justified with value
- **Fix:** Improve value communication, grandfather pricing

### Issue 3: [e.g., Discount dependency]
- **Problem:** Average discount 30%, list price rarely paid
- **Impact:** Eroded price positioning, negotiation expectation
- **Root cause:** Sales comp incentivizes discounting
- **Fix:** Reduce list price, limit discounting authority

## Optimization Opportunities

### Opportunity 1: Add Usage-Based Overage
- **Change:** Base tier + overage charges for excess usage
- **Impact:** Capture expansion revenue from high-usage customers
- **Risk:** May cause sticker shock
- **Recommendation:** Test with 10% of new customers

### Opportunity 2: Introduce Annual Discounting
- **Change:** 20% discount for annual pre-pay
- **Impact:** Improve cash flow, reduce churn
- **Risk:** Revenue timing shift
- **Recommendation:** Implement immediately

### Opportunity 3: Value Metric Change
- **Change:** Switch from per-user to per-project pricing
- **Impact:** Better value alignment, higher ACV
- **Risk:** Customer confusion, conversion impact
- **Recommendation:** A/B test with new customers

### Opportunity 4: Premium Tier Addition
- **Change:** Add enterprise tier at 2x mid-tier price
- **Impact:** Capture enterprise budget, anchor existing tiers
- **Risk:** Unclear differentiation
- **Recommendation:** Build if 20% of customers request missing features

## Pricing Experiment Plan

**Test 1: [Name]**
- Hypothesis: [What you expect]
- Change: [What to modify]
- Segment: [Who to test with]
- Metrics: [What to measure]
- Duration: [How long]
- Success criteria: [How to decide]

## Expected Impact

**Revenue impact:**
- Scenario 1 (Conservative): +X%
- Scenario 2 (Expected): +Y%
- Scenario 3 (Optimistic): +Z%

**Implementation timeline:**
- Month 1: [Actions]
- Month 2: [Actions]
- Month 3: [Measure and iterate]
```

## Pricing Research Methods

**Method 1: Competitive Price Scraping**
- **Sources:** Competitor pricing pages
- **Tools:** WebFetch to retrieve pricing pages
- **Approach:** Systematic documentation of prices, tiers, features
- **Frequency:** Quarterly or when competitors change pricing

**Method 2: Review Mining for Price Perception**
- **Sources:** G2, Capterra, TrustRadius reviews
- **Tools:** WebSearch for reviews
- **Focus:** Price-related comments (value, too expensive, cheap)
- **Insight:** Understand price sensitivity and value perception

**Method 3: Customer Economics Research**
- **Approach:** Research target customer budgets, alternatives
- **Tools:** WebSearch for industry salary data, tool budgets
- **Goal:** Understand ability and willingness to pay

**Method 4: Value Quantification**
- **Approach:** Calculate ROI customer achieves
- **Method:** Time saved × hourly rate, or revenue increase, cost reduction
- **Goal:** Determine how much value created to price against

## Validation Checklist

Before finalizing pricing analysis:

- [ ] Competitive pricing researched across 5+ competitors
- [ ] Pricing normalized for apples-to-apples comparison
- [ ] Features mapped to price points
- [ ] Pricing model options evaluated
- [ ] Value metric identified and justified
- [ ] Willingness-to-pay assessed by segment
- [ ] Current pricing issues identified
- [ ] Optimization opportunities prioritized
- [ ] Revenue impact estimated
- [ ] Implementation plan outlined
- [ ] Pricing positioned strategically (value/premium/budget)

## Examples

**Example 1: SaaS Pricing Benchmark**

Input: "Analyze competitive pricing for project management software"

Process:
1. Identify top 10 competitors
2. Research pricing pages for each
3. Document pricing tiers, models, per-user costs
4. Calculate median and range per tier
5. Map features to price points
6. Identify pricing patterns
7. Recommend pricing position

Output: Competitive pricing benchmark with recommended pricing range and positioning

**Example 2: Pricing Model Selection**

Input: "Should we use per-user or usage-based pricing for our API product?"

Process:
1. Analyze both models (pros/cons)
2. Assess value metric alignment (usage aligns better for API)
3. Review competitive norms in API space
4. Evaluate revenue predictability tradeoffs
5. Consider customer preference and predictability
6. Model revenue scenarios
7. Recommend usage-based with volume tiers

Output: Pricing model recommendation with rationale and implementation guidance

## Additional Notes

- Pricing is both art and science - data informs, but judgment decides
- Test pricing changes with new customers before changing existing
- Price increases require strong value justification
- Always grandfather existing customers or phase in gradually
- Packaging (what's in each tier) is as important as price
- Discounting should be strategic, not default
- Monitor competitor pricing changes quarterly
- Combine with analyzing-customers to understand price sensitivity
- Link to competitive-intelligence for competitive pricing moves
- Use analyzing-business-models to ensure pricing supports unit economics
