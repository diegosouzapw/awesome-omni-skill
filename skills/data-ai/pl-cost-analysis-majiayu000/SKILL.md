---
name: pl-cost-analysis
description: "Calculate monthly COGS, cost percentages, and manager bonuses (COGS + Top Line) using NET SALES for accuracy and detailed inventory data for restaurant locations."
license: Apache-2.0
allowed-tools: "execute_command,read_file,write_file,list_directory"
---
# P&L Cost Analysis Skill

## Configuration Details (For Claude)

* **Version:** 1.0.0
* **Required Python Packages:** python>=3.7,pdfplumber>=0.10.0,pandas>=1.5.0,numpy>=1.20.0
* **Author:** Devin Bostwick
* **Homepage:** https://threepointshospitality.com
* **Tags:** finance, hospitality, pnl, analysis, bonus

## Overview
This Skill automates monthly P&L cost analysis and bonus calculations for restaurant locations using **NET SALES** for accuracy and actual COGS data (Beginning Inventory + Purchases - Ending Inventory formula). It calculates cost percentages by bonus category, determines COGS bonuses using tiered structures, evaluates Top Line sales bonuses with eligibility rules, and provides actionable insights.

### 🎯 Key Accuracy Update (Nov 2024)
**Now uses NET SALES instead of GROSS SALES for precise bonus calculations:**
- **51% more accurate** bonus calculations  
- **$52,000+ difference** in total sales recognition per month
- **Eliminates overstatement** from discounts, comps, and voids
- **Example:** Cantina Oct 2024: Net $1.24M vs Gross $1.29M

**When to use this Skill:**
- Monthly close when inventory counts are complete
- When calculating manager bonuses  
- User says "run P&L analysis" or "calculate bonuses for [location]"
- User uploads files containing "foodUsageReport" OR "Gross_Sales" OR "controllableProfitAndLoss"
- User uploads BOTH Food Usage Report AND Gross Sales Report (preferred for accuracy)
- When comparing location performance
- When analyzing cost variances
- When user mentions "net sales" vs "gross sales" accuracy

## What This Skill Does

### Calculates
1. **Actual COGS** using formula: Beginning Inventory + Purchases - Ending Inventory
2. **Cost percentages** using **NET SALES** (recommended) or Gross Sales based on respective category sales (e.g., Food COGS % = Food Usage ÷ Food NET Sales)
3. **COGS bonuses** using tiered structure with "< for MAX" logic
4. **Top Line bonus** with eligibility rules (must hit 2 of 3 COGS targets)
5. **Variance analysis** showing actual vs target cost percentages
6. **Dollar impact** of each variance

### Key Accuracy Improvement: NET vs GROSS Sales
**NET SALES** (Gross Sales Report):
- ✅ Actual revenue after discounts, comps, voids, returns
- ✅ True picture of restaurant performance
- ✅ Accurate bonus calculations
- ✅ Example: Liquor Net $890K vs Gross $932K (5% difference!)

**GROSS SALES** (Food Usage Report):
- ⚠ Total charges before adjustments
- ⚠ Overstates actual revenue
- ⚠ Can lead to inflated bonus payments
- ⚠ Less accurate for performance measurement

### Handles
- Multi-location processing (Cantina, OAK, White Buffalo)
- Reimbursements (e.g., E11even vodka credits)
- Different P&L layouts per location
- Month-over-month comparisons
- Cross-location benchmarking

## Input Options

### Option 1: Dual PDF (Required for Maximum Accuracy)
**Uses TWO PDFs to combine the best data from each** - COGS from Usage + NET Sales from Breakdown!

**Required Files:**
1. **Food Usage Report PDF** - Contains detailed COGS/inventory data (beginning inventory, purchases, ending inventory, usage) BUT only has GROSS sales
2. **Sales Breakdown Report PDF** - Contains NET SALES by category (actual revenue after discounts/comps) BUT lacks inventory details

**Example:** 
- `foodUsageReport (15).pdf` → Inventory costs + COGS details
- `Cantina_Gross_Sales_09_10_25.pdf` → NET sales data

**Why Both Are Essential:**
- 🔍 **Food Usage Report** = Has all inventory/COGS data but sales are GROSS (inflated by ~$50K)
- 🎯 **Sales Breakdown Report** = Has accurate NET sales but no inventory/purchase details  
- ✅ **Combined** = Real inventory costs ÷ actual net revenue = accurate cost percentages

