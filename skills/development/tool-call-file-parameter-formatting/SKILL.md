---
name: tool-call-file-parameter-formatting
description: Formats file and URL parameters for tool calls. You must analyze the target tool's parameter names and descriptions to choose the correct format (base64, text, or URL ref).
metadata:
  version: "1.0"
---

# File Parameter Formatting

## Purpose
Standardize file and URL inputs for tool calls using the `file:{prefix}::{path_or_url}` schema. You must determine the correct prefix by strictly analyzing the target tool's parameter definition.

## Instructions

When a tool parameter requires a file or URL, follow this process:

### 1. Analyze Tool Parameter Definition
Inspect the **Parameter Name** and **Parameter Description** in the tool definition to determine the requirement:

*   **Requirement: Base64 Content**
    *   *Clues:* Description mentions "content," "base64," "encoded," "file data".
    *   *Action:* Use `base64` prefix.
*   **Requirement: Plain Text**
    *   *Purpose:* *   *Purpose:* Use this mode to indicate the tool expects the actual text file content as the parameter. When a file is marked with the `text` prefix, the tool-call processor will read the file's contents and pass that text as the tool call parameter.
    *   *Clues:* Description mentions "text," "string content," "read file," or the file type is code/markdown/logs.
    *   *Action:* Use `text` prefix.
*   **Requirement: URL/Path Reference**
    *   *Clues:* Parameter name contains `url`, `link`, `uri`, or description says "pass the link" or "reference."
    *   *Action:* Use `url` prefix.

### 2. Format the Value
Construct the string using the format: `file:{prefix}::{path_or_url}`

*   **Prefix:** One of `base64`, `text`, or `url` (determined from step 1)
*   **Separator:** Always use `::`
*   **Path:**
    *   For system files: `files/path/to/file.ext`
    *   For web resources: `https://example.com/resource`

## Thinking Process
Before executing the tool call, perform this check:

1.  **Inspect Definition:** "I am calling tool `[tool_name]`. The parameter `[param_name]` has the description: `[description]`."
2.  **Deduce Type:** "Based on the name/description, this tool expects [raw content | text string | URL reference]."
3.  **Apply Format:** "Therefore, I will use the `[base64 | text | url]` prefix."

## Examples

### Example 1: Visual Analysis Tool
*   **Tool Definition:**
    *   `name`: `analyze_image`
    *   `parameter`: `image_data`
    *   `description`: "The base64 encoded contents of the image file."
*   **Reasoning:** Description explicitly asks for "encoded contents."
*   **Result:** `file:base64::files/images/chart.png`

### Example 2: Browser Tool
*   **Tool Definition:**
    *   `name`: `web_browser`
    *   `parameter`: `target_url`
    *   `description`: "The URL to navigate to."
*   **Reasoning:** Parameter name is `target_url` and expects a navigation link.
*   **Result:** `file:url::https://google.com`

### Example 3: Code Reader
*   **Tool Definition:**
    *   `name`: `read_script`
    *   `parameter`: `script_content`
    *   `description`: "The text content of the python script."
*   **Reasoning:** Description asks for "text content."
*   **Result:** `file:text::files/scripts/main.py`

### Example 4: Format Hint in Tool Name
*   **Tool Definition:**
    *   `name`: `base64_image_processor`
    *   `parameter`: `image_input`
    *   `description`: "Image to process."
*   **Reasoning:** Tool name contains "base64," indicating it expects encoded content.
*   **Result:** `file:base64::files/photos/portrait.jpg`

### Example 5: Format Hint in Tool Description
*   **Tool Definition:**
    *   `name`: `document_analyzer`
    *   `description`: "Analyzes documents by reading their text content directly."
    *   `parameter`: `document`
    *   `description`: "Document to analyze."
*   **Reasoning:** Tool description mentions "reading text content directly," suggesting text mode optimization.
*   **Result:** `file:text::files/reports/annual_report.txt`

### Example 6: Format Hint in Argument Name
*   **Tool Definition:**
    *   `name`: `fetch_resource`
    *   `parameter`: `resource_url`
    *   `description`: "The resource to fetch."
*   **Reasoning:** Parameter name ends with `_url`, indicating a URL reference is expected.
*   **Result:** `file:url::https://api.example.com/data.json`

### Example 7: Format Hint in Argument Description
*   **Tool Definition:**
    *   `name`: `process_file`
    *   `parameter`: `input_file`
    *   `description`: "Pass the base64-encoded file data for processing."
*   **Reasoning:** Description explicitly states "base64-encoded file data."
*   **Result:** `file:base64::files/uploads/document.pdf`

## Common Mistakes
*   ❌ Ignoring the tool description and defaulting to `url` for local files.
*   ❌ Passing `file:base64::...` to a tool that only asks for a URL string (parameter name `url`).
*   ❌ Passing raw paths (`files/doc.pdf`) without the `file:...` schema.