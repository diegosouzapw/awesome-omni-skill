---
name: eda
description: Exploratory Data Analysis for tabular data. Use when analyzing column distributions, checking data quality, examining class balance, detecting missing patterns, or generating summary statistics for datasets.
---

# Exploratory Data Analysis (EDA)

Analyze tabular datasets to understand distributions, data quality, and patterns.

## When to Use

- Understanding a new dataset before modeling
- Checking data quality (missing values, outliers, duplicates)
- Analyzing target variable distribution
- Identifying class imbalance
- Generating summary statistics

## Analysis Process

1. **Connect to data** - Verify access and inspect schema
2. **Analyze target variable first** - Understand class balance
3. **Check each column** - Distribution, missing data, cardinality
4. **Document findings** - Save reports for reproducibility

## Available Analyses

| Analysis | Description |
|----------|-------------|
| Column Distribution | Value counts, percentages, cardinality assessment |
| Missing Data | Null counts, patterns (MCAR/MAR/MNAR) |
| Class Balance | Imbalance detection for classification targets |
| Summary Stats | Count, unique, nulls per column |

## Column Distribution Analysis

For detailed analysis methodology and output format:
- See [references/eda-analysis.md](references/eda-analysis.md)

### Quick Reference

**Cardinality Levels:**
| Level | Criteria | Action |
|-------|----------|--------|
| Low | ≤10 unique | Good for categorical encoding |
| Medium | 11-100 or <1% of rows | May need encoding strategy |
| High | >100 and <50% of rows | Consider grouping/binning |
| Very High | >50% of rows | Likely identifier, exclude |

**Missing Data Thresholds:**
| Percentage | Assessment |
|------------|------------|
| 0% | No missing data |
| <1% | Minimal - safe to drop or impute |
| 1-5% | Some - consider imputation strategy |
| >5% | Significant - investigate pattern |

**Class Imbalance:**
- >80% in top class: Imbalance detected
- >95% in top class: Extreme imbalance

## Output Format

```markdown
# Column Distribution: {column_name}

- **source**: path/to/data
- **column**: column_name

## Summary
- Total rows: N
- Null/missing: N (X%)
- Unique values: N
- Cardinality: Low|Medium|High|Very High

## Distribution
| Value | Count | Percentage | Cumulative |
|-------|-------|------------|------------|

## Observations
- Auto-generated insights
```

## Best Practices

1. Start with schema inspection before deep analysis
2. Check target variable first for classification tasks
3. Missing data may not be random - investigate patterns
4. Save reports for reproducibility
