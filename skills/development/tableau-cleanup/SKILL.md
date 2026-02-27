---
name: tableau-cleanup
description: Clean up Tableau workbooks by standardizing captions, adding comments, and organizing into folders.
---

# Tableau Workbook Cleanup

Clean up Tableau workbooks (.twb/.twbx) by editing XML. Run validation, fix errors, repeat until clean.

## Scratchpad

Use `.cleanup/` directory. Track progress in `.cleanup/status.json`.

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/backup_workbook.py <input>` | Backup before editing |
| `scripts/extract_twbx.py <input.twbx>` | Unzip packaged workbook |
| `scripts/list_calculations.py <file.twb>` | List all calcs as JSON |
| `scripts/validate_cleanup.py <file.twb>` | **Check all rules, output errors** |
| `scripts/validate_xml.py <file.twb>` | Check XML validity |
| `scripts/repackage_twbx.py <dir> <output.twbx>` | Repackage to .twbx |

## Safety Rules

1. **Backup first** - Always run backup_workbook.py
2. **Never modify `name` attributes** - Only edit `caption`
3. **Escape XML**: `&` -> `&amp;`, `'` -> `&apos;`
4. **Create `_cleaned` copy** - Don't overwrite original

## What You CAN and CANNOT Edit

### CAN EDIT (Safe)

| Element | Attribute | What to do |
|---------|-----------|------------|
| `<column>` | `caption` | Change to Title Case, remove underscores, remove c_ prefix |
| `<calculation>` | `formula` | ADD `// comment` at START (only if formula attribute already exists) |
| `<folders-common>` | (whole element) | CREATE if missing, add folders |
| `<folder>` | `name` | Use exact names from Folder Rules with HTML entity codes |
| `<folder-item>` | `name`, `type` | Reference calculation names |
| `<layout>` | `show-structure` | Set to `'true'` |

### CANNOT EDIT (Will Corrupt Workbook)

| Element | What NOT to do | Why |
|---------|----------------|-----|
| `<column>` | Change `name` attribute | Breaks all references to this field |
| `<calculation class='categorical-bin'>` | Add `formula` attribute | Bin/group calcs use XML structure, not formulas |
| `<calculation class='quantitative-bin'>` | Add `formula` attribute | Same - bins don't have formulas |
| Any `<calculation>` without `formula` | Add `formula` attribute | These are special calc types |
| Formulas with `&#13;&#10;` | Remove or change these | These are valid XML-encoded newlines |
| Formulas with `&amp;` | Change to `&amp;amp;` | Already properly encoded |
| `<column>` | Change `datatype`, `role`, `type` | Breaks field behavior |
| `<datasource>` | Change `name` or structure | Breaks data connections |

### STOP Conditions

If you encounter these, STOP and report - do NOT try to fix:
1. Validator says "newline not XML-encoded" but you see `&#13;&#10;` in raw XML - Validator bug
2. Validator says "unescaped &" but you see `&amp;` in raw XML - Validator bug
3. Validator says "missing comment" on a calc with no `formula` attribute - Can't add comment
4. Any error on `categorical-bin` or `quantitative-bin` calculations - Skip these

### Example: What a Proper Edit Looks Like

**BEFORE:**
```xml
<column caption='c_total_sales' name='[Calculation_123]'>
  <calculation formula='SUM([Sales])' />
</column>
```

**AFTER:**
```xml
<column caption='Total Sales' name='[Calculation_123]'>
  <calculation formula='// Aggregates all sales for the selected period&#13;&#10;SUM([Sales])' />
</column>
```

**NOTICE:**
- `caption` changed (safe)
- `formula` has comment ADDED at start (safe)
- `name` attribute UNCHANGED (critical!)
- `&#13;&#10;` used for newline (correct XML encoding)

## Reference Documents

Before starting, read these guides in the skill's `resources/` folder:
- `resources/comment-guide.md` - How to write meaningful comments (REQUIRED for M3)
- `resources/good-comments.md` - 50+ real examples by category
- `resources/xml-folders-guide.md` - How to create folders in XML

## Workflow

1. Backup workbook
2. Extract if .twbx
3. Run `validate_cleanup.py` to see all errors
4. Fix errors one category at a time
5. Run validation again
6. Repeat until 0 errors
7. Repackage if .twbx
8. Report changes

## Caption Rules

- Title Case with spaces (no underscores)
- No `c_` prefix
- Preserve acronyms: ID, YTD, MTD, KPI, ROI, YOY, MOM, WOW, LOD
- No double parentheses `()()`

## Comment Rules

Add `//` comment explaining PURPOSE at start of formula:
```xml
formula='// Flags at-risk accounts for dashboard highlight&#13;&#10;[Score] < 50'
```

Use `&#13;&#10;` for newlines. Escape `&` as `&amp;`.

### Comment Quality Requirements (M3 Validation)

Comments MUST:
- Be **15+ characters** of explanation
- Explain **WHY** (purpose), not just WHAT (formula description)
- NOT just restate the caption

Comments that FAIL M3:
- `// Calculated field` - too generic
- `// Sum` - too short (only 3 chars)
- `// Total Revenue` (if caption is "Total Revenue") - restates caption

See `resources/comment-guide.md` for detailed guidance.

### Batch Processing (Recommended)

Use `scripts/batch_comments.py` to process calculations 10 at a time:

```bash
python batch_comments.py workbook.twb init    # Create batches
python batch_comments.py workbook.twb next    # Show next 10 calcs
python batch_comments.py workbook.twb done 1  # Mark batch complete
python batch_comments.py workbook.twb status  # Check progress
```

This ensures you READ each formula before commenting.

## Folder Rules

Insert `<folders-common>` BEFORE `<layout>` with exactly 6 broad folders (use HTML entity codes):

```xml
<folders-common>
  <folder name='&#x1F4CA; Metrics'>
    <folder-item name='[Calculation_XXX]' type='field' />
  </folder>
</folders-common>
```

**6 Folders Only** (use HTML entity codes to avoid encoding issues):
| Folder | Entity Code | Contains |
|--------|-------------|----------|
| Metrics | `&#x1F4CA;` | KPIs, totals, margins, revenue, percentages, averages, growth |
| Dates | `&#x1F4C5;` | Date calcs, periods, fiscal, YTD/MTD/QTD, year, month, quarter |
| Filters | `&#x1F6A6;` | Booleans, flags, is_, has_, visibility, include/exclude, parameters |
| Display | `&#x1F3A8;` | Labels, tooltips, formatting, colors, text, UI elements, rankings |
| Projections | `&#x1F52E;` | Forecasts, targets, goals, budgets, predictions, estimates |
| Security | `&#x1F512;` | RLS, user-based filters, permissions, access control |

**IMPORTANT:** Use `&#x` entity format, NOT raw emoji characters (prevents encoding corruption).

**Note:** LOD calcs (FIXED/INCLUDE/EXCLUDE) go in the folder matching their PURPOSE, not technique.

## Report

```
=== Tableau Cleanup Complete ===
Workbook: <name>
Errors fixed: X
Output: <path>_cleaned.twbx
```
