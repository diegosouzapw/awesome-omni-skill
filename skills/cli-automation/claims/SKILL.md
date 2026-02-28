---
name: claims
description: Process expense claims by matching YNAB transactions with uploaded receipts. Use when the user mentions claims, expenses, reimbursements, receipts, or YNAB TODOs.
---

# Claim Processing Workflow

Process expense claims by matching YNAB transactions with uploaded receipts.

## Instructions

You are helping the user process expense claims. Follow this workflow.

**Parallelization Strategy**: Use sub-agents (Task tool) throughout to maximize speed:
- **Downloading/identifying receipts**: Spawn parallel agents to process all receipts concurrently
- **Post-claim cleanup**: Run cleanup tasks in background agents while showing next claim
- This significantly speeds up claim processing, especially with many receipts

### 1. Load Configuration

Use the Read tool to read `.env` in the project root. Extract these values:
- `YNAB_API_KEY` - API key for YNAB
- `YNAB_BUDGET_ID` - Budget ID to query
- `R2_WORKER_URL` - URL of the receipt upload worker
- `R2_PASSWORD` - Password for receipt worker auth

If `.env` is missing or incomplete, ask the user to set it up using `.env.example` as a template.

**Important**: When using these values in curl commands, substitute them directly into the command (don't rely on shell variable expansion from `source .env` as it doesn't handle comments well).

### 2. Fetch YNAB Transactions

Use curl to fetch transactions marked with "TODO" in the memo:

```bash
curl -s -H "Authorization: Bearer <YNAB_API_KEY>" \
  "https://api.ynab.com/v1/budgets/<YNAB_BUDGET_ID>/transactions" \
  | jq '[.data.transactions[] | select(.memo) | select(.memo | ascii_downcase | contains("todo"))]'
```

Note: Filter for `amount < 0` (outflows) to avoid duplicate transfer entries.

**Important: Subtransactions / Split Transactions**
Some TODO claims live inside split transactions as subtransactions. The top-level memo filter won't catch these. To find them:
1. Also search subtransaction memos: `select(.subtransactions[] | .memo | ascii_downcase | contains("todo"))`
2. Subtransaction IDs have format `{parent_id}_st_{index}_{date}` — fetching them as regular transactions returns null
3. EZ-Link transfers and credit card payments may show positive amounts (inflows on the receiving account) — use absolute value for claiming
4. When the web UI / R2 receipts show `linkedClaimId` that doesn't match any top-level TODO, check if it's a subtransaction ID

Parse the response to extract:
- `id` - Transaction ID (for updating later)
- `date` - Transaction date
- `amount` - Amount in milliunits (divide by 1000 for actual amount)
- `payee_name` - Merchant/payee
- `memo` - Contains "TODO: description"
- `category_name` - Category

**IMPORTANT: The YNAB amount is always the claim amount.** Even if the receipt shows a different amount (e.g. foreign currency before conversion, or a different total), the SGD amount from YNAB is what gets submitted to Volopay. Do not flag YNAB/receipt amount mismatches as warnings — the YNAB amount is the source of truth.

### 3. Fetch Pending Receipts

List receipts from R2:

```bash
curl -s -H "X-Auth-Token: <R2_PASSWORD>" "<R2_WORKER_URL>/list" | jq '.receipts'
```

**Response includes link metadata** (if user pre-linked via web UI):
```json
{
  "key": "2025-01-01_120000_abc12345_receipt.pdf",
  "size": 12345,
  "uploaded": "2025-01-01T12:00:00.000Z",
  "originalName": "receipt.pdf",
  "linkedClaimId": "ynab-transaction-id",      // If pre-linked
  "linkedClaimDescription": "ChatGPT"          // Claim description
}
```

**Pre-linked receipts**: When `linkedClaimId` is present, auto-match this receipt to the corresponding YNAB TODO - skip manual matching for these.

### 4. Identify All Receipts

**Before matching, download and read ALL receipts to identify their contents.** Don't rely solely on filenames - many receipts have generic names like "Receipt-1234.pdf" or "unnamed.png".

