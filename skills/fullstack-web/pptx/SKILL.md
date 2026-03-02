---
name: pptx
description: "Generates PowerPoint presentations from templates with consistent styling across tables, charts, and Mermaid diagrams. Use when creating PPTX files, working with template-based presentations, or applying unified styling from style.yaml. Supports Python, R, and native PowerPoint shapes."
---

# PPTX Skill (Template-based)

## 0. Scope & Prerequisites

This skill is **template-first** and uses **style.yaml as Single Source of Truth**.

- ✅ **ALWAYS reference TEMPLATE.md for layout selection** - different layouts have different placeholder indices
- ✅ Always begin with `template.pptx`
- ✅ Extract styles from `Chart.crtx` and `template.pptx` into `style.yaml`
- ✅ Use consistent styling across Python, R, and Mermaid

---

## 1. Working Directory Structure

To keep the skill directory clean, all working files should be placed in a separate project directory:

```
{project}/
└── powerpoint/          # All PowerPoint-related files (auto-created)
    ├── outline.md       # Content definition (input, human-edited)
    ├── generate_*.py    # Generation script (AI-created, preserved for reference)
    ├── output.pptx      # Final output
    └── processing/      # Temporary files and generation logs (safe to delete)
        ├── snapshot/    # Generation-time snapshots (for audit/reproducibility)
        │   ├── template.pptx    # Template used at generation time
        │   ├── template.crtx    # Chart template used at generation time
        │   ├── style.yaml       # Style config used at generation time
        │   ├── TEMPLATE.md      # Layout documentation used at generation time
        │   └── timestamp.txt    # Generation timestamp and skill version
        ├── pptx_generation.log  # Debug logs (auto-generated)
        ├── charts/      # R-generated SVG/PNG (optional)
        ├── diagrams/    # Mermaid-generated SVG (optional)
        └── temp/        # Other temporary files (optional)
```

### Directory Roles

