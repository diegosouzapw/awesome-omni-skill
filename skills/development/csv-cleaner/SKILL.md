---
name: csv-cleaner
description: Clean and normalize CSV data by analyzing structure, detecting issues (missing values, duplicates, type inconsistencies), and applying transformations. Use when users need to prepare messy CSV files for analysis or import.
---

# CSV Cleaner Skill

You are a data cleaning specialist. Use this skill to clean and normalize CSV data.

## Setup

Before running scripts, install dependencies:
```bash
pip install -r requirements.txt
```

## How to Use This Skill

1. **Start**: Read `knowledge/index.md` for overview
2. **Analyze**: Run `python scripts/analyze.py <input.csv>` to get data profile
3. **Learn**: Based on issues found, read relevant knowledge files
4. **Clean**: Run cleaning operations using `scripts/clean.py`
5. **Output**: Generate cleaned CSV, report, and schema

## Available Scripts

### analyze.py
```bash
python scripts/analyze.py input.csv [--output analysis.json]
```
Returns JSON with:
- Column names, types, stats
- Missing value counts
- Duplicate detection
- Semantic type inference (email, phone, date, etc.)

### clean.py
```bash
python scripts/clean.py input.csv output.csv --operations ops.json
```
Operations file format:
```json
{
  "operations": [
    {"type": "fill_missing", "column": "age", "strategy": "median"},
    {"type": "normalize_strings", "column": "name", "ops": ["trim", "lowercase"]},
    {"type": "standardize_dates", "column": "created_at", "format": "%Y-%m-%d"}
  ]
}
```

### validate.py
```bash
python scripts/validate.py input.csv --schema schema.json
```
Validates data against JSON Schema, reports violations.

## Workflow

1. Run `analyze.py` on input CSV
2. Review output, identify issues
3. Read knowledge files for relevant topics:
   - Missing values → `knowledge/operations/missing-values.md`
   - Duplicates → `knowledge/operations/duplicates.md`
   - String issues → `knowledge/types/strings.md`
   - Date parsing → `knowledge/types/dates.md`
4. Build operations JSON based on knowledge
5. Run `clean.py` with operations
6. Generate report and schema

## Decision Making

When unsure which strategy to use, consult the knowledge files.
They contain decision trees and best practices for each scenario.

## Available Operations

| Operation | Description | Required Params |
|-----------|-------------|-----------------|
| `fill_missing` | Fill null values | `column`, `strategy` (mean/median/mode/constant/forward/backward) |
| `drop_missing` | Drop rows with nulls | `columns` (list), `how` (any/all) |
| `remove_duplicates` | Remove duplicate rows | `columns` (optional), `keep` (first/last/none) |
| `normalize_strings` | Clean string columns | `column`, `ops` (trim/lowercase/uppercase/remove_special) |
| `standardize_dates` | Parse and format dates | `column`, `format` (strftime format) |
| `normalize_phones` | Convert to E.164 format | `column`, `country` (default: US) |
| `cap_outliers` | Cap extreme values | `column`, `method` (iqr/zscore), `multiplier` |

## Knowledge Base Structure

```
knowledge/
├── index.md                 # Start here
├── operations/
│   ├── missing-values.md    # Handling nulls
│   ├── duplicates.md        # Deduplication
│   ├── outliers.md          # Outlier detection
│   └── normalization.md     # General patterns
├── types/
│   ├── strings.md           # Text cleaning
│   ├── numbers.md           # Numeric formatting
│   ├── dates.md             # Date parsing
│   ├── emails.md            # Email validation
│   └── phones.md            # Phone normalization
├── validation/
│   └── index.md             # JSON Schema rules
└── csv/
    └── edge-cases.md        # Encoding, quoting
```

Read only what you need based on detected issues.
