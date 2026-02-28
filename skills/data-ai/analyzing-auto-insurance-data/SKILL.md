---
name: analyzing-auto-insurance-data
description: Analyzes vehicle insurance daily reports and signing lists. Use when user asks to analyze insurance data, generate business reports, check institution performance, monitor policy trends, or detect business anomalies. Handles Excel/CSV files with fields like premium, institution, customer type, and renewal status.
allowed-tools: Read, Edit, Grep, Glob
---

# Vehicle Insurance Business Data Analysis

You are a specialized assistant for analyzing vehicle insurance business data. Your role is to process signing lists, generate statistical reports, and provide actionable business insights.

## When to Use This Skill

Activate this skill when the user requests:
- Analysis of insurance signing lists or daily reports
- Statistical summaries of premium, policy count, or institution performance
- Comparison across time periods, institutions, or customer segments
- Anomaly detection in business metrics
- Trends in renewal rates, customer types, or product combinations

## Step-by-Step Analysis Workflow

Follow these steps when conducting analysis:

### Step 1: Data Loading and Validation

**Preferred approach: CSV files**
1. If user provides Excel file, first convert it to CSV format using Python
2. Read the CSV file directly using the Read tool (much more efficient for AI)
3. Parse the CSV content and validate structure
4. Verify the presence of required fields (投保确认时间, 三级机构, 业务员, 总保费, etc.)
5. Check data types and formats
6. Report any missing critical fields to the user

**Why CSV is better:**
- AI can directly read and parse CSV text content
- No need for external libraries (pandas/openpyxl)
- Faster processing and lower context usage
- More transparent data structure

**For Excel files:**
- Use Bash tool with Python to convert Excel → CSV first
- Save CSV to temporary location
- Then read CSV using Read tool

### Step 2: Data Cleaning
1. Handle missing values appropriately:
   - Missing 三级机构: Look up from staff-institution mapping table
   - Missing 团队简称: Acceptable, leave as null
   - Missing 总保费: Flag as invalid record
2. Process special values:
   - Negative premium: Keep (indicates policy cancellation/adjustment)
   - Zero commission: Normal, no action needed
   - Negative policy amount: Flag as anomaly
3. Parse date fields and ensure chronological order

### Step 3: Load Reference Data

**Best practice: Convert mapping table to structured format first**

1. Convert `业务员机构团队对照表20251104.xlsx` to CSV using Python
2. Read the CSV using Read tool to get structured text data
3. Parse and create a lookup dictionary: {业务员: {三级机构, 四级机构, 团队简称}}
4. Use this mapping to correct institution assignments in the signing list

**Alternative: Pre-converted JSON format**
- Suggest user to maintain `staff_mapping.json` for faster loading
- JSON format example:
```json
{
  "200049147向轩颉": {"三级机构": "达州", "四级机构": "达州", "团队简称": null},
  "210011936赵莎莎": {"三级机构": "达州", "四级机构": "达州", "团队简称": "达州业务三部"}
}
```
- This allows direct Read tool usage without conversion step

### Step 4: Calculate Core Metrics
Compute these essential statistics:
- Total policy count
- Total premium (总保费总计)
- Average premium per policy
- Commission total and ratio
- Daily premium trends
- Institution-level aggregations
- Customer type distribution
- Renewal status breakdown
- Product combination analysis

### Step 5: Dimensional Analysis
Conduct analysis across these dimensions:

**Time Dimension**
- Daily business volume and premium
- Week-over-week comparisons (same weekday across 3 weeks)
- Weekday vs. weekend patterns

**Institution Dimension**
- Level-3 institution performance (using corrected mapping)
- Level-4 institution performance
- Institution concentration (single institution exceeding 40% is high risk)
- Geographic analysis (Chengdu vs. other cities)

**Customer Dimension**
- 9 customer categories distribution
- Renewal status: 转保 (transfer), 续保 (renewal), 新保 (new)
- 5 product combinations analysis

**Team Dimension** (when available)
- Team-level performance within each institution
- Top-performing teams and agents

### Step 6: Anomaly Detection
Check for these business anomalies:

**Priority: High**
- Daily premium fluctuation exceeding ±10%
- Weekend business surge over 10x normal level
- Single institution concentration above 40%

**Priority: Medium**
- Unusual customer type shifts
- Abnormal commission ratios
- Significant changes in renewal rates

**Priority: Low**
- Minor product mix changes
- Small team performance variations

### Step 7: Generate Report
Structure the output report with:
1. Executive summary (3-5 key findings)
2. Core metrics table
3. Dimensional analysis results
4. Anomaly alerts (if any)
5. Actionable recommendations

## Data Requirements

### Expected File Formats (Priority Order)

**Tier 1 - Highly Recommended (AI-friendly):**
- **CSV files** (UTF-8 encoding with BOM, comma-delimited)
- **JSON files** (for configuration and mapping data)
- **Plain text** structured data

**Tier 2 - Acceptable (requires conversion):**
- Excel files (.xlsx, .xls) - will be converted to CSV first

**Why this priority matters:**
1. **CSV/JSON**: AI can directly read and parse as text → Fast and efficient
2. **Excel**: Binary format, requires Python conversion → Slower, more steps
3. **Best practice**: Ask users to export Excel as CSV before uploading

**Recommended workflow for users:**
1. Open Excel file
2. File → Save As → CSV UTF-8 (Comma delimited) (*.csv)
3. Upload the CSV file instead of Excel

### Core Data Fields

| Field Name | Description | Data Type | Required |
|------------|-------------|-----------|----------|
| 投保确认时间 | Policy confirmation time | Datetime | Yes |
| 报告日期 | Report date | Date | Yes |
| 三级机构 | Level-3 institution | String | Yes* |
| 四级机构 | Level-4 institution | String | No |
| 业务员 | Sales agent (format: ID+Name) | String | Yes |
| 客户类别 | Customer type | String | Yes |
| 险别组合 | Product combination | String | Yes |
| 续保情况 | Renewal status | String | Yes |
| 总保费 | Total premium | Numeric | Yes |
| 手续费 | Commission | Numeric | No |
| 签单保额 | Policy amount | Numeric | No |

*三级机构: If missing in data, look up from staff-institution mapping table

### Staff-Institution Mapping Table

File: `业务员机构团队对照表20251104.xlsx`

This reference file contains 229 records with the structure:

| Column | Field Name | Example Value | Purpose |
|--------|------------|---------------|---------|
| 2 | 序号 | 1, 2, 3... | Index number |
| 3 | 三级机构 | 达州, 德阳, 天府 | Level-3 institution name |
| 4 | 四级机构 | 达州, 德阳 | Level-4 institution name |
| 5 | 团队简称 | 达州业务三部 | Team short name (nullable) |
| 6 | 业务员 | 200049147向轩颉 | Agent ID+Name |

**Usage:**
1. Extract the 业务员 field from signing list
2. Look up corresponding 三级机构 from this mapping table
3. Use the mapped institution (not the one from signing list if different)
4. This ensures accurate institutional attribution

## Business Rules and Thresholds

### Data Cleaning Rules
1. **Negative premium**: Retain in analysis (caused by policy adjustments/cancellations)
2. **Zero commission**: Normal occurrence, no action required
3. **Negative policy amount**: Flag as data anomaly, recommend verification

### Alert Thresholds
| Metric | Threshold | Priority | Action |
|--------|-----------|----------|--------|
| Daily premium change | ±10% | High | Alert user |
| Weekend surge ratio | >10x | High | Alert user |
| Institution concentration | >40% | High | Alert user |
| Commission ratio | <3% or >8% | Medium | Note in report |
| Renewal rate drop | >15% | Medium | Note in report |

### Customer Categories (9 Types)
Primary focus areas:
- **非营业个人客车** (Non-commercial personal vehicles): 53.8% of premium, highest value
- **摩托车** (Motorcycles): 24.8% of premium, second largest
- **非营业货车** (Non-commercial trucks): 5.6% of premium, third segment

### Product Combinations (5 Types)
- **单交** (Compulsory only): 48.1% - opportunity for commercial insurance upsell
- **交商** (Compulsory + Commercial): Target for growth
- Others: Specialized combinations

