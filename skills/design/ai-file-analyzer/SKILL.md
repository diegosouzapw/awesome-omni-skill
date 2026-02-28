---
name: ai-file-analyzer
description: Analyze Adobe Illustrator (.ai) files to extract design information including text content, fonts, color palettes, vector paths, and generate high-resolution preview images. Use when analyzing logo files, design assets, or any Adobe Illustrator documents that need programmatic inspection.
---

# Adobe Illustrator File Analyzer

Extract comprehensive design information from Adobe Illustrator (.ai) files including text, fonts, colors, and visual previews.

## When to Use This Skill

Use this skill when:
- Analyzing existing logo files (.ai) to understand design elements
- Extracting brand colors from design files
- Identifying fonts used in artwork
- Generating preview images from Illustrator files without opening Adobe Illustrator
- Understanding the structure and content of .ai files for design decisions
- Preparing design specifications or style guides from source files

## How It Works

Adobe Illustrator files (.ai) have been PDF-based since Creative Suite versions. This means .ai files can be read and analyzed using PDF processing libraries like PyMuPDF, without requiring Adobe Illustrator itself.

The analyzer extracts:
1. **Text content** - All text elements and labels
2. **Font information** - Font families and sizes used
3. **Color palette** - All colors with usage frequency
4. **Preview images** - High-resolution PNG previews (300 DPI)
5. **Metadata** - Artboard dimensions, creation info
6. **Design complexity** - Vector paths, images, structure

## Quick Start

### Install Dependencies

```bash
pip install pymupdf pillow
```

### Analyze AI File

```bash
python scripts/analyze_ai_file.py <path/to/logo.ai> --output-dir ./analysis
```

## Output Structure

The script creates the following output:

```
analysis/
├── analysis_report.md      # Human-readable summary
├── analysis_data.json      # Complete structured data
└── previews/               # High-resolution previews
    ├── artboard_1_preview.png
    ├── artboard_2_preview.png
    └── ...
```

## Usage in Claude Code

When a user provides an .ai file for analysis:

1. **Check dependencies**: Verify PyMuPDF and Pillow are installed
2. **Run analyzer**: Execute `scripts/analyze_ai_file.py` with the .ai file path
3. **Review outputs**: Read the markdown report and JSON data
4. **Analyze previews**: Inspect generated preview images
5. **Extract insights**: Identify key design elements for decision-making

Example workflow:

```bash
# Step 1: Analyze the AI file
python scripts/analyze_ai_file.py "existing_logo.ai" --output-dir ./logo_analysis

# Step 2: Read the summary report
cat ./logo_analysis/analysis_report.md

# Step 3: Check the color palette
cat ./logo_analysis/analysis_data.json | grep -A 10 '"colors"'

# Step 4: View preview images
ls ./logo_analysis/previews/
```

## Extracted Information

### Text Content

- All text elements from all artboards
- Unique text strings (brand names, slogans, etc.)
- Text organized by artboard

Example output:
```json
{
  "unique_texts": ["COMPANY NAME", "Est. 2022", "Quality Products"],
  "total_text_elements": 5
}
```

### Font Information

- Font family names
- Font sizes used
- All unique font/size combinations

Example output:
```json
[
  ["Helvetica-Bold", 48.0],
  ["Arial-Regular", 12.0],
  ["CustomFont-Light", 24.0]
]
```

### Color Palette

