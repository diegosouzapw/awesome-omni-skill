---
name: lcsc-supplier
description: ALL ALL LCSC RELATED OPERATIONS, including price checks, product searches, and BOM/CSV file generation. NEVER perform these actions manually - ALWAYS use this skill.
user-invocable: false
---

# Skill Instructions

When this skill is invoked, let me define these specific terms for you. When I refer to these terms I will enclose them in square brackets for clarity (e.g. [TRIGGER_REASON]):

SKILL INVOCATION CONTEXT
------------------------
TRIGGER_REASON: <describe why you called the skill>
TRIGGER_PROMPT: <If $ARGUMENTS is defined then use this, otherwise this is the exact prompt that triggered the skill, do not include any contextual information>
TRIGGER_CONTEXT: <**MUST BE 'NONE' UNLESS:** additional context that when combined with the TRIGGER_PROMPT, led to the invocation of this skill. This may include previous messages in the conversation, or any other relevant context that is not included in the TRIGGER_PROMPT but was still relevant to the decision to invoke this skill>
CONVERSATION_HISTORY: <**VERBATIM SNAPSHOT (DO NOT INTERPRET OR CHANGE)** of the full conversation history up until the point of invocation, including the TRIGGER_PROMPT. Prefix the user's conversation with 'User: ' and Claude's conversation with 'Claude: ' for clarity>

# LCSC Supplier - Forked Subagent Instructions

You are executing as a **forked subagent** with access to the full conversation history before this skill was invoked.

## MANDATORY START PROCEDURE — READ THIS FIRST

New Definition - ACTION_INTENT: <The identified intent based primarily on the [TRIGGER_PROMPT] which is the **sole authority** of what operation to perform but supported by [CONVERSATION_HISTORY]>

**IMMEDIATELY upon loading, you MUST:**

1. **Classify the task** — Use the TASK CLASSIFICATION rules below
2. **Bail if no match** — If you cannot classify the task, respond with "Unable to classify task. No matching workflow found." and do NOT continue.
3. **State the task** — Begin your response: "Task identified: [BOM creation / pricing check / search]"
4. **Execute immediately** — Jump to the correct section and execute. Do not re-analyze.

## TASK CLASSIFICATION — (Mandatory)

**Check the [ACTION_INTENT] against these workflows. Evaluate against all possible matches first and select the best match unless the [ACTION_INTENT] does not clearly fit any workflow.**
**DD NOT execute more than one workflow. If multiple match, select the most specific one.**

### Workflow 1: BOM Creation

When the [ACTION_INTENT] is about creating BOMs or CSV files for LCSC, where LCSC is either explicitly mentioned or implied from conversation context/history.

### Rule 2: Search

If the [ACTION_INTENT] relates to searching "search" or "find" WITHOUT specific LCSC codes** → Task is **SEARCH**

- Jump to "Search Workflow" (Step 3 in the execution steps below)

### Rule 3: Pricing Check (DEFAULT)

When the [ACTION_INTENT] is to check prices, stock, or details of specific parts where LCSC codes are mentioned or implied from conversation context/history.

- Jump to "Pricing Check Workflow" (Step 3 in the execution steps below)

---

## BOM Creation Workflow

**This section is self-contained. Execute ONLY these steps. Do NOT run pricing scripts.**

### BOM Step 1: Extract parts data

**If args reference a file path:**

1. Read the file using the Read tool
2. Parse the markdown table to extract:
   - **LCSC ID** column → part codes (e.g., C107107)
   - **Qty** column → quantities (e.g., 100)
3. **Skip** any rows where LCSC ID is "-" or missing (these are Mouser-only items)
4. **Skip** any rows that are section headers (e.g., rows containing only bold text)
5. **Skip** any struck-through/deleted rows (containing `~~`)

**If args contain inline part codes:**

- Parse format: "C137394:100,C1525:50"

### BOM Step 2: Generate the BOM CSV file

Build the parts string and run the create-bom-file script:

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/create-bom-file
./run.sh --parts="C107107:100,C7471456:25,..." --output=/opt/src/mcp/electronics-assistant/data/boms/lcsc-bom-YYYYMMDD.csv --useLcscCodes=true 2>/dev/null
```

**IMPORTANT:**

- Use absolute path for `--output`
- Use `--useLcscCodes=true` when input contains LCSC codes (C######)
- Default output directory: `data/boms/`
- Filename format: `lcsc-bom-{description}-{YYYYMMDD}.csv`

### BOM Step 3: Report results

Format your response as:

```markdown
## LCSC BOM File Created

