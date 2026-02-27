---
name: citation-link-validator
description: Validates footnote links in articles to prevent broken 404 URLs. Use when Claude needs to generate content with reference citations (research reports, technical documentation, academic articles). Supports two modes - (1) Real-time validation mode - validates each URL during content generation, ensuring zero broken links; (2) Post-validation mode - checks all footnotes in existing documents. Suitable for high-quality citation scenarios including web search data compilation, literature citation, and fact-checking tasks.
---

# Citation Link Validator

This skill provides footnote link validation functionality to ensure all reference links in generated Markdown documents are valid and accessible.

## Core Features

1. **Real-time Single URL Validation**: Instantly verify each link during content generation
2. **Batch Document Validation**: Concurrently check all footnotes in an entire document
3. **Smart Filtering**: Automatically exclude broken links, ensuring zero-failure output
4. **Detailed Reports**: Color-coded display of valid/invalid/suspicious links

## Usage Scenarios & Mode Selection

### Mode A: Real-time Validation Mode (Recommended)

**When to Use**: 
- User requests new research reports or technical documentation
- Need to cite sources from web search results
- Requirements include "ensure all links are valid" or "no broken links"

**Trigger Keywords**:
- "generate report with references"
- "cite sources and ensure links are valid"
- "no broken links"
- "validate all cited URLs"

### Mode B: Post-validation Mode

**When to Use**:
- User uploads existing Markdown documents with footnotes
- Need to check link status in existing articles
- Regular document maintenance

**Trigger Keywords**:
- "check links in this document"
- "validate footnotes in my uploaded file"
- "which links are broken"

---

## Mode A: Real-time Validation Workflow

### Core Principles

**Core Philosophy**: Validate first, then add — never include unvalidated URLs in the final article

**Workflow**:
1. Use `web_search` to find relevant data
2. Extract candidate URLs from search results
3. **Before adding footnotes**, validate each URL for accessibility
4. Only add verified URLs to footnotes
5. If URL is broken, immediately search for alternative sources
6. Ensure all footnotes in final article are valid links

### Step-by-Step Guide

#### Step 1: Search and Obtain Candidate URLs

```python
# Use web_search to find relevant topics
search_results = web_search("AI ethics research site:edu OR site:org")

# Extract URLs from results
candidate_urls = [result['url'] for result in search_results]
```

#### Step 2: Real-time Validation of Each URL

Use the `check_single_url.py` script to validate individual URLs:

```bash
python /mnt/skills/user/citation-link-validator/scripts/check_single_url.py "<URL>"
```

**Return Values**:
- Exit code 0: URL is valid (HTTP status code < 400)
- Exit code 1: URL is broken or inaccessible

**Python Call Example**:
```python
import subprocess

def is_url_valid(url: str, timeout: int = 10) -> bool:
    """Validate if a single URL is valid"""
    result = subprocess.run(
        ['python', '/mnt/skills/user/citation-link-validator/scripts/check_single_url.py',
         url, '--timeout', str(timeout), '--quiet'],
        capture_output=True
    )
    return result.returncode == 0

# Usage example
url = "https://www.nature.com/articles/s41586-023-12345-6"
if is_url_valid(url):
    print(f"URL is valid: {url}")
else:
    print(f"URL is broken: {url}")
```

#### Step 3: Filter and Build Footnote List

```python
valid_footnotes = []
footnote_counter = 1

for result in search_results:
    url = result['url']
    title = result['title']
    
    # Real-time validation
    if is_url_valid(url):
        # URL is valid, add to footnotes
        valid_footnotes.append({
            'id': footnote_counter,
            'title': title,
            'url': url,
            'description': result.get('description', '')
        })
        footnote_counter += 1
    else:
        # URL is broken, log and skip (or search for alternatives)
        print(f"Skipping broken link: {url}")
```

#### Step 4: Generate Final Article

