---
name: care-program-effectiveness
description: Measure the clinical, utilization, and financial impact of care management programs and population health interventions. Use when evaluating program outcomes, conducting pre/post analyses, comparing intervention cohorts, or reporting program ROI to stakeholders.

metadata:
  display_name: "Care Program Effectiveness"
  short_description: "Evaluate care management program outcomes and ROI impact"
  default_prompt: "Analyze my care program effectiveness and recommend clear next actions"
  version: "1.0.1"
  tags:
    - healthcare
  icon_path: "assets/icon.png"
---

# Care Program Effectiveness

## Overview

This skill evaluates the effectiveness of care management programs — disease management, care coordination, transitional care, remote patient monitoring, and behavioral health integration — by measuring outcomes against matched comparison groups. It applies quasi-experimental methods, difference-in-differences analysis, and propensity score matching to isolate program impact from secular trends.

## When to Use

- Evaluating whether a care management program achieved its intended outcomes
- Conducting annual program effectiveness reviews for leadership or board reporting
- Comparing intervention cohorts to matched controls for causal attribution
- Quantifying cost avoidance and utilization reduction attributable to programs
- Deciding whether to expand, modify, or sunset a care program

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| Program enrollment data | Enrollment dates, program type, engagement level | Program roster |
| Claims/encounter data | Pre- and post-enrollment claims for participants and comparison | Claims detail |
| Clinical outcomes | Lab values, vitals, functional assessments | Clinical data |
| Cost data | Allowed amounts, PMPM, total cost of care | Financial tables |
| Eligibility data | Continuous enrollment, LOB, demographics | Enrollment file |
| Program operational data | Touchpoints, interventions delivered, care plans | Program logs |

## Methodology

### Step 1 — Define Program Cohort and Measurement Periods

Establish clear cohort boundaries:

- **Enrollment criteria**: Minimum program engagement threshold (e.g., ≥ 2 care manager contacts)
- **Pre-period**: 12 months before program enrollment (baseline measurement)
- **Post-period**: 12 months after program enrollment (outcome measurement)
- **Wash-out period**: Exclude first 30-60 days post-enrollment to allow intervention ramp-up
- **Continuous enrollment**: Require continuous eligibility through pre and post periods
- **Exclusion criteria**: Deceased during measurement, disenrolled, < minimum engagement

### Step 2 — Construct Comparison Group

Build a matched comparison cohort using propensity score methods:

- **Matching variables**: Age, sex, baseline RAF score, prior-year cost, prior-year utilization (IP admits, ED visits), chronic condition count, dual-eligible status
- **Matching method**: 1:1 nearest-neighbor propensity score matching with caliper ≤ 0.2 SD
- **Balance diagnostics**: Verify standardized mean differences < 0.1 on all matching variables
- **Sensitivity analysis**: Test robustness with alternative methods (exact matching, inverse probability weighting)

If a formal comparison group is unavailable, use pre-post with trend adjustment as a secondary approach, acknowledging weaker causal inference.

### Step 3 — Measure Outcome Domains

Evaluate across four outcome domains:

**Clinical Outcomes**:
- Disease-specific metrics (HbA1c change for diabetes, BP control for hypertension)
- Composite quality scores (percentage meeting all HEDIS targets)
- Medication adherence (PDC ≥ 80%)

**Utilization Outcomes**:
- Inpatient admissions per 1,000 (all-cause and condition-specific)
- ED visits per 1,000 (all-cause and avoidable per NYU algorithm)
- 30-day readmission rate
- PCP and specialist visit frequency

**Cost Outcomes**:
- Total cost of care PMPM (allowed amount, including all service categories)
- Category-specific cost (IP, ED, outpatient, pharmacy, ancillary)
- Cost trend (program vs. comparison growth rate)

**Patient Experience Outcomes** (if available):
- Patient satisfaction scores, self-reported health status, functional assessments

### Step 4 — Calculate Program Impact

Apply difference-in-differences (DID) methodology:

```
Impact = (Post_program - Pre_program) - (Post_comparison - Pre_comparison)
```

