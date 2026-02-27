---
name: pdf-utilities
version: "1.0.0"
category: data
description: "Read, extract, edit, and manipulate PDF documents including table extraction, page manipulation, fillable forms, and comments."
globs:
  - src/assetutilities/modules/pdf_utilities/**
alwaysApply: false
---

# PDF Utilities Skill

## Overview

This skill provides comprehensive PDF processing capabilities including reading PDFs with multiple library backends (tabula, camelot, PyPDF2), extracting tables to DataFrames, editing/extracting page ranges, handling fillable forms, and managing PDF comments. All operations are driven by YAML configuration.

## Key Components

### ReadPDF Class (read_pdf.py)
Multi-backend PDF reading with table extraction:
- `read_pdf(cfg, file_index)` - Route to appropriate backend based on config
- `from_pdf_tabula(cfg, file_index)` - Extract tables using tabula-py
- `from_pdf_camelot(cfg, file_index)` - Extract tables using camelot
- `from_pdf_PyPDF2(cfg, file_index)` - Read PDF pages using PyPDF2

### EditPDF Class (edit_pdf.py)
PDF page manipulation and extraction:
- `edit_pdf(cfg, file_index)` - Process PDF files based on configuration
- `from_pdf_PyPDF2(cfg, file_index)` - Extract page ranges to new PDF files
- `process_cfg_files(cfg)` - Process multiple PDF files from config

### Additional Modules
- `fillable_pdf.py` - Handle fillable PDF forms (fill fields, extract data)
- `pdf_comments.py` - Add, read, and manipulate PDF annotations
- `pdf_reports.py` - Generate PDF reports from data

## Usage Patterns

### Table Extraction Configuration
```yaml
pdf:
  io: pdf_read
  reader: tabula  # or camelot, PyPDF2
  files:
    - path: "input.pdf"
      pages: [1, 2, 3]
      area: [0, 0, 100, 100]  # Optional: specific region
```

### Page Extraction Configuration
```yaml
pdf:
  io: pdf_edit
  files:
    - path: "source.pdf"
      output: "extracted_pages.pdf"
      page_start: 1
      page_end: 5
```

### Common Workflows
1. **Table Extraction**: PDF → tabula/camelot → DataFrame → CSV/Excel
2. **Page Extraction**: Multi-page PDF → Extract range → New PDF
3. **Form Processing**: Fillable PDF → Fill fields → Save completed form
4. **Report Generation**: DataFrame → Generate styled PDF report

## Module Location
- Read: `src/assetutilities/modules/pdf_utilities/read_pdf.py`
- Edit: `src/assetutilities/modules/pdf_utilities/edit_pdf.py`
- Forms: `src/assetutilities/modules/pdf_utilities/fillable_pdf.py`
- Comments: `src/assetutilities/modules/pdf_utilities/pdf_comments.py`
- Reports: `src/assetutilities/modules/pdf_utilities/pdf_reports.py`

## Dependencies
- PyPDF2 (PDF reading and manipulation)
- tabula-py (table extraction with Java backend)
- camelot-py (table extraction)
- reportlab (PDF generation, optional)