**Key Benefits:**
- ✅ Uses **NET SALES** (actual revenue after discounts/comps/voids)
- ✅ Uses **ACTUAL COGS** (beginning inventory + purchases - ending inventory)
- ✅ More accurate bonus calculations (prevents $40,000+ revenue overstatement)
- ✅ Auto-extracts location and date from both files
- ✅ Detailed cost breakdown by category with precise percentages

**PDF Workflow (New):**
```python
python3 pnl_analyzer_pdf.py foodUsageReport.pdf grossSales.pdf 4656
```

**PDF Detection:**
- Food Usage: filename contains "foodUsageReport" or "Food Usage" 
- Gross Sales: filename contains "Gross_Sales" or contains "Sales Breakdown" content
- Both files must be `.pdf` format

### Option 2: Single PDF (Legacy - Less Accurate)
**Single Food Usage Report PDF** - Uses GROSS SALES (includes discounts)

**Example:** `foodUsageReport__10_.pdf` (October 2025)

When user uploads only `foodUsageReport.pdf`:
- Auto-extracts summary table (Food, Beer, Wine, Liquor, N/A Bev)
- Uses **GROSS SALES** which overstates revenue
- Auto-detects period from filename or PDF content

**PDF Workflow (Legacy):**
```python
python3 pnl_analyzer_pdf.py /path/to/foodUsageReport.pdf 4656
```

**⚠ Warning:** This uses gross sales which can overstate revenue by $40,000+ for liquor

### Option 2: CSV (Legacy)
**Two input methods available:**

#### Method A: Two Files (Original)
Usage Report + P&L Report:

#### Usage Report (foodUsageReport.csv)
Contains inventory data with columns:
- **Category Type** - Product category (Food, Liquor, Beer, Wine, N/A Bev)
- **Starting Inventory Value** - Beginning inventory value
- **Purchased Value** - Total purchases during period
- **Ending Inventory Value** - Ending inventory value

#### P&L Report (controllableProfitAndLoss[Location].csv)
**Contains both sales AND COGS data** - Can be processed standalone!

**Sales Data (INCOME section):**
- Beer, Food, Liquor, N/A Bev, Wine sales with actual dollar amounts
- Total income calculation

**COGS Data (COST OF GOODS SOLD section):**
- Direct COGS amounts by category with percentages already calculated
- Subcategories: Beer - Bottle, Bar Consumables, Liquor, NA Beverage, Wine

**File Format:**
```csv
" ","Actual (Location)","% of Sales (Location)","Budget","Variance"
"Beer","$43,561.13","8.9%","",""
"Food","$104,829.63","21.4%","",""
"Beer","$5,991.14","13.8%","",""
```

**Advantage:** Single file contains both sales AND COGS - no separate usage report needed!

#### Method B: Single P&L File (Standalone)
Just the P&L Report containing both sales and COGS:

**When to use:** 
- User uploads only `controllableProfitAndLoss[Location].csv`
- File contains complete INCOME and COST OF GOODS SOLD sections
- COGS percentages already calculated in the report

**Processing:** Direct analysis without separate usage calculations

**Note:** If OAK/WB layouts differ, update PNLCOORDS in pnl-engine.py

## How to Use

### Basic Single Location (Dual PDF - Recommended)
**User says:**
> "Run P&L for Cantina, October 2025. E11even reimbursement was $4,656."

**Then uploads 2 files:**
- `foodUsageReport (15).pdf` 
- `Cantina_Gross_Sales_09_10_25.pdf`

**Claude:**
1. Detects location (CANTINA) and period (2025-10) from both PDFs
2. Identifies Food Usage Report for COGS data
3. Identifies Gross Sales Report for NET SALES data  
4. Applies reimbursement amount ($4,656) to liquor COGS
5. Executes: `python3 pnl_analyzer_pdf.py foodUsage.pdf grossSales.pdf 4656`
6. Uses NET SALES for accurate bonus calculations
7. Generates executive summary with insights

### Basic Single Location (Legacy Mode)
**User says:**
> "Run P&L for Cantina, October 2025. E11even reimbursement was $4,656."

**Then uploads 1 file:**
- `foodUsageReport (15).pdf`