**Use sub-agents for parallel processing**: Spawn multiple Task tool agents (subagent_type="general-purpose") to download and identify receipts concurrently. Each agent handles one receipt:

```
Task 1: "Download receipt [key1] from R2, convert if HEIC, read and extract: merchant, date, amount, invoice#. Return structured summary."
Task 2: "Download receipt [key2] from R2, convert if HEIC, read and extract: merchant, date, amount, invoice#. Return structured summary."
...etc
```

Launch all agents in a single message (parallel tool calls) for maximum speed.

For each receipt, the agent should:
1. Download to /tmp/claims/:
   ```bash
   mkdir -p /tmp/claims
   curl -s -H "X-Auth-Token: <R2_PASSWORD>" "<R2_WORKER_URL>/receipt/[key]" -o /tmp/claims/[filename]
   ```

2. **For HEIC/image files**: Convert if needed:
   ```bash
   sips -Z 1500 /tmp/claims/file.heic --out /tmp/claims/file.jpg
   ```

3. **Read the receipt** using the Read tool to extract:
   - Merchant name
   - Date
   - Amount
   - Any invoice/order number

4. Return structured data for the manifest.

Collect all agent results and build the receipt manifest for matching.

### 5. Match Analysis

Compare TODOs against **identified** receipts and show a summary:

**Matching priority:**
1. **Pre-linked receipts** - If `linkedClaimId` matches a TODO's transaction ID, use that receipt (highest priority)
2. **Date proximity** - Within 3 days
3. **Amount match** - Exact or within 10%

**Present the overview:**
```
=== CLAIMS OVERVIEW ===

🔗 PRE-LINKED (X items) - user already matched via web UI:
   - [date] [description] $[amount] ← [receipt name]
   ...

✅ READY TO PROCESS (X items) - have matching receipts:
   - [date] [description] $[amount]
   ...

❌ MISSING RECEIPTS (Y items) - need to find:
   - 3x Cold Storage (~$40-60 each, Oct-Nov)
   - 2x Grab rides (~$30-40, Nov)
   - 1x GitHub ($133, Nov 4)
   ...

📎 UNMATCHED RECEIPTS (Z items) - uploaded but no matching TODO:
   - [filename] [date]
   ...
```

**Ask the user:**
1. Process ready items now?
2. Or pause to find missing receipts first?

### 6. Group and Order Claims

**Sorting strategy** (maintains claiming momentum by keeping similar items together):

1. **Group by merchant first** - All Cold Storage claims together, all Grab claims together, etc.

2. **Within each merchant, sub-group by description similarity** - Infer from the TODO description what type of expense it is:
   - e.g., "groceries", "household items", "snacks" might cluster together
   - e.g., "team lunch", "client dinner" might cluster together
   - This lets user stay in the same mental context when filling claim forms

3. **Within sub-groups, sort by date** - Chronological order within similar items

**Example ordering**:
```
Cold Storage (5 items):
  - Groceries: Oct 1, Oct 8, Oct 15
  - Household: Oct 5, Oct 12
Grab (3 items):
  - Work commute: Oct 2, Oct 9
  - Client meeting: Oct 7
GitHub (1 item):
  - Subscription: Nov 4
```

Present this grouping to user and confirm the processing order before starting.

### 7. Process Each Claim

For each TODO transaction:

1. **Show transaction details**:
   - Date: [date]
   - Payee: [payee_name]
   - Amount: [amount / 1000] (with currency)
   - Description: [memo without "TODO:" prefix]
   - Category: [category_name]

2. **Find matching receipt(s)**:
   - **Pre-linked**: If receipt has `linkedClaimId` matching this transaction, use it automatically (skip manual matching)
   - Otherwise, match by: date proximity (within 3 days), amount match (exact or close)
   - Show top matches and let user confirm