```python
# Generate article content (using valid footnotes)
article = generate_article_content(valid_footnotes)

# Generate footnote section
footnote_section = "\n## References\n\n"
for fn in valid_footnotes:
    footnote_section += f'[^{fn["id"]}]: [{fn["title"]}]({fn["url"]}) "{fn["description"]}"\n'

final_article = article + "\n" + footnote_section
```

### Complete Example: Generating AI Ethics Report

```python
import subprocess

def is_url_valid(url: str) -> bool:
    """Validate if URL is valid"""
    result = subprocess.run(
        ['python', '/mnt/skills/user/citation-link-validator/scripts/check_single_url.py',
         url, '--quiet'],
        capture_output=True,
        timeout=15
    )
    return result.returncode == 0

# 1. Search for AI ethics-related data
search_results = [
    {'url': 'https://www.nature.com/articles/ai-ethics', 'title': 'AI Ethics Paper'},
    {'url': 'https://invalid-domain-404.com/article', 'title': 'Invalid Source'},
    {'url': 'https://www.unesco.org/ai-ethics', 'title': 'UNESCO AI Ethics'},
]

# 2. Validate and filter
valid_sources = []
for result in search_results:
    if is_url_valid(result['url']):
        valid_sources.append(result)
        print(f"✓ Valid: {result['title']}")
    else:
        print(f"✗ Broken: {result['title']} - Searching for alternatives...")
        # If broken, can perform additional searches for alternative URLs

# 3. Generate article (only includes valid sources)
article = f"""
# AI Ethics Development Report

According to recent research[^1], artificial intelligence ethics has become a global focus.
UNESCO has published relevant guidelines[^2].

## References

"""
for i, source in enumerate(valid_sources, 1):
    article += f'[^{i}]: [{source["title"]}]({source["url"]})\n'

print(article)
```

**Output Result**:
```
✓ Valid: AI Ethics Paper
✗ Broken: Invalid Source - Searching for alternatives...
✓ Valid: UNESCO AI Ethics

[Article contains only 2 valid footnotes]
```

### Strategies for Handling Broken URLs

When validation fails, you have the following options:

#### Option 1: Search for Alternative Sources (Recommended)

```python
if not is_url_valid(url):
    # Perform additional search using the same topic
    alternative_results = web_search(f"{title} {topic}")
    for alt in alternative_results:
        if is_url_valid(alt['url']):
            # Found valid alternative source
            url = alt['url']
            break
```

#### Option 2: Skip That Source

```python
if not is_url_valid(url):
    print(f"Skipping broken source: {title}")
    continue  # Don't add to footnotes
```

#### Option 3: Use Archived Version

```python
if not is_url_valid(url):
    # Try Internet Archive
    archive_url = f"https://web.archive.org/web/{url}"
    if is_url_valid(archive_url):
        url = archive_url
```

### Batch Validation Optimization

For multiple candidate URLs, you can validate concurrently:

```python
from concurrent.futures import ThreadPoolExecutor

def validate_batch(urls: list[str]) -> dict[str, bool]:
    """Concurrently validate multiple URLs"""
    results = {}
    with ThreadPoolExecutor(max_workers=5) as executor:
        futures = {executor.submit(is_url_valid, url): url for url in urls}
        for future in futures:
            url = futures[future]
            results[url] = future.result()
    return results

# Usage example
candidate_urls = ["https://example1.com", "https://example2.com", ...]
validation_results = validate_batch(candidate_urls)

# Only use valid URLs
valid_urls = [url for url, is_valid in validation_results.items() if is_valid]
```

---

## Mode B: Post-validation Workflow

### Use Cases

- Check user-uploaded Markdown documents
- Regular maintenance of existing article library
- Batch detection of multiple files

### Step 1: Save Document

```python
# If document is already uploaded
document_path = "/mnt/user-data/uploads/report.md"

# If need to save newly generated content
with open('/home/claude/article.md', 'w') as f:
    f.write(article_content)
```

### Step 2: Execute Batch Validation

