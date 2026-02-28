---
name: reddit-engagement
description: Research, monitor, and engage with Reddit communities — find relevant discussions, analyze sentiment, draft posts and comments, and track subreddit activity.
alwaysApply: false
---

# Reddit Engagement

You have access to the Reddit MCP tools for reading and writing to Reddit. Use these capabilities to help the user engage with Reddit communities effectively.

## Available Tools

### Read Operations
- **reddit_search_posts** — Find posts by keyword across all of Reddit or within a specific subreddit
- **reddit_get_subreddit** — Browse a subreddit's feed (hot, new, top, rising)
- **reddit_get_comments** — Read the comment thread on a specific post
- **reddit_get_subreddit_info** — Get subreddit metadata (subscribers, description, rules)
- **reddit_search_subreddits** — Discover subreddits by topic or name
- **reddit_get_user** — Look up a user's public profile
- **reddit_get_me** — Check the authenticated account

### Write Operations (require user confirmation)
- **reddit_submit_post** — Create a new text or link post
- **reddit_comment** — Reply to a post or comment
- **reddit_vote** — Upvote, downvote, or remove vote

## Workflow Patterns

### Research & Monitoring
1. Use `reddit_search_subreddits` to find relevant communities
2. Use `reddit_get_subreddit` with sort=new or sort=top to monitor activity
3. Use `reddit_get_comments` to deep-dive into discussions
4. Summarize findings with links

### Content Creation
1. Research the subreddit first — check rules, flair requirements, posting norms
2. Draft the post title and body, matching the community's tone
3. Always confirm with the user before using write tools
4. After posting, provide the permalink for tracking

### Community Analysis
1. Use `reddit_get_subreddit_info` to understand community size and description
2. Use `reddit_search_posts` with different time filters to see trending topics
3. Read comment threads to gauge sentiment
4. Report findings in a structured format

## Important Guidelines

- **Always confirm before write operations** — never post, comment, or vote without explicit user approval
- **Respect subreddit rules** — check r/subreddit info before posting
- **Be transparent** — if content is AI-assisted, follow the subreddit's disclosure rules
- **Rate limits** — Reddit allows ~60 requests/minute; space out bulk operations
- **Fullnames** — Posts use `t3_` prefix, comments use `t1_` prefix. These are returned in search/feed results.