### Renewal Analysis
- **转保** (Transfer): Policies from other insurers
- **续保** (Renewal): Existing customer renewals - track retention rate
- **新保** (New): First-time policies

## Output Format Examples

### Daily Report Summary
```markdown
## Vehicle Insurance Business Report
**Report Period**: [Start Date] to [End Date]

### Executive Summary
- Total Policies: [count] policies
- Total Premium: ¥[amount] million
- Average Premium: ¥[avg] per policy
- Top Institution: [name] ([percentage]%)

### Key Findings
1. [Finding 1 with data support]
2. [Finding 2 with data support]
3. [Finding 3 with data support]

### Anomaly Alerts
**High Priority:**
- [Alert 1 if any]

**Medium Priority:**
- [Alert 2 if any]

### Recommendations
1. [Actionable recommendation 1]
2. [Actionable recommendation 2]
```

### Statistical Table Format
```
| Dimension | Metric | Count | Premium (万元) | Share (%) |
|-----------|--------|-------|---------------|-----------|
| Overall   | Total  | XXX   | XXX.XX        | 100.00    |
| Level-3 A | -      | XXX   | XXX.XX        | XX.XX     |
| Level-3 B | -      | XXX   | XXX.XX        | XX.XX     |
```

## Common Analysis Scenarios

### Scenario 1: Weekly Performance Review
**User request**: "Analyze the last 3 weeks of insurance data, compare daily trends"

**Your approach**:
1. Load data and filter to recent 3 weeks
2. Calculate daily metrics (count, premium, avg)
3. Group by weekday for same-day comparisons across weeks
4. Identify weekly patterns and anomalies
5. Generate trend charts and summary statistics

### Scenario 2: Institution Performance Comparison
**User request**: "Compare performance across all institutions this month"

**Your approach**:
1. Load staff-institution mapping first
2. Correct institution assignments using mapping
3. Group by 三级机构 and calculate aggregates
4. Rank by total premium and policy count
5. Calculate concentration ratios
6. Flag high concentration if single institution >40%

### Scenario 3: Customer Segment Deep Dive
**User request**: "Analyze motorcycle customer segment in detail"

**Your approach**:
1. Filter data to 客户类别 = '摩托车'
2. Calculate segment contribution to total business
3. Analyze renewal rate for this segment
4. Break down product combinations within segment
5. Compare to previous periods if historical data available
6. Provide segment-specific insights

### Scenario 4: Anomaly Monitoring
**User request**: "Run business anomaly check on today's data"

**Your approach**:
1. Load today's data and previous 7 days for baseline
2. Calculate day-over-day premium change
3. Check if change exceeds ±10% threshold
4. Verify institution concentration
5. Check for weekend anomalies if applicable
6. Generate prioritized alert report

## Important Considerations

1. **Always use the staff-institution mapping** to determine the correct 三级机构 for each agent. The institution in the signing list may be incorrect.

2. **Preserve negative premium values** in calculations as they represent legitimate business adjustments (policy cancellations, refunds).

3. **Context matters**: A 15% premium drop on Monday after weekend is normal; the same drop mid-week is anomalous.

4. **Focus on actionable insights**: Don't just report numbers, explain what they mean for the business.

5. **Data quality**: Always report data quality issues (missing fields, anomalous values) to the user.

6. **Trend context**: When possible, compare current metrics to historical baselines (previous week, month, or year).

## Related Files

### Documentation
- [业务规则与数据洞察.md](业务规则与数据洞察.md) - Comprehensive business rules documentation
- [excel_analysis_report.md](excel_analysis_report.md) - Example analysis report

### Scripts
- [scripts/convert_excel_to_csv.py](scripts/convert_excel_to_csv.py) - Convert Excel to CSV/JSON (recommended)
- [数据分析预警规则.py](数据分析预警规则.py) - Automated monitoring script

### Data Files
- `staff_mapping.json` - Pre-converted staff-institution mapping (228 agents)
- `业务员机构团队对照表20251104.xlsx` - Original mapping Excel file

## Quick Start for Users

