---
name: nano-pdf
cluster: community
description: "PDF processing: extraction, text mining, form filling, manipulation, OCR integration"
tags: ["pdf","document","extraction","ocr"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "pdf extract text mine form fill manipulate ocr"
---

# nano-pdf

## Purpose
This skill provides tools for PDF processing, including text extraction, mining, form filling, manipulation, and OCR integration, to handle document workflows efficiently.

## When to Use
Use this skill for tasks involving PDF data extraction (e.g., from scanned documents), text analysis in reports, automating form submissions, merging/splitting files, or applying OCR to non-text PDFs. Apply it in data pipelines, document automation scripts, or when integrating with OCR services for unstructured data.

## Key Capabilities
- Text extraction: Pulls plain text or structured data from PDFs, supporting encrypted files with passwords; uses OCR via Tesseract integration for image-based PDFs.
- Text mining: Analyzes extracted text for keywords, sentiment, or patterns; e.g., counts occurrences of phrases in a document.
- Form filling: Populates interactive PDF forms with JSON data; supports flattening forms to static PDFs.
- Manipulation: Merges, splits, rotates, or watermarks PDFs; handles up to 500-page documents efficiently.
- OCR integration: Converts scanned PDFs to searchable text using external APIs; requires Tesseract or similar engine configuration.

## Usage Patterns
Invoke via CLI for quick scripts or API for server-side integration. For batch processing, chain commands in a shell script; for web apps, use API calls in loops. Always specify input/output paths explicitly. Pattern: Extract text first, then mine or manipulate as needed. For OCR-heavy tasks, preprocess images before PDF operations.

## Common Commands/API
CLI commands use `nano-pdf` binary; API endpoints are under `https://api.opencclaw.com/nano-pdf/`. Authentication requires `$NANO_PDF_API_KEY` environment variable.

- Extract text: `nano-pdf extract --file input.pdf --output text.txt --ocr true` (adds OCR if text is not selectable).
- Mine text: `nano-pdf mine --input text.txt --keywords "AI,robot" --output results.json` (outputs keyword frequencies).
- Fill form: `nano-pdf fill --template form.pdf --data '{"field1": "value"}' --output filled.pdf`.
- Manipulate PDF: `nano-pdf merge --files file1.pdf file2.pdf --output combined.pdf`.
- API endpoint for extraction: POST /extract with body `{"file": "base64encoded_content", "ocr": true}` and header `Authorization: Bearer $NANO_PDF_API_KEY`.
- Code snippet (Python):
  ```
  import requests
  response = requests.post('https://api.opencclaw.com/nano-pdf/extract', headers={'Authorization': f'Bearer {os.environ["NANO_PDF_API_KEY"]}'}, json={'file': 'base64data'})
  print(response.json()['text'])
  ```
- Config format: JSON for API bodies, e.g., `{"file": "path", "options": {"ocr_engine": "tesseract", "language": "en"}}`; CLI uses flag-based configs like `--config config.json`.

## Integration Notes
Integrate by setting `$NANO_PDF_API_KEY` for authenticated requests; for local use, install via `pip install nano-pdf` and import as a module. Combine with other tools: pipe CLI output to NLP libraries for mining, or use in Node.js via HTTP requests. For OCR, ensure Tesseract is installed and configured in your environment path. Test integrations in a sandbox to verify API rate limits (e.g., 100 requests/min).

## Error Handling
Check for common errors like file not found (exit code 404), invalid API keys (401), or OCR failures (e.g., no Tesseract installed). Use try-except in code: 
```
try:
    result = nano_pdf.extract('input.pdf')
except FileNotFoundError:
    print("Error: File does not exist.")
except Exception as e:
    print(f"API Error: {e} - Check $NANO_PDF_API_KEY.")
```
For CLI, parse stderr output; retry transient errors (e.g., network issues) with exponential backoff. Always validate inputs, like ensuring PDFs are not corrupted before processing.

## Example 1: Extract and Mine Text from a PDF
To extract text from a scanned invoice PDF and mine for product names:
1. Run: `nano-pdf extract --file invoice.pdf --output invoice_text.txt --ocr true`
2. Then: `nano-pdf mine --input invoice_text.txt --keywords "product" --output analysis.json`
This produces a JSON with keyword occurrences for further processing.

## Example 2: Fill and Manipulate a Form PDF
To fill a job application form and merge it with a cover letter:
1. Prepare data in JSON: `{"name": "John Doe", "position": "Engineer"}`
2. Execute: `nano-pdf fill --template application.pdf --data application_data.json --output filled_app.pdf`
3. Merge: `nano-pdf merge --files filled_app.pdf cover_letter.pdf --output final_packet.pdf`
Output is a single PDF ready for submission.

## Graph Relationships
- Related to: "ocr-tool" (for enhanced OCR capabilities), "document-parser" (for broader file type support), "text-analyzer" (for advanced mining integrations).
- Clusters: Connected via "community" cluster to skills like "data-extraction" and "automation-utils".
