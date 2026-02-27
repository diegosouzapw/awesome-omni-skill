---
name: pdf-smart-split
description: Intelligent PDF splitting skill. Split multi-page scanned PDF documents into separate files based on content analysis. Features: (1) Convert PDF to high-quality images (2) Analyze and intelligently name images by content (3) Auto-detect and remove duplicate pages (4) Identify and reorder scrambled pages (5) Merge related images into separate PDFs by content (6) Generate summary reports. Use for: bond filing documents, application materials, contracts and agreements that need intelligent content-based splitting.
---

# PDF Smart Split Skill

Split multi-page scanned PDF documents into separate files based on content analysis. Supports duplicate detection, page reordering, and intelligent naming.

## Workflow

### 1. PDF to Images
```python
from pdf2image import convert_from_path
images = convert_from_path(pdf_path, dpi=200, thread_count=4)
```
- DPI=200 for quality
- Multi-threading for speed

### 2. Duplicate Detection
```python
import hashlib
def get_image_hash(path):
    with open(path, 'rb') as f:
        return hashlib.md5(f.read()).hexdigest()
```
- Use MD5 hash comparison
- Record duplicates for summary

### 3. Content Analysis and Naming
Use `view_file` tool to analyze each image:
- File type (cover, content, signature page)
- Document category (application, agreement, statement)
- Page sequence number

Naming format: `sequence_filetype_description.jpg`

### 4. Page Reordering
Analyze content continuity:
- Check table sequence numbers
- Check chapter numbering progression
- Ensure signature pages follow content

Reorder by logical content, not physical order.

### 5. Merge Images to PDF
```python
from PIL import Image
def images_to_pdf(image_paths, output_pdf):
    images = [Image.open(p).convert('RGB') for p in image_paths]
    images[0].save(output_pdf, "PDF", save_all=True, append_images=images[1:])
```
- Merge signature pages with their content
- Combine continuous content into single files

### 6. Generate Summary
Summary should include:
- Source file info (name, original pages)
- Duplicate removal records (if any)
- Page reorder records (if any)
- Split file list (name, original pages, page count, description)
- Output directory paths

## Script Usage

### Full Processing
```bash
python scripts/convert_dedup.py <pdf_path> [output_dir]
python scripts/merge_images.py <image1> <image2> ... <output.pdf>
```

## Output Structure
```
filename_images/     # Intelligently named images
filename_split_PDF/  # Content-grouped PDFs
```

## Requirements
- Install: `pdf2image`, `Pillow`
- Windows needs `poppler`: `conda install -c conda-forge poppler`
