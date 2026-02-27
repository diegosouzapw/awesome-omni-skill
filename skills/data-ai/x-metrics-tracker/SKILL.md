---
name: x-metrics-tracker
description: Track X account metrics (followers, posts, following) with daily snapshots via browser automation. Use when you need to monitor X profile growth, capture daily analytics, or view historical follower/post trends.
---

# X Metrics Tracker

Track @eva_uncensored X account metrics daily: posts, replies, impressions, follower count.

## Usage

### Capture Metrics (Agent Workflow)
1. Navigate to `https://x.com/eva_uncensored` via browser relay
2. Extract metrics from snapshot:
   - Posts count: from "XX posts" text
   - Followers: from "XXX Followers" link
   - Following: from "X Following" link
3. Save snapshot:
```bash
node ~/clawd/skills/x-metrics-tracker/capture.js '{"followers":138,"following":3,"posts":72}'
```

### View History
```bash
~/clawd/skills/x-metrics-tracker/history.sh [days]
```

### Data Location
- Daily snapshots: `~/clawd/skills/x-metrics-tracker/data/YYYY-MM-DD.json`
- Summary: `~/clawd/skills/x-metrics-tracker/data/summary.json`

## Metrics Tracked
- `followers`: Current follower count
- `following`: Current following count  
- `posts`: Total post count
- `date`: Snapshot date (auto-added)
- `timestamp`: Full ISO timestamp (auto-added)

## Automation
Add to HEARTBEAT.md for daily tracking (recommend: once per day, morning):

```markdown
### X Metrics (Daily)
- Navigate to `https://x.com/eva_uncensored`
- Extract followers, posts, following from profile
- Run: `node ~/clawd/skills/x-metrics-tracker/capture.js '{"followers":N,"posts":N,"following":N}'`
```

## Browser Relay Required
This skill uses browser automation to scrape X profile data since API access is limited.
