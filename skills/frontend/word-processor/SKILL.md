---
name: word-processor
description: Process Microsoft Word documents (.doc and .docx files) by reading content, creating new documents, editing existing ones, and converting between formats (Word, PDF, Markdown, HTML). Use this skill when users mention Word documents, provide .doc/.docx file paths, or request operations like "read this Word file", "create a Word document", "edit this .docx", or "convert Word to PDF/Markdown".
---

# Word Processor

## Overview

Process Microsoft Word documents with full support for reading, creating, editing, and format conversion. Handle both legacy .doc and modern .docx formats seamlessly.

## Quick Start

To determine which operation to perform:

1. **Reading** - User provides a Word file path or mentions reading/extracting content
2. **Creating** - User requests a new Word document with specific content
3. **Editing** - User wants to modify an existing Word document
4. **Converting** - User needs to convert Word to another format (PDF, Markdown, HTML, Text)

## Reading Word Documents

Extract text, tables, and metadata from Word documents.

### Usage

```bash
python3 scripts/read_word.py <file_path> [--format json|markdown|text]
```

### Output Formats

- **json** (default) - Structured data with paragraphs, tables, and metadata
- **markdown** - Human-readable Markdown format
- **text** - Plain text content only

### Example

When user asks "Read this Word file" or provides a .doc/.docx path:

```bash
python3 scripts/read_word.py ~/Documents/report.docx --format markdown
```

### File Format Support

- **.docx** - Full support via python-docx (requires: `pip install python-docx`)
- **.doc** - Legacy format via antiword or textract (requires: `brew install antiword` or `pip install textract`)

## Creating Word Documents

Generate new Word documents from structured content.

### Usage

```bash
python3 scripts/create_word.py <output_path> <content_json>
```

### Content Structure

Provide content as JSON with these optional fields:

```json
{
  "title": "Document Title",
  "subtitle": "Optional subtitle",
  "metadata": {
    "author": "Author Name",
    "subject": "Document Subject"
  },
  "paragraphs": [
    "Simple paragraph text",
    {
      "text": "Styled paragraph",
      "style": "Heading 1",
      "bold": true,
      "alignment": "center"
    }
  ],
  "tables": [
    [
      ["Header 1", "Header 2"],
      ["Row 1 Col 1", "Row 1 Col 2"]
    ]
  ],
  "lists": [
    {
      "ordered": false,
      "items": ["Item 1", "Item 2", "Item 3"]
    }
  ]
}
```

### Example Workflow

When user requests "Create a Word document with a title, introduction, and conclusion":

1. Structure the content as JSON
2. Execute the script:
   ```bash
   python3 scripts/create_word.py ~/Documents/output.docx '{"title":"My Report","paragraphs":["Introduction text","Conclusion text"]}'
   ```

### Supported Formatting

**Text Formatting (applies to runs):**
- `bold`: true/false - Bold text
- `italic`: true/false - Italic text
- `underline`: true/false - Underlined text
- `font_size`: number - Font size in points (e.g., 12, 14, 16)
- `font_name`: string - Font family (e.g., "Arial", "Times New Roman", "Calibri")
- `font_color`: string or array - Text color as hex "#FF0000" or RGB array [255, 0, 0]

**Paragraph Formatting:**
- `alignment`: "left" | "center" | "right" | "justify"
- `line_spacing`: number - Line spacing (1.0 = single, 1.5, 2.0 = double, etc.)
- `space_before`: number - Space before paragraph in points
- `space_after`: number - Space after paragraph in points
- `left_indent`: number - Left indent in inches
- `right_indent`: number - Right indent in inches
- `first_line_indent`: number - First line indent in inches

**Styles:**
- "Heading 1", "Heading 2", "Heading 3" - Predefined heading styles
- Lists: ordered (numbered) and unordered (bullets)

### Advanced Formatting Example

Create a document with custom fonts and spacing:

```json
{
  "title": "Professional Report",
  "title_format": {
    "font_name": "Arial",
    "font_size": 24,
    "font_color": "#2E4053"
  },
  "paragraphs": [
    {
      "text": "Executive Summary",
      "style": "Heading 1",
      "font_name": "Arial",
      "font_size": 18,
      "font_color": "#1A5490"
    },
    {
      "text": "This report provides key insights...",
      "font_name": "Times New Roman",
      "font_size": 12,
      "line_spacing": 1.5,
      "space_after": 10,
      "first_line_indent": 0.5,
      "alignment": "justify"
    }
  ],
  "tables": [
    [
      [
        {"text": "Product", "bold": true, "font_size": 11},
        {"text": "Revenue", "bold": true, "font_size": 11}
      ],
      [
        {"text": "Widget A", "font_name": "Arial"},
        {"text": "$50,000", "font_color": "#27AE60"}
      ]
    ]
  ]
}

## Editing Word Documents

Modify existing Word documents with operations like find/replace, adding content, or updating metadata.

### Usage

```bash
python3 scripts/edit_word.py <input_path> <output_path> <operations_json>
```

### Available Operations

**Replace Text:**
```json
{
  "type": "replace_text",
  "old_text": "original text",
  "new_text": "replacement text"
}
```

**Add Paragraph:**
```json
{
  "type": "add_paragraph",
  "text": "New paragraph content",
  "position": "start",
  "style": "Normal"
}
```

**Add Heading:**
```json
{
  "type": "add_heading",
  "text": "New Section",
  "level": 1
}
```

**Update Metadata:**
```json
{
  "type": "update_metadata",
  "metadata": {
    "title": "New Title",
    "author": "New Author"
  }
}
```

**Delete Paragraph:**
```json
{
  "type": "delete_paragraph",
  "index": 0
}
```

**Format Specific Paragraph:**
```json
{
  "type": "format_paragraph",
  "index": 2,
  "run_format": {
    "font_name": "Arial",
    "font_size": 14,
    "bold": true,
    "font_color": "#FF0000"
  },
  "paragraph_format": {
    "alignment": "center",
    "line_spacing": 1.5,
    "space_after": 12
  }
}
```

**Format All Paragraphs:**
```json
{
  "type": "format_all_paragraphs",
  "run_format": {
    "font_name": "Times New Roman",
    "font_size": 12
  },
  "paragraph_format": {
    "line_spacing": 1.5,
    "first_line_indent": 0.5
  }
}
```

**Format Text Containing Specific String:**
```json
{
  "type": "format_text_contains",
  "search_text": "Important",
  "run_format": {
    "bold": true,
    "font_color": "#FF0000",
    "font_size": 14
  },
  "paragraph_format": {
    "alignment": "center"
  }
}
```

### Example

When user says "Replace all instances of 'draft' with 'final' in this document":

```bash
python3 scripts/edit_word.py input.docx output.docx '[{"type":"replace_text","old_text":"draft","new_text":"final"}]'
```

Multiple operations can be combined in a single array.

## Converting Formats

Convert Word documents to PDF, Markdown, HTML, or plain text.

### Usage

```bash
python3 scripts/convert_format.py <input_path> <output_path> <format>
```

### Supported Formats

- **pdf** - Requires docx2pdf (`pip install docx2pdf`) or LibreOffice
- **markdown** / **md** - Convert to Markdown with preserved structure
- **html** - Generate HTML with basic styling
- **txt** / **text** - Extract plain text only

### Example

When user requests "Convert this Word document to PDF":

```bash
python3 scripts/convert_format.py report.docx report.pdf pdf
```

When user asks "Convert to Markdown so I can edit in a text editor":

```bash
python3 scripts/convert_format.py report.docx report.md markdown
```

## Dependencies

Install required Python packages:

```bash
pip install python-docx
```

Optional dependencies for additional features:

```bash
# For .doc file support
brew install antiword  # macOS
# or
pip install textract

# For PDF conversion
pip install docx2pdf
# or install LibreOffice
```

## Common Workflows

### Workflow 1: Extract and Analyze Content

User: "What's in this Word file?"

1. Read the document: `python3 scripts/read_word.py file.docx --format markdown`
2. Present the content in readable format
3. Provide analysis or answer questions based on content

### Workflow 2: Create from Scratch

User: "Create a Word document with these sections: Introduction, Methods, Results, Conclusion"

1. Structure content as JSON with headings and paragraphs
2. Execute create script with the structured data
3. Confirm creation and provide file path

### Workflow 3: Modify Existing Document

User: "Update the author name and add a new section to this document"

1. Read existing document to understand structure
2. Prepare edit operations (update metadata, add heading, add paragraphs)
3. Execute edit script with all operations
4. Confirm changes made

### Workflow 4: Format Conversion

User: "Convert all my Word documents to PDF"

1. Identify .docx files in the specified directory
2. For each file, run convert script to PDF format
3. Report conversion results

## Error Handling

Scripts return JSON error messages with solutions when issues occur:

- Missing dependencies → Install instructions provided
- File not found → Verify path and existence
- Unsupported format → List of supported formats shown
- Invalid JSON → JSON parsing error details

Always check script output for `"success": true` before confirming completion.
