---
name: anonymise
description: Anonymise CSV files by removing personal identifying information and adding datetime stamps. Use when user wants to process a new CSV file or strip PII from data.
---

# CSV Anonymiser

When the user wants to anonymise a CSV file or process new data:

## 1. Check .gitignore

First, verify that CSV files are excluded from git:

```bash
cat .gitignore
```

If `*.csv` is not present, add it:

```bash
echo "*.csv" >> .gitignore
```

## 2. Identify the CSV File

The script can auto-detect new CSV files:
- **Auto-detection**: Files without a timestamp pattern (YYYYMMDD_HHMMSS) in their name
- **New files**: Any CSV file that hasn't been processed yet
- If multiple new files exist, the script will list them and ask user to specify which one

## 3. Remove Personal Identifying Information

Columns to remove are configured in `~/.agents/skills/anonymise/config.txt`:
- One column header per line
- Lines starting with `#` are comments
- Default includes `"Record - Airtable ID"` (PII)

Process the file:
1. Read the CSV file
2. Remove all columns listed in `~/.agents/skills/anonymise/config.txt`
3. Keep all other columns intact
4. Preserve the header row (minus the removed columns)

## 4. Add Datetime Stamp

The script will:
1. Create a new file with format: `YYYYMMDD_HHMMSS.csv` (just the timestamp)
2. Delete the original file

Example: `ILR data 25_26-Social value data for Dingley.csv` becomes `20231216_143022.csv`

## 5. Run the Anonymisation Script

Execute the Python script located in the same directory as this skill:

**Auto-detect mode** (finds CSV files without timestamps):
```bash
python3 ~/.agents/skills/anonymise/anonymise.py
```

**Specific file mode**:
```bash
python3 ~/.agents/skills/anonymise/anonymise.py "filename.csv"
```

If multiple new files are found in auto-detect mode, the script will list them and you'll need to specify which one to process.

## 6. Confirm Completion

After processing:
1. Confirm the anonymised file was created with timestamp
2. Confirm the original file was removed
3. Confirm .gitignore includes `*.csv`
4. Remind user that CSV files won't be committed to git

## Notes

- Uses British English spelling throughout
- Handles UTF-8 BOM encoding (`utf-8-sig`) for compatibility
- Preserves all data except the first column
- Assumes consistent CSV structure across files
- Files are automatically excluded from version control
