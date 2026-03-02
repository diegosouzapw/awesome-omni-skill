---
name: fecfile
description: Analyze FEC (Federal Election Commission) campaign finance filings. Use when working with FEC filing IDs, campaign finance data, contributions, disbursements, or political committee financial reports. Provides the proper workflow for the fec-api MCP tools (search_committees, get_filings).
compatibility: Requires uv and access to the internet
license: MIT
metadata:
  author: Matt Hodges
  version: "2.0.2"
---

# FEC Filing Analysis

This skill enables analysis of Federal Election Commission campaign finance filings.

## Requirements

- [uv](https://docs.astral.sh/uv/) must be installed
- Python 3.9+

Dependencies are automatically installed when running scripts with `uv run`.

## First-Time Check

The first time this skill is invoked in a session, verify that `uv` is installed by running:

```bash
uv --version
```

If this command fails or `uv` is not found, do not proceed. Instead, inform the user that `uv` is required but not installed, and direct them to the installation guide: https://docs.astral.sh/uv/getting-started/installation/

## Quick Start

**Always start by checking the filing size:**
```bash
uv run scripts/fetch_filing.py <FILING_ID> --summary-only
```

Based on the summary, decide how to proceed—see **Handling Large Filings** below for filtering and streaming strategies. Small filings can be fetched directly; large filings require pre-filtering or streaming.

**Fetching data:**
```bash
uv run scripts/fetch_filing.py <FILING_ID>                   # Full filing (small filings only)
uv run scripts/fetch_filing.py <FILING_ID> --schedule A      # Only contributions
uv run scripts/fetch_filing.py <FILING_ID> --schedule B      # Only disbursements
uv run scripts/fetch_filing.py <FILING_ID> --schedules A,B   # Multiple schedules
```

The `fecfile` library is installed automatically by uv.

## Field Name Policy

**IMPORTANT**: Do not guess at field names. Before referencing any field names in responses:

1. For form-level fields (summary data, cash flow, totals): Read `references/FORMS.md`
2. For itemization fields (contributors, payees, expenditures): Read `references/SCHEDULES.md`

These files contain the authoritative field mappings. If a field name isn't documented there, verify it exists in the actual JSON output before using it.

## Handling Large Filings

FEC filings vary enormously in size. Small filings (like state party monthly reports) may have only a few dozen itemizations and can be used directly. However, major committees like ActBlue, WinRed, and presidential campaigns can have hundreds of thousands of itemizations in a single filing. **Do not dump large filing data directly into the context window. Avoid streaming large filings to stdout.**

### Checking Size

Before pulling full schedules, use `--summary-only` to assess the filing:

```bash
uv run scripts/fetch_filing.py <ID> --summary-only
```

The summary includes financial totals that help gauge filing size without parsing itemizations:

| Field | Description |
|-------|-------------|
| `col_a_individuals_itemized` | Itemized individual contributions (this period) |
| `col_a_total_contributions` | Total contributions (this period) |
| `col_a_total_disbursements` | Total disbursements (this period) |
| `col_b_individuals_itemized` | Itemized individual contributions (year-to-date) |
| `col_b_total_contributions` | Total contributions (year-to-date) |
| `col_b_total_disbursements` | Total disbursements (year-to-date) |

These are dollar totals, not item counts, but combined with the committee name they help you decide:
- **Small state/local party with modest totals**: Probably safe to pull full schedules
- **ActBlue, WinRed, or presidential campaign with millions in totals**: Use streaming or post-filter

If you need to verify exact counts before processing, stream with an early cutoff:

```bash
uv run scripts/fetch_filing.py <ID> --stream --schedule A | python3 -c "
import sys
count = 0
limit = 256
for line in sys.stdin:
    count += 1
    if count >= limit:
        print(f'Schedule A: {limit}+ items (stopped counting)')
        sys.exit(0)
print(f'Schedule A: {count} items')
"
```

If itemization counts are in the hundreds or more, you must post-filter before presenting results. Even smaller filings may benefit from post-filtering to aggregate or focus the output.

### Pre-Filtering at Parse Time

Use CLI flags to filter before data is loaded into memory:

| Flag | Effect |
|------|--------|
| `--summary-only` | Only filing summary (no itemizations) |
| `--schedule A` | Only Schedule A (contributions) |
| `--schedule B` | Only Schedule B (disbursements) |
| `--schedule C` | Only Schedule C (loans) |
| `--schedule D` | Only Schedule D (debts) |
| `--schedule E` | Only Schedule E (independent expenditures) |
| `--schedules A,B` | Multiple schedules (comma-separated) |

Schedules you don't request are never parsed.

### Post-Filtering with Pandas

Use Python/pandas to aggregate, filter, and limit results:

```bash
cat > /tmp/analysis.py << 'EOF'
# /// script
# requires-python = ">=3.9"
# dependencies = ["pandas>=2.3.0"]
# ///
import json, sys
import pandas as pd

data = json.load(sys.stdin)
df = pd.DataFrame(data.get('itemizations', {}).get('Schedule A', []))
# Aggregate and limit output
print(df.groupby('contributor_state')['contribution_amount'].agg(['count', 'sum']).sort_values('sum', ascending=False).to_string())
EOF

uv run scripts/fetch_filing.py <ID> --schedule A 2>&1 | uv run /tmp/analysis.py
```

### Streaming Mode (Producer/Consumer Model)

For truly massive filings where even a single schedule is too large to hold in memory, use `--stream` to output JSONL (one JSON object per line):

```bash
uv run scripts/fetch_filing.py <ID> --stream --schedule A
```

Each line has the format: `{"data_type": "...", "data": {...}}`

**How streaming works:**

The producer (fetch_filing.py) outputs one record at a time without loading the full filing. A consumer script reads one line at a time and aggregates incrementally. Neither side ever holds all records in memory.

Example streaming aggregation:

```bash
uv run scripts/fetch_filing.py <ID> --stream --schedule A | python3 -c "
import json, sys
from collections import defaultdict
totals = defaultdict(float)
counts = defaultdict(int)
for line in sys.stdin:
    rec = json.loads(line)
    if rec['data_type'] == 'itemization':
        state = rec['data'].get('contributor_state', 'Unknown')
        amt = float(rec['data'].get('contribution_amount', 0))
        totals[state] += amt
        counts[state] += 1
for state in sorted(totals, key=lambda s: -totals[s]):
    print(f'{state}: {counts[state]} contributions, \${totals[state]:,.2f}')
"
```

This processes hundreds of thousands of records using constant memory.

### Guidelines

1. **Small filings** - Can be used directly without filtering
2. **Large filings** - Pre-filter with `--summary-only` or `--schedule X`, then check size
3. **Massive results** - Post-filter with pandas to aggregate, filter, and limit output
4. **Streaming mode** - Use `--stream` with inline Python consumers for constant-memory processing
5. **Limit output** - Use `.head()`, `.nlargest()`, `.nsmallest()` to cap results

## Finding Filings by Candidate/Committee Name

When the user asks about a candidate or committee's filings without providing a filing ID, use the MCP tools to discover the filing ID.

### MCP Tools

The `fec-api` MCP server provides two tools:

- **`search_committees`**: Search for committees by name → returns committee IDs
- **`get_filings`**: Get filings for a committee ID → returns filing IDs and metadata

The MCP server loads the FEC API key from the system keyring on first tool use, keeping it secure and hidden from the conversation. The API key is never visible to the model.

### API Key Security

**IMPORTANT**: Never output or log the FEC API key. The key is loaded on first tool use, cached in memory, and never exposed to the model.

The key can be accidentally exposed in:
- Error messages from HTTP clients (which may include the full URL)
- Debug output or logging
- Custom scripts that print request parameters

The MCP server sanitizes error output to prevent key exposure.

### Workflow Example

**"What are the top expenditures in Utah Republican Party's most recent filing?"**

**Step 1: Find the committee**

Use `search_committees` tool with query "Utah Republican Party":

```json
[
  {
    "id": "C00089482",
    "is_active": true,
    "name": "UTAH REPUBLICAN PARTY"
  },
  {
    "id": "C00174144",
    "is_active": false,
    "name": "UTAH COUNTY REPUBLICAN PARTY/FEC ACCT"
  }
]
```

Choose the appropriate `id` based on the user's query. Users may not know the exact name of the committee they're searching for. You may need to run multiple searches with alternate committee name queries to find the user's desired committee.

**Step 2: Get recent filings**

Use `get_filings` tool with committee_id "C00089482":

```json
[
  {
    "filing_id": 1896830,
    "form_type": "F3X",
    "receipt_date": "2025-06-20T00:00:00",
    "coverage_start_date": "2025-05-01",
    "coverage_end_date": "2025-05-31",
    "total_receipts": 42655.8,
    "total_disbursements": 21283.49,
    "amendment_indicator": "N"
  },
  {
    "filing_id": null,
    "form_type": "FRQ",
    "receipt_date": "2025-05-21T00:00:00",
    "coverage_start_date": "2025-03-01",
    "coverage_end_date": "2025-03-31",
    "total_receipts": null,
    "total_disbursements": null,
    "amendment_indicator": null
  },
  {
    "filing_id": 1893645,
    "form_type": "F3X",
    "receipt_date": "2025-05-20T00:00:00",
    "coverage_start_date": "2025-04-01",
    "coverage_end_date": "2025-04-30",
    "total_receipts": 25100.23,
    "total_disbursements": 15024.56,
    "amendment_indicator": "N"
  },
  {
    "filing_id": 1889675,
    "form_type": "F3X",
    "receipt_date": "2025-04-20T00:00:00",
    "coverage_start_date": "2025-03-01",
    "coverage_end_date": "2025-03-31",
    "total_receipts": 33363.33,
    "total_disbursements": 37921.03,
    "amendment_indicator": "N"
  }
]
```

Choose the appropriate `filing_id` based on the user's query. You may need to broaden the limit flag depending on the initial results, or select more than one `filing_id` depending on the user's query.

**Step 3: Check filing size**
```bash
uv run scripts/fetch_filing.py 1896830 --summary-only
```

**Step 4: Post-filter to get top 10 expenditures**
```bash
cat > /tmp/top_expenditures.py << 'EOF'
# /// script
# requires-python = ">=3.9"
# dependencies = ["pandas>=2.3.0"]
# ///
import json, sys
import pandas as pd

data = json.load(sys.stdin)
df = pd.DataFrame(data.get('itemizations', {}).get('Schedule B', []))
org  = df["payee_organization_name"].astype("string").str.strip().replace("", pd.NA)
last = df["payee_last_name"].astype("string").str.strip().replace("", pd.NA)
first= df["payee_first_name"].astype("string").str.strip().replace("", pd.NA)

# "Last, First" when both exist; otherwise fall back to whichever exists
person = (last + ", " + first).where(last.notna() & first.notna())
person = person.combine_first(last).combine_first(first)

payee_name = org.combine_first(person)

top10 = (
    df.assign(payee_name=payee_name)
      .nlargest(10, "expenditure_amount")[
          ["payee_name", "expenditure_amount", "expenditure_purpose_descrip", "expenditure_date"]
      ]
)
print(top10.to_string())
EOF

uv run scripts/fetch_filing.py 1896830 --schedule B 2>&1 | uv run /tmp/top_expenditures.py
```

### MCP Tool Reference

**search_committees**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Committee name or partial name to search |
| `limit` | integer | No | Maximum results (default: 20) |

**get_filings**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `committee_id` | string | Yes | FEC committee ID (e.g., C00089482) |
| `limit` | integer | No | Maximum results (default: 10) |
| `form_type` | string | No | Filter by form: F3, F3P, F3X |
| `cycle` | integer | No | Filter by two-year election cycle (e.g., 2024) |
| `report_type` | string | No | Filter by report period: Q1, Q2, Q3, YE, MY, 12G, 30G |
| `sort` | string | No | Sort field with '-' prefix for descending (default: -receipt_date) |
| `include_amended` | boolean | No | Include superseded amendments (default: false) |

**Sorting options:**

| Category | Fields |
|----------|--------|
| Date/time | `receipt_date`, `coverage_start_date`, `coverage_end_date` |
| Financial | `total_receipts`, `total_disbursements` |
| Other | `report_year`, `cycle` |

**When to use different sort options:**

| Sort | Use when... |
|------|-------------|
| `-receipt_date` | You want the most recently filed documents (default) |
| `-coverage_end_date` | You want filings by reporting period (e.g., "most recent quarter") |
| `-total_receipts` | You want filings with the highest fundraising totals first |

Note: `-receipt_date` can have ties when multiple filings arrive the same day. `-coverage_end_date` is useful for finding the latest reporting period but doesn't account for amendments filed later.

## Finding Filing IDs (Manual)

If the FEC API is not set up, filing IDs can be found via:
1. **FEC Website**: Visit [fec.gov](https://www.fec.gov) and search for a committee
2. **Direct URLs**: Filing IDs appear in URLs like `https://docquery.fec.gov/dcdev/posted/1690664.fec`

## Response Style

When analyzing FEC filings:
- Start with your best judgment about whether this filing has unusual aspects (no activity is not unusual)
- Write in a simple, direct style
- Group related information together in coherent sections

## Form Types

See [FORMS.md](references/FORMS.md) for detailed guidance on:
- **F1/F1A**: Committee registration/organization
- **F2/F2A**: Candidate declarations
- **F3/F3P/F3X**: Financial reports
- **F99**: Miscellaneous text filings

## Schedules & Field Mappings

See [SCHEDULES.md](references/SCHEDULES.md) for detailed field mappings for:
- **Schedule A**: Individual contributions
- **Schedule B**: Disbursements/expenditures
- **Schedule C**: Loans
- **Schedule D**: Debts
- **Schedule E**: Independent expenditures

## Amendment Detection

Check the `amendment_indicator` field:
- `A` = Standard Amendment
- `T` = Termination Amendment
- Empty/None = Original Filing

If it's an amendment, look for `previous_report_amendment_indicator` for the original filing ID.

## Coverage Periods

Use `coverage_from_date` and `coverage_through_date` fields.
- Format: Usually YYYY-MM-DD
- Calculate days covered: (end_date - start_date) + 1
- Context: Quarterly reports ~90 days, Monthly ~30 days, Pre-election varies

## Financial Summary Fields

For financial filings (F3, F3P, F3X):
- **Receipts**: `col_a_total_receipts`
- **Disbursements**: `col_a_total_disbursements`
- **Cash on Hand**: `col_a_cash_on_hand_close_of_period`
- **Debts**: `col_a_debts_to` and `col_a_debts_by`

## Data Quality Notes

- Contributions/expenditures $200+ must be itemized with details
- Smaller amounts may appear in summary totals but not itemized
- FEC Committee ID format is usually C########

## Example Queries

Once you have filing data, you can answer questions like:
- "What are the total receipts and disbursements?"
- "Who are the top 10 contributors?"
- "What are the largest expenditures?"
- "What contributions came from California?"
- "How much was spent on advertising?"