```bash
python /mnt/skills/user/citation-link-validator/scripts/verify_links.py <markdown_file>
```

**Complete Command Example**:
```bash
python /mnt/skills/user/citation-link-validator/scripts/verify_links.py \
    /home/claude/report.md \
    --timeout 15 \
    --max-workers 10
```

### Step 3: Interpret Validation Report

The report outputs three categories of links:

```
======================================================================
Link Validation Report
======================================================================

✓ Valid Links (5 items):
  [^1] Nature Journal
       https://www.nature.com/articles/example (Status: 200)
  ...

✗ Broken Links (2 items):
  [^3] Old Article Link
       https://old-site.com/article
       Error: HTTP 404: Not Found
  ...

⚠ Suspicious Links (1 item):
  [^7] Unstable Website
       https://slow-site.com/page
       Warning: Connection error: [Errno 110] Connection timed out
  ...

======================================================================
Statistics Summary:
  Total Footnotes: 8
  Valid: 5
  Broken: 2
  Suspicious: 1
  Success Rate: 62.5%
======================================================================
```

### Step 4: Fix Broken Links

Based on report results:

```python
# For broken links, search for alternative sources
failed_footnotes = [
    {'id': 3, 'title': 'Old Article Link', 'url': 'https://old-site.com/article'}
]

for fn in failed_footnotes:
    # Search for alternative sources
    search_results = web_search(f"{fn['title']} {topic}")
    
    for result in search_results:
        if is_url_valid(result['url']):
            # Found valid alternative, update document
            update_footnote(fn['id'], result['url'])
            break
```

### Step 5: Re-validate

```bash
python /mnt/skills/user/citation-link-validator/scripts/verify_links.py /home/claude/report.md
```

Ensure all links are marked as green (valid).

---

## Footnote Format Specifications

### Standard Format

```markdown
[^number]: [Title](URL) "Description"
```

**Components**:
- `[^number]`: Footnote number, must be numeric (e.g., `[^1]`, `[^123]`)
- `[Title]`: Source title, enclosed in square brackets
- `(URL)`: Complete URL, enclosed in parentheses, must include `https://` or `http://`
- `"Description"`: Optional description text, enclosed in double quotes

### Correct Examples

```markdown
[^1]: [Nature Journal](https://www.nature.com/articles/s41586-023-12345-6) "AI ethics research paper, published in 2023"
[^2]: [UNESCO Official Website](https://www.unesco.org/en/artificial-intelligence/recommendation-ethics)
[^3]: [White House Memo](https://www.whitehouse.gov/wp-content/uploads/2023/10/Blueprint-for-an-AI-Bill-of-Rights.pdf) "AI Bill of Rights Blueprint"
```

### Incorrect Examples

```markdown
[1] Title https://example.com  ← Missing [^] symbols and brackets
[^1]: Title (https://example.com)  ← Title missing square brackets
[^1]: [Title](example.com)  ← URL missing https://
[^1]: [Title] (https://example.com)  ← Space between brackets and parentheses
```

---

## Best Practices

### In Real-time Validation Mode

1. **Prioritize Authoritative Sources**: 
   - Use `site:edu`, `site:gov`, `site:org` filters when searching
   - Prioritize academic journals, government agencies, international organizations

2. **Validate in Batches Then Select**:
   ```python
   # Get multiple candidate sources
   candidates = web_search(f"{topic} site:edu OR site:org", num_results=10)
   
   # Validate all, then select the 5 most reliable
   valid_candidates = [c for c in candidates if is_url_valid(c['url'])]
   best_sources = valid_candidates[:5]
   ```

3. **Set Validation Timeout**:
   - Default 10 seconds is usually sufficient
   - For slower sites, can increase to 15-20 seconds
   - Avoid excessively long timeouts that slow the entire process

4. **Log Validation Process**:
   ```python
   print(f"Validating: {url}")
   if is_url_valid(url):
       print(f"✓ Validation passed")
   else:
       print(f"✗ Validation failed, searching for alternatives...")
   ```

