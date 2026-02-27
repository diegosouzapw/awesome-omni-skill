---
name: fermi-estimation
description: Break down unknowable quantities into estimable components to reach order-of-magnitude accuracy when making quick decisions without precise data
---

# Fermi Estimation

**Category:** Decision-Making & Strategic Thinking
**Source:** Enrico Fermi (1901-1954) / Physics / Google Interview Process
**Practitioner Score:** 46/50 (Tier 1 Canonical)

## Overview

Fermi Estimation (also called order-of-magnitude estimation or back-of-the-envelope calculation) is a technique for making surprisingly accurate approximations using minimal data by decomposing complex problems into smaller, estimable parts. Named after physicist Enrico Fermi, who was legendary for his ability to estimate seemingly impossible quantities (e.g., "How many piano tuners in Chicago?").

**Core Insight:** When facing unknown quantities, breaking the problem into multiplication of smaller estimates converges toward accuracy. Even if each component is off by 2-3x, errors tend to cancel out, yielding results within the correct order of magnitude.

**Power:** Enables quick decision-making when precise data is unavailable, provides sanity checks for detailed analyses, and develops intuition about numerical relationships.

## When to Use

- **Quick feasibility checks** - Is this market opportunity worth pursuing?
- **Strategy decisions** - Rough-sizing of market, revenue, cost before deep analysis
- **Sanity testing** - Does this projection/claim pass the smell test?
- **Resource planning** - Ballpark staffing, budget, capacity needs
- **Problem-solving** - Google-style interview questions and real-world unknowns
- **Forecasting** - Superforecasting decomposition (Step 2 of Tetlock method)

**Anti-patterns:**
- Precise calculations already available (use those instead)
- Life-critical decisions (medical, safety, legal)
- Situations where 10x error is unacceptable
- Contract negotiations (need exact figures)

## How to Execute

### Step 1: Clarify the Question
**Action:** Define exactly what you're estimating
- **Scope boundaries:** Geography, time period, population segment
- **Success criteria:** Within 1x? 2x? 10x of actual?
- **Example:** "Piano tuners in Chicago" → full-time professionals, present day
- **Output:** Clear, bounded question

### Step 2: Break Down into Estimable Components
**Action:** Decompose into multiplication of simpler factors
- **Identify variables:** What do you need to know?
- **Chain logic:** Variable A × Variable B × Variable C = Answer
- **Example:** Piano tuners = (Pianos in city) ÷ (Pianos per tuner can service per year)
- **Output:** Mathematical breakdown with 3-7 variables

### Step 3: Estimate Each Component
**Action:** Make reasonable assumptions for each factor
- **Use round numbers:** 3M instead of 2.7M (easier mental math)
- **Anchor to knowns:** "About the size of..." "Roughly X% of..."
- **Bounds:** Lower and upper limits help triangulate
- **Output:** Numeric estimate for each component

**Piano Tuners Example:**
- Chicago population: ~3 million people
- People per household: ~3
- Households with pianos: ~1 in 20 (5%)
- **Pianos in Chicago:** 3M ÷ 3 × 0.05 = 50,000 pianos
- Tunings per piano per year: ~1
- Days per tuner per year: ~250
- Pianos per tuner per day: ~4
- **Pianos per tuner per year:** 250 × 4 = 1,000
- **Piano tuners:** 50,000 ÷ 1,000 = **50 tuners**

### Step 4: Perform the Calculation
**Action:** Multiply/divide components using mental math
- **Round aggressively:** 9 → 10, 48 → 50
- **Powers of 10:** Track orders of magnitude separately
- **Example:** (5 × 10⁴) ÷ (1 × 10³) = 5 × 10¹ = 50
- **Output:** Numeric result

### Step 5: Sanity Check
**Action:** Does the answer make intuitive sense?
- **Compare to related quantities:** More or less than expected?
- **Test extremes:** What if assumptions were 2x off?
- **Example:** 50 piano tuners → ~1 per 60,000 people (seems reasonable)
- **Output:** Validated or revised estimate

### Step 6: Express as Order of Magnitude
**Action:** Report result with appropriate precision
- **Round to 1 significant figure:** "About 50" or "order of 10²"
- **Range:** "Between 30 and 100"
- **Confidence:** "Probably within 2-3x of actual"
- **Output:** Order-of-magnitude estimate with uncertainty