- Calculate DID for each outcome metric
- Test statistical significance using regression with clustered standard errors
- Report 95% confidence intervals for all impact estimates
- Calculate effect sizes (Cohen's d) for clinical outcomes

### Step 5 — Assess Dose-Response Relationship

Evaluate whether greater program engagement produces larger effects:

- Stratify by engagement level (low: 1-2 contacts, medium: 3-6, high: 7+)
- Test for dose-response gradient across engagement tiers
- Identify minimum effective dose (engagement threshold for measurable impact)
- Analyze engagement predictors (demographics, condition type, referral source)

### Step 6 — Calculate Cost-Effectiveness

Determine program financial performance:

- **Gross savings**: Total cost avoidance in intervention group vs. comparison
- **Program cost**: Staffing, technology, overhead allocated to program
- **Net savings**: Gross savings minus program cost
- **ROI**: (Net savings / Program cost) × 100
- **Cost per quality-adjusted outcome**: Program cost / number of members achieving target

### Step 7 — Synthesize and Recommend

Produce an integrated assessment:

- Summarize findings across all four outcome domains
- Rate program effectiveness: highly effective, moderately effective, inconclusive, or ineffective
- Identify sub-populations with strongest and weakest response
- Recommend: expand, maintain, modify (specify modifications), or sunset
- Define metrics and timeline for next evaluation cycle

## Output Specification

```
Program Effectiveness Report:
├── Executive Summary (program description, key findings, recommendation)
├── Methodology Overview (study design, matching, measurement periods)
├── Cohort Description (enrollment, demographics, baseline equivalence)
├── Clinical Outcomes (pre/post, DID, significance, effect size)
├── Utilization Outcomes (admits, ED, readmissions — DID with CI)
├── Cost Outcomes (PMPM, total cost, cost avoidance)
├── Dose-Response Analysis (engagement tiers, response gradient)
├── ROI Calculation (gross savings, net savings, cost per outcome)
├── Sub-Group Analysis (by condition, acuity, demographics)
├── Limitations and Sensitivity Analyses
└── Recommendations and Next Steps
```

## Analysis Framework

### Effectiveness Rating Criteria

| Rating | Clinical | Utilization | Cost |
|--------|----------|-------------|------|
| Highly effective | ≥ 10% improvement, p < 0.01 | ≥ 15% reduction in IP/ED | ROI > 2:1 |
| Moderately effective | 5-10% improvement, p < 0.05 | 5-15% reduction | ROI 1:1-2:1 |
| Inconclusive | < 5% change or p > 0.05 | < 5% change | ROI < 1:1 |
| Ineffective | No improvement or worsening | No reduction or increase | Net cost |

### Common Program Benchmarks

- Disease management programs: 8-15% IP reduction, ROI 1.5:1-3:1
- Transitional care: 20-30% readmission reduction in first 90 days
- Remote patient monitoring: 15-25% ED reduction for enrolled CHF patients
- Care coordination: 5-10% total cost reduction for high-risk members

## Examples

**Example 1 — CHF Care Management Program**
Evaluate a CHF disease management program with 1,200 enrolled members over 18 months. Match 1:1 against 1,200 non-enrolled CHF patients. DID analysis shows: IP admissions −22% (p=0.003), ED visits −18% (p=0.01), 30-day readmissions −31% (p=0.001), total cost PMPM −$142 (p=0.008). Program cost $1.8M, gross savings $3.1M, net savings $1.3M, ROI 1.72:1. Recommend expansion to all CHF patients with EF < 40%.

**Example 2 — Diabetes Prevention Program**
Assess a DPP-modeled lifestyle intervention for 800 pre-diabetic members. After 12 months, 34% of participants achieved ≥ 5% weight loss vs. 12% in comparison (p < 0.001). Diabetes conversion rate 4.2% vs. 8.8% in comparison (p=0.02). Program cost per prevented diabetes case: $3,200 vs. estimated 5-year diabetes care cost of $48,000.

## Guidelines

- Propensity score matching requires sufficient overlap in covariate distributions; report overlap statistics
- Pre-post analysis without a comparison group is insufficient for causal claims — label clearly as associational
- Account for regression to the mean, particularly for programs targeting high-cost members
- Report all outcomes including null or negative findings to avoid publication bias
- Use intent-to-treat analysis as primary; per-protocol as sensitivity analysis

## Validation Checklist

- [ ] Comparison group is well-balanced on all matching variables (SMD < 0.1)
- [ ] Pre-period trends are parallel between groups (parallel trends assumption)
- [ ] Continuous enrollment requirement applied consistently to both groups
- [ ] Program costs include all direct and allocated indirect costs
- [ ] Statistical tests are appropriate for outcome data types
- [ ] Sensitivity analyses confirm robustness of primary findings
- [ ] Limitations are clearly documented

## HIPAA Compliance

This skill processes Protected Health Information (PHI) for program evaluation, which constitutes health care operations under HIPAA (45 CFR §164.506). All outputs must comply with HIPAA Privacy and Security Rules. Apply minimum necessary standards to data access. De-identify comparison group outputs. Individual-level program data must be maintained in access-controlled systems. Report aggregate outcomes only in external communications, applying minimum cell-size suppression (n ≥ 11) for sub-group analyses.