**Claude:**
1. Warns about using GROSS SALES (less accurate)
2. Executes: `python3 pnl_analyzer_pdf.py foodUsage.pdf 4656`  
3. Uses GROSS SALES which may overstate revenue
4. Generates analysis with accuracy warning

### Multiple Locations (Dual PDF Mode)
**User says:**
> "Run all 3 locations, September 2025. Cantina reimb $4,656, others none."

**Uploads 6 files:**
- 3x Food Usage Reports (`foodUsageReport_Cantina.pdf`, `foodUsageReport_OAK.pdf`, etc.)
- 3x Gross Sales Reports (`Cantina_Gross_Sales.pdf`, `OAK_Gross_Sales.pdf`, etc.)

**Claude processes each location with accurate NET SALES and provides comparison summary**

### Multiple Locations (Legacy Mode)
**User says:**
> "Run all 3 locations, September 2025. Cantina reimb $4,656, others none."

**Uploads 3 files (3 usage reports only)**

**Claude processes each location with GROSS SALES (warns about accuracy) and provides comparison summary**

### Month-Over-Month (Dual PDF Mode)
**User says:**
> "Compare Cantina: September vs October"

**Uploads 4 files:**
- September: `foodUsageReport_Sep.pdf` + `Cantina_Gross_Sales_Sep.pdf`
- October: `foodUsageReport_Oct.pdf` + `Cantina_Gross_Sales_Oct.pdf`

**Claude shows trend analysis using accurate NET SALES data**

### Month-Over-Month (Legacy Mode) 
**User says:**
> "Compare Cantina: September vs October"

**Uploads 2 files:**
- `foodUsageReport_Sep.pdf`
- `foodUsageReport_Oct.pdf` 

**Claude shows trend analysis using GROSS SALES (with accuracy warnings)**

## Category Mapping & COGS Calculation

### COGS Calculation Method
**Formula:** Beginning Inventory + Purchases - Ending Inventory = Usage
**COGS Percentage:** Usage ÷ Category Sales = COGS %

**Example:**
- Food Usage: $46,844.37
- Food Sales: $136,177.17  
- Food COGS %: $46,844.37 ÷ $136,177.17 = 34.4%

### From Usage Report to Bonus Categories

**Food bucket:**
- Category Type = "Food"
- COGS % = Food Usage ÷ Food Sales

**Liquor/NA Bev bucket:**
- Category Type = "Liquor" + "N/A Bev"
- Combined COGS % = (Liquor Usage + NA Usage - Reimbursements) ÷ (Liquor Sales + NA Sales)
- **Always show combined totals for bonus calculations**

**Beer/Wine bucket:**
- Category Type = "Beer" + "Wine"  
- Combined COGS % = (Beer Usage + Wine Usage) ÷ (Beer Sales + Wine Sales)
- **Always show combined totals for bonus calculations**

## Cost Targets by Location

See `resources/bonus_reference.md` for complete tier structure.

**Cantina:**
- Food: 29%
- Liquor/NA Bev: 13%
- Beer/Wine: 25%

**OAK:**
- Food: 30%
- Liquor/NA Bev: 16%
- Beer/Wine: 26%

**White Buffalo:**
- Food: N/A (no target defined)
- Liquor/NA Bev: Custom tiers
- Beer/Wine: Custom tiers

## Reimbursement Handling

Reimbursements apply **only to Liquor/NA Bev COGS**, never to Food or Beer/Wine.

When a reimbursement is provided, the report shows three lines:
1. Liquor/NA Bev *(Before reimbursement)*
2. - Reimbursement (amount)
3. **Liquor/NA Bev (TRUE COST)** *(used for % and bonus calculations)*

Common reimbursements:
- E11even vodka credits ($4,000-$5,000 typical)
- Vendor promotional allowances
- Damaged goods credits

If no reimbursement mentioned, default to $0.

## Executive Summary Format

Structure responses like this:

