---
name: docx-to-pdf
description: Convert Word documents (DOCX files) to PDF format in batch. Use when the user uploads one or more Word documents and asks to convert them to PDF, export to PDF, or save as PDF. The tool processes multiple files efficiently and provides a summary of successful conversions and any failures.
---

# DOCX to PDF Converter

## Overview

This skill converts Word documents (.docx files) to PDF format using LibreOffice's conversion engine. It's designed for batch processing - you can convert multiple files at once without manually opening each file.

**Key Features:**
- Batch conversion of multiple DOCX files
- Preserves original filenames (changes only the extension)
- Provides clear summary of successful and failed conversions
- Uses LibreOffice's conversion engine for high-quality output

## Workflow

### Step 1: Run the conversion script

Execute the conversion script with all uploaded DOCX files:

```bash
python3 /mnt/skills/user/docx-to-pdf/scripts/convert_to_pdf.py /mnt/user-data/uploads/*.docx
```

Or for specific files:

```bash
python3 /mnt/skills/user/docx-to-pdf/scripts/convert_to_pdf.py /mnt/user-data/uploads/file1.docx /mnt/user-data/uploads/file2.docx
```

The script automatically:
- Converts each DOCX file to PDF using LibreOffice
- Preserves the original filename (e.g., `contract.docx` → `contract.pdf`)
- Saves all PDFs directly to `/mnt/user-data/outputs/`
- Tracks successful conversions and failures
- Provides a summary at the end

### Step 2: Present files to user

The script saves PDFs directly to `/mnt/user-data/outputs/`, so use the `present_files` tool to share them with the user along with the conversion summary.

## Output

- **PDF files**: Same name as original DOCX, saved in `/mnt/user-data/outputs/`
- **Console output**: Summary showing:
  - Total files processed
  - Number of successful conversions
  - List of any files that failed to convert

## Error Handling

If a file fails to convert:
- The script continues processing remaining files
- The failed filename is logged
- At the end, all failures are reported to the user
- Common failure reasons: corrupted files, unsupported features, file access issues

## Technical Notes

- Uses LibreOffice headless mode for conversion
- Conversion quality: ~90-95% identical to Word's "Save as PDF"
- Works with standard DOCX formatting including:
  - Text, paragraphs, headings
  - Tables and borders
  - Images and shapes
  - Basic fonts and styles
- May have minor differences with:
  - Complex embedded objects
  - Rare font combinations
  - Advanced Word-specific features

## Limitations

- Only processes `.docx` files (not `.doc` or other formats)
- Requires LibreOffice to be installed on the system
- Output quality may differ slightly from Microsoft Word's native PDF export
- Very large files (>100MB) may take longer to process
