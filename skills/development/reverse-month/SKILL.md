---
name: reverse-month
description: Convert dates from various formats to "reverse month" format (YYYY-MM), which is the ISO date format containing only the year and month components. Use when users need to extract year-month from dates, standardize month formats, or prepare dates for monthly aggregations and reporting.
---

# Reverse Month Converter

## Overview

Convert dates from various human-readable formats into standardized "reverse month" format (YYYY-MM). This skill handles flexible date input parsing and provides consistent, machine-readable month output by extracting only the year and month components from a full date.

The reverse month format is the year and month portion of an ISO date (YYYY-MM), making it ideal for monthly reporting, time-series aggregations, and calendar operations.

## Usage

Use the `convert_month.py` script to convert any date string to YYYY-MM format:

```bash
python scripts/convert_month.py "21 Oct 2025"
# Output: 2025-10
```

### Supported Input Formats

The script accepts a wide variety of date formats, including:

- **Month-day-year**: "21 Oct 2025", "October 21, 2025", "Oct 21, 2025"
- **Slash-separated**: "21/10/2025", "10/21/2025"
- **Dash-separated**: "21-10-2025", "2025-10-21"
- **ISO format**: "2025-10-21"
- **Other common formats**: Most standard date representations

### Examples

```bash
# Human-readable format
python scripts/convert_month.py "21 Oct 2025"
# → 2025-10

# Full month name
python scripts/convert_month.py "October 21, 2025"
# → 2025-10

# Slash format
python scripts/convert_month.py "21/10/2025"
# → 2025-10

# Already in ISO format
python scripts/convert_month.py "2025-10-21"
# → 2025-10

# Short format
python scripts/convert_month.py "Oct 2025"
# → 2025-10
```

### Error Handling

If the date string cannot be parsed, the script returns an error message:

```bash
python scripts/convert_month.py "invalid date"
# Error: Unable to parse date: 'invalid date'. Error: Unknown string format: invalid date
```

## Common Use Cases

- Monthly reporting and aggregations
- Calendar month filtering
- Time-series data grouping by month
- Standardizing month identifiers across different date formats
- Extracting billing or subscription periods

## Relationship to reverse-date Skill

This skill builds upon the reverse-date skill concept. While reverse-date converts dates to full ISO format (YYYY-MM-DD), reverse-month extracts only the year-month portion (YYYY-MM), providing a coarser granularity suitable for monthly operations.

## Implementation Details

The script uses Python's `dateutil.parser` library for flexible date parsing, which handles many common date formats automatically. The output is always in ISO 8601 year-month format (YYYY-MM).
```