```
📊 [LOCATION] P&L ANALYSIS — [MONTH YEAR]

━━━ SALES PERFORMANCE ━━━
Total Sales: $XXX,XXX
├─ Food: $XX,XXX
├─ Liquor: $XX,XXX  
├─ Beer: $XX,XXX
├─ Wine: $XX,XXX
└─ N/A Bev: $XX,XXX

━━━ COST PERFORMANCE ━━━

Food COGS: XX.X% (Target: XX%)
├─ Usage: $XX,XXX ÷ Sales: $XX,XXX
├─ Variance: ±X.X%
├─ $ Impact: $X,XXX
└─ Bonus: $X,XXX ✓/✗

Liquor COGS: XX.X% (Target: XX%)
├─ Raw Usage: $XX,XXX
├─ Reimbursement: -$X,XXX [if applicable]
├─ Adjusted Usage: $XX,XXX
├─ COGS %: $XX,XXX ÷ $XX,XXX = XX.X%
├─ Variance: ±X.X%
├─ $ Impact: $X,XXX
└─ Bonus: $X,XXX ✓/✗

Beer COGS: XX.X% (Target: XX%)
├─ Usage: $XX,XXX ÷ Sales: $XX,XXX
├─ Variance: ±X.X%
├─ $ Impact: $X,XXX
└─ Bonus: $X,XXX ✓/✗

Wine COGS: XX.X% (Target: XX%)
├─ Usage: $XX,XXX ÷ Sales: $XX,XXX
├─ Variance: ±X.X%
├─ $ Impact: $X,XXX
└─ Bonus: $X,XXX ✓/✗

N/A Bev COGS: XX.X%
├─ Usage: $XX,XXX ÷ Sales: $XX,XXX
└─ Note: High % typical (ice/mixers)

━━━ BONUS CATEGORY TOTALS ━━━

Liquor/NA Bev Combined: XX.X% (Target: XX%)
├─ Combined Sales: $XXX,XXX (Liquor + NA Bev)
├─ Combined Usage: $XX,XXX (adjusted for reimbursements)
├─ Variance: ±X.X%
├─ $ Impact: $X,XXX
└─ Bonus: $X,XXX ✓/✗

━━━ BONUS SUMMARY ━━━
COGS Bonuses: $XX,XXX
Top Line Bonus: $X,XXX (XX% tier)
├─ Eligibility: [Qualified/Not Qualified]
└─ Reason: [X of 3 COGS targets met]

TOTAL BONUS: $XX,XXX

━━━ KEY INSIGHTS ━━━
[3-5 bullet points]
```

## Insight Generation Rules

### When Costs Beat Target (Negative Variance)
- "🎯 [Category] crushed the target—XX.X% vs XX% goal. Saved $X,XXX this month."
- "Strong cost control on [category]. Team is X.X% under target."

### When Costs Miss Target (Positive Variance)
- "⚠️ [Category] ran XX.X% over target (XX.X% vs XX%). Cost you $X,XXX this month."
- "Focus area: [Category] needs tightening—X.X% above goal."

### Top Line Bonus
**If Qualified:**
- "Top Line bonus unlocked: $X,XXX at the XX% tier. Hit $XXX,XXX in sales (baseline: $XXX,XXX)."

**If Not Qualified (2+ COGS missed):**
- "Top Line bonus blocked—X categories missed target. Need 2 of 3 COGS goals met for eligibility."

### Action Items (Always Include 2-3)
- "Tighten [category] cost controls to hit XX% target next month."
- "Continue current [category] practices—performing X.X% under target."
- "Review high-cost items in [category] to close the $X,XXX gap."
- "Push sales by $X,XXX to hit next Top Line tier (XX%)."

## Bonus Tier Translation

When showing bonuses, explain the tier achieved in plain English.

**Example for Cantina Liquor/NA at 12.1%:**
✅ "Hit the $1,800 tier (12.5-13.49%). Just 0.6% away from the $8,400 tier—that's about $2,000 in savings needed."

**Example for OAK Food at 31%:**
✅ "Missed all bonus tiers—need to get under 30.5% (target: 30%). Cost $1,700 in bonus potential."

## Script Execution

The Skill uses three processing options:

**1. pnl_analyzer_pdf.py** - PDF processor (RECOMMENDED)
- Reads foodUsageReport.pdf directly
- Extracts summary table via pdfplumber
- Calculates COGS and bonuses
- Outputs JSON + terminal report
- Single command execution

**Usage:**
```python
python3 pnl_analyzer_pdf.py /path/to/foodUsageReport.pdf 4656
```

**2. pnl-engine.py** - CSV calculation engine (LEGACY)
- **Two-file mode:** Reads separate usage CSV and P&L CSV, calculates COGS using Beginning + Purchases - Ending formula
- **Single P&L mode:** Reads complete P&L CSV with pre-calculated COGS percentages  
- Maps categories to bonus buckets
- Applies reimbursements to Liquor/NA Bev only
- Determines bonus tiers
- Evaluates Top Line eligibility
- Outputs detailed CSV and summary text