### Option 1: Use CSV (Recommended)
```bash
# Convert your Excel file to CSV first
python scripts/convert_excel_to_csv.py your_data.xlsx

# Then ask AI to analyze the CSV file
# "Please analyze your_data.csv"
```

### Option 2: Use JSON Mapping (Fastest)
```bash
# For staff mapping, convert to JSON once
python scripts/convert_excel_to_csv.py 业务员机构团队对照表.xlsx --mapping

# This creates staff_mapping.json which AI can read directly
```

### Option 3: Direct Excel (Slower)
```
# AI will convert Excel to CSV first, then analyze
# "Please analyze your_data.xlsx"
```

**Recommendation**: Always use Option 1 or 2 for best performance

## Core Data Processing Logic Reference

### DataProcessor Class Overview

The backend implements a comprehensive `DataProcessor` class ([backend/data_processor.py](backend/data_processor.py)) that handles all data operations. Key responsibilities:

#### 1. Staff-Institution Mapping (`_build_name_to_info`)
**Location**: [backend/data_processor.py:24-58](backend/data_processor.py#L24-L58)

**Purpose**: Build name → institution/team mapping from staff mapping file

**Logic**:
- Extract Chinese name from "工号+姓名" format (e.g., "200049147向轩颉" → "向轩颉")
- Create mapping: `{姓名: {三级机构, 四级机构, 团队简称}}`
- Detect conflicts: same name with different institution/team assignments
- Returns: `(name_to_info dict, conflicts list)`

**Business Rule**: When same name appears multiple times with different info, keep last record and flag as conflict

#### 2. Policy Mapping (`get_policy_mapping`)
**Location**: [backend/data_processor.py:60-98](backend/data_processor.py#L60-L98)

**Purpose**: Create unique policy number → staff → institution/team chain

**Returns**:
```python
{
  'policy_to_staff': {保单号: 业务员姓名},
  'staff_to_info': {姓名: {三级机构, 四级机构, 团队简称}},
  'conflicts': [姓名列表]
}
```

**Use Case**: Ensures data consistency when filtering by policy number

#### 3. KPI Three-Window Calculation (`get_kpi_windows`)
**Location**: [backend/data_processor.py:559-658](backend/data_processor.py#L559-L658)

**Purpose**: Calculate KPI metrics across 3 time windows anchored to a specific date

**Three Windows**:
1. **Day**: Specified date only
2. **Last 7 days**: 7-day window ending on specified date (inclusive)
3. **Last 30 days**: 30-day window ending on specified date (inclusive)

**Metrics per Window**:
- Total premium (`签单/批改保费`)
- Policy count (`签单数量`)
- Commission (`手续费含税`)

**Returns**:
```python
{
  'anchor_date': 'YYYY-MM-DD',
  'premium': {'day': float, 'last7d': float, 'last30d': float},
  'policy_count': {'day': int, 'last7d': int, 'last30d': int},
  'commission': {'day': float, 'last7d': float, 'last30d': float},
  'target_gap_day': float,
  'validation': {...}
}
```

**Key Implementation Details**:
- Anchor date defaults to latest date in dataset
- Uses `pd.to_datetime` for date parsing with error handling
- Applies filters BEFORE calculating windows
- Includes data validation results

#### 4. Week Comparison (`get_week_comparison`)
**Location**: [backend/data_processor.py:408-557](backend/data_processor.py#L408-L557)

**Purpose**: Compare same weekdays across 3 consecutive 7-day periods

**Algorithm**:
1. Anchor to latest date (or specified date)
2. Define 3 periods:
   - Period 0 (D): [anchor - 6 days, anchor]
   - Period 1 (D-7): [anchor - 13 days, anchor - 7 days]
   - Period 2 (D-14): [anchor - 20 days, anchor - 14 days]
3. For each period, extract 7 consecutive days aligned by weekday
4. Calculate daily premium or count for each day
5. Generate chart-ready series data

**Returns**:
```python
{
  'latest_date': '2025-11-05',
  'x_axis': ['周三', '周四', '周五', '周六', '周日', '周一', '周二'],
  'series': [
    {
      'name': 'D-14 (10-22): 781万',
      'data': [110234.5, 95023.1, ...],  # 7 daily values
      'dates': ['2025-10-22', '2025-10-23', ...],
      'code': 'D-14',
      'total_value': 7814320.5,
      'period_index': 2
    },
    ...
  ],
  'validation': {...}
}
```

**Metrics**:
- `metric='premium'`: Sum of `签单/批改保费` per day
- `metric='count'`: Count of policies with premium ≥ 50 per day

**X-Axis Logic**: Weekday labels start from first day of Period 0 (D), maintaining consistent weekday alignment across all 3 periods

**Why This Matters**: Enables apple-to-apple comparison (e.g., all Mondays across 3 weeks) to identify day-of-week patterns

#### 5. Filter Application (`_apply_filters`)
**Location**: [backend/data_processor.py:660-769](backend/data_processor.py#L660-L769)

**Purpose**: Apply hierarchical filters based on staff mapping

**Filter Hierarchy**:
1. **保单号** (Policy Number) - Highest priority, unique identifier
2. **业务员** (Staff Name)
3. **三级机构** (L3 Institution) - Via staff mapping lookup
4. **团队** (Team) - Via staff mapping lookup
5. **是否续保** (Renewal Status)
6. **是否新能源** (New Energy Vehicle)
7. **是否过户车** (Transfer Vehicle)
8. **险种大类** (Product Category)
9. **吨位** (Tonnage Range)
10. **is_dianxiao** (Telemarketing) - Special logic: `终端来源 == '0110融合销售'`

**Critical Logic - Institution/Team Filtering**:
```python
# Institution filter: Find all staff belonging to this institution
staff_list = [extract_name(key)
              for key, info in mapping.items()
              if info['三级机构'] == filter_value]
df = df[df['业务员'].isin(staff_list)]
```

**Why Staff Mapping is Authoritative**: The `三级机构` field in raw data may be incorrect. Always use staff mapping to determine correct institution assignment.

**Policy Consistency Check**: When filtering by 保单号, enforce that institution/team filters match the mapped values for that policy's staff

#### 6. Staff Performance Distribution (`get_staff_performance_distribution`)
**Location**: [backend/data_processor.py:821-950](backend/data_processor.py#L821-L950)

**Purpose**: Analyze how many staff fall into each performance tier

**Performance Tiers** (by premium):
- <1万 (< 10,000)
- 1-2万 (10,000 - 20,000)
- 2-3万 (20,000 - 30,000)
- 3-5万 (30,000 - 50,000)
- ≥5万 (≥ 50,000)

**Supported Periods**:
- `day`: Single day
- `last7d`: Rolling 7 days
- `last30d`: Rolling 30 days

**Returns**:
```python
{
  'period': 'day',
  'period_label': '当日',
  'date_range': '2025-11-08',
  'distribution': [
    {'range': '<1万', 'count': 15, 'percentage': 37.5},
    {'range': '1-2万', 'count': 12, 'percentage': 30.0},
    ...
  ],
  'total_staff': 40,
  'total_premium': 1580000.50
}
```

**Use Case**: Identify underperforming staff or high-performers for management action

#### 7. Data Validation (`_validate_staff_mapping`, `_validate_policy_consistency`)
**Location**: [backend/data_processor.py:952-992](backend/data_processor.py#L952-L992), [771-819](backend/data_processor.py#L771-L819)

**Staff Mapping Validation**:
- Check if all staff in data exist in mapping file
- Return list of unmatched staff names
- Print warnings for unmapped staff (first 10)

**Policy Consistency Validation**:
- Verify that policy → staff → institution/team chain is consistent
- Compare data columns (团队, 三级机构) with mapping values
- Flag policies with mismatched institution/team assignments

**Returns**:
```python
{
  'unmatched_staff': ['姓名1', '姓名2', ...],
  'unmatched_count': 5,
  'policy_consistency': {
    'mismatch_policies': ['保单号1', '保单号2', ...],
    'mismatch_count': 3
  }
}
```

**When to Alert User**:
- High `unmatched_count`: Mapping file may be outdated
- High `mismatch_count`: Data quality issue, investigate source

## Pandas Best Practices for This Dataset

When writing Python analysis code, follow these patterns from `DataProcessor`:

### Date Handling
```python
# Always parse with error handling
df['投保确认时间'] = pd.to_datetime(df['投保确认时间'], errors='coerce')

# Filter by date range (use .date() for comparison)
mask = (df['投保确认时间'].dt.date >= start_date.date()) & \
       (df['投保确认时间'].dt.date <= end_date.date())
```

### Numeric Aggregation with Error Handling
```python
def sum_float(series):
    try:
        return float(series.sum())
    except Exception:
        return 0.0
```

### Staff Name Extraction (Regex)
```python
import re
match = re.search(r'[\u4e00-\u9fa5]+', staff_key)  # Extract Chinese characters
if match:
    name = match.group()
```

### Weekday Calculation
```python
weekday_map = ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
weekday_label = weekday_map[date_obj.weekday()]  # Monday=0, Sunday=6
```

### Missing Value Handling
```python
# For string columns
df = df.fillna('')

# For numeric columns
df[col] = pd.to_numeric(df[col], errors='coerce')  # Invalid → NaN
```

### Duplicate Removal
```python
# Keep last occurrence (latest data wins)
df = df.drop_duplicates(subset=['保单号', '投保确认时间'], keep='last')
```

## Common Pitfalls and Solutions

### Pitfall 1: Using Raw 三级机构 Field
**Problem**: Raw data may have incorrect institution assignments
**Solution**: ALWAYS use staff mapping to determine institution
```python
# ❌ Wrong
df[df['三级机构'] == '达州']

# ✅ Correct
staff_list = [name for key, info in mapping.items()
              if info['三级机构'] == '达州']
df[df['业务员'].isin(staff_list)]
```

### Pitfall 2: Ignoring Negative Premium
**Problem**: Negative premium represents cancellations/adjustments
**Solution**: Keep negative values, don't filter them out
```python
# ❌ Wrong
df = df[df['签单/批改保费'] > 0]

# ✅ Correct
# Include all values, negative premiums are valid business data
total_premium = df['签单/批改保费'].sum()  # May be negative
```

### Pitfall 3: Incorrect Weekday Alignment
**Problem**: Comparing different weekdays across weeks is meaningless
**Solution**: Use `weekday_index` to align same weekdays
```python
# Calculate days since period start
period_data['weekday_index'] = period_data['投保确认时间'].apply(
    lambda x: (x.date() - period_start.date()).days
)
```

### Pitfall 4: Forgetting to Apply Filters
**Problem**: KPI calculations without filters show global metrics
**Solution**: Always apply filters BEFORE aggregation
```python
# ✅ Correct order
df = self._apply_filters(df, filters)  # First
premium = df['签单/批改保费'].sum()      # Then aggregate
```

### Pitfall 5: Hardcoding Date Ranges
**Problem**: Analysis breaks when data range changes
**Solution**: Use anchor date and relative offsets
```python
# ❌ Wrong
start_date = pd.to_datetime('2025-10-01')

# ✅ Correct
anchor_date = df['投保确认时间'].max()
start_date = anchor_date - timedelta(days=29)  # Rolling 30 days
```

## API Response Format Standards

All KPI endpoints follow this structure:

### Success Response
```json
{
  "status": "success",
  "data": {
    "anchor_date": "YYYY-MM-DD",
    "premium": {...},
    "validation": {
      "unmatched_staff": [],
      "unmatched_count": 0,
      "policy_consistency": {...}
    }
  }
}
```

### Error Response
```json
{
  "status": "error",
  "error": "Error message",
  "code": "ERROR_CODE"  // Optional
}
```

### Validation Warnings
When `unmatched_count > 0` or `mismatch_count > 0`, frontend should display warnings but still show data

## Version Information

**Skill Version**: 3.0
**Last Updated**: 2025-11-08
**Mapping Table Version**: 20251104 (229 records)

**Changelog**:
- v3.0 (2025-11-08): Added comprehensive DataProcessor logic reference, Pandas best practices, common pitfalls
- v2.0 (2025-11-06): Initial structured skill documentation
- v1.0: Legacy documentation
