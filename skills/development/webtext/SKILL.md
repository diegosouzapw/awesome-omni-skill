---
name: WebText
description: Fetch a web page (single URL) and return readable plain text extracted from HTML.
metadata: {"openclaw":{"requires":{"bins":["python"]}}}
---

Use this skill when the user wants the plain text of a referenced web page.

How to run:
- Use the Exec Tool to run the Python script in this skill folder.
- The script prints a JSON object to stdout: { url, chars, text }

Install deps (once, on the host):
- python -m pip install -r "{baseDir}/requirements.txt"

Command template (Windows-friendly):
- python "{baseDir}/WebText.py" --url "<URL>" --max-chars 20000 --timeout-ms 15000

Return the JSON stdout to the user. If the request fails (HTTP error, timeout, unsupported content-type), return the error message.
