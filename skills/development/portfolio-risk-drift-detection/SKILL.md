---
name: portfolio-risk-drift-detection
description: Detect and explain risk drift in lending portfolios over time using vintage analysis, migration matrices, and concentration metrics. Use when monitoring portfolio credit quality trends, preparing board risk reports, conducting stress testing, or when risk metrics deviate from appetite thresholds.

metadata:
  display_name: "Portfolio Risk Drift Detection"
  short_description: "Detect credit risk drift in lending portfolios over time"
  default_prompt: "Review my portfolio risk drift and highlight top risks and next actions"
  version: "1.0.0"
  tags:
    - financial-services
  icon_path: "assets/icon.png"
---

# Portfolio Risk Drift Detection

## Overview

Monitor and quantify changes in lending portfolio risk profiles over time. This skill applies vintage analysis, credit migration matrices, concentration tracking, and early warning indicators to detect drift from the institution's risk appetite before losses materialize. Outputs include trend decomposition, attribution of drift drivers, and actionable recommendations for portfolio management committees.

## When to Use

- Quarterly portfolio risk reporting to board and risk committees
- Detecting origination quality deterioration across vintages
- Monitoring concentration limit breaches (geography, industry, borrower)
- Stress testing portfolio resilience under adverse scenarios
- Evaluating impact of credit policy changes on portfolio composition
- Responding to early warning signals from delinquency or migration trends

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| Loan tape | Full portfolio with origination and performance data | Loan-level extract |
| Vintage tags | Origination quarter/year for each loan | Date field |
| Risk ratings | Internal risk grades or PD bands | Rating scale |
| Performance data | Delinquency status, charge-offs, modifications | Monthly snapshots |
| Risk appetite | Board-approved limits and thresholds | Policy document |
| Macro indicators | Unemployment, HPI, GDP, interest rate forecasts | Economic data feed |

## Methodology

### Step 1 — Vintage Curve Analysis

Construct cumulative default and loss curves by origination vintage:

- Plot cumulative net charge-off (NCO) rate by months-on-book for each vintage
- Compare recent vintages against seasoned benchmarks at equivalent age points
- Calculate the **vintage deviation index**: ratio of current vintage NCO to benchmark NCO at the same month-on-book
- Flag vintages where the deviation index exceeds 1.2 (20% worse than benchmark)
- Decompose deviations into volume effect (mix shift) vs. rate effect (quality shift)

### Step 2 — Credit Migration Matrix Construction

Build transition matrices showing rating movement between periods:

- Construct NxN migration matrix for each reporting period (quarterly recommended)
- Calculate upgrade rate, downgrade rate, and stability rate for each grade
- Compute the **weighted average migration score** (WAMS): positive = portfolio upgrading, negative = degrading
- Compare current migration patterns against historical norms (3-year, 5-year averages)
- Identify grades with abnormal downgrade velocity (>1.5x historical rate)

### Step 3 — Concentration Drift Tracking

Monitor portfolio concentration across key dimensions:

- **Geographic**: Exposure by state/MSA vs. limits
- **Industry/sector**: NAICS-code concentration vs. diversification targets
- **Borrower size**: Single-name concentration and top-10/top-25 exposure
- **Product mix**: Fixed vs. variable, secured vs. unsecured, term vs. revolving
- **Risk grade distribution**: Percentage in pass, watch, substandard, doubtful

Calculate the Herfindahl-Hirschman Index (HHI) for each dimension and track trend. Flag any dimension where HHI increases by >10% quarter-over-quarter or breaches the policy limit.

### Step 4 — Early Warning Indicator Dashboard

Compute and trend the following leading indicators:

- **30-day delinquency rate** by vintage and product (earliest signal)
- **Roll rate analysis**: Probability of transitioning from current → 30 DPD → 60 DPD → 90 DPD → charge-off
- **Probability of Default (PD) migration**: Weighted average PD of the portfolio vs. prior periods
- **Loss Given Default (LGD) trend**: Recovery rates on realized defaults
- **Expected Credit Loss (ECL)** under CECL/IFRS 9: Lifetime loss estimate movement
- **Criticized asset ratio**: Classified + special mention as percentage of total loans

### Step 5 — Attribution Analysis

Decompose observed drift into root causes:

- **Origination quality**: Are new bookings riskier than historical averages?
- **Seasoning effect**: Are older loans performing as expected at their age?
- **Economic environment**: Can macro factor shifts explain the observed migration?
- **Policy changes**: Did recent credit policy modifications alter the risk profile?
- **Portfolio runoff**: Are payoffs concentrated in lower-risk segments, leaving a riskier residual pool?

Use a decomposition framework: Total drift = Origination effect + Seasoning effect + Economic effect + Mix effect + Residual

### Step 6 — Stress Scenario Overlay

