---
name: data-management
version: "1.0.0"
category: data
description: "Comprehensive DataFrame loading, filtering, transformation, and data pipeline management from Excel, CSV, and multiple sources with YAML-driven configuration."
globs:
  - src/assetutilities/common/data_management.py
  - src/assetutilities/common/data.py
alwaysApply: false
---

# Data Management Skill

## Overview

This skill provides comprehensive data management capabilities including loading data from Excel/CSV files, filtering DataFrames by column values, applying transformations, managing data arrays, and building data pipelines. All operations are configurable via YAML files for reproducible data workflows.

## Key Components

### DataManagement Class (data_management.py)
High-level data pipeline management:
- `router(cfg)` - Route data operations based on configuration
- `get_df_data(cfg)` - Load DataFrame from configuration
- `get_df_array_from_cfg(cfg)` - Load multiple DataFrames as array
- `get_filtered_df(data_set_cfg, df)` - Apply filters to DataFrame
- `get_transformed_df(data_set_cfg, df)` - Apply transformations to DataFrame

### ReadFromExcel Class (data.py)
Excel file reading with sheet selection:
- `from_xlsx(cfg, file_index)` - Read Excel files with configurable sheet selection
- Supports multiple sheets, header row configuration, data range selection

### ReadFromCSV Class (data.py)
CSV file reading with encoding detection:
- `to_df(cfg, file_index)` - Read CSV to DataFrame
- Automatic encoding detection with chardet
- Configurable delimiter, header options

### ReadData Class (data.py)
Advanced data reading operations:
- `df_filter_by_column_values(cfg, df, file_index)` - Filter DataFrame by column values
- `xlsx_to_df_by_keyword_search(cfg)` - Read Excel by keyword-based row search
- `get_data_from_xlsx_and_csv(cfg)` - Unified Excel/CSV reading

## Usage Patterns

### Data Loading Configuration
```yaml
data:
  files:
    - path: "data.xlsx"
      sheet_name: "Sheet1"
      header_row: 0
      columns: ["A", "B", "C"]
```

### Filtering Configuration
```yaml
data:
  filter:
    column: "status"
    values: ["active", "pending"]
    operator: "in"  # in, equals, gt, lt, contains
```

### Transformation Configuration
```yaml
data:
  transform:
    - type: "rename"
      mapping: {"old_col": "new_col"}
    - type: "add_column"
      name: "calculated"
      expression: "col_a + col_b"
```

### Common Workflows
1. **Excel Pipeline**: Load Excel → Filter rows → Transform columns → Export
2. **Multi-Source Merge**: Load CSV + Excel → Merge on key → Validate → Save
3. **Data Validation**: Load data → Apply filters → Check constraints → Report
4. **Batch Processing**: Config with file list → Process each → Aggregate results

## Module Location
- Pipeline: `src/assetutilities/common/data_management.py`
- Readers: `src/assetutilities/common/data.py`

## Dependencies
- pandas (DataFrame operations)
- openpyxl (Excel reading)
- chardet (encoding detection)
- numpy (numerical operations)
