---
name: final-tax-return-filing
description: Drafts U.S. final tax return filing packets for death or corporate dissolution closings, including final-status designation, short-period reporting, liquidation/distribution items, schedule selection, attachment build, and filing mechanics. Use when handling a "final return", "deceased taxpayer", "estate final 1040", "corporate dissolution filing", "short year return", or any closing-phase tax compliance task.
tags:
  - analysis
  - corporate
  - drafting
  - regulatory
---

# Final Tax Return Filing

Builds an audit-ready final filing package (or review pack) to close taxpayer obligations at death or during entity liquidation/dissolution.

## Prerequisites

1. Confirm U.S. tax year, filing regime (calendar/fiscal), and taxpayer class (decedent estate vs dissolving entity).
2. Obtain legal identifiers and dates: legal name, SSN/EIN, last known address, date of death or formal dissolution.
3. Verify legal authority:
   - executor/personal representative/administrator authority for decedent matters
   - board or partner authorization for dissolving entities
4. Gather source returns/forms and records:
   - 1040/1120/1065 prior year, W-2, 1099, K-1, W-3, 1096
   - bank/brokerage ledgers, payroll, general ledger, balance sheet, asset transfer docs
5. Determine whether filing is a final short-period return and define reporting cutoff date.
6. Confirm whether any extension request is already filed or still possible [VERIFY].

## Output Structure / Process

### 1) Intake and status setup
Use this opening block to anchor the filing.

| Field | Value | Source |
|---|---|---|
| Taxpayer type | Decedent estate / entity liquidation | Death certificate, dissolution filing |
| Final period end date | Date of death or dissolution | Vital records, corporate minutes |
| Reporting date range | Jan 1 → end date | Financial close records |
| Primary return form | 1040 or 1120/1120S/1065 | Tax year and taxpayer type |
| Authority | Executor / trustee / liquidator name + title + proof | Court order / corporate document |
| Mailing/return instructions | Primary tax address/contact | IRS entity notice + return packet |

### 2) Identity and filing status block
1. Populate legal name and ID exactly as official source docs show.
2. Add final-status indicators:
   - Individual: final status and date markers
   - Entity: final return designation and liquidation context
3. Insert signer section with title that reflects legal authority.
4. Flag jurisdiction-specific signature and filing constraints.

### 3) Income capture (by reporting stream)
```text
For each income source, capture:
- Gross amount
- Gross-up / timing test (pre-final-date only)
- Source doc and payer
- Schedule/form line
- Reclassification needed (estate/entity)
- Review note
```

Prioritize in this order:
1. Wage/withholding income (W-2)
2. Interest/dividends (1099-INT/1099-DIV and threshold-based Schedule B)
3. Business income/loss (Schedule C / partner or shareholder passthrough items)
4. Capital activity (Sale/basis/holding period schedule flow)
5. Other unusual income (legal, settlement, cancellation, cessation adjustments)

### 4) Deductions and credits
1. Select deduction route (standard vs itemized) using final-period rules.
2. Build medical, SALT, mortgage, charitable, casualty where applicable.
3. Validate statutory caps/limits and substantiation quality.
4. Build credit matrix:
   - Eligibility
   - Income/phaseout test
   - Required supporting schedules
   - Filing-year rule consistency
5. For entity returns, confirm separation between ordinary cessation expenses and non-deductible capital/distribution-related items.

### 5) Schedule and attachment matrix
| Trigger | Required artifact | Completion rule |
|---|---|---|
| Itemized deductions | Schedule A | Supporting receipts + limits |
| Interest/dividend volume | Schedule B | Payer detail complete |
| Business operations | Schedule C or entity schedules | Cost method consistency |
| Capital gains/losses | Schedule D flow | Basis and term classification |
| Liquidation/distributions | Entity/pass-through disclosures | Recipient-level allocation reconciliation |
| Estate refund on non-spouse | Form 1310 | Include authority proof [VERIFY] |
| Non-standard treatment | Written statement packet | Include legal basis and rationale |

### 6) Filing mechanics
1. Determine due date from taxpayer type and filing status; adjust weekends/holidays.
2. Choose e-file vs paper based on attachment and IRS form limitations.
3. Reconfirm payment/refund instructions:
   - Direct payment routing
   - Refund routing (estate, representative, trust account)
4. Assemble filing packet order and attach proof list.
5. File only after final control checklist passes.

### 7) Final quality controls (mandatory)
1. Recompute all arithmetic and schedule rollups.
2. Trace each reporting line back to a source doc.
3. Reconcile withholding, estimated taxes, and credits.
4. Verify final return designation appears consistently.
5. Confirm no post-termination income is included.
6. Check signatory authority and signature block completeness.
7. Produce exception list for unresolved items and escalation requests.

## Guidelines

- Scope is U.S. returns only; defer to state-specific requirements for estimated/state schedules.
- Treat any tax date, due date matrix, or IRS destination as year-dependent; verify from latest IRS publications before filing [VERIFY].
- Do not infer authority from informal emails; require explicit legal proof before final signature.
- Use conservative classification for liquidation/distribution and basis-sensitive items; add explicit documentation where treatment is arguable.
- Escalate when significant valuation, multi-state nexus, or substantial tax-risk interpretations arise.
- Retain filing proof and support records for at least the minimum administrative period, or longer if estate or dispute exposure remains.
