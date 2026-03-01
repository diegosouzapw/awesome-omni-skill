---
name: tencent-cloud-pptx
description: "Create professional Tencent Cloud themed presentations from markdown content. Use when users request: (1) Creating presentations with Tencent Cloud branding, (2) Converting markdown documents to PowerPoint slides, (3) Generating slides with automatic content structuring, (4) Creating bilingual (Chinese/English) technical presentations, (5) Adding AI-generated images to presentation slides. Keywords to watch: 腾讯云, Tencent Cloud, markdown to PPT, presentation generation, slides with images."
---

# Tencent Cloud Presentation Builder

Create beautiful, professional presentations with Tencent Cloud branding from markdown content. This skill intelligently structures content into slides, suggests image placements, generates AI images, and produces polished PowerPoint presentations.

## Overview

This skill transforms markdown documents into Tencent Cloud branded presentations through an intelligent multi-step workflow:

1. **Parse markdown** → Analyze structure and intelligently split into slides
2. **Generate images** → Create AI-generated visuals with Tencent Cloud styling
3. **Build presentation** → Convert to PowerPoint with professional design
4. **Validate output** → Ensure visual quality and brand consistency

## When to Use This Skill

Use this skill when users request:
- Converting markdown documents to Tencent Cloud themed presentations
- Creating professional slide decks with automatic content structuring
- Generating presentations with AI images based on content
- Building bilingual (Chinese/English) technical presentations
- Producing brand-consistent PowerPoint files from text content

Trigger phrases: "create a Tencent Cloud presentation", "markdown to PPT", "generate slides with 腾讯云 theme", "build presentation from this document"

## Complete Workflow

### Step 1: Parse Markdown Content

Use `parse_markdown.py` to analyze markdown and structure it into slides.

```bash
python scripts/parse_markdown.py <markdown_file>
```

**What it does:**
- Analyzes markdown structure (headers, lists, paragraphs)
- Intelligently splits content into slides based on density
- Identifies where images should be placed
- Generates image prompts based on content
- Outputs structured JSON with slide data

**Output:** `<filename>_slides.json` containing:
- Slide types (title, section, content)
- Content for each slide
- Image placeholders with sizes and prompts
- Metadata for presentation generation

**Example:**
```bash
python scripts/parse_markdown.py content.md
# Creates: content_slides.json
```

The parser automatically:
- Makes the first H1 a title slide
- Converts other H1s to section dividers
- Creates content slides from H2 headers
- Splits long sections into multiple slides (>6 items)
- Suggests images for technical/visual content

### Step 2: Generate AI Images

Use `generate_images.py` to create AI-generated images for slides.

```bash
# Generate all images from slides JSON
python scripts/generate_images.py <slides_json_file>

# Generate single image
python scripts/generate_images.py --single "<prompt>" "<size>" <output_path>
```

**What it does:**
- Reads image requirements from slides JSON
- Generates images using text-to-image model (or creates placeholders)
- Applies Tencent Cloud styling (blue accents, professional aesthetic)
- Saves images with appropriate naming convention