### Step 7: Refine if Stakes Warrant
**Action:** If decision is important, improve weak assumptions
- **Lookup 1-2 key facts:** Chicago population, average household size
- **Sensitivity analysis:** Which variables matter most?
- **Iterate:** Recalculate with better inputs
- **Output:** Tightened estimate (still order-of-magnitude)

## Real-World Examples

**Enrico Fermi - Manhattan Project:**
- Estimated yield of first nuclear test by dropping paper scraps
- Calculated blast energy from scrap displacement
- Result: Within 2x of actual yield (before any instruments measured)

**Google Interviews:**
- "How many golf balls fit in a school bus?"
- "How much would you charge to wash all windows in Seattle?"
- Purpose: Test structured thinking and numerical reasoning

**Startup Market Sizing:**
- TAM estimation: (Target users) × (willingness to pay) × (capture rate)
- Example: "B2B SaaS for dentists" → 200K practices × $100/mo × 5% = $10M addressable
- Result: Quick go/no-go decision before expensive market research

**Engineering Sanity Checks:**
- "Can our server handle 10M users?" → requests/sec × bytes/request × response time
- Catches order-of-magnitude errors before deployment
- Result: Fast validation of architectural decisions

## Integration Points

**Complements:**
- **Superforecasting:** Core tool for "Fermi-style decomposition" (Commandment 2)
- **Expected Value:** Estimate probabilities and outcomes for EV calculation
- **Base Rate Analysis:** Use reference classes to estimate components
- **Sensitivity Analysis:** Identify which factors most affect result

**Powers:**
- **Pre-analysis:** Fermi estimate before committing to expensive research
- **Validation:** Check if detailed model results are order-of-magnitude correct
- **Teaching:** Develops numerical intuition and structured thinking

## Common Pitfalls

**Pitfall 1: Forgetting to Decompose**
- **Warning sign:** Stuck trying to estimate the final answer directly
- **Fix:** Always break into components first

**Pitfall 2: False Precision**
- **Warning sign:** Calculating to 3 decimal places from rough estimates
- **Fix:** Round aggressively, report as order of magnitude

**Pitfall 3: Correlated Errors**
- **Warning sign:** All assumptions biased in same direction (all too high or too low)
- **Fix:** Deliberately vary bias direction, use bounds

**Pitfall 4: Missing Conversion Factors**
- **Warning sign:** Mixing units (per day vs. per year, metric vs. imperial)
- **Fix:** Write out units explicitly, convert carefully

**Pitfall 5: Over-Complicated Breakdown**
- **Warning sign:** 15+ variables in decomposition
- **Fix:** Simplify to 3-7 key factors, merge correlated components

## Practice Problems

**Beginner:**
- How many gas stations in the United States?
- How many pizzas are delivered in your city per year?
- How many words have you spoken in your lifetime?

**Intermediate:**
- What's the total market size for electric bicycles in Europe?
- How much computing power does Google use globally?
- How many software engineers are there in the world?

**Advanced:**
- Estimate Tesla's electricity cost per vehicle manufactured
- What's the carbon footprint of global video streaming?
- How much would it cost to provide clean water to everyone on Earth?

## Validation Checklist

- [ ] Problem decomposed into 3-7 estimable components
- [ ] Each component estimated with reasonable assumptions
- [ ] Calculation performed with appropriate rounding
- [ ] Result expressed as order of magnitude (1 sig fig)
- [ ] Sanity check performed (does answer make sense?)
- [ ] Key assumptions documented (for later refinement if needed)
- [ ] Uncertainty acknowledged (within 2-3x of actual)

## Tips for Improvement

**Build Reference Library:**
- Population figures (US: 330M, world: 8B)
- Economic data (US GDP: $25T, median income: $70K)
- Common conversions (people per household, work days/year)

**Practice Regularly:**
- Daily estimation puzzles
- Compare estimates to actual values when revealed
- Track which types of assumptions you consistently over/under-estimate

**Calibrate Intuition:**
- "How many X in Y?" questions build pattern recognition
- Over time, estimates get faster and more accurate
- Meta-skill: knowing which factors matter most

## Further Reading

- **"Street-Fighting Mathematics"** - Sanjoy Mahajan (MIT, systematic estimation)
- **"Guesstimation"** - Lawrence Weinstein & John Adam (practice problems)
- **"How to Solve It"** - George Pólya (problem decomposition strategies)
- **Farnam Street:** "Fermi Thinking" article
- **Good Judgment Project:** Uses Fermi as core forecasting tool