- Hex color codes (#RRGGBB)
- Usage frequency for each color
- Sorted by most-used colors first

Example output:
```json
[
  {"color": "#000000", "occurrences": 25},
  {"color": "#ff6600", "occurrences": 12},
  {"color": "#ffffff", "occurrences": 8}
]
```

### Artboard Dimensions

- Width and height in points (pt)
- Width and height in millimeters (mm)
- Rotation angle
- Image count

Example output:
```json
{
  "artboard_number": 1,
  "width": 612.0,
  "height": 792.0,
  "width_mm": 216.0,
  "height_mm": 279.4
}
```

### Preview Images

- PNG format with transparency
- 300 DPI resolution (print-quality)
- One image per artboard

## Advanced Features

### High-Resolution Previews

The analyzer generates 300 DPI previews, suitable for:
- Print quality inspection
- Design presentations
- Web mockups
- Client presentations

### Color Frequency Analysis

Colors are ranked by usage, helping identify:
- Primary brand colors (most used)
- Accent colors (less frequent)
- Unused colors in palette

### Design Complexity Score

The analyzer calculates complexity based on:
- Number of vector paths
- Number of text elements
- Number of embedded images

Higher scores indicate more complex designs.

## Integration with Logo Design Workflows

### Analyzing Existing Brand Logos

When creating a new logo based on existing brand assets:

1. **Extract brand colors**:
   ```bash
   python scripts/analyze_ai_file.py "old_logo.ai" --output-dir ./brand_analysis
   # Review analysis_data.json for color palette
   ```

2. **Identify typography**:
   ```bash
   # Check fonts section in analysis_report.md
   cat ./brand_analysis/analysis_report.md | grep -A 20 "## Fonts Used"
   ```

3. **Review design elements**:
   ```bash
   # View preview images
   open ./brand_analysis/previews/artboard_1_preview.png
   ```

4. **Use findings for new design**:
   - Maintain brand colors from palette
   - Consider existing typography style
   - Reference design complexity level

### Brand Guidelines Extraction

For logo design contests requiring brand adherence:

1. Analyze existing logo file
2. Extract official brand colors → Use in new design
3. Identify font styles → Match typography approach
4. Review design simplicity → Maintain brand consistency

## Troubleshooting

### File Not Recognized

If the analyzer fails to open the .ai file:

```bash
# Verify file integrity
file logo.ai

# Check if it's truly a PDF-based AI file
pymupdf logo.ai
```

Older .ai files (pre-CS) may use EPS format and require conversion.

### Missing Color Information

If few colors are detected:

- Colors may be defined as vector fill/stroke (not text)
- Consider manual inspection of preview images
- Some colors might be in embedded images

### Large File Size

For very large .ai files (>50MB):

- Processing may take longer
- Preview generation is memory-intensive
- Consider processing individual artboards

## Technical Details

### Why PyMuPDF?

PyMuPDF (fitz) is ideal for .ai files because:
- ⚡ Fast performance (C++ backend)
- 📄 Native PDF handling (AI files are PDF-based)
- 🖼️ High-quality image rendering
- 💾 Low memory footprint
- 🔧 No external dependencies (like Ghostscript)

### AI File Format Background

Since Adobe Creative Suite (CS):
- .ai files contain embedded PDF
- PDF portion is readable by standard PDF tools
- Proprietary AI data is in separate layer
- PyMuPDF accesses the PDF layer

### Limitations

What the analyzer **can** extract:
- ✅ Text content and fonts
- ✅ Basic color information
- ✅ Artboard dimensions
- ✅ Visual previews (raster)
- ✅ Embedded images

What the analyzer **cannot** extract:
- ❌ Editable vector paths (proprietary format)
- ❌ Layer structure (AI-specific)
- ❌ Blend modes and effects (AI-specific)
- ❌ Symbols and brushes (AI-specific)

For full editing capabilities, Adobe Illustrator is still required.

## Example Outputs

### Logo Analysis for Contest

```markdown
# AI File Analysis Report

## File Information
- **Filename**: company_logo.ai
- **Artboards**: 2
- **Creator**: Adobe Illustrator 27.0
- **Created**: 2024-08-02

## Artboards
### Artboard 1
- **Dimensions**: 500x500 pt (176.39x176.39 mm)
- **Images**: 0

### Artboard 2
- **Dimensions**: 1000x300 pt (352.78x105.83 mm)
- **Images**: 1

## Text Content
1. COMPANY NAME
2. Innovation Through Technology

## Fonts Used
- Montserrat-Bold (48.0pt)
- Montserrat-Regular (18.0pt)

## Color Palette
- `#1a1a1a` (used 15 times)
- `#ff6b35` (used 8 times)
- `#ffffff` (used 5 times)

## Design Analysis
- **Contains text**: Yes
- **Contains images**: Yes
- **Contains vector paths**: Yes
- **Complexity score**: 127
```

## Integration Tips

### For Logo Design Projects

After analyzing an existing brand logo:

1. **Color consistency**: Use extracted hex codes in design tools
2. **Typography reference**: Match font style and weight
3. **Size reference**: Use artboard dimensions for aspect ratios
4. **Visual comparison**: Compare new designs against preview images

### For Brand Guidelines

After analyzing design assets:

1. **Document colors**: Create palette from frequency data
2. **Typography specs**: List fonts with sizes and weights
3. **Usage examples**: Use preview images in guidelines
4. **File specifications**: Include dimension data

### For Design Handoff

Before submitting final designs:

1. Analyze your own .ai file
2. Verify all required elements present
3. Check color consistency
4. Generate previews for review
5. Include analysis report with submission
