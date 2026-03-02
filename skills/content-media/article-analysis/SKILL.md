---
name: article-analysis
description: Analyze blog posts and web articles by fetching content from URLs. Use when the user mentions blog post, article, Substack, Medium, web page, newsletter, or provides a URL to analyze.
---

# Web Article Analysis

Analyze blog posts, newsletters, and web articles to generate structured reports with summaries, key points, and insights.

## When to Use

Activate this skill when the user:
- Mentions "blog", "article", "post", "Substack", "Medium", "newsletter"
- Provides a URL they want analyzed
- Asks to summarize or analyze a web page
- Wants to extract insights from online content

## Instructions

1. **Get the URL** - Ask the user for the URL if not provided
2. **Fetch the webpage content** using WebFetch
3. If fetch fails:
   - Inform user: "Could not fetch content from [url]"
   - Suggest: "Try copying the content manually to inbox/ and use /analyze"
   - Stop here
4. **Read the analysis prompt** from `prompts/article.md`
5. **Extract the article title** from the page content
6. **Generate analysis** following the prompt structure exactly
7. **Save the report** to `reports/articles/YYYY-MM-DD_sanitized-title.md` where:
   - YYYY-MM-DD is today's date
   - sanitized-title is the title in lowercase, spaces replaced with hyphens, special chars removed
8. **Update the activity log** at `logs/YYYY-MM-DD.md`:
   - Create file if it doesn't exist
   - Add entry under "## Articles Read" section
   - Format: `- [Title](../reports/articles/filename.md) - HH:MM`
9. **Confirm to user** what was saved and where

## Report Format

Include this header in the report:
```markdown
# [Article Title]

**Source**: [URL]
**Date**: YYYY-MM-DD
**Type**: Article

---

[Analysis content following prompts/article.md structure]

---

## My Notes

[Empty space for user notes]
```

## Error Handling

- If URL is invalid: Ask user for correct URL
- If WebFetch fails: Suggest manual copy to inbox/
- If prompts/article.md missing: Use prompts/default.md or basic structure

## Related

- Slash command equivalent: `/read <url>`
- Prompt file: `prompts/article.md`
- Output location: `reports/articles/`
