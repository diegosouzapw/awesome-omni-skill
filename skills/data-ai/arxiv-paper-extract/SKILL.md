---
name: arxiv-paper-extract
description: Extract, translate and save arXiv CS.CV papers for a specific date. Use when user asks to fetch arXiv papers, download paper lists, extract CV papers, translate paper titles to Chinese, or save paper metadata from arxiv.org/list/cs.CV.
allowed-tools:
  - Bash
  - Read
  - Write
  - Task
---

# arXiv CS.CV Paper Extraction Skill

Extract papers from arXiv Computer Vision category for a specific date, translate titles to Chinese using a subagent, and save to JSON.

## Workflow

### Step 1: Download the arXiv page

```bash
curl -s "https://arxiv.org/list/cs.CV/recent?skip=0&show=2000" -o /tmp/arxiv_cs_cv.html
```

### Step 2: Parse papers for the target date

Use Python to extract papers. The HTML structure uses `<h3>` tags for date headers (e.g., `<h3>Thu, 25 Dec 2025`).

```python
import re
from html import unescape

with open('/tmp/arxiv_cs_cv.html', 'r', encoding='utf-8') as f:
    html = f.read()

# Find the target date section
# Format: "<h3>Day, DD Mon YYYY" e.g. "<h3>Thu, 25 Dec 2025"
date_pattern = r'<h3>{weekday}, {day} {month} {year}'
# Example: r'<h3>Thu, 25 Dec 2025'

start_match = re.search(date_pattern, html)
if start_match:
    start_pos = start_match.start()
    # Find next date section (previous day)
    remaining = html[start_pos + 10:]
    end_match = re.search(r'<h3>\w+, \d+ \w+ \d{4}', remaining)
    if end_match:
        section = html[start_pos:start_pos + 10 + end_match.start()]
    else:
        section = html[start_pos:]

# Extract paper IDs and titles
id_pattern = r'<a href ="/abs/(\d+\.\d+)"'
title_pattern = r"<div class='list-title mathjax'>(.*?)</div>"

ids = re.findall(id_pattern, section)
titles_raw = re.findall(title_pattern, section, re.DOTALL)

papers = []
for paper_id, title_html in zip(ids, titles_raw):
    title = re.sub(r'<[^>]+>', '', title_html)
    title = re.sub(r'\s+', ' ', title).strip()
    title = unescape(title)
    # Remove "Title:" prefix if present
    if title.startswith("Title:"):
        title = title[6:].strip()
    papers.append({
        "id": f"https://arxiv.org/abs/{paper_id}",
        "title_en": title,
        "title_cn": ""
    })
```

### Step 3: Save extracted papers to temporary JSON

Save the extracted papers first, then delegate translation to a subagent.

```python
import json
from pathlib import Path

# Format: YYYY-MM-DD.json (e.g., 2025-12-25.json)
# Convert month name to number: Dec -> 12
month_map = {"Jan": "01", "Feb": "02", "Mar": "03", "Apr": "04", "May": "05", "Jun": "06",
             "Jul": "07", "Aug": "08", "Sep": "09", "Oct": "10", "Nov": "11", "Dec": "12"}
month_num = month_map["{month}"]
filename = f"{year}-{month_num}-{day:02d}.json"

output_path = Path.home() / "paper_list" / filename
output_path.parent.mkdir(parents=True, exist_ok=True)

with open(output_path, 'w', encoding='utf-8') as f:
    json.dump(papers, f, ensure_ascii=False, indent=2)
```

### Step 4: Translate titles using Task subagent (CRITICAL)

**MUST use the Task tool to spawn a `general` subagent for translation.** This is the recommended approach for batch translation tasks.

Call the Task tool with the following parameters:

```
Tool: Task
Parameters:
  description: "Translate arXiv paper titles to Chinese"
  subagent_type: "general"
  prompt: |
    Read the JSON file at ~/paper_list/{YYYY-MM-DD}.json containing arXiv paper metadata.
    
    For each paper, translate the "title_en" field to Chinese and update the "title_cn" field.
    
    Translation guidelines:
    1. Keep model names as-is: SAM, SAM2, NeRF, CLIP, Transformer, Mamba, Diffusion, etc.
    2. Keep acronyms unchanged: VLM, LLM, 3D, 2D, 6DoF, MRI, CT, GAN, CNN, RNN, etc.
    3. Translate descriptive phrases accurately
    4. Maintain colon separator for titled papers (use Chinese colon：)
    5. Remove any "Title:" prefix from English titles before translating
    
    Save the updated JSON back to the same file.
    
    Return the count of translated papers and show 3-5 example translations.
```

## Output Format

File naming: `YYYY-MM-DD.json` (e.g., `2025-12-25.json`)
Location: `~/paper_list/`

```json
[
  {
    "id": "https://arxiv.org/abs/2512.21338",
    "title_en": "HiStream: Efficient High-Resolution Video Generation via Redundancy-Eliminated Streaming",
    "title_cn": "HiStream：通过消除冗余的流式传输实现高效高分辨率视频生成"
  }
]
```

## Date Format Reference

arXiv uses format: `{Weekday}, {Day} {Month} {Year}`
- Weekday: Mon, Tue, Wed, Thu, Fri
- Day: 1-31 (no leading zero)
- Month: Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec
- Year: 2025

Example: `Thu, 25 Dec 2025`

## Notes

- arXiv cs.CV typically has 50-150 papers per day
- The page includes both primary cs.CV papers and cross-listed papers
- Cross-listed papers appear after the main cs.CV section
- Ensure all papers are captured by checking the count matches the arXiv listing
- **Always use Task subagent for translation** - do not attempt to translate manually in the main workflow