- **File**: `data/boms/lcsc-bom-YYYYMMDD.csv`
- **Parts**: X line items
- **Total quantity**: Y pieces
- **Skipped**: Z items (Mouser-only / deleted / no LCSC ID)
- **Status**: Ready to upload

### Next Steps

1. Upload BOM to LCSC: https://www.lcsc.com/user/order/bom-order
```

**BOM creation is now complete. Return your results. Do NOT continue to other steps.**

---

## Pricing / Search Task Identification (Non-BOM Tasks Only)

**Only reach this section if the task is NOT BOM creation.**

Use the conversation history to find supporting details:

- Specific LCSC codes mentioned
- File paths containing part lists
- Quantity context
- Number of items to check (e.g., "first 10 items")

## Step 2: Identify LCSC Codes or Search Terms

### For Pricing Checks

Extract LCSC codes from:

- Direct codes in args: "C137394, C144507, C137364"
- File references: Read the file and extract codes from LCSC Code column
- Quantity context: Note if specific quantity tier is mentioned (default: 100+)

### For Searches

Extract search criteria:

- Keywords: Component type, value, package, specifications
- Max results: How many to return (default: 100)

## Step 3: Execute Playwright Scripts (Pricing / Search Only)

**CRITICAL**: Use the Bash tool to execute scripts directly. Do NOT invoke other skills.

### Check Pricing Script

Location: `playwright/lcsc.com/check-pricing/`

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/check-pricing
./run.sh --lcscCode="C137394" 2>/dev/null
```

Run for each LCSC code. Output is JSON to stdout.

### Search Script

Location: `playwright/lcsc.com/search/`

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/search
./run.sh --keywords="1206 resistor 10k" --maxResults=50 2>/dev/null
```

### Create BOM Script

Location: `playwright/lcsc.com/create-bom-file/`

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/create-bom-file
./run.sh --parts="C137394:100,C1525:50" --output=data/boms/output.csv --useLcscCodes=true 2>/dev/null
```

**Note**: This script does NOT use browser automation - it's fast CSV generation.

## Step 4: Parse JSON Outputs

All scripts return JSON in this format:

```json
{
  "success": true|false,
  "data": { /* script-specific data */ },
  "metadata": { "url": "...", "timestamp": "...", "duration": 1234 },
  "error": { "message": "...", "screenshot": "..." }
}
```

Extract from `data` field:

- **Pricing**: `lcscCode`, `mpn`, `manufacturer`, `description`, `stock`, `pricing[]`
- **Search**: `keywords`, `total_found`, `products[]`

## Step 5: Apply Pricing Rules (CRITICAL)

For all pricing operations, **ALWAYS** enforce these rules:

### Rule 1: Match Order Quantity Tier

Select the correct price tier based on quantity being ordered.

Example: For qty=100 with tiers [1+=$0.01, 100+=$0.005, 500+=$0.003], use $0.005.

### Rule 2: Check MOQ (Minimum Order Quantity)

The first pricing tier indicates MOQ. If requested quantity < MOQ, the part CANNOT be ordered.

Example: Tiers start at 500+ → MOQ is 500. Cannot order qty=100.

### Rule 3: Verify Stock

Check `stock` field. "0", "N/A", or empty = OUT OF STOCK = cannot order.

### Checklist Before Recommending a Part

- ✅ Price tier matches order quantity
- ✅ MOQ ≤ requested quantity
- ✅ Item is in stock (stock > 0)
- ✅ All conditions met

## Step 6: Save Results to JSON File

**MANDATORY**: Before formatting results, save all raw JSON data to a file.

Use `jq -s` to consolidate all script outputs into a single JSON array:

```bash
OUTFILE="temp/lcsc-pricing-$(date +%Y%m%d-%H%M%S).json"
jq -s '.' \
  <(./playwright/lcsc.com/check-pricing/run.sh --lcscCode="C107107" 2>/dev/null) \
  <(./playwright/lcsc.com/check-pricing/run.sh --lcscCode="C107108" 2>/dev/null) \
  > "$OUTFILE"
```

**File naming convention:**

- Pricing: `temp/lcsc-pricing-YYYYMMDD-HHMMSS.json`
- Search: `temp/lcsc-search-YYYYMMDD-HHMMSS.json`
- BOM: Already saved by script to specified path

