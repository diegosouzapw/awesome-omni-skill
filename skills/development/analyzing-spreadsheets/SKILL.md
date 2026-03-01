---
name: Analyzing Spreadsheets
description: Analyzes Excel spreadsheets, summarizes trends, and recommends charts when users mention spreadsheets, Excel workbooks, or .xlsx files.
---

# Analyzing Spreadsheets

## When to use
- User shares an Excel workbook or asks about spreadsheet analysis
- Tasks include summarizing metrics, spotting anomalies, or drafting charts
- Data lives in tabular form (CSV or XLSX)

## Workflow
1. **Inspect workbook structure**
   ```python
   import pandas as pd
   xl = pd.ExcelFile("input.xlsx")
   xl.sheet_names
   ```
2. **Load relevant sheets**
   ```python
   df = pd.read_excel("input.xlsx", sheet_name="Sheet1")
   df.head()
   ```
3. **Clean and validate**
   - Drop empty columns/rows
   - Normalize date formats with `pd.to_datetime`
   - Verify numeric columns with `df.describe()`
4. **Analyze and summarize**
   - Use groupby/pivot patterns from [reference/pandas-recipes.md](reference/pandas-recipes.md)
   - Highlight KPIs, trends, and outliers
5. **Recommend visuals**
   - Suggest chart types (line for time series, bar for categorical comparisons, heatmap for correlations)
   - Provide short rationale per recommendation

## Output expectations
- Concise summary (1–3 paragraphs) covering key findings
- Bullet list of insights with supporting numbers
- Optional chart suggestions with column mappings

## Validation checklist
- [ ] Loaded the correct sheet(s) and reported row/column counts
- [ ] Highlighted missing or unusual data
- [ ] Referenced actual values from the workbook
- [ ] Included next-step recommendations (e.g., further slicing, charting)

## Additional resources
- [reference/pandas-recipes.md](reference/pandas-recipes.md) – common aggregation patterns
- `python -m pip install pandas openpyxl` – install requirements if missing (Claude Code already includes pandas)

