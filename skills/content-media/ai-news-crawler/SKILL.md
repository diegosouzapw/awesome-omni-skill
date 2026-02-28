---
name: ai-news-crawler
description: Trigger phrase \"watching news!\"; Create a JSON file named with the current timestamp only in the current directory, crawl 10 AI-related news in descending order of time and output Chinese summaries
---

# Purpose
Crawl the latest AI-related news from multiple websites, merge and deduplicate them, select 10 in descending order of time, unify summaries into Chinese, and write to a JSON file.

# When to Trigger
When the request contains the phrase "Go, Little Plane".

# Inputs
- optional keywords?: Topic keywords (optional)
- optional time_window?: Time window (default: 144h)

# Output
- news_json_file_only

# Constraints
- Create only one file: <YYYYMMDD_HHMMSS>.json in the current working directory; Do not output any text or paths in the conversation.
- Each record must include fields: title, source, url, published_at (ISO8601), summary_zh, language (zh/en/mixed).
- Fixed quantity: 10 entries, sorted in descending order of published_at; Strict deduplication (using url and title as keys).
- Summaries for English content must be in Chinese; Retain the original language identifier in the language field.
- Avoid crawling paywalled or login-required pages; Exclude ads and sponsored content; Do not write sensitive information or keys.
- If the publication time is missing, use the explicit time on the page; If unavailable, exclude the entry or mark published_at: null and lower its priority—do not guess.

# Candidate Sources
- The Verge / AI tag
- TechCrunch / AI tag
- Liangziwei (Quantum Bits) / AI section
- 36Kr / Artificial Intelligence tag
- Official Announcements / Anthropic, Google AI Blog (select latest announcements)

# Steps
1. Confirm time_window and keywords (if provided).
2. Iterate through candidate sources, crawl recent content lists, and parse title/url/published_at and body summaries.
3. Condense and translate English summaries into Chinese, maintaining neutrality and information density.
4. Normalize timestamps to ISO8601, merge lists, and deduplicate by url/title.
5. Sort in descending order of published_at, select the top 10 entries; Write to <YYYYMMDD_HHMMSS>.json as a JSON array.
6. Do not output any text in the conversation; Only create the file.

# Output Example (Structure)
[
  {
    "title": "示例标题",
    "source": "TechCrunch",
    "url": "https://example.com/news/123",
    "published_at": "2025-11-16T08:30:00Z",
    "summary_zh": "用中文概括要点，80-150字，覆盖背景、事件与影响。",
    "language": "en"
  }
]