### In Post-validation Mode

1. **Regular Re-validation**:
   - New articles: Validate before publishing
   - Existing articles: Validate quarterly
   - Important documents: Validate monthly

2. **Batch Processing**:
   ```bash
   # Validate entire directory
   for file in /home/claude/articles/*.md; do
       python verify_links.py "$file"
   done
   ```

3. **Keep Correction Records**:
   - Note last validation date in document
   - Record which links were replaced
   - Keep backups before corrections

---

## Technical Details

### Validation Logic

#### check_single_url.py (Real-time Validation)

- **Request Method**: HEAD (saves bandwidth)
- **Timeout**: Default 10 seconds, customizable
- **User-Agent**: Simulates Chrome browser
- **Return**: Exit code 0 (valid) or 1 (broken)

**Determination Criteria**:
| HTTP Status Code | Result |
|-----------------|---------|
| 200-299 | Valid |
| 300-399 | Valid (redirect) |
| 400-499 | Broken (client error) |
| 500-599 | Broken (server error) |
| No status code | Broken (connection error) |

#### verify_links.py (Batch Validation)

- **Request Method**: HEAD
- **Concurrency**: Default 5 threads, customizable
- **Output**: Colored terminal report
- **Categories**: Valid/Broken/Suspicious

### Regular Expression

Footnote parsing pattern:
```python
pattern = r'\[\^(\d+)\]:\s*\[([^\]]+)\]\(([^)]+)\)(?:\s*"([^"]*)")?'
```

**Match Rules**:
- `\[\^(\d+)\]`: Capture numeric ID
- `\[([^\]]+)\]`: Capture title (excluding `]`)
- `\(([^)]+)\)`: Capture URL (excluding `)`)
- `(?:\s*"([^"]*)")?`: Optional description text

---

## FAQ

### Q1: Will real-time validation slow down article generation?

**A**: Slight impact, but can be optimized:
- Single URL validation typically 1-2 seconds
- Batch concurrent validation can speed up
- Validation cost is much lower than fixing broken links later

### Q2: How to handle sites requiring login?

**A**: 
- Validation will fail (returns 401 or 403)
- Recommend noting "subscription required" or "login required" in footnote description
- Prioritize finding publicly accessible alternative sources

### Q3: Can validated links still break later?

**A**: Yes. Validation only confirms accessibility at that moment. Recommendations:
- Prioritize citing stable authoritative websites
- Note validation date in document
- Regular re-validation (recommend quarterly)

### Q4: Can I validate only specific domain links?

**A**: Yes. Filter before generating footnotes:
```python
trusted_domains = ['nature.com', 'unesco.org', 'gov']
if any(domain in url for domain in trusted_domains):
    if is_url_valid(url):
        # Add to footnotes
```

### Q5: Validation failed but link is actually valid?

**A**: Possible reasons:
- Site has anti-scraping mechanisms
- Requires JavaScript rendering
- Geographic restrictions or IP blocking

Solutions:
- Use `web_fetch` tool for cross-validation
- Increase timeout: `--timeout 20`
- Can choose to keep after manual confirmation

---

## Summary

### Recommended Workflow

**When Generating New Content (Mode A)**:
```
1. web_search for data
2. Extract candidate URLs
3. Validate each URL in real-time
4. Only add valid URLs
5. Search for alternatives if broken
6. Generate final article (zero-failure guarantee)
```

**When Checking Existing Documents (Mode B)**:
```
1. Read document
2. Batch validate all footnotes
3. Review validation report
4. Search for alternative sources to fix broken links
5. Re-validate until all pass
```

### Core Value

- **Quality Improvement**: Ensure all cited sources are reliable and accessible
- **Time Saving**: Automated validation, avoid manual clicking
- **Zero Failure**: Real-time mode ensures no broken links in output
- **Professionalism**: Avoid readers encountering 404 errors, enhance document credibility