**Image generation details:**
- **API Integration**: Configure `BANANA_API_KEY` and `IMAGE_API_ENDPOINT` environment variables
- **Fallback**: Creates gradient placeholders with Tencent Cloud branding if API unavailable
- **Sizes**: Supports any resolution (common: 1920×1080, 960×540, 600×400)
- **Style**: Professional, modern, tech-focused with Tencent Blue (#006EFF) accents

**Output:** Images saved to `images/slide_X_image_Y.png`

**Example:**
```bash
python scripts/generate_images.py content_slides.json
# Creates: images/slide_2_image_1.png, images/slide_4_image_1.png, etc.
```

### Step 3: Generate HTML Slides

Use `generate_presentation.py` to create styled HTML slides.

```bash
python scripts/generate_presentation.py <slides_json_file> [output_dir]
```

**What it does:**
- Generates professional HTML slides with Tencent Cloud design
- Applies brand colors, typography, and layouts
- Links images to appropriate slides
- Creates CSS stylesheet with design system
- Generates conversion script for PowerPoint

**Output in `output_dir/`:**
- `styles.css` - Tencent Cloud design system
- `slide_1.html`, `slide_2.html`, etc. - Individual slides
- `convert.js` - Node.js conversion script

**Example:**
```bash
python scripts/generate_presentation.py content_slides.json ./my_presentation
```

### Step 4: Convert to PowerPoint

Convert HTML slides to PowerPoint using html2pptx.

```bash
# 1. Extract html2pptx library (one-time setup)
cd <output_dir>
mkdir -p html2pptx
tar -xzf /mnt/skills/public/pptx/html2pptx.tgz -C html2pptx

# 2. Run conversion
NODE_PATH="$(npm root -g)" node convert.js
```

**What it does:**
- Converts HTML slides to native PowerPoint format
- Preserves styling, layouts, and images
- Creates `presentation.pptx` ready for delivery

**Output:** `presentation.pptx` - Final Tencent Cloud branded presentation

### Step 5: Visual Validation (Recommended)

Always validate the presentation visually before delivery.

```bash
# Convert to PDF
soffice --headless --convert-to pdf presentation.pptx

# Convert to images for inspection
pdftoppm -jpeg -r 150 presentation.pdf slide

# View the images to check for:
# - Text cutoff or overlap
# - Image positioning issues
# - Color contrast problems
# - Alignment issues
```

**CRITICAL:** Fix any visual defects before delivering to user. Common fixes:
1. Increase margins/padding
2. Reduce font sizes
3. Adjust image dimensions
4. Rethink layout if needed

## Design System

### Brand Colors

The skill uses Tencent Cloud's official color palette:

- **Primary Blue**: #006EFF - Main brand color
- **Deep Blue**: #0052D9 - Secondary accent
- **Light Blue**: #E0EBFF - Background highlights
- **Tech Green**: #00A870 - Success indicators
- **Energy Orange**: #FF6A00 - Warnings/highlights
- **Dark Gray**: #1F2329 - Main text
- **White**: #FFFFFF - Clean backgrounds

### Typography

**Font Family:**
- **Chinese**: Alimama ShuHeiTi (阿里妈妈数黑体), Microsoft YaHei
- **English**: Inter, Segoe UI, Helvetica

**Font Weights & Styles:**
- **Heavy (900)**: Main titles, hero text - 微软粗体
- **Bold (700)**: Headers, emphasis - 粗体
- **Medium (500)**: Subtle emphasis
- **Regular (400)**: Body text
- **Italic**: Accents, quotes - 斜体

**Size Scale:**
- **56px (Heavy)**: Title slide main titles
- **48px (Heavy)**: Section slide headers
- **36px (Bold)**: Content slide headers
- **20px (Regular)**: Body text, bullet points
- **18px (Regular)**: Paragraphs
- **Italic**: Used for subtitles and subtle emphasis

### Slide Layouts

**Title Slide:**
- Full-screen gradient background (blue gradient)
- Centered large title (56px, white, bold)
- Subtitle below (24px, light blue)
- Minimal, impactful design

**Section Slide:**
- Blue gradient background
- Left-aligned large title (48px, white)
- Optional subtitle (20px, light blue)
- Clean separation between sections

**Content Slide:**
- White background
- Blue header bar with title (36px)
- Left content area with bullet points (20px)
- Optional right-side image area (400px wide)
- Professional spacing and hierarchy

**Image Slide:**
- White background with header
- Large centered image area
- Clean presentation of visual content

## Advanced Usage

### Customizing Image Prompts

Edit the slides JSON before image generation to customize prompts:

```json
{
  "slides": [
    {
      "images": [
        {
          "size": "1920x1080",
          "prompt": "Custom prompt here",
          "requirements": "Additional style requirements",
          "position": "full"
        }
      ]
    }
  ]
}
```

### Manual Slide Editing

After generating HTML slides, you can manually edit them:

1. Open `slide_X.html` in a text editor
2. Modify content, add elements, adjust styling
3. Re-run conversion script to update PowerPoint

### Using External Images

Replace generated images with your own:

```bash
# Copy your image to the images directory
cp my_image.png images/slide_2_image_1.png

# Re-run conversion
NODE_PATH="$(npm root -g)" node convert.js
```

## Best Practices

### Content Structure
- Keep titles concise (< 8 words)
- Limit bullet points to 5-6 per slide
- Use clear, scannable language
- Break long sections into multiple slides

### Image Usage
- Use images strategically for complex concepts
- Ensure images support the message
- Maintain consistent visual style
- Check image resolution (minimum 1920×1080 for full slides)

### Brand Consistency
- Always use official Tencent Cloud colors
- Follow typography guidelines
- Maintain professional tone
- Use provided templates and styles

### Quality Assurance
1. **Visual Check**: Always convert to images and inspect
2. **Text Verification**: Ensure no cutoff or overlap
3. **Brand Compliance**: Verify colors and fonts
4. **Content Review**: Check for accuracy and clarity

## Troubleshooting

### Issue: Parser creates too many slides
**Solution:** Increase content density per section in markdown. Combine related H2 sections.

### Issue: Images not generating
**Solution:** 
1. Check if API credentials are set (`BANANA_API_KEY`)
2. Skill will use gradient placeholders automatically
3. Manually add images to `images/` directory

### Issue: Conversion fails
**Solution:**
1. Ensure html2pptx library is extracted
2. Check Node.js modules are installed: `npm list -g pptxgenjs`
3. Verify NODE_PATH is set correctly

### Issue: Text overflow in PowerPoint
**Solution:**
1. Reduce font sizes in CSS
2. Increase slide padding
3. Split content into multiple slides
4. Reduce content per slide

## Examples

### Basic Workflow

```bash
# 1. Parse markdown
python scripts/parse_markdown.py project.md

# 2. Generate images
python scripts/generate_images.py project_slides.json

# 3. Create HTML slides
python scripts/generate_presentation.py project_slides.json ./output

# 4. Convert to PowerPoint
cd output
mkdir -p html2pptx && tar -xzf /mnt/skills/public/pptx/html2pptx.tgz -C html2pptx
NODE_PATH="$(npm root -g)" node convert.js

# 5. Validate
soffice --headless --convert-to pdf presentation.pptx
pdftoppm -jpeg -r 150 presentation.pdf check
# View check-*.jpg files
```

### Quick Single-Image Generation

```bash
python scripts/generate_images.py --single \
  "Modern cloud infrastructure with Tencent Blue" \
  "1920x1080" \
  hero_image.png
```

## Reference Files

- **Brand Guidelines**: `references/tencent-cloud-brand.md` - Complete brand specifications
- **Example Markdown**: `assets/example.md` - Sample markdown for testing
- **Color Palette**: See brand guidelines for hex codes
- **Typography**: See brand guidelines for font specifications

## Dependencies

Ensure these are installed:
- Python 3.8+
- Node.js 14+
- LibreOffice (for PDF conversion)
- Poppler utils (for image conversion)
- Python packages: json, pathlib (built-in)
- Node packages: pptxgenjs, playwright (installed globally)
- PIL/Pillow (for placeholder generation): `pip install Pillow`

## Tips for Success

1. **Start with good markdown**: Well-structured markdown = better slides
2. **Review JSON output**: Check parsed structure before image generation
3. **Customize when needed**: Don't hesitate to edit HTML or JSON
4. **Validate visually**: Always check the final output looks professional
5. **Iterate quickly**: The workflow supports rapid iteration

## Integration with Other Skills

This skill works well with:
- **docx skill**: Convert presentations to Word documents
- **pdf skill**: Extract content from PDF reports for slides
- **web-artifacts-builder**: Create interactive web versions
- **brand-guidelines**: Ensure consistency across materials
