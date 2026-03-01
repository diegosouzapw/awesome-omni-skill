---
name: weekly-bank-exec-brief
description: Auto-generate weekly executive summaries for bank leadership covering financial performance, risk metrics, market developments, and strategic initiatives. Use when preparing CEO/CFO weekly briefs, executive dashboard narratives, or condensing multi-source banking data into concise leadership communications.

metadata:
  display_name: "Weekly Bank Exec Brief"
  short_description: "Generate weekly executive performance briefs for bank leaders"
  default_prompt: "Summarize my weekly bank exec with key findings and next steps"
  version: "1.0.0"
  tags:
    - financial-services
  icon_path: "assets/icon.png"
---

# Weekly Bank Executive Brief

## Overview

Produces a structured weekly executive summary for bank C-suite and senior leadership that synthesizes financial performance, risk positions, market context, regulatory developments, and strategic initiative progress into a concise, decision-oriented brief. Designed to replace ad-hoc reporting with a standardized format that enables rapid executive consumption and action.

## When to Use

- Generating the weekly CEO/CFO management information brief
- Synthesizing data from multiple source systems into executive-ready commentary
- Producing board-ready weekly performance snapshots
- Summarizing week-over-week changes in key financial and risk metrics
- Tracking strategic initiative milestones and escalation items

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| Financial performance | Week-to-date and MTD P&L, NII, fee income, provisions, expenses | Financial summary |
| Balance sheet snapshot | Total assets, loans, deposits, wholesale funding, capital ratios | Key line items |
| Risk dashboard | LCR, credit quality indicators (NPL, coverage), market risk (VaR), operational risk events | Risk metrics |
| Market data | Key rates, equity indices, credit spreads, FX rates | Market observations |
| Regulatory/compliance | Regulatory developments, examination findings, compliance items | Narrative updates |
| Strategic initiatives | Status of key projects, milestones achieved/missed, budget tracking | Project status |
| Competitive intelligence | Notable peer actions, market share data, industry news | Curated updates |

## Methodology

### Step 1 — Establish the Performance Baseline

Extract and validate the core financial metrics for the reporting week:

**P&L metrics:**
- Net interest income (NII): WTD actual vs. budget, variance in $ and %
- Non-interest income: Fee income by line (wealth, cards, trading), WTD vs. budget
- Provision for credit losses: WTD charge, reserve build/release, comparison to run-rate
- Operating expenses: WTD actual vs. budget, efficiency ratio (expenses / revenue)
- Pre-tax pre-provision income (PTPP): Revenue − Expenses before provisions

**Balance sheet metrics:**
- Total assets, total loans (by major category), total deposits (by type)
- Week-over-week changes in absolute and percentage terms
- Loan-to-deposit ratio, asset growth annualized

**Return metrics:**
- ROAA and ROAE (annualized from WTD/MTD figures)
- NIM (annualized from WTD NII / average earning assets)
- Efficiency ratio (non-interest expense / total revenue)

### Step 2 — Assess Risk Positions

Summarize the risk posture across all major risk types:

**Credit risk:**
- NPL ratio, NPL migration (new formations, cures, charge-offs)
- Provision coverage ratio (allowance / NPLs)
- Watch list movements (upgrades, downgrades, new additions)
- Large exposure or single-name concentration changes

**Liquidity risk:**
- LCR level and week-over-week change
- Deposit flow trends (inflows vs. outflows by channel)
- Wholesale funding maturity profile (near-term maturities)

**Market risk:**
- Trading VaR utilization vs. limit
- Notable P&L from market risk positions
- Investment portfolio unrealized gain/loss (AOCI impact)

**Operational risk:**
- Significant operational risk events or near-misses
- Technology incidents, cyber events, or fraud cases
- Key risk indicators (KRIs) breaching thresholds

### Step 3 — Contextualize with Market Developments

Provide market context relevant to the bank's positioning:
- Central bank actions or guidance (rate decisions, QT pace, forward guidance)
- Yield curve movements (level, slope, inversion/steepening)
- Credit market conditions (IG/HY spreads, new issuance volumes, default activity)
- Equity market performance (bank index vs. broad market)
- FX movements impacting international operations
- Commodity prices relevant to lending portfolios (oil for energy lending, agriculture for ag lending)

Link each market development to a specific bank impact (e.g., "The 15bps flattening of the 2s10s curve will compress projected Q2 NIM by approximately 2bps").

### Step 4 — Report Regulatory and Compliance Developments

Summarize relevant regulatory activity:
- Pending rule changes with estimated implementation timeline and impact
- Examination findings or supervisory feedback received
- Compliance remediation progress on outstanding matters
- Industry enforcement actions relevant to the bank's operations
- Upcoming regulatory deadlines or submissions

### Step 5 — Track Strategic Initiative Progress

For each active strategic initiative, report:
- **Status**: On track / At risk / Delayed / Completed
- **Key milestone**: What was planned vs. achieved this week
- **Budget**: Spend to date vs. plan, forecast at completion
- **Escalation items**: Issues requiring executive decision or resource reallocation
- **Next week outlook**: Expected milestones and dependencies

