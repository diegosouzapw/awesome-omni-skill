---
name: Data Cleaner
description: Use this skill when the user needs to analyze, clean, or prepare datasets. Helps with listing columns, detecting data types (text, categorical, ordinal, numeric), identifying data quality issues, and cleaning values that don't fit expected patterns. Invoke when users mention data cleaning, data quality, column analysis, type detection, or preparing datasets.
allowed-tools: Read, Bash, Grep, Glob
---

# Data Cleaning Skill

This skill helps analyze and clean datasets by detecting data types, identifying quality issues, and suggesting or applying corrections.

## Core Capabilities

1. **Sample Rows**: View random sample of rows from a dataset using pandas.sample()
2. **Column Analysis**: List all columns with basic statistics and sample values
3. **Type Detection**: Automatically detect if columns are:
   - Numeric (integer, float)
   - Categorical (limited unique values)
   - Ordinal (ordered categories)
   - Text (free-form text)
   - DateTime
   - Boolean
4. **Data Quality Reports**: Comprehensive quality analysis with severity levels and completeness scores
5. **Value Mapping Generation**: Auto-generate standardization functions for categorical data
6. **Value Cleaning**: Fix common issues like extra whitespace, inconsistent casing, invalid values
7. **Validation Reports**: Compare before/after cleaning to verify transformations

## Instructions

When the user requests data cleaning assistance:

1. **Identify the dataset**: Ask for the file path if not provided
2. **Generate quality report**: Use `scripts/data_quality_report.py` for comprehensive quality analysis
3. **Analyze columns**: Use `scripts/analyze_columns.py` to get an overview of all columns
4. **Detect types**: Use `scripts/detect_types.py` to determine the data type of each column
5. **Generate value mappings**: Use `scripts/value_mapping_generator.py` for categorical columns needing standardization
6. **Present findings**: Show the user:
   - Data quality grade and issues
   - Column names and detected types
   - Suggested value mappings
   - Sample problematic values
7. **Suggest fixes**: Recommend cleaning strategies based on issues found
8. **Apply cleaning**: If user approves, use `scripts/clean_values.py` to fix issues
9. **Validate results**: Use `scripts/validation_report.py` to compare before/after and confirm changes

## Supported File Formats

All scripts support multiple file formats with automatic detection:
- **CSV** (.csv) - Comma-separated values
- **Excel** (.xlsx, .xls) - Microsoft Excel files
- **JSON** (.json) - JSON arrays or objects
- **JSONL** (.jsonl) - JSON Lines format (one JSON object per line)

File format is auto-detected from the extension. You can also explicitly specify the format using the `--format` parameter.

## Using the Python Scripts

All scripts should be run using `uv` for fast, dependency-managed execution:

```bash
uv run python scripts/<script_name>.py [arguments]
```

Or use `uvx` to run scripts with automatic dependency installation:

```bash
uvx --from . python scripts/<script_name>.py [arguments]
```

**Note**: The first time you run a script, `uv` will automatically install the required dependencies (pandas, numpy, openpyxl) in an isolated environment. Subsequent runs will be much faster.

### sample_rows.py
View a random sample of rows from a dataset.

**Usage**:
```bash
uv run python scripts/sample_rows.py <file_path> [--n 10] [--format csv|excel|json|jsonl] [--output table|json|csv] [--seed SEED]
```

**Options**:
- `--n`: Number of rows to sample (default: 10)
- `--format`: Input file format - csv, excel, json, or jsonl (auto-detected if not specified)
- `--output`: Output format - table (human-readable), json, or csv (default: table)
- `--seed`: Random seed for reproducibility (optional)

**Output**: Random sample of rows in the specified format

**Examples**:
```bash
# Sample 5 random rows from CSV
python scripts/sample_rows.py data.csv --n 5

# Sample from JSON Lines file
python scripts/sample_rows.py data.jsonl --n 10

# Sample with reproducible results
python scripts/sample_rows.py data.csv --n 10 --seed 42

# Output as JSON
python scripts/sample_rows.py data.xlsx --n 20 --output json
```

### analyze_columns.py
Analyzes all columns in a dataset and provides summary statistics.

**Usage**:
```bash
uv run python scripts/analyze_columns.py <file_path> [--format csv|excel|json|jsonl]
```

**Output**: JSON with column names, types, null counts, unique counts, and sample values

**Examples**:
```bash
# Analyze CSV file
python scripts/analyze_columns.py customers.csv

# Analyze JSONL file
python scripts/analyze_columns.py events.jsonl
```

### detect_types.py
Detects the semantic type of each column (numeric, categorical, ordinal, text, datetime).

**Usage**:
```bash
uv run python scripts/detect_types.py <file_path> [--format csv|excel|json|jsonl]
```

**Output**: JSON mapping columns to detected types with confidence scores

