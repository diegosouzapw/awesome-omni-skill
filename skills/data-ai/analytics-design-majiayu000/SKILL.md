---
name: analytics-design
description: Design Looker Studio reports and dashboards. Use when creating Looker Studio reports, designing data sources, or writing visualization specifications. Trigger on "Looker Studio", "レポート設計", "ダッシュボード".
---

# Analytics Design

## Workflow

1. **Purpose**: Define what to visualize (link to design doc with KPIs)
2. **Data Sources**: Design SQL queries and field definitions
3. **Report Structure**: Define pages and their purposes
4. **Visualization**: Specify charts with dimensions, metrics, filters

## Templates

See [references/looker-studio-template.md](references/looker-studio-template.md) for report design template.

## Looker Studio Best Practices

### Data Source Design

- One data source per analytical purpose
- Pre-aggregate in SQL for performance
- Include bucket fields for distribution analysis
- Include sort-order fields for proper chart ordering
- Descriptive data source names

### Report Structure

- Separate pages by time granularity (daily/monthly)
- Group related metrics per page
- Consistent filter scopes within pages

### Visualization

- Scorecards for KPI current values
- Time series for trends
- Stacked area/bar for category breakdown over time
- Tables for detailed data
- Add comparison periods where relevant
