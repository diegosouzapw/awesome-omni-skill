---
name: powerpoint
description: Create, edit, and manipulate PowerPoint presentations from the command line. Add slides, text, images, tables, charts, shapes, and apply professional designs. Use when the user wants to create or modify .pptx files.
---

# PowerPoint CLI

For installation and requirements, see [README.md](README.md).

## Batch Mode (recommended)

The server is **stateful** — create a presentation, then build it up, then save. Use the batch runner to execute multiple commands in a single session:

```bash
echo '<json>' | bun ~/.pi/agent/skills/powerpoint/powerpoint-batch.ts
bun ~/.pi/agent/skills/powerpoint/powerpoint-batch.ts recipe.json
bun ~/.pi/agent/skills/powerpoint/powerpoint-batch.ts --recipe '<json>'
```

Input format — a JSON object with a `commands` array:

```json
{
  "commands": [
    { "tool": "create_presentation" },
    { "tool": "add_slide", "args": { "title": "Hello World" } },
    { "tool": "manage_text", "args": { "slide_index": 0, "operation": "add", "left": 1, "top": 2, "width": 8, "height": 1, "text": "Body text", "font_size": 18 } },
    { "tool": "save_presentation", "args": { "file_path": "./output.pptx" } }
  ]
}
```

Output: JSON array of `{ tool, result }` objects (or `{ tool, error }` on failure).

The batch runner auto-injects `presentation_id` into all commands after `create_presentation`/`open_presentation`, so you don't need to track it yourself.

## Single-Command Mode

For one-off read operations, use the single-command CLI:

```bash
bun ~/.pi/agent/skills/powerpoint/powerpoint-cli.js <command> [flags]
bun ~/.pi/agent/skills/powerpoint/powerpoint-cli.js --help
```

Useful for inspection commands that don't need state:

```bash
bun <cli> list-slide-templates
bun <cli> get-template-info --template-id title_slide
bun <cli> get-server-info
```

## Available Tools

### Lifecycle
- `create_presentation` — create empty presentation
- `create_presentation_from_template` — create from .pptx template: `{ "template_path": "./template.pptx" }`
- `open_presentation` — open existing file: `{ "file_path": "./deck.pptx" }`
- `save_presentation` — save to file: `{ "file_path": "./output.pptx" }`
- `get_presentation_info` — slide count, dimensions, properties
- `set_core_properties` — set title, author, keywords, etc.

### Slides
- `add_slide` — add a slide: `{ "layout_index": 1, "title": "Slide Title" }`
  - layout_index: 0=Title, 1=Title+Content, 2=Section, 5=Blank, etc.
  - Optional: `background_type` ("solid"/"gradient"), `background_colors`, `color_scheme`
- `get_slide_info` — shapes, layout info: `{ "slide_index": 0 }`
- `extract_slide_text` — all text from a slide: `{ "slide_index": 0 }`
- `extract_presentation_text` — all text from all slides

### Text Content
- `populate_placeholder` — fill a layout placeholder: `{ "slide_index": 0, "placeholder_idx": 1, "text": "Hello" }`
- `add_bullet_points` — bullets in a placeholder: `{ "slide_index": 0, "placeholder_idx": 1, "bullet_points": ["A", "B", "C"] }`
- `manage_text` — add/format text boxes:
  ```json
  { "slide_index": 0, "operation": "add", "left": 1, "top": 2, "width": 8, "height": 1, "text": "Hello", "font_size": 24, "bold": true, "color": [0, 0, 0] }
  ```
  Operations: `add`, `format`, `validate`, `format_runs`

### Images
- `manage_image` — add/enhance images:
  ```json
  { "slide_index": 0, "operation": "add", "image_source": "./photo.png", "left": 1, "top": 2, "width": 4, "height": 3 }
  ```
  `source_type`: "file" (default), "url", "base64"

### Tables
- `add_table` — add a data table:
  ```json
  { "slide_index": 0, "rows": 3, "cols": 4, "left": 1, "top": 2, "width": 8, "height": 3, "data": [["H1","H2","H3","H4"],["a","b","c","d"],["e","f","g","h"]] }
  ```
- `format_table_cell` — style a cell: `{ "slide_index": 0, "shape_index": 0, "row": 0, "col": 0, "bold": true, "bg_color": [68, 114, 196] }`

### Charts
- `add_chart`:
  ```json
  { "slide_index": 0, "chart_type": "bar", "left": 1, "top": 2, "width": 8, "height": 4, "categories": ["Q1","Q2","Q3"], "series_names": ["Revenue"], "series_values": [[100, 200, 300]] }
  ```
  Note: `series_values` is an array of arrays (one inner array per series).
  Chart types: `bar`, `column`, `line`, `pie`, `scatter`, `area`, `doughnut`
- `update_chart_data` — replace data on existing chart

### Shapes & Connectors
- `add_shape`:
  ```json
  { "slide_index": 0, "shape_type": "rectangle", "left": 1, "top": 1, "width": 3, "height": 2, "fill_color": "4472C4", "text": "Box" }
  ```
  Types: `rectangle`, `rounded_rectangle`, `oval`, `triangle`, `diamond`, `arrow_right`, etc.
- `add_connector` — lines/arrows between points:
  ```json
  { "slide_index": 0, "connector_type": "straight", "start_x": 2, "start_y": 3, "end_x": 6, "end_y": 3 }
  ```

### Design & Styling
- `apply_professional_design` — apply themes: `{ "operation": "apply_theme", "color_scheme": "modern_blue" }`
  - Operations: `apply_theme`, `design_slide`, `enhance_slide`
- `apply_picture_effects` — shadow, reflection, etc.
- `optimize_slide_text` — auto-resize fonts: `{ "slide_index": 0 }`
- `manage_fonts` — analyze/optimize: `{ "operation": "analyze", "font_path": "./deck.pptx" }`

### Templates
- `list_slide_templates` — list available layout templates
- `get_template_info` — details about a template: `{ "template_id": "title_slide" }`
- `apply_slide_template` — apply template to existing slide
- `create_slide_from_template` — new slide from template
- `create_presentation_from_templates` — full deck from template sequence
- `auto_generate_presentation` — auto-generate from topic:
  ```json
  { "topic": "Q4 Results", "slide_count": 8, "presentation_type": "business", "color_scheme": "modern_blue" }
  ```

### Other
- `manage_hyperlinks` — add/remove/list links
- `manage_slide_transitions` — set/remove transitions
- `manage_slide_masters` — inspect master layouts

## Reference

- Positions (`left`, `top`) and sizes (`width`, `height`) are in **inches**. Standard slide is 10×7.5.
- Slide indices are **0-based**.
- Colors: hex RGB without `#` for shape fills (e.g., `"4472C4"`), RGB arrays `[r, g, b]` for text/cell colors.
- Color schemes: `modern_blue`, `corporate_gray`, `elegant_green`, `warm_red`.
- Tool names accept both `snake_case` and `kebab-case`.