3. **Download and open the receipt**:
   ```bash
   mkdir -p /tmp/claims
   curl -s -H "X-Auth-Token: <R2_PASSWORD>" "<R2_WORKER_URL>/receipt/[key]" -o /tmp/claims/[filename]
   ```

   **For HEIC files**: Convert to JPEG for easier viewing, then delete the HEIC:
   ```bash
   sips -Z 1500 /tmp/claims/file.heic --out /tmp/claims/file.jpg
   trash /tmp/claims/file.heic
   ```

   **Rename for clarity**: Rename the local file to a descriptive format:
   `[claim#] - [merchant] [date] [amount].[ext]`

   Example: `1 - stratechery-dithering 25-oct 150.pdf`

   Then open the renamed file. Also use the Read tool to view and extract details.

   **Cleanup**: After each claim, delete the processed local file immediately to keep /tmp/claims clean. Only the current claim's receipt should be in the folder.

4. **Extract from receipt**:
   - Merchant name
   - Date
   - Total amount
   - Tax breakdown (GST/VAT if visible)
   - Any other relevant details

5. **Present formatted claim summary**:
   ```
   === CLAIM SUMMARY ===
   Date: [date]
   Merchant: [merchant]
   Description: [description from memo]
   Amount: S$[YNAB amount]
          (or for foreign currency: US$[receipt amount] (S$[YNAB amount] at exchange rate of [rate]))
   Tax: [tax amount if found, or "included" / "not shown"]
   Receipt: file:///tmp/claims/[filename]
   Folder:  file:///tmp/claims/
   ```

   **Copy merchant to clipboard**: Run `echo -n "[merchant]" | pbcopy` so user can paste it easily. Use the registered company name, not the trade name:
   - For Singapore vendors: Look for "Pte Ltd" or "LLP" (e.g., "Kap Kia Pte Ltd" not "Yeast Side")
   - For US vendors: Look for "LLC", "Inc.", "Corp" (e.g., "OpenAI, LLC" not "OpenAI")
   - Use the EXACT name as registered, including punctuation

   **Foreign currency**: If the receipt is in a foreign currency, show both amounts with the implied exchange rate: `US$X (S$Y at rate Z)`. The YNAB SGD amount is always the claim amount — never substitute the receipt amount.

6. **Wait for user confirmation**. When user says "done":

   **For speed**: Show the next claim's details FIRST, then run cleanup via background sub-agent:
   - Present the next claim summary immediately
   - Open the next receipt
   - **Spawn a background sub-agent** (Task tool with `run_in_background: true`) to handle cleanup for the completed claim

   **Background cleanup agent prompt**:
   ```
   "Complete claim cleanup for transaction [TRANSACTION_ID]:
   1. Update YNAB memo from 'TODO: X' to 'CLAIMED: X' via PUT to transactions API
   2. Delete receipt [key] from R2 via DELETE endpoint
   3. Delete local file /tmp/claims/[filename] using trash command
   Credentials: YNAB_API_KEY=[key], R2_WORKER_URL=[url], R2_PASSWORD=[pwd]"
   ```

   This runs cleanup concurrently while user reviews the next claim. No need to wait for cleanup to complete before proceeding.

   Cleanup tasks (for reference):
   - Update YNAB memo from "TODO: X" to "CLAIMED: X":
     ```bash
     curl -s -X PUT -H "Authorization: Bearer <YNAB_API_KEY>" \
       -H "Content-Type: application/json" \
       -d '{"transaction": {"memo": "CLAIMED: [description]"}}' \
       "https://api.ynab.com/v1/budgets/<YNAB_BUDGET_ID>/transactions/<TRANSACTION_ID>"
     ```
   - Delete receipt from R2:
     ```bash
     curl -s -X DELETE -H "X-Auth-Token: <R2_PASSWORD>" "<R2_WORKER_URL>/receipt/[key]"
     ```
   - Delete local receipt file (keeps /tmp/claims clean for easier uploads):
     ```bash
     trash /tmp/claims/[filename]
     ```

7. Move to the next claim.

### 8. Handle Edge Cases

