---
name: adf-json-example
description: Fetch raw ADF JSON data from a Confluence page URL. Use this skill when you need to see real-world ADF examples, understand how Confluence represents specific elements, debug ADF parsing, or create test samples.
---

# ADF JSON Example Query Skill

This skill fetches the raw Atlassian Document Format (ADF) JSON data from a Confluence page. Use this when you need to understand the actual ADF structure of a real Confluence page, inspect how specific elements are represented in ADF, or debug ADF parsing issues.

## Purpose

When working with ADF data, you often need to see real-world examples of how Confluence represents content. This skill:

- Fetches the complete ADF JSON body from any accessible Confluence page
- Displays the raw JSON structure with proper formatting
- Caches results locally to avoid repeated API calls

## Prerequisites

The Confluence API credentials must be configured. See `atlas_doc_parser/tests/data/client.py` for authentication setup.

## Command

Fetch ADF JSON from a Confluence page URL:

```bash
.venv/bin/python .claude/skills/adf-json-example/scripts/adf_json_example_cli.py get_example "<confluence_page_url>"
```

### Parameters

- `confluence_page_url`: Full URL to a Confluence page (e.g., `https://your-domain.atlassian.net/wiki/spaces/SPACE/pages/123456/Page+Title`)

### Example

```bash
.venv/bin/python .claude/skills/adf-json-example/scripts/adf_json_example_cli.py get_example "https://sanhehu.atlassian.net/wiki/spaces/GitHubMacHuGWU/pages/654049306/Mark+-+strong"
```

This outputs the full ADF JSON structure of the page, showing how nodes and marks are organized.

## Output

The command prints formatted JSON to stdout, containing the full ADF document structure with:

- `version`: ADF format version (typically 1)
- `type`: Document type (typically "doc")
- `content`: Array of top-level nodes (paragraphs, headings, tables, etc.)

Each node contains its type, attributes, and nested content/marks.
