---
name: lecture-transcript-slide-matcher
description: Combines YouTube lecture transcripts with PDF slides to create an interactive HTML page. Matches each slide to corresponding transcript segments, organized by key concepts. Use when users want to create synchronized lecture notes from transcript text files and slide PDFs.
---

# Lecture Transcript and Slide Matcher

Combines YouTube lecture transcripts (txt files) with corresponding PDF slides to create an interactive HTML page with synchronized content organized by key concepts.

## Overview

This skill processes lecture materials and generates an HTML page with:
1. Left-hand table of contents (TOC) with key concepts
2. Main content area with slides and transcript segments for each concept
3. Automatic transcript cleaning (removes fillers, formats paragraphs)
4. Visual separation between sections

## Workflow

The matching process involves these steps:

1. **Convert transcript** - Standardize timestamp format in transcript
2. **Analyze content** - Extract information from transcript and PDF
3. **Create mapping** - Match concepts to slides and transcript segments
4. **Generate HTML** - Produce the final interactive page

## Step 1: Covert transcript

Run the conversion script to standardize the transcript timestamp format:

```bash
python scripts/convert_transcript.py <transcript_input.txt> <transcript_output.pdf>
```

This script:
- Reads timestamps from separate lines
- Converts them to [MM:SS] or [H:MM:SS] format
- Attaches timestamps inline with text
- Outputs a new transcript text file

## Step 2: Analyze Content

Run the analysis script to understand the lecture materials:

```bash
python scripts/analyze_content.py <transcript.txt> <slides.pdf> [output_analysis.json]
```

This script:
- Parses all transcript segments with timestamps
- Extracts text previews from each PDF slide
- Creates a mapping template
- Outputs `content_analysis.json` with all information

**What to do:**
1. Run the analysis script
2. Review the output JSON file
3. Examine transcript segments and slide previews
4. Identify the key concepts in the lecture

## Step 3: Create Mapping

Create a `mapping.json` file that connects concepts to slides and transcript segments.

**Option A: Let Claude create the mapping**

After running the analysis script, ask Claude to create the mapping by providing:
- The `content_analysis.json` file
- The original transcript file (for full text)
- Instructions on how to identify key concepts

Claude will analyze the content and create a comprehensive mapping.

**Option B: Manual creation**

Use the template in `content_analysis.json` as a starting point. See `references/mapping_schema.md` for complete documentation.

### Mapping Structure

```json
[
  {
    "title": "Key concept or insight",
    "slide_indices": [0, 1, 2],
    "transcript_segments": [
      {
        "start_time": "MM:SS or HH:MM:SS",
        "end_time": "MM:SS or HH:MM:SS",
        "text": "Full transcript text from this time range"
      }
    ]
  }
]
```

**Key points:**
- Use 0-based indexing for slides (first slide = 0)
- Timestamps must match format in transcript: `[HH:MM:SS]` or `[MM:SS]`
- Include full transcript text, not summaries
- Each TOC item represents one coherent concept
- Multiple slides and transcript segments can map to one concept

See `references/mapping_schema.md` for detailed schema documentation and examples.

## Step 4: Generate HTML

Run the generation script to create the final HTML page:

```bash
python scripts/match_lecture_content.py <transcript.txt> <slides.pdf> <mapping.json> [output.html]
```

The script:
- Parses the transcript and extracts all segments
- Converts PDF pages to images (embedded as base64)
- Reads the mapping JSON
- Generates an interactive HTML page with:
  - Left panel with TOC (clickable navigation)
  - Main area with sections for each concept
  - Slides displayed as images
  - Cleaned and formatted transcript segments
  - Visual separation between sections

**Output:** `lecture_output.html` (or specified filename)

## Transcript Format Requirements

The transcript must use timestamp markers:

```
[00:15] Welcome to today's lecture on machine learning.
[00:45] We'll start by discussing supervised learning...
[02:30] Now let's look at an example with house prices...
```

Supported timestamp formats:
- `[HH:MM:SS]` - Hours, minutes, seconds
- `[MM:SS]` - Minutes, seconds
- `[H:MM:SS]` - Single-digit hours

## Automatic Transcript Cleaning

The script automatically:
- Removes filler words (um, uh, like, you know, etc.)
- Removes conversational artifacts ([inaudible], [laughter], etc.)
- Condenses multiple spaces
- Breaks text into readable paragraphs (50 words per paragraph)
- Displays only start and end timestamps for continuous segments

## HTML Output Features

### Table of Contents (Left Panel)
- Clickable items for navigation
- Highlights current section on scroll
- Fixed width, scrollable
- Responsive (collapses on mobile)

### Content Area
- One section per TOC item
- Section title as header
- Slides displayed as images
- Transcript segments below slides
- Time range badges for each segment
- Visual separators between sections
- Smooth scrolling

### Styling
- Clean, professional appearance
- Blue accent colors
- Readable typography
- Shadow effects for slides
- Highlighted transcript containers

## Best Practices

### Identifying Key Concepts

**Good concept granularity:**
- "Linear Regression: Mathematical Formulation"
- "Gradient Descent Algorithm"
- "Neural Networks: Forward Propagation"

**Too broad:**
- "Machine Learning Overview" (entire lecture)

**Too narrow:**
- "Definition of Theta" (single term)

### Creating Effective Mappings

1. **One concept per TOC item**: Each entry should represent one coherent idea
2. **Logical ordering**: Follow lecture sequence
3. **Complete coverage**: Include all major concepts
4. **Accurate alignment**: Ensure slides and transcript truly correspond
5. **Full transcript text**: Don't summarize; include everything from the time range

### Handling Edge Cases

**Concept spans non-contiguous slides:**
```json
{
  "title": "Example: Housing Price Prediction",
  "slide_indices": [5, 8, 12],
  "transcript_segments": [...]
}
```

**Multiple transcript segments per concept:**
```json
{
  "title": "Backpropagation",
  "slide_indices": [15],
  "transcript_segments": [
    {"start_time": "20:00", "end_time": "22:30", "text": "..."},
    {"start_time": "23:00", "end_time": "25:45", "text": "..."}
  ]
}
```

**No slides for a concept (discussion only):**
```json
{
  "title": "Q&A: Common Misconceptions",
  "slide_indices": [],
  "transcript_segments": [...]
}
```

## Dependencies

The scripts require PyMuPDF for PDF processing:

```bash
pip install pymupdf --break-system-packages
```

Claude handles installation automatically when needed.

## Example Usage

Complete workflow example:

```bash
# Step 1: Analyze
python scripts/analyze_content.py lecture.txt slides.pdf analysis.json

# Step 2: Create mapping (manually or with Claude's help)
# Edit analysis.json or create new mapping.json

# Step 3: Generate HTML
python scripts/match_lecture_content.py lecture.txt slides.pdf mapping.json output.html
```

## Reference Files

- `references/mapping_schema.md` - Complete JSON schema documentation with examples
- `references/example_mapping.json` - Sample mapping for a machine learning lecture

## Troubleshooting

**"PyMuPDF not installed"**
Run: `pip install pymupdf --break-system-packages`

**Timestamps don't match**
Ensure timestamps in mapping.json exactly match those in the transcript file.

**Slides not displaying**
Verify slide_indices are 0-based (first slide = 0, not 1).

**Text looks messy**
The cleaning is automatic. If issues persist, check for unusual formatting in the transcript.

**Missing concepts**
Review the analysis output to ensure all relevant transcript segments and slides are covered.
