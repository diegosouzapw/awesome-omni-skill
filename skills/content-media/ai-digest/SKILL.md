---
name: ai-digest
description: Digest AI/tech articles into structured learning documents. Use when user says "digest this article", "/ai-digest", "analyze this AI news", or provides URL/content to summarize.
version: 2.0.0
---

# AI Digest

Analyzes AI/tech articles, blog posts, or content and creates structured learning documents optimized for quick reference and practical application.
Output is blog-ready with YAML frontmatter for Astro Content Collections.

---

## Execution Algorithm

### Step 1: Parse Input

Extract from user message:

| Element | What to Extract |
|---------|----------------|
| **URL** | Any URL (https://...) |
| **Focus** | User's specific interest (e.g., "focus on API changes") |
| **Direct Content** | Text content if no URL provided |

**Examples:**
- `/ai-digest https://anthropic.com/news/...` → URL extracted
- `/ai-digest "Focus on breaking changes" https://...` → URL + Focus
- `/ai-digest "Analyze: [pasted content]"` → Direct content

---

### Step 2: Resolve Content

Determine the content source using the following priority:

**Priority 1 — Direct content provided:**
If the user pasted article text (beyond just a URL), use it directly. Skip WebFetch entirely. This is the most reliable path.

**Priority 2 — Fetchable URL only:**
If only a URL is provided (no pasted content), attempt WebFetch:

```python
WebFetch(
    url=extracted_url,
    prompt="""
    Extract the main content of this article.

    Focus on:
    - Main topic and key points
    - Technical details and specifications
    - Code examples
    - Changes or new features
    - Practical applications

    Ignore navigation, ads, and footer content.
    """
)
```

**Priority 3 — Auth-gated URL with no content:**
If the URL is from a known auth-gated domain AND no direct content is provided, do NOT attempt WebFetch. Instead, ask the user to paste the content.

Known auth-gated domains (non-exhaustive):
- `linkedin.com` — requires login
- `x.com`, `twitter.com` — requires login
- `medium.com` — may be paywalled
- `substack.com` — may be paywalled
- `notion.so` — requires access
- `docs.google.com` — requires access

**When both URL + direct content are provided:**
Use the direct content for analysis. Retain the URL only as a source reference in the output document.

---

### Step 3: Analyze Content

Analyze the fetched content to identify:

| Category | What to Extract |
|----------|----------------|
| **Topic & Context** | What is this about? Why does it matter? |
| **Key Changes/Concepts** | New features, updates, or core concepts |
| **Practical Applications** | How to use this in real projects |
| **Code Examples** | Runnable code snippets |
| **Migration/Upgrade Notes** | Breaking changes, migration steps |
| **Limitations** | Known issues, constraints, gotchas |

Also extract metadata for frontmatter:

| Field | How to Derive |
|-------|---------------|
| **title** | Article title or main topic |
| **description** | 1-2 sentence summary under 160 characters |
| **tags** | 3-7 lowercase kebab-case keywords from content topics |
| **source** | Original article URL |
| **lang** | Always `en` (output is always in English) |

**Consider user's focus** if provided (e.g., "API changes only").

---

### Step 4: Read Configuration

Read `~/.claude/skills/learning-summary/config.yaml`:

```yaml
learning_repo: "/Users/jaykim/Documents/Projects/ai-learning"
auto_commit: false
auto_push: false
```

**Why reuse learning-summary config?**
- Keeps all learning documents in one repository
- Shares git settings (auto_commit, auto_push) for consistency
- No duplicate configuration needed
- Users only configure once

**Output directory**: Always use `"blog/src/content/digests"` subfolder to separate article digests from conversation summaries.

**If config not found**: Ask user for learning repository path. You can create the config file or use the provided path for this session only.

---

### Step 5: Generate Filename

Create descriptive filename:

**IMPORTANT**: Run `date '+%Y-%m-%d'` to get the exact current date. Never estimate.

**Format**: `YYYY-MM-DD-ai-[topic-slug].md`

**Examples**:
- `2026-01-25-ai-claude-sonnet-4-5-release.md`
- `2026-01-25-ai-openai-gpt5-features.md`
- `2026-01-25-ai-langchain-updates.md`

**Topic slug rules:**
- Lowercase, kebab-case
- Max 4-5 words
- Descriptive and searchable

---

### Step 6: Generate Document

**First, classify the content type** to determine which template variant to use:

| Content Type | Characteristics | Template Variant |
|-------------|-----------------|------------------|
| **Technical** | Code changes, API updates, library releases, tutorials | Full template (with code examples, Before/After) |
| **Strategic/Opinion** | Industry analysis, trend pieces, strategy frameworks | Strategy template (with frameworks, comparison tables) |
| **News/Announcement** | Product launches, funding, partnerships | News template (concise, fact-focused) |

#### Template: Core (always included)

```markdown
---
title: "[Article Title or Main Topic]"
date: YYYY-MM-DD
description: "[1-2 sentence summary under 160 chars]"
category: digests
tags: ["ai", "topic", "subtopic"]
source: "https://original-article-url"
lang: en
draft: false
---

## Summary

[1-2 paragraph summary in English]
[What this is about and why it matters]

## Key Concepts

### [Concept 1]
- **What**: [Description]
- **Why**: [Reasoning or benefit]
- **Impact**: [Who/what is affected]

### [Concept 2]
...

## Practical Applications

### Use Case 1: [Scenario]
[How to apply this in real projects]

### Use Case 2: [Scenario]
...

## Limitations & Gotchas

- [Warning 1]
- [Warning 2]
- [Tip 1]

## References

- [Original article URL]
- [Related documentation]
- [Related tools/libraries]

## Next Steps

- [ ] [Action item 1]
- [ ] [Action item 2]
- [ ] [Topic to explore further]

---

**Notes**:
[Personal notes or context]
```

**Important formatting rules**:
- Do NOT include `# Title` heading in body (the blog layout renders title from frontmatter)
- Do NOT use blockquote metadata (`> **Source**: ...`) — all metadata goes in frontmatter
- Body starts with `## 요약 (Summary)` or first applicable section
- `category` is always `digests`
- `tags` are lowercase, kebab-case, as a YAML array
- `source` is the original article URL (omit if direct content with no URL)
- `description` should be under 160 characters for SEO
- Empty sections should be omitted entirely

#### Template: Technical Extensions (add when content type is Technical)

Include these sections when the article contains code, API changes, or migration guides:

```markdown
## Code Examples

### Example 1: [What it demonstrates]
```language
[Runnable code]
```

**Explanation**: [What this code does]

## Before/After Comparison

### Before (Old Way)
```language
[Old way]
```

### After (New Way)
```language
[New way]
```

**Key Differences**: [Key differences]
```

#### Template: Strategy Extensions (add when content type is Strategic/Opinion)

Include these sections when the article discusses frameworks, trends, or strategic analysis:

```markdown
## Key Framework

[Visual or structured representation of the article's main framework]
[Use tables, diagrams (text-based), or hierarchical lists]

## Case Comparisons

| Item | [Case A] | [Case B] |
|------|----------|----------|
| ... | ... | ... |
```

**Section inclusion rules:**
- Always: Summary, Key Concepts, Practical Applications, References
- Technical content: + Code Examples, Before/After Comparison
- Strategic content: + Key Framework, Case Comparisons
- Optional: Limitations & Gotchas (if important warnings exist), Next Steps
- Do not include empty sections. If a section has no meaningful content, omit it.

---

### Step 7: Save Document

**Save location**: `{learning_repo}/blog/src/content/digests/{filename}`

**Example**: `/Users/jaykim/Documents/Projects/ai-learning/blog/src/content/digests/2026-01-25-ai-claude-sonnet-4-5-release.md`

**Deduplication check**: Before saving, use Glob to check if a file with the same date and similar topic slug already exists in the digests directory. If a match is found, ask the user whether to overwrite, append, or create with a suffix (e.g., `-2`).

**Save**: Use the Write tool directly. Do NOT run `mkdir -p` — the `blog/src/content/digests/` directory should already exist in a configured learning repo. If Write fails due to a missing directory, then create it.

---

### Step 8: Git Operations (if auto_commit)

Only if `auto_commit: true` in config:

```bash
cd "$learning_repo"
git add "blog/src/content/digests/$filename"
git commit -m "Add AI digest: $topic

Co-Authored-By: Claude <model> <noreply@anthropic.com>"
```

**Note**: Replace `<model>` with the actual model name being used (e.g., "Opus 4.6", "Sonnet 4.5"). Do not hardcode a specific model name.

If `auto_push: true`, also run:
```bash
git push
```

When pushed to main, GitHub Actions will automatically build and deploy the blog.

---

### Step 9: Confirm to User

Show the user:
- **Saved**: Full path of saved file
- **Captured**: Brief summary of what was captured (bulleted list of key topics)
- **Source**: Original URL (if applicable)
- **Git**: Commit status (only if auto_commit enabled)
- **Blog**: URL where post will appear after deploy

Do not use emojis in the confirmation output. Keep it concise and scannable.

---

## Trigger Phrases

**English:**
- "digest this article"
- "analyze this AI news"
- "summarize this tech blog"
- "/ai-digest [URL or content]"

**Korean:**
- "이 글 정리해줘"
- "AI 뉴스 요약"
- "기술 블로그 분석"

---

## Quick Reference

### When to Use

**Use this skill when**:
- User provides URL to AI/tech article
- User pastes content to analyze
- Need to capture rapidly changing AI updates
- Want structured notes for future reference

**Skip when**:
- General conversation summary (use learning-summary)
- Code review (use other tools)
- Already documented elsewhere

---

## Error Handling

| Scenario | Response |
|----------|----------|
| Auth-gated URL, no content | Ask user to paste the article content. Do NOT attempt WebFetch. |
| Auth-gated URL + content provided | Use direct content. Retain URL as source reference only. |
| WebFetch fails | `Error: Failed to fetch content. You can paste the content directly.` |
| Config not found | Ask user for ai-learning repository path |
| Write fails (missing directory) | Create directory with `mkdir -p`, then retry Write |
| Write permission denied | `Error: Cannot write to {path}. Check permissions.` |
| Git not initialized | `Warning: Not a git repository. Document saved but not committed.` |
| Empty content | `Error: No content to analyze. Please provide URL or text.` |
| Duplicate file exists | Ask user: overwrite, append, or save with `-2` suffix |

---

## Configuration

Reuses `~/.claude/skills/learning-summary/config.yaml`:

```yaml
# Dedicated AI learning repository (absolute path)
learning_repo: "/Users/jaykim/Documents/Projects/ai-learning"

# Auto-commit to git
auto_commit: false

# Auto-push to remote (requires auto_commit: true)
auto_push: false
```

**Output directory**: Always uses `blog/src/content/digests/` subfolder (hardcoded).

**No separate config needed** - shares learning_repo and git settings with learning-summary for consistency.

---

## Examples

### Example 1: URL Digest (fetchable)

```
User: /ai-digest https://anthropic.com/news/claude-sonnet-4-5

Claude: [Fetches content via WebFetch, classifies as Technical, generates document]

Saved: .../blog/src/content/digests/2026-01-25-ai-claude-sonnet-4-5-release.md

Captured:
- New Claude Sonnet 4.5 features and performance benchmarks
- Migration guide from previous versions
- Code examples showing before/after API changes
```

### Example 2: Auth-gated URL + pasted content

```
User: /ai-digest https://linkedin.com/posts/someone/...
[pasted article text]

Claude: [Detects LinkedIn URL + direct content -> uses direct content, skips WebFetch]
[Classifies as Strategic/Opinion, uses strategy template variant]

Saved: .../blog/src/content/digests/2026-01-25-ai-agent-economy-strategy.md
Source: https://linkedin.com/posts/someone/...

Captured:
- Agent economy framework and 3 survival conditions
- Case comparisons (Supabase vs competitors, Resend vs SendGrid)
```

### Example 3: Focused Analysis

```
User: /ai-digest "Focus on breaking changes only" https://openai.com/blog/gpt-5

Claude: [Fetches content, applies user focus filter to analysis]

Saved with focus on:
- API breaking changes and deprecated parameters
- Migration steps with before/after code examples
```

### Example 4: Auth-gated URL only (no content)

```
User: /ai-digest https://linkedin.com/posts/someone/...

Claude: LinkedIn requires authentication, so I can't fetch this content directly.
Please paste the article text and I'll digest it.
```

---

## Related Skills

- `learning-summary`: For conversation summaries (saves to `blog/src/content/learnings/`)
- `project-insight`: For codebase analysis

---

## Tips

1. **Batch processing**: Digest multiple articles in one session
2. **Tag consistently**: Use consistent tags for searchability
3. **Update existing**: If topic already documented, offer to update
4. **Link related**: Suggest related articles from ai-learning repo