**Examples**:
```bash
# Detect types in CSV
python scripts/detect_types.py data.csv

# Detect types in JSON file
python scripts/detect_types.py data.json
```

### clean_values.py
Cleans specific columns based on detected issues.

**Usage**:
```bash
uv run python scripts/clean_values.py <input_file> <output_file> [--operations json_string] [--input-format csv|excel|json|jsonl] [--output-format csv|excel|json|jsonl]
```

**Options**:
- `--operations`: JSON string defining cleaning operations
- `--input-format`: Input file format (auto-detected if not specified)
- `--output-format`: Output file format (auto-detected if not specified)

**Operations JSON format**:
```json
{
  "column_name": {
    "operation": "trim|lowercase|uppercase|remove_special|fill_missing|convert_type",
    "params": {}
  }
}
```

**Examples**:
```bash
# Clean CSV file and output as CSV
python scripts/clean_values.py data.csv cleaned.csv --operations '{"name":{"operation":"trim"}}'

# Clean JSONL file and convert to JSON
python scripts/clean_values.py logs.jsonl cleaned.json --input-format jsonl --output-format json --operations '{"status":{"operation":"lowercase"}}'

# Clean JSON and output as JSONL
python scripts/clean_values.py data.json output.jsonl --output-format jsonl
```

### data_quality_report.py
Generates a comprehensive data quality report with severity levels and completeness scores.

**Usage**:
```bash
uv run python scripts/data_quality_report.py <file_path> [--format csv|excel|json|jsonl] [--output report.json]
```

**Output**: JSON report with:
- Overall quality grade (A-F)
- Per-column completeness scores
- Missing values analysis
- Formatting issues
- Outliers detection
- Data type consistency checks

**Examples**:
```bash
# Generate quality report for CSV
python scripts/data_quality_report.py data.csv --output report.json

# Generate report for JSONL file
python scripts/data_quality_report.py logs.jsonl
```

### value_mapping_generator.py
Auto-generates standardization mappings and Python functions for categorical columns.

**Usage**:
```bash
uv run python scripts/value_mapping_generator.py <file_path> [--column COLUMN] [--threshold 20] [--format csv|excel|json|jsonl] [--output-functions functions.py]
```

**Output**: JSON with:
- Suggested value mappings
- Groups of similar values
- Auto-generated Python standardization functions
- Before/after value counts

**Options**:
- `--column`: Analyze specific column only
- `--threshold`: Max unique values to consider categorical (default: 20)
- `--format`: File format - csv, excel, json, or jsonl (auto-detected if not specified)
- `--output-functions`: Write Python functions to file

**Examples**:
```bash
# Generate mappings for all categorical columns
python scripts/value_mapping_generator.py survey.csv

# Generate mappings for specific column in JSONL file
python scripts/value_mapping_generator.py events.jsonl --column user_type
```

### validation_report.py
Compares original and cleaned datasets to validate transformations.

**Usage**:
```bash
uv run python scripts/validation_report.py <original_file> <cleaned_file> [--format csv|excel|json|jsonl] [--output validation.json]
```

**Output**: JSON report with:
- Transformation examples for each column
- Data loss analysis
- Before/after distribution comparisons
- Validation status (pass/review_needed)
- Recommendations

**Examples**:
```bash
# Validate CSV cleaning
python scripts/validation_report.py original.csv cleaned.csv

# Validate JSONL cleaning
python scripts/validation_report.py original.jsonl cleaned.jsonl --output validation.json
```

## Workflow Examples

### Basic Workflow
1. User: "I need to clean my customer data"
2. Get file path from user
3. Run `sample_rows.py` to show user a preview of their data
4. Run `data_quality_report.py` to assess overall quality
5. Run `analyze_columns.py` to see all columns
6. Run `detect_types.py` to determine types
7. Present findings and ask user which columns to clean
8. Run `clean_values.py` with appropriate operations
9. Run `validation_report.py` to verify changes
10. Confirm cleaning completed and show summary

### Advanced Workflow (with auto-generated functions)
1. User: "Generate cleaning functions for my survey data"
2. Run `data_quality_report.py` for quality overview
3. Run `value_mapping_generator.py` for categorical columns
4. Show user the generated standardization functions
5. User can copy functions into their own cleaning script
6. Apply cleaning using the generated functions
7. Validate with `validation_report.py`

## Best Practices

- Always show sample values before suggesting changes
- Explain why certain types were detected
- Ask for confirmation before modifying data
- Create backups or save to new files when cleaning
- Support all file formats: CSV, Excel, JSON, and JSONL
- For JSON/JSONL files, pandas expects records-oriented format (list of objects)
- JSONL format is ideal for streaming or large datasets (one JSON object per line)
- You can convert between formats using clean_values.py with `--input-format` and `--output-format`
- Provide clear summaries of changes made