**3. pnl-wrapper.py** - CSV auto-detection (LEGACY)
- Searches for files in /mnt/user-data/uploads/
- Auto-detects usage and P&L reports
- Builds command for pnl-engine.py
- Handles errors gracefully

Execute like this:

**Auto-detection (searches uploads folder):**
```python
python3 pnl-wrapper.py --location CANTINA --period 2025-10 --reimb 4656
```

**Two-file method:**
```python
python3 pnl-wrapper.py --location CANTINA --period 2025-10 --usage "/path/to/usage.csv" --pnl "/path/to/pnl.csv" --reimb 4656
```

**Single P&L method:**
```python
python3 pnl-wrapper.py --location WB --period 2025-11 --pnl "/path/to/controllableProfitAndLoss[White Buffalo].csv" --reimb 0
```

## Output Files

**PDF Workflow** creates:
1. `pnl_analysis_complete.json` - Machine-readable analysis
2. Terminal report (formatted text)

**CSV Workflow** creates:
1. `pnl_detail_[LOCATION]_[PERIOD].csv` - Full breakdown
2. `pnl_summary_[LOCATION]_[PERIOD].txt` - Text summary

**Dashboard** generates:
- Interactive HTML results (in-browser)
- Exportable PDF reports (future)

After generating output, offer to:
- "Show me the detailed CSV"
- "Compare to last month"
- "Run analysis for another location"

## Error Handling

## Interactive Dashboard

**pl-dashboard-final.html** provides:
- Drag-and-drop PDF upload
- Real-time bonus calculations
- **Bonus Forecast Calculator** ("What if Food = 28%?")
- Toggle bonus visibility on/off
- Color-coded insights
- Multi-month comparison support

**When to use:**
- Manager wants to model scenarios
- Need visual presentation
- Training team on targets
- Monthly planning sessions

**How to deploy:**
1. Open HTML file in browser
2. User drops PDF
3. Sets reimbursement (optional)
4. Clicks "Analyze P&L"
5. Uses forecast sliders to model next month

**Output:** Interactive charts + downloadable reports

## Error Handling

### Missing Files
If only 1 file uploaded (CSV mode):
**First check if it's a complete P&L report:**
- If `controllableProfitAndLoss[Location].csv` with both INCOME and COGS sections → Process standalone
- If only `foodUsageReport.csv` → Request P&L file

> "I can process this in two ways:
> 
> **Option 1:** Upload both files:
> 1. Usage Report (foodUsageReport.csv) 
> 2. P&L Report (controllableProfitAndLoss[Location].csv)
> 
> **Option 2:** Just the P&L Report if it contains both sales and COGS data (like your White Buffalo file)"

If no files uploaded:
> "Please upload either:
> - Single PDF: foodUsageReport.pdf (recommended)  
> - Complete P&L CSV: controllableProfitAndLoss[Location].csv (with sales + COGS)
> - Two CSVs: usage + P&L reports (legacy method)"

### Script Errors
If pnl-engine.py fails:
- Read the error message
- Explain in plain English
- Suggest fix

### Ambiguous Location
If location unclear:
> "Which location is this for? (Cantina / OAK / White Buffalo)"

## Multi-Location Workflow

If user uploads 6 files at once (3 usage + 3 P&L):
1. Ask: "Run all 3 locations?"
2. Process each sequentially
3. Generate comparison summary:
   - "Best performer: [Location] - $XX,XXX total bonus"
   - "Needs attention: [Location] - X categories over target"

## Tone & Style

- **Confident and direct** - "You're $2,000 over target" not "It appears costs may be slightly elevated"
- **Action-oriented** - Always end with "Next steps" or "Focus areas"
- **Celebrate wins** - Use ✓, 🎯, or bold for achievements
- **Flag problems clearly** - Use ⚠️ or bold for misses
- **No fluff** - Skip phrases like "based on the data provided" or "it's worth noting"

## Important Notes