- **No matching receipt**: Flag for manual review, ask user if they want to skip or mark without receipt
- **Multiple matches**: Show all options and let user pick
- **Unmatched receipts**: At the end, list any receipts that weren't matched to transactions
- **Multiple YNAB transactions per receipt** (e.g. `linkedClaimDescription: "2 claims linked"`): Sometimes the user splits one ride/expense into two YNAB transactions but has only one receipt. In this case:
  1. Present both transactions together, showing their combined total matches the receipt
  2. Ask user if they want to submit one combined Volopay claim for the receipt total
  3. If combined: submit one claim for the full amount, then mark BOTH YNAB transactions as CLAIMED
  4. Only delete the R2 receipt after all related transactions are marked CLAIMED
  5. Do NOT attempt to submit two separate Volopay claims with the same receipt amount split

### 9. Summary

When all claims are processed:

1. **Wait for background cleanup agents**: Use TaskOutput to verify all background cleanup tasks completed successfully. Report any failures.

2. **Show summary**:
   - Number of claims processed
   - Any skipped items
   - Any orphaned receipts remaining
   - Any cleanup failures that need manual attention

---

## Volopay Form Automation (Playwright)

Use the Playwright script in `scripts/volopay-submit.ts` to automate Volopay claim submission.

### Usage

```bash
cd scripts
npm run submit -- claim.json
```

Or pipe JSON directly:
```bash
echo '{"merchant":"...","amount":99.99,...}' | npm run submit
```

### Claim JSON Format

```json
{
  "merchant": "Lovable Labs Incorporated",
  "amount": 33.39,
  "date": "2025-12-20",
  "volopayCategory": "Software",
  "memo": "Lovable AI subscription",
  "xeroCategory": "Computer Software (463)",
  "xeroTaxCode": "OPINPUT:Out Of Scope Purchases",
  "xeroBizUnit": "Classes",
  "receiptPath": "/tmp/claims/receipt.pdf"
}
```

### Tax Code Logic

**CRITICAL: Only use INPUTY24 if the receipt explicitly shows a GST line item with amount. Never assume GST.**

| Condition | Tax Code |
|-----------|----------|
| Receipt shows explicit GST amount (e.g., "GST 9%: $X.XX") | INPUTY24:Standard-Rated Purchases |
| No GST breakdown + Foreign currency (USD) | OPINPUT:Out Of Scope Purchases |
| No GST breakdown + SGD | NRINPUT:Purchases from Non-GST Registered Suppliers |

**WARNING**: "Inclusive of taxes" does NOT mean GST is shown. You must see an actual GST line item to use INPUTY24.

### Volopay Category Mapping

| Expense Type | Volopay Category |
|--------------|------------------|
| Software/SaaS | Software |
| Hardware/Equipment | Equipment & hardware |
| Food/Meals | Entertainment |

### Xero Category Mapping

| Expense Type | Xero Category |
|--------------|---------------|
| Software/SaaS | Computer Software (463) |
| Software for class (IMDA VIBE, "for class") | Cost of Sales (320) |
| Hardware | Computer Hardware & Accessories (464) |
| Books | Books, Magazines, Journals (460) |
| Transport (local) | Local Public Transport (incl Taxi) (451) |
| Transport (overseas) | Overseas Transport (452) |
| Phone/Internet | Telephone & Internet (467) |

**Note**: Most software uses "Computer Software (463)". Only use "Cost of Sales (320)" when the YNAB memo explicitly mentions IMDA VIBE or "for class".

### Script Behaviour

1. Opens headed Chromium browser
2. Auto-login via Google SSO (saves session for reuse)
3. Fills all form fields automatically
4. Uploads receipt file
5. **Pauses for review** before submit - user clicks Continue manually
6. Saves auth state for future runs

---

## Quick Reference

**YNAB API**: https://api.ynab.com/v1/
**Transaction amounts**: In milliunits (divide by 1000)
**Negative amounts**: Outflows (expenses)
**Positive amounts**: Inflows

**Receipt filename format**: `YYYY-MM-DD_HHMMSS_originalname.ext`
**Volopay URL**: `${VOLOPAY_URL}/my-volopay/reimbursement/claims?createReimbursement=true` (configured in .env)