Limit to 5-7 highest-priority initiatives to maintain executive focus.

### Step 6 — Identify Escalation Items and Decisions Required

Extract items requiring executive attention:
- Limit breaches or near-breaches requiring approval
- Material variances from plan requiring corrective action
- Competitive threats requiring strategic response
- Risk events requiring disclosure or communication decisions
- Resource conflicts between strategic initiatives

Each escalation item should state: the issue, its impact, recommended action, and decision deadline.

### Step 7 — Compose the Executive Brief

Assemble the brief following the standardized template with these principles:
- **Lead with the headline**: One sentence capturing the week's overall theme
- **Quantify everything**: Every statement supported by a specific number
- **Highlight changes**: Emphasize what changed this week, not static positions
- **Call out actions**: Clearly separate information from items requiring decisions
- **Keep it scannable**: Use tables, bullets, and traffic-light indicators for rapid consumption
- Target length: 2-3 pages (approximately 800-1200 words)

## Output Specification

```markdown
# Weekly Executive Brief — Week Ending [Date]

## Week in One Sentence
[Single headline capturing the dominant theme of the week]

## Financial Performance Snapshot
| Metric | WTD Actual | WTD Budget | Variance | MTD Actual | MTD Budget | YTD Trend |
|--------|-----------|-----------|----------|-----------|-----------|-----------|
| NII | | | | | | |
| Fee Income | | | | | | |
| PTPP | | | | | | |
| Provision | | | | | | |
| Net Income | | | | | | |

NIM: [X]% | Efficiency Ratio: [Y]% | ROAE (ann.): [Z]%

## Balance Sheet
| | Current | WoW Δ | MoM Δ |
|----------|---------|-------|-------|
| Total Assets | | | |
| Loans | | | |
| Deposits | | | |
| LDR | | | |

## Risk Dashboard
| Risk Type | Key Metric | Current | WoW Δ | Limit | Status |
|-----------|-----------|---------|-------|-------|--------|
| Credit | NPL Ratio | | | | 🟢/🟡/🔴 |
| Liquidity | LCR | | | | 🟢/🟡/🔴 |
| Market | VaR Util. | | | | 🟢/🟡/🔴 |
| Capital | CET1 | | | | 🟢/🟡/🔴 |

## Market Context
- [Key market development 1 with bank-specific impact]
- [Key market development 2 with bank-specific impact]
- [Key market development 3 with bank-specific impact]

## Strategic Initiatives
| Initiative | Status | This Week | Next Week | Escalation |
|-----------|--------|-----------|-----------|------------|

## Regulatory & Compliance
- [Notable item with action/timeline]

## Decisions Required
1. [Decision item with context, impact, recommendation, and deadline]

## Looking Ahead
[Key events, data releases, or milestones expected next week]
```

## Analysis Framework

Apply the **So-What Test** to every data point:
- Don't just state that NIM is 3.42% — explain whether it's above or below target and why
- Don't just report LCR at 135% — note whether the trend is positive or negative and what's driving it
- Every metric should answer: "What does this mean for the bank this week, and does anyone need to do anything about it?"

The brief should enable an executive to walk into any meeting fully informed in under 5 minutes of reading.

## Examples

**Example — Week Headline:**
"NII outperformed budget by $8M (3.2%) on stronger loan repricing, but NPL formations in CRE office spiked $42M, pushing the watch list to $1.2B and triggering enhanced monitoring under the early-warning framework."

**Example — Market Context with Bank Impact:**
"The Fed held rates unchanged but signaled a higher-for-longer stance, causing 2Y Treasury yields to rise 12bps. For the bank, this is net positive: $14B of floating-rate commercial loans will reprice higher, adding an estimated $3.2M to weekly NII, while the deposit book reprices with a 3-month lag at an expected beta of 0.45."

## Guidelines

- Maintain a strict 2-3 page length; executives will not read longer briefs
- Lead every section with the most important change, not the most positive news
- Use consistent WoW (week-over-week) comparison methodology
- Apply traffic-light indicators (green/amber/red) for rapid visual scanning of risk metrics
- Separate facts from opinions; forecasts must be labeled as such
- Include "Decisions Required" section only when genuine decisions are needed — do not pad it
- Date-stamp all market data and financial figures to the specific reporting date
- Distribute by Monday 8AM to be actionable for the week ahead

## Validation Checklist

- [ ] All financial metrics tie to source systems and are internally consistent
- [ ] Week-over-week changes calculated correctly against verified prior-week figures
- [ ] Risk metrics mapped to correct limits with accurate traffic-light status
- [ ] Market developments linked to specific bank impacts (no generic commentary)
- [ ] Strategic initiatives reflect actual project management status, not aspirational
- [ ] Escalation items include specific recommended actions and deadlines
- [ ] Brief does not exceed 3 pages / 1200 words
- [ ] Every data point passes the "so-what" test
- [ ] Forward-looking statements qualified with assumptions
- [ ] Brief formatted for rapid scanning with tables, bullets, and visual indicators
