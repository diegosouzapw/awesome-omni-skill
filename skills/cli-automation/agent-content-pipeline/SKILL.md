---
name: agent-content-pipeline
description: Safe content workflow (drafts/reviewed/revised/approved/posted) with human-in-the-loop approval. pi rewrites drafts automatically on feedback. Post to LinkedIn, X, Reddit, dev.to, and Hashnode. Use when setting up a content pipeline, drafting content, managing review threads, or posting approved content.
---

# Content Pipeline Skill

Safe content automation with human-in-the-loop approval. Draft → Review → Revise (via pi) → Approve → Post.

## Setup

```bash
npm install -g agent-content-pipeline
content init .
```

This creates:
- `drafts/` — write your drafts here
- `reviewed/` — human reviewed, pi is rewriting
- `revised/` — rewritten, ready for another look
- `approved/` — human-approved, ready to post
- `posted/` — archive after posting
- `templates/` — platform templates

## Your Permissions

✅ **Can do:**
- Write to `drafts/`
- Read all content directories
- Add notes: `content thread <file> --from agent`
- Run `content list` to see pending content

❌ **Cannot do:**
- Move files between folders (the pipeline does this automatically)
- Move files to `approved/` or `posted/` (human only)
- Post content
- Set `status: approved`

## How the Pipeline Works

When the human reviews a draft and gives feedback, **pi automatically rewrites the draft** and moves it to `revised/`. You do not need to revise drafts yourself.

```
drafts/ → reviewed/ → revised/ → approved/ → posted/
  you       human      pi          human       human
  write     reviews    rewrites    approves    posts
```

Your job is to write good initial drafts. The rewrite loop is handled automatically.

## Creating a Draft

File naming: `YYYY-MM-DD-<platform>-<slug>.md`

```yaml
---
platform: linkedin    # linkedin | x | reddit | devto | hashnode
title: "Required for reddit, devto, hashnode"
status: draft
subreddit: programming  # Required for Reddit
tags: [tag1, tag2]      # Optional, used by devto/hashnode
---

Your content here.
```

## Platform Guidelines

### LinkedIn
- Professional but human
- 1-3 paragraphs ideal
- End with question or CTA
- 3-5 hashtags at end

### X (Twitter)
- 280 chars per tweet
- Use `---` to separate tweets in a thread
- Punchy, direct
- 1-2 hashtags max

### Reddit (experimental)
- Title required in frontmatter
- Match each subreddit's rules and tone

### dev.to / Hashnode
- Full markdown article
- Title required in frontmatter
- Tags as array of slugs

## Commands Reference

```bash
content list                    # Show all folders with timestamps
content review <file>           # Review: give feedback (pi rewrites) or approve
content edit <file>             # Open in editor
content post <file>             # Post (prompts for confirmation)
content post <file> --dry-run   # Preview without posting
content thread <file>           # Add a note to the feedback thread
content platforms               # List available platforms
```

## Security Model

- ✅ Agent drafts content
- ❌ Agent cannot approve or post (human only)

Posting is handled manually via CLI.