## Step 7: Format and Return Results

Return results in markdown format optimized for the invoking agent to present to the user.

**CRITICAL**: Always include the path to the saved JSON file in your response.

### For Pricing Checks

Return a comparison table:

```markdown
## LCSC Pricing Check Results

**📄 Raw data saved to**: `temp/lcsc-pricing-20260214-235959.json`

### Price Comparison

| # | Value | LCSC Code | MPN | Manufacturer | Price (100+) | Stock | Status |
|---|-------|-----------|-----|--------------|--------------|-------|---------|
| 1 | 0Ω    | C137394   | RC1206FR-070RL | YAGEO | $0.0043 | 198,300 | ✅ Available |
| 2 | 1Ω    | C144507   | RC1206FR-071RL | YAGEO | $0.0179 | 16,300  | ✅ Available |

### Summary

- **Total items checked**: 10
- **Available**: 9 items
- **Issues**: 1 item (low stock)
- **Total cost**: $1.23 for 1000 pcs

### Key Findings

- All items in stock with adequate quantities
- MOQ requirements met for all parts
- No significant price changes from file data
```

**Status indicators:**

- ✅ Available (in stock, MOQ met, good price)
- ⚠️ Warning (low stock <500, high MOQ, price increase >20%)
- ❌ Unavailable (out of stock, MOQ not met)

### For Searches

```markdown
## LCSC Search Results

**📄 Raw data saved to**: `temp/lcsc-search-20260214-235959.json`

**Query**: "1206 resistor 10k"
**Total found**: 42 products
**Showing**: Top 20 results

| # | LCSC Code | MPN | Manufacturer | Description | Price | Stock |
|---|-----------|-----|--------------|-------------|-------|-------|
| 1 | C137394 | RC1206FR-0710KL | YAGEO | ±1% 10kΩ 1206 | $0.0028 | 15,000 |

### Recommendations

- **Best price**: C137394 at $0.0028 (100+)
- **Best stock**: C137394 with 15,000 units
```

### For BOM Creation

**See "BOM Creation Workflow" section above — BOM tasks are fully self-contained there.**

## Important Notes

- **Work directory**: `/opt/src/mcp/electronics-assistant/`
- **Scripts**: Always use `2>/dev/null` to suppress stderr (progress logs)
- **Deprecated**: Never use `scripts/lcsc_tool.py` (old, slow, deprecated)
- **Storage**: Save BOMs to `data/boms/` (gitignored directory)
- **JSON Output**: **MANDATORY** - Always save raw JSON results to `temp/lcsc-{operation}-{timestamp}.json`
- **Efficiency**: Run pricing checks in parallel when checking multiple parts
- **Errors**: If a script fails, check `error.screenshot` path and report issue
- **Result Format**: Always include JSON file path in markdown response

## Quick Reference

**Check single price and save to file:**

```bash
cd /opt/src/mcp/electronics-assistant
OUTFILE="temp/lcsc-pricing-$(date +%Y%m%d-%H%M%S).json"
./playwright/lcsc.com/check-pricing/run.sh --lcscCode="C137394" 2>/dev/null > "$OUTFILE"
```

**Batch pricing (efficient):**
Run multiple check-pricing scripts in parallel, capture results, then save to consolidated JSON:

```bash
cd /opt/src/mcp/electronics-assistant
OUTFILE="temp/lcsc-pricing-$(date +%Y%m%d-%H%M%S).json"
jq -s '.' \
  <(./playwright/lcsc.com/check-pricing/run.sh --lcscCode="C137394" 2>/dev/null) \
  <(./playwright/lcsc.com/check-pricing/run.sh --lcscCode="C144507" 2>/dev/null) \
  > "$OUTFILE"
```

**Search components:**

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/search && ./run.sh --keywords="search term" 2>/dev/null
```

**Create BOM:**

```bash
cd /opt/src/mcp/electronics-assistant/playwright/lcsc.com/create-bom-file && ./run.sh --parts="C137394:100" --output=data/boms/file.csv --useLcscCodes=true 2>/dev/null
```

## Script Documentation

For detailed reference material on each LCSC operation, see:

- `playwright/lcsc.com/check-pricing/README.md` - Pricing check script documentation
- `playwright/lcsc.com/search/README.md` - Search script documentation
- `playwright/lcsc.com/create-bom-file/README.md` - BOM creation script documentation
- `docs/Pricing-Guidelines.md` - MOQ and pricing rules
