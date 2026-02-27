---
name: heath-ledger
description: "AI bookkeeping agent for Mercury bank accounts. Pulls transactions, categorizes them (rule-based + AI), and generates Excel workbooks with P&L, Balance Sheet, Cash Flow, and transaction detail. Use when the user wants to do bookkeeping, generate financial statements, categorize bank transactions, connect Mercury, or produce monthly/quarterly/annual books. Triggers on: bookkeeping, P&L, profit and loss, balance sheet, cash flow, financial statements, Mercury bank, categorize transactions, generate books, monthly close."
---

# Heath Ledger

AI bookkeeping skill for Mercury bank accounts.

## Quick Start

1. `scripts/init_db.mjs` — creates DB + seeds ~90 universal vendor→category rules
2. `scripts/connect_mercury.sh <MERCURY_API_TOKEN> [entity_name]` — discovers accounts
3. *(Optional)* `scripts/connect_stripe.sh <entity_id> <stripe_api_key>` — connect Stripe for exact revenue + fees
4. *(If Stripe connected)* `scripts/pull_stripe_revenue.sh <entity_id> <start_date> <end_date>` — pull monthly revenue data
5. `scripts/pull_transactions.sh <entity_id> <start_date> <end_date>`
5. `scripts/categorize.sh <entity_id>` — rule-based first, AI for unknowns
6. Review ambiguous items, correct with `scripts/set_category.sh`
7. `scripts/generate_books.sh <entity_id> <start_date> <end_date> [output_path]`

## Setup Flow

### Mercury API Key (Required)
Get from Mercury Dashboard → Settings → API Tokens. The token gives read-only access to transactions.

### Stripe API Key (Optional but Recommended)
Without Stripe API: Mercury shows **net** Stripe deposits (revenue minus fees). The system estimates gross revenue using a configurable fee rate (default 2.3% + $0.30).

With Stripe API: You get **exact** gross revenue, exact fees, and proper refund tracking. Always prefer this when available.

To connect: `scripts/connect_stripe.sh <entity_id> <stripe_api_key>`
Then pull data: `scripts/pull_stripe_revenue.sh <entity_id> <start_date> <end_date>`

The P&L generator automatically uses Stripe data when available, falling back to Mercury estimates otherwise.

### Entity Settings
Configure per-entity via the `entity_settings` table:

| Setting | Default | Description |
|---------|---------|-------------|
| `accounting_basis` | `accrual` | `accrual` or `cash` — cash basis uses posted dates only |
| `month_offset` | `1` | Fiscal year month offset (1 = calendar year) |
| `stripe_fee_rate` | `0.023` | Stripe percentage fee for gross-up calculation |
| `stripe_fee_fixed` | `0.30` | Stripe fixed fee per transaction |
| `amortization_monthly` | `null` | Monthly amortization amount for acquired assets |

## Workflow

1. **Connect Mercury** — `scripts/connect_mercury.sh <token> [name]` discovers accounts, creates entity
2. **Pull transactions** — `scripts/pull_transactions.sh <entity_id> <start_date> <end_date>`
3. **Categorize** — `scripts/categorize.sh <entity_id> [max_transactions]` — rule-based first, then AI for unknowns
4. **Review ambiguous** — Script outputs low-confidence items. Ask user, then update with `scripts/set_category.sh <transaction_id> <category> [subcategory]`
5. **Generate books** — `scripts/generate_books.sh <entity_id> <start_date> <end_date> [output_path]`

## Scripts Reference

All scripts are in `scripts/`. Run with bash or node. Database is SQLite at `data/heath.db`.

| Script | Purpose |
|--------|---------|
| `init_db.mjs` | Create/migrate SQLite database + seed rules |
| `connect_mercury.sh` | Connect Mercury API, discover accounts |
| `pull_transactions.sh` | Pull transactions for date range |
| `categorize.sh` | Categorize transactions (rules + AI) |
| `set_category.sh` | Manually set category for a transaction |
| `add_rule.sh` | Add/update a categorization rule |
| `generate_books.sh` | Generate Excel workbook |
| `list_entities.sh` | List all entities |
| `connect_stripe.sh` | Connect Stripe API to an entity |
| `pull_stripe_revenue.sh` | Pull Stripe balance transactions by month |
| `status.sh` | Show entity status (accounts, tx counts) |

## Chart of Accounts

See `references/chart-of-accounts.md` for the full chart with P&L sections and cash flow classifications.

## Learning & Compounding System

Heath Ledger gets smarter over time through a layered rule system:

### Rule Hierarchy
1. **Entity-specific rules** (highest priority) — per-company overrides
2. **Global rules** (`entity_id = NULL`) — apply to all entities
3. **Seed rules** — universal vendor mappings shipped with the skill
4. **AI categorization** — used when no rule matches

### How Learning Works
- Every manual correction creates or updates a categorization rule
- Rules track `usage_count` — heavily-used rules are more reliable
- `source` field tracks provenance: `seed`, `ai`, `human`, `manual`
- Human-confirmed rules get `confidence: 0.95-1.0`
- AI-generated rules start at `0.85` and can be promoted
- Entity-specific rules can be promoted to global when they prove universal

### The Compounding Effect
After categorizing ~5,000 transactions across 2 entities, the system now auto-categorizes **~95%** of transactions without AI. Each new entity benefits from all previous learnings.

## Known Limitations

### Stripe Net vs Gross (Without Stripe API)
Mercury deposits from Stripe are **net** amounts (revenue minus ~2.9% + $0.30 fees). Without the Stripe API:
- We estimate gross revenue using configurable fee rates
- This creates "synthetic" Stripe Fee entries
- Accuracy depends on your actual Stripe fee rate (varies by plan, card type, international)
- **Solution**: Connect Stripe API for exact numbers

### Deel Fee Splitting
Deel combines platform fees and contractor payroll in one transaction stream. Pattern:
- **Small fixed amounts** (~$2-5) → Deel Platform Fee → categorize as "Software expenses"
- **Larger variable amounts** → Contractor Payroll → categorize as "Wages & Salaries"
- The system learns this pattern but may need initial human guidance

### Mercury API Limitations
- Only returns posted transactions (not pending)
- Some counterparty names are truncated or normalized differently
- Wire descriptions may include reference numbers that create duplicate rules

### Multi-Currency
- Wise transfers create both a debit (USD) and may show FX fees separately
- International wire fees from Mercury appear as separate line items
- FX gains/losses are not tracked (would need multi-currency ledger)

## AI Categorization

The `categorize.sh` script calls the host agent's model via stdin/stdout JSON protocol. It sends transaction batches and expects category assignments back. The script writes a prompt to stdout that the agent should process and return results for.

When AI confidence < 0.85, transactions are flagged as ambiguous for user review.

## Key Details

- **Cash or accrual basis** — configurable per entity
- **Multiple entities supported** — each with own connections and rules
- **Rules persist** — categorization rules saved to SQLite, reused across runs
- **Seed rules** — ~90 universal vendor mappings loaded on init
- **Excel output** — 4-tab workbook: P&L, Balance Sheet, Cash Flow, Transaction Detail
