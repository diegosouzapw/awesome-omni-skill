---
name: annotate-talk
description: Create annotated blog posts from technical talks with slides. Use when asked to convert a video presentation to a blog post, create written content from a talk, or annotate slides with transcript.
---

# Annotated Talk Generator

Use the `ai-annotate-talk` CLI tool to create blog posts from technical talks.

## Usage

```bash
ai-annotate-talk "https://youtu.be/VIDEO_ID" slides.pdf output_images/
ai-annotate-talk "https://youtu.be/VIDEO_ID" slides.pdf output_images/ --output post.md
ai-annotate-talk "https://youtu.be/VIDEO_ID" slides.pdf output_images/ --transcript custom.txt
ai-annotate-talk "https://youtu.be/VIDEO_ID" slides.pdf output_images/ --prompt context.txt
```

## Requirements

- `GEMINI_API_KEY` environment variable
- `JINA_READER_KEY` environment variable
- `poppler-utils` installed (`brew install poppler` on macOS)
- The `hamel_tools` package must be installed: `pip install hamel_tools`

## Output

Markdown blog post with embedded slide images and synchronized annotations.
