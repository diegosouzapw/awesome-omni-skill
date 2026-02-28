---
name: analyzing-backtests
description: Analyzes algorithmic trading backtest results from Jupyter notebooks and generates summary reports. Use when the user wants to analyze or summarize backtest notebooks.
allowed-tools: Read, Bash, Glob, Grep
---

# Backtest Analysis Skill

Analyze a Jupyter notebook containing algorithmic trading backtest results and generate a comprehensive summary report.

## Analysis Steps

1. **Version Control Information**
   - Run `git status` to check current state
   - Run `git log -1 --format="%H %ci"` for latest commit hash and date
   - Note any uncommitted changes

2. **Read the Notebook**
   - Use Read tool to load the specified .ipynb file
   - Parse cells for code, markdown, and outputs

3. **Extract Key Information**

   **Model/Strategy Details:**
   - Strategy name, type, and configuration
   - Key hyperparameters
   - Training and testing period information

   **Date Coverage:**
   - Backtest period (start, end, duration)

   **Performance Metrics:**
   - Monetary results: returns, capital, drawdowns, trade statistics
   - Statistical analysis: risk metrics, benchmark comparisons, distributions
   - Extract whatever metrics are available in the notebook

4. **Generate Report**

Output a structured markdown report:

```markdown
# Backtest Analysis Report

**Notebook:** [filename]
**Generated:** [date]
**Git Commit:** [hash] ([date])
**Uncommitted Changes:** [yes/no]

## Strategy
[Name and brief description]

**Configuration:**
- [Key parameters]

## Period
- **Dates:** [start] to [end] ([duration])

## Performance

| Metric | Value | Benchmark |
|--------|-------|-----------|
| Total Return | X% | X% |
| Annualized Return | X% | X% |
| Max Drawdown | X% | X% |
| Sharpe Ratio | X.XX | X.XX |
| Win Rate | X% | - |
| Total Trades | X | - |

## Risk Metrics
| Metric | Value |
|--------|-------|
| Volatility | X% |
| Alpha | X% |
| Beta | X.XX |

## Key Findings
- [Notable observations]
- [Strengths and weaknesses]

## Concerns/Recommendations
- [Any issues or suggestions]
```

## Instructions

- Extract all available metrics from the notebook
- Mark unavailable metrics as "N/A"
- Provide brief analysis, not just data
- Flag unusual results or potential issues
- Keep report concise but comprehensive