- **COGS percentages calculated per category** - Usage ÷ Category Sales (not total sales)
- **P&L CSV files contain pre-calculated COGS %** - Use these directly when available
- **Reimbursements only apply to Liquor/NA bucket** (never Food or Beer/Wine)
- **"< for MAX" rule** - The highest bonus tier uses < (less than), not ≤
- **Top Line eligibility** - 2+ COGS misses = disqualified, regardless of sales level
- **WB has no Food target** - Skip food bonus calculations for White Buffalo
- **Script coordinates** - P&L values in column 2; rows: Beer=3, Food=4, Liquor=5, NA Bev=6, Wine=10
- **Multi-period reports** - Calculate monthly averages for comparison to targets
- **Single P&L processing** - When user uploads only controllableProfitAndLoss.csv, extract both sales and COGS directly

## Examples

### Example 1: Real Cantina Results (Sep-Oct 2025)

**User:**
> "Run full COGS report for Cantina, 2-month period Sep 1 through Oct 31. E11even reimbursement = $14,832."
> [Uploads foodUsageReport.pdf]

**Claude:**
[Detects PDF, calculates 2-month COGS with proper category breakdown]

📊 CANTINA P&L ANALYSIS — SEP-OCT 2025 (2 MONTHS)

━━━ SALES PERFORMANCE ━━━
Total Sales: $1,294,583.99
├─ Food: $136,177.17
├─ Liquor: $932,449.82
├─ Beer: $211,900.00
├─ Wine: $5,112.00
└─ N/A Bev: $8,945.00

━━━ COST PERFORMANCE ━━━

Food COGS: 34.4% (Target: 29%) ⚠️
├─ Usage: $46,844.37 ÷ Sales: $136,177.17
├─ Variance: +5.4%
├─ $ Impact: $7,353 over target
└─ Bonus: $0 (missed all tiers)

Liquor COGS: 11.4% (Target: 13%) ✅
├─ Raw Usage: $121,461.88
├─ E11even Reimbursement: -$14,832.00
├─ Adjusted Usage: $106,629.88
├─ COGS %: $106,629.88 ÷ $932,449.82 = 11.4%
├─ Variance: -1.6%
├─ $ Impact: $14,919 under target
└─ Bonus: $X,XXX (hit tier)

Beer COGS: 19.9% (Target: 25%) ✅
├─ Usage: $42,078.63 ÷ Sales: $211,900.00
├─ Variance: -5.1%
├─ $ Impact: $10,807 under target
└─ Bonus: $X,XXX (hit tier)

Wine COGS: 26.2% (Target: 25%) ⚠️
├─ Usage: $1,339.96 ÷ Sales: $5,112.00
├─ Variance: +1.2%
├─ $ Impact: $61 over target

N/A Bev COGS: 79.5%
├─ Usage: $7,112.59 ÷ Sales: $8,945.00
└─ Note: High % typical (ice/mixers)

━━━ BONUS CATEGORY TOTALS ━━━

Liquor/NA Bev Combined: 11.3% (Target: 13%) ✅
├─ Combined Sales: $941,394.82 (Liquor + NA Bev)
├─ Combined Usage: $106,629.88 (after $14,832 reimbursement)
├─ Variance: -1.7%
├─ $ Impact: $16,004 under target
└─ Bonus: $X,XXX (hit tier)

Beer/Wine Combined: 20.2% (Target: 25%) ✅
├─ Combined Sales: $217,012.00 (Beer + Wine)
├─ Combined Usage: $43,418.59
├─ Variance: -4.8%
├─ $ Impact: $10,417 under target
└─ Bonus: $X,XXX (hit tier)

━━━ TOTALS ━━━
Total Adjusted COGS: $204,005.43 (15.8%)
Monthly Average: $102,002.72 (15.8%)

Potential if all targets hit: $14,070
Opportunity cost: $10,560

Key Insights:
- ⚠️ Food 9.2% over target—largest issue ($7,288 waste)
- ⚠️ Liquor just 1% over but $6,129 impact
- ✓ Beer/Wine crushing it—3.2% under target
- 🔴 Top Line blocked (need 2 of 3 COGS targets)

November Action Plan:
1. 🔥 Food cost emergency—review portions, waste, comps
2. 🎯 Liquor control—check pours, verify E11even credit
3. ✅ Replicate Beer/Wine practices across categories
4. 📈 Fix 2 categories to unlock Top Line eligibility

### Example 2: Multi-Location Comparison

**User:**
> "Compare all 3 locations for September."
> [Uploads 6 files]

**Claude:**
[Processes all 3]

SEPTEMBER 2025 — ALL LOCATIONS