Project portfolio performance under stress:

- Apply regulatory stress scenarios (CCAR severely adverse) to current portfolio
- Calculate stressed PD, LGD, and ECL under each scenario
- Compare stressed losses to capital buffers and loan loss reserves
- Assess whether observed drift has increased vulnerability to stress
- Quantify the incremental capital needed if drift continues at current trajectory

### Step 7 — Report Generation and Recommendations

Produce a comprehensive drift report with:

- Portfolio risk dashboard with traffic-light indicators
- Trend charts for all key metrics (12-quarter minimum horizon)
- Specific drift findings with root cause attribution
- Comparison to risk appetite thresholds (within / approaching / breached)
- Recommended management actions (tighten origination, increase reserves, hedging)

## Output Specification

```
## Portfolio Risk Drift Report — [Period]

### Risk Dashboard
| Metric | Current | Prior Qtr | YoY Change | Appetite Limit | Status |
|--------|---------|-----------|------------|----------------|--------|
| WA PD | X.XX% | X.XX% | +/- X bps | X.XX% | [G/Y/R] |
| WA LGD | XX.X% | XX.X% | +/- X pp | XX.X% | [G/Y/R] |
| NCO Rate | X.XX% | X.XX% | +/- X bps | X.XX% | [G/Y/R] |
| 30+ DPD | X.XX% | X.XX% | +/- X bps | X.XX% | [G/Y/R] |
| HHI (Geo) | X,XXX | X,XXX | +/- XXX | X,XXX | [G/Y/R] |
| Criticized Ratio | X.XX% | X.XX% | +/- X bps | X.XX% | [G/Y/R] |

### Vintage Analysis
- Worst-performing vintage: [Quarter] — [X]% above benchmark at [N] MOB
- Vintages on watch: [List]

### Migration Summary
- Weighted average migration score: [X.XX] (negative = deteriorating)
- Grades with elevated downgrade velocity: [List]

### Drift Attribution
- Origination quality: [X]% of total drift
- Economic environment: [X]% of total drift
- Mix/runoff effect: [X]% of total drift
- Policy change impact: [X]% of total drift

### Stress Test Impact
- Baseline ECL: $[X]M
- Severely adverse ECL: $[X]M (+[X]%)
- Capital adequacy under stress: [Adequate / Marginal / Insufficient]

### Recommendations
1. [Action — responsible party — timeline]
2. [Action — responsible party — timeline]
3. [Action — responsible party — timeline]
```

## Analysis Framework

Apply the **VMIC** framework:

- **V**intage — Compare cohort performance curves against benchmarks
- **M**igration — Track credit grade transitions for directional drift
- **I**ndicators — Monitor leading delinquency and loss signals
- **C**oncentration — Measure diversification changes across dimensions

## Examples

**Example 1 — Origination Quality Deterioration**

Finding: Q3 2025 vintage auto loans show 30-day delinquency of 4.2% at 6 MOB vs. 2.8% benchmark (deviation index: 1.50). Attribution: 68% origination quality (average FICO dropped 22 points post-policy change), 32% economic (regional unemployment spike). Recommendation: Revert minimum FICO to pre-change threshold, apply geographic risk overlay for affected MSAs.

**Example 2 — Concentration Breach**

Finding: CRE office exposure reached 342% of Tier 1 capital (limit: 300%). HHI for sector concentration increased 18% QoQ. Migration matrix shows 12% of CRE office loans downgraded in the quarter (historical norm: 4%). Recommendation: Pause new CRE office originations, accelerate workout for substandard credits, present remediation plan to board within 30 days.

## Guidelines

- Use at least 12 quarters of historical data for trend analysis and benchmark construction
- Apply both absolute thresholds and rate-of-change triggers for early warning
- Separate portfolio drift from expected seasoning patterns using age-adjusted metrics
- Validate vintage curves against actual loss emergence before using as benchmarks
- Coordinate findings with ALCO, credit risk committee, and finance (reserve adequacy)
- Present migration matrices at the individual grade level, not aggregated pass/fail
- Document all data sources, exclusions, and methodology assumptions

## Validation Checklist

- [ ] Vintage curves are age-adjusted for fair comparison across cohorts
- [ ] Migration matrices are complete (all grades represented, rows sum to 100%)
- [ ] Concentration metrics reflect current outstandings, not commitments alone
- [ ] Early warning indicators use consistent definitions across periods
- [ ] Attribution analysis accounts for all material drift drivers
- [ ] Stress scenarios are current (aligned with latest CCAR/DFAST guidance)
- [ ] Risk appetite thresholds reflect board-approved limits, not informal targets
- [ ] Report includes both quantitative metrics and narrative context
- [ ] Data quality issues are disclosed and their impact on conclusions assessed
- [ ] Recommendations are prioritized by urgency and expected impact
