---
name: x-engagement
description: Auto-find and reply to people with relevant pain points when you post on X. Two rounds max (8 replies total) to stay safe from throttling.
---

# X Engagement Skill 🎯

Automatically find people discussing pain points related to your X post and reply with helpful context + link.

## Trigger

Run manually after posting, or hook to post notifications:
```bash
# Manual trigger
x-engage <post-url>
```

## Workflow

### 1. Analyze Post
- Extract topic, pain point addressed, keywords
- Identify the solution being offered

### 2. Search for Targets
Find people who:
- Are complaining about the problem you solved
- Are discussing the topic with questions
- Are building similar things and would benefit
- Are new users who need the tip

Search queries to try:
- Direct pain: `"[problem] frustrating" OR "[problem] annoying"`
- Questions: `"how to [topic]" OR "[topic] help"`
- Ecosystem: `@[relevant_account] [topic]`
- New users: `"just installed [product]" OR "trying [product]"`

### 3. Filter Targets
**Good targets:**
- Recent posts (< 24h ideal, < 48h max)
- Have engagement (not bots)
- Genuine pain/question (not promotional)
- Not already replied to by us

**Skip:**
- Promotional/spam accounts
- Posts with 0 engagement and suspicious patterns
- Already have 50+ replies (won't be seen)
- Competitors or hostile accounts

### 4. Craft Replies
Each reply must:
- Be contextual to THEIR specific post
- Offer genuine value (not just a link drop)
- Include link to your post naturally
- Be under 280 chars
- Sound human, not templated

**Reply formula:**
`[Acknowledge their point] + [How this helps] + [Link]`

**Examples:**
- "The context struggle is real. This saves state BEFORE it fills — game changer. [link]"
- "Already built exactly this. [link]"
- "Pro tip for new users: grab this early. Trust me. [link]"

### 5. Post Replies (Rate Limited)

**Round 1:** 4 replies
**Round 2:** 4 replies (after 30+ min gap)
**Total:** 8 max per post

**Spacing:** 3-5 seconds between replies within a round.

## Safety Limits

| Metric | Limit |
|--------|-------|
| Replies per day | 8-10 max |
| Rounds per post | 2 |
| Replies per round | 4 |
| Gap between rounds | 30+ minutes |
| Gap between replies | 3-5 seconds |

## Commands

```bash
# Full engagement run
bird search "[keywords]" -n 15
bird reply <target-url> "message"

# Check your recent posts
bird user-tweets @YourHandle -n 5

# Monitor replies to your post
bird replies <your-post-url>
```

## Tracking

Log engagements to `memory/x-engagement-log.md`:
```markdown
## [Date] - [Post Topic]
Post: [url]
Round 1:
- @user1: [their topic] → [reply url]
- @user2: [their topic] → [reply url]
Round 2:
- @user3: [their topic] → [reply url]
```

## Example Run

**Your post:** "Your AI agent mid-task: 'What was I doing again?' We fixed this. session-continuity writes state BEFORE context fills."

**Search queries:**
- `"agent forgets" OR "context lost" OR "mid-task amnesia"`
- `"OpenClaw memory" OR "Clawdbot context"`
- `"just installed OpenClaw"`
- `@openclaw skill`

**Targets found:**
1. @user complaining about manual context management
2. @user wanting to build memory agent
3. @user just installed OpenClaw
4. @user making OpenClaw tutorial

**Replies posted:** 4 contextual replies linking your post.

## Notes

- Quality > quantity. 4 great replies beat 10 generic ones.
- Read their post carefully. Reference something specific.
- Don't reply to the same person twice.
- If rate limited, stop immediately and wait 24h.