Best Performer: Cantina
├─ Total Bonus: $14,200
├─ All categories under target
└─ Top Line: Qualified ($3,500)

Needs Attention: OAK
├─ Total Bonus: $2,000
├─ Food 1.2% over (missed $1,700 tier)
└─ Top Line: Not qualified (2 misses)

White Buffalo:
├─ Total Bonus: $8,800
├─ Liquor/NA strong (9.8% vs 11.5%)
└─ Beer/Wine at target

Action Items:
- OAK: Focus on food cost—get under 30.5% to unlock bonuses
- WB: Push sales $8K to hit next Top Line tier
- Cantina: Maintain current practices—everything on target

## Feature Prompts

**When asked "What can this skill do?" provide these 10 example prompts:**

1. **"Run full COGS analysis for [Location], [Month/Period]. E11even reimbursement = $X,XXX."**
   - Complete breakdown with bonus calculations and actionable insights

2. **"Compare all 3 locations for [Month]. Show me who's winning and who needs help."**
   - Multi-location performance comparison with rankings

3. **"What if Cantina's food cost was 28% instead of 34%? Show me the bonus impact."**
   - Scenario modeling for bonus optimization

4. **"Analyze the last 3 months for OAK. What's the trend?"**
   - Month-over-month trend analysis with pattern identification

5. **"Calculate bonuses if we hit all targets vs. current performance for White Buffalo."**
   - Potential vs. actual bonus comparison with opportunity cost

6. **"Show me which category is costing us the most money across all locations."**
   - Cross-location variance analysis with dollar impact prioritization

7. **"Break down the $14,832 E11even reimbursement impact on Cantina's liquor bonus tiers."**
   - Detailed reimbursement effect analysis with tier explanations

8. **"What specific changes does each location need to unlock their next bonus tier?"**
   - Actionable recommendations with exact targets and dollar amounts

9. **"Compare Cantina's September vs October and predict November performance."**
   - Trend analysis with forward-looking recommendations

10. **"Show me a complete executive summary I can present to ownership about all locations."**
    - Professional report format with key insights and strategic recommendations

**Usage:** Simply say any of these prompts after uploading your foodUsageReport PDF(s) or CSV files.

## Success Criteria

After using this Skill, users should:
- ✓ Know exact bonus amounts in <2 minutes
- ✓ Understand which categories need attention
- ✓ Have clear action items for next month
- ✓ Be able to explain bonuses to managers confidently

## Resources

See `resources/bonus_reference.md` for complete bonus tier structure, targets, and Top Line thresholds for all locations.

---

## Quick Reference for Claude

### File Detection & Execution
**When user uploads files, auto-detect and execute:**

1. **Dual PDF Mode (Preferred)** - If user uploads BOTH:
   - `*foodUsageReport*.pdf` AND `*Gross*Sales*.pdf`
   - Execute: `python3 pnl_analyzer_pdf.py foodUsage.pdf grossSales.pdf [reimbursement]`
   - Result: Uses **NET SALES** - most accurate

2. **Single PDF Mode (Legacy)** - If user uploads ONLY:
   - `*foodUsageReport*.pdf`
   - Execute: `python3 pnl_analyzer_pdf.py foodUsage.pdf [reimbursement]`
   - Result: Uses **GROSS SALES** - warn about accuracy

3. **CSV Mode (Legacy)** - If user uploads:
   - `*foodUsageReport*.csv` AND `*controllableProfitAndLoss*.csv`
   - Execute: `python3 pnl_wrapper.py --location [LOC] --period [YYYY-MM] --reimb [AMT]`
   - Result: Uses P&L report sales data

### Auto-Detection Patterns
- **Food Usage PDF**: `*foodUsageReport*.pdf`, `*Food*Usage*.pdf`
- **Gross Sales PDF**: `*Gross*Sales*.pdf`, `*Sales*Breakdown*.pdf`
- **Usage CSV**: `*foodUsageReport*.csv`, `*usage*.csv`
- **P&L CSV**: `*controllableProfitAndLoss*.csv`, `*P&L*.csv`

### Key Messages to User
- ✅ **"Using NET SALES for accurate calculations"** (dual PDF mode)
- ⚠️ **"Warning: Using GROSS SALES - upload Gross Sales PDF for better accuracy"** (single PDF)
- 📊 **Show both total sales amounts and bonus difference when comparing modes**