- **powerpoint/outline.md** - Markdown file defining slide content and structure (human-edited input, high-level design)
- **powerpoint/generate_*.py** - Python script that generates the presentation (AI-created from outline.md, preserved for reference)
- **powerpoint/output.pptx** - Final generated PowerPoint presentation (output)
- **powerpoint/processing/** - Temporary/intermediate files and logs (can be safely deleted, but useful for audit)
  - **snapshot/** - Snapshots of templates/styles used at generation time (for reproducibility and audit)
  - **pptx_generation.log** - Detailed debug and error logs (auto-generated)

### Setup

No manual setup required! The generation script automatically:
- Creates `powerpoint/processing/` directory structure
- Copies templates to `processing/snapshot/` for audit logging
- Initializes logging to `processing/pptx_generation.log`

Optional: Create subdirectories for R charts or Mermaid diagrams if needed:
```bash
mkdir -p powerpoint/processing/{charts,diagrams,temp}
```

### Logging (Automatic)

All PPTX generation activities are automatically logged to `powerpoint/processing/pptx_generation.log`:

- **Auto-detection**: Finds `powerpoint/processing/` directory automatically
- **Console**: Shows warnings/errors only
- **Log file**: Records all debug information, validation errors, and styling issues

No manual setup required - logging initializes on first use of table/chart creation functions.

---

## 1.5. outline.md Format

The `outline.md` file defines presentation content at a high level. AI reads this file and generates a corresponding Python script (`generate_*.py`) that creates the PowerPoint presentation.

### Format Specification

**IMPORTANT**: Every slide MUST specify its layout explicitly using the `**Layout**:` field.

```markdown
# Presentation Title

---

## Slide 1: [Slide Title]
**Layout**: 0 (00_Title)

- タイトル: [Main Title Text]
- サブタイトル: [Subtitle Text]
- 副題: [Additional Subtitle] (optional)

---

## Slide 2: [Section Title]
**Layout**: 2 (02_Section)

- タイトル: [Section Title]

---

## Slide 3: [Table Slide Title]
**Layout**: 7 (Handout_Single_Table_Pos)

- タイトル: [Slide Title]
- KeyMessage: [Key message]

### [Table Title]
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1A  | Data 1B  | Data 1C  |
| Data 2A  | Data 2B  | Data 2C  |

---

## Slide 4: [Chart Slide Title]
**Layout**: 5 (Handout_Single_Chart_Pos)

- タイトル: [Slide Title]
- KeyMessage: [Key message]

### [Chart Description]
- グラフ: [Chart type and description]
- データ: [Data description or actual values]

---

## Slide 5: [Text Content Slide]
**Layout**: 11 (Handout_Single_Object_Pos)

- タイトル: [Slide Title]
- KeyMessage: [Key message]

### [Content]
- **[Point 1]**
  - [Detail 1]
  - [Detail 2]

- **[Point 2]**
  - [Detail 1]
  - [Detail 2]
```

**Layout Selection Guidelines**:

- **0 (00_Title)** - Opening slide with title and subtitle
- **1 (01_Contents)** - Table of contents or text-focused slides
- **2 (02_Section)** - Section dividers
- **5 (Handout_Single_Chart_Pos)** - Full-width chart with key message
- **7 (Handout_Single_Table_Pos)** - Full-width table with key message
- **11 (Handout_Single_Object_Pos)** - Full-width content (text, diagrams, etc.)

For complete layout reference, see `~/.claude/skills/pptx/templates/TEMPLATE.md`

### AI Workflow

When AI **creates** `outline.md`:

1. **MUST specify Layout for every slide** - Reference TEMPLATE.md to choose appropriate layout
2. **Use layout name format**: `**Layout**: [number] ([name])`
   - Example: `**Layout**: 7 (Handout_Single_Table_Pos)`
3. **Select layout based on content type**:
   - Title slide → Layout 0
   - Section divider → Layout 2
   - Table → Layout 7
   - Chart → Layout 5
   - Text/Objects → Layout 11

When AI **processes** `outline.md` to generate presentation:

1. **Reads outline.md** - Extracts layout number and content
2. **References TEMPLATE.md** - Looks up placeholder indices for specified layout
3. **Generates generate_*.py** - Creates Python script with:
   - Layout from outline.md (e.g., `prs.slide_layouts[7]`)
   - Correct placeholder indices from TEMPLATE.md (e.g., `placeholders[16]` for TABLE)
   - Structured data specifications (table_spec, chart_spec)
   - Styling via `create_styled_table()` and `create_styled_chart()`
4. **Executes script** - Runs `python generate_*.py` to create `output.pptx`

### Information AI Supplements

AI automatically determines technical details not explicitly in `outline.md`:

- Placeholder indices (from TEMPLATE.md based on specified layout)
- Content structure (converting bullet lists to table_spec)
- Chart types (inferring from context)
- Specific data values (if not provided)

**Note**: Layout numbers are now **explicitly specified** in outline.md, not inferred by AI.

### Example Mapping

**outline.md** (AI creates with Layout specified):
```markdown
## Slide 2: 課題認識
**Layout**: 7 (Handout_Single_Table_Pos)

- タイトル: こんな課題はありませんか？
- KeyMessage: 新規事業の成否は「人」に依存するが...

### 3つの課題
1. **異動・採用の判断**
   - この候補者はイノベーションに向いているか？
```

**AI generates in generate_*.py** (based on specified Layout 7):
```python
# Uses Layout 7 as specified in outline.md
slide = prs.slides.add_slide(prs.slide_layouts[7])
slide.shapes.title.text = "こんな課題はありませんか？"

# References TEMPLATE.md: Layout 7 has KeyMessage at idx=13
slide.placeholders[13].text = "新規事業の成否は..."

table_spec = {
    'data': [
        ['課題', '具体的な悩み'],  # AI structures bullet list as table
        ['異動・採用の判断', '• この候補者は...'],
        ...
    ],
    'header_row': True
}

# References TEMPLATE.md: Layout 7 has TABLE at idx=16
create_styled_table(slide, slide.placeholders[16], table_spec)
```

### Human Editing Workflow

When a human edits `outline.md`:

1. **Change Layout**: Modify `**Layout**: [number] ([name])`
   - Example: Change from Layout 7 (table) to Layout 5 (chart)
2. **Adjust Content**: Update content to match new layout
3. **Regenerate**: Ask AI to regenerate `generate_*.py` from modified outline.md
4. **Execute**: Run `python generate_*.py` to create updated presentation

---

## 2. Files and Roles

### Core Files

- **templates/template.pptx** - Slide layouts, theme colors/fonts (human-edited, shared across projects)
- **templates/template.crtx** - Chart template with styling (human-edited, shared across projects)
- **templates/style.yaml** - Master style definitions (auto-generated from templates, shared across projects)
- **templates/TEMPLATE.md** - Layout documentation (auto-generated from template.pptx, used by AI for layout selection)
- **{project}/powerpoint/processing/snapshot/*** - Snapshots of templates/styles/docs used at generation time (auto-copied for audit)

### Scripts

- **scripts/extract_style.py** - Generate style.yaml from templates
- **scripts/style_config.py** - Python style loader
- **scripts/style_config.R** - R style loader
- **scripts/mermaid_to_shapes.py** - Mermaid → native PowerPoint shapes
- **scripts/native_objects.py** - Native table/chart/diagram creation (with validation & logging)
- **scripts/crtx_utils.py** - Chart.crtx utilities (with detailed error logging)
- **scripts/logging_utils.py** - Auto-configured logging to processing/pptx_generation.log
- **scripts/layout_registry.py** - Layout management
- **scripts/generate_template.py** - TEMPLATE.md auto-generation (maintenance tool)

---

## 3. Style System

### Generate Master Style (templates/style.yaml)

When you update `template.pptx` or `template.crtx`, regenerate the master style:

```bash
cd ~/.claude/skills/pptx
python scripts/extract_style.py
```

This extracts styling from:

- `templates/template.crtx` - Series colors, axes, legend, data labels
- `templates/template.pptx` Slide 1 - Table styling
- `templates/template.pptx` Slide 2 - Flowchart/diagram styling

Output: `templates/style.yaml` (master template)

### Generation-Time Snapshots

For audit and reproducibility, the system snapshots templates/styles at generation time:

- **Auto-snapshot**: On each generation, copies current templates to `powerpoint/processing/snapshot/`
  - `template.pptx` - The template file used
  - `template.crtx` - The chart template used
  - `style.yaml` - The style configuration used
  - `timestamp.txt` - Generation timestamp and skill version
- **Purpose**: Audit trail showing exactly which templates produced the output
- **Regeneration**: Always uses latest templates from `~/.claude/skills/pptx/templates/` (not the snapshot)
- **Benefit**: You can diff snapshots to see how template changes affect output over time

### style.yaml Structure

```yaml
colors:
  primary: "#4F4F70"
  series:
    - type: rgb
      value: "#4F4F70"
    - type: theme
      value: bg1
      brightness: -0.25

category_axis:
  visible: true
  font:
    size_pt: 11
    color_type: theme
    color_value: tx1
    brightness: 0.35

value_axis:
  visible: false

legend:
  position: bottom
  font:
    size_pt: 11

table:
  header:
    fill_theme: bg1
    fill_brightness: -0.5
  body:
    column_brightness: [-0.15, -0.05, -0.05, -0.05]

flowchart:
  node:
    fill: "#4F4F70"
    shadow:
      enabled: false
  connector:
    type: elbow
    dash_style: solid
```

---

## 4. Usage

### Creating Presentations (REQUIRED)

**IMPORTANT**: Always use `native_objects.py` for creating tables and charts. This ensures:

- Complete styling from `.crtx` template is applied
- Automatic data validation
- Detailed error logging to `powerpoint/processing/pptx_generation.log`

**CRITICAL**: Different layouts have different placeholder indices. Always check TEMPLATE.md or use the debug script to find the correct idx for your layout.

```python
import sys
import os
sys.path.insert(0, os.path.expanduser('~/.claude/skills/pptx'))

from pptx import Presentation
from scripts.native_objects import create_styled_table, create_styled_chart

# Load template
skill_dir = os.path.expanduser('~/.claude/skills/pptx')
template_path = os.path.join(skill_dir, 'templates', 'template.pptx')
prs = Presentation(template_path)

# Example 1: Create a chart slide (use Layout 5)
slide = prs.slides.add_slide(prs.slide_layouts[5])  # Handout_Single_Chart_Pos
slide.shapes.title.text = "Sales Report"
slide.placeholders[13].text = "Q1-Q4 performance analysis"
# Chart placeholder is idx=15 for this layout
chart_spec = {
    'chart_kind': 'column',  # 'line', 'bar', 'pie'
    'categories': ['Q1', 'Q2', 'Q3', 'Q4'],
    'series': [
        {'name': 'Sales', 'values': [100, 120, 110, 130]},
        {'name': 'Cost', 'values': [80, 90, 85, 95]}
    ]
}
create_styled_chart(slide, slide.placeholders[15], chart_spec)

# Example 2: Create a table slide (use Layout 7)
slide = prs.slides.add_slide(prs.slide_layouts[7])  # Handout_Single_Table_Pos
slide.shapes.title.text = "Summary Data"
slide.placeholders[13].text = "Key metrics overview"
# Table placeholder is idx=16 for this layout
table_spec = {
    'data': [
        ['項目', '値A', '値B'],
        ['データ1', '100', '200'],
        ['データ2', '150', '250']
    ],
    'header_row': True
}
create_styled_table(slide, slide.placeholders[16], table_spec)

prs.save('powerpoint/output.pptx')
```

### Reading Styles (Advanced)

For custom styling beyond native objects, use `StyleConfig`:

```python
from scripts.style_config import StyleConfig

# Auto-detects: templates/style.yaml (master)
style = StyleConfig.load()
primary = style.colors['primary']  # '#4F4F70'
table_config = style.table

# Or specify path explicitly:
style = StyleConfig.load('~/.claude/skills/pptx/templates/style.yaml')
```

**StyleConfig.load() behavior:**
- Always loads from `~/.claude/skills/pptx/templates/style.yaml` (master template)
- Generation snapshots are saved to `processing/snapshot/style.yaml` for audit only
- No project-specific customization - all styling comes from the master template

**WARNING**: Using `StyleConfig` directly requires manual application of all styles. Prefer `native_objects.py` instead.

### R

```r
source("scripts/style_config.R")
style <- load_style("style.yaml")
colors <- get_series_colors(style, 3)
```

### Mermaid → Native Shapes

```python
from scripts.mermaid_to_shapes import create_flowchart_shapes

code = """flowchart LR
    A[Start] --> B{Decision}
    B -->|Yes| C[Action]
    B -->|No| D[End]"""

create_flowchart_shapes(slide, placeholder, code)
```

---

## 5. Render Modes

| Type    | Mode   | Description                 | Editable |
| ------- | ------ | --------------------------- | -------- |
| TABLE   | NATIVE | python-pptx table           | ✅       |
| CHART   | NATIVE | python-pptx with Chart.crtx | ✅       |
| DIAGRAM | NATIVE | Mermaid → native shapes     | ✅       |

**Note:** All rendering uses NATIVE mode for maximum editability in PowerPoint.

---

## 6. Template Layouts

**IMPORTANT**: For complete layout reference, see **[TEMPLATE.md](TEMPLATE.md)**.

TEMPLATE.md provides:
- All 124 available layouts with detailed descriptions
- Naming convention: `{Usage}_{Layout}_{Content}_{Variant}`
- Selection guidelines for Handout vs Preso layouts
- AI guidelines for outline.md creation

### Quick Reference

**Foundation Layouts**:
- `0`: `00_Title` - Opening slide
- `1`: `01_Contents` - Table of contents
- `2`: `02_Section` - Section divider

**Common Layouts**:
- `0`: `00_Title` - Title slide
- `5`: `Handout_Single_Chart_Pos` - Full-width chart with key message
- `7`: `Handout_Single_Table_Pos` - Full-width table with key message
- `11`: `Handout_Single_Object_Pos` - Full-width object (for Mermaid diagrams)
- `66`: `Preso_Single_Chart_Pos` - Presentation mode chart

**Key Placeholder Indices** (vary by layout - check TEMPLATE.md):
- `idx=0`: TITLE (most layouts)
- `idx=13`: KeyMessage (most content layouts)
- `idx=15`: CHART (chart layouts like 5, 6)
- `idx=16`: TABLE (table layouts like 7, 8)
- `idx=1`: OBJECT (object layouts like 11, 12)

**IMPORTANT**: Always reference TEMPLATE.md for exact placeholder indices for each layout.

---

## 7. Dependencies

### Python

```bash
pip install python-pptx lxml pyyaml pillow
```

### R

```r
install.packages(c("ggplot2", "yaml", "dplyr", "tidyr"))
```

### Mermaid (optional)

```bash
npm install -g @mermaid-js/mermaid-cli
```

---

## 8. Workflow Example

### Complete Example

```python
#!/usr/bin/env python3
import sys
import os
sys.path.insert(0, os.path.expanduser('~/.claude/skills/pptx'))

from pptx import Presentation
from pptx.util import Pt
from pptx.enum.text import PP_ALIGN
from scripts.native_objects import create_styled_table, create_styled_chart

# Setup (run once in project directory)
# mkdir -p powerpoint/processing/{charts,diagrams,temp}
# cp ~/.claude/skills/pptx/templates/template.pptx powerpoint/

# Load template
prs = Presentation('template.pptx')

# Delete existing slides
while len(prs.slides) > 0:
    rId = prs.slides._sldIdLst[0].rId
    prs.part.drop_rel(rId)
    del prs.slides._sldIdLst[0]

# Slide 1: Title slide
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "Presentation Title"
slide.placeholders[1].text = "Subtitle\nDate"

# Slide 2: Chart slide (use Layout 5 for charts)
slide = prs.slides.add_slide(prs.slide_layouts[5])  # Handout_Single_Chart_Pos
slide.shapes.title.text = "Chart Example"
slide.placeholders[13].text = "Key message about this chart"
# Chart placeholder is idx=15
chart_spec = {
    'chart_kind': 'column',
    'categories': ['Q1', 'Q2', 'Q3', 'Q4'],
    'series': [
        {'name': 'Sales', 'values': [100, 120, 110, 130]},
        {'name': 'Cost', 'values': [80, 90, 85, 95]}
    ]
}
create_styled_chart(slide, slide.placeholders[15], chart_spec)

# Slide 3: Table slide (use Layout 7 for tables)
slide = prs.slides.add_slide(prs.slide_layouts[7])  # Handout_Single_Table_Pos
slide.shapes.title.text = "Table Example"
slide.placeholders[13].text = "Summary statistics"
# Table placeholder is idx=16
table_spec = {
    'data': [
        ['Item', 'Value A', 'Value B', 'Total'],
        ['Product 1', '100', '200', '300'],
        ['Product 2', '150', '250', '400']
    ],
    'header_row': True
}
create_styled_table(slide, slide.placeholders[16], table_spec)

# Save
prs.save('powerpoint/output.pptx')
print("✅ Presentation created: powerpoint/output.pptx")
print("📋 Check logs: cat powerpoint/processing/pptx_generation.log")
```

### R Charts (Advanced)

For complex ggplot2 charts, use R with style.yaml:

```r
source("~/.claude/skills/pptx/scripts/style_config.R")
style <- load_style("powerpoint/processing/style.yaml")

p <- ggplot(data, aes(x, y)) +
  geom_bar(fill = get_primary_color(style)) +
  theme_style(style)

# Save as PNG and insert manually into PowerPoint
ggsave("powerpoint/processing/charts/chart.png", p, width = 10, height = 6, dpi = 300)
```

---

## 9. Troubleshooting

### Check Logs

If tables or charts fail to generate correctly:

```bash
cat powerpoint/processing/pptx_generation.log
```

### Common Issues

**Table creation fails**

- Log shows: `Row X has Y columns, expected Z` → Check data array consistency
- Log shows: `Table spec.data is empty` → Verify data is not empty

**Chart creation fails**

- Log shows: `Series 'X' contains non-numeric value` → All chart values must be numbers
- Log shows: `Chart.crtx not found` → Template path issue (auto-fixed in latest version)
- Log shows: `Unknown theme color 'accentX'` → Check style.yaml theme color definitions

**Styling not applied**

- Log shows: `Failed to apply category axis styling` → Check template.crtx compatibility
- Console shows warnings → Check `powerpoint/processing/pptx_generation.log` for details

### Error Prevention

All input data is now validated:

- Table: Column count consistency, non-empty data
- Chart: Numeric values, matching series/category lengths, non-empty series
- Template paths use absolute paths (no longer dependent on working directory)
