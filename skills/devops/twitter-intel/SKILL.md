---
name: twitter-intel
description: Real-time X/Twitter intelligence - analyze accounts, track topics, and monitor keywords using live data. Use when you need current social media insights, competitor monitoring, or audience research.
---

# Twitter Intel

Get real-time X/Twitter intelligence without API keys. Analyze accounts, track trending topics, and monitor keywords with live data from X.

## When to Use This Skill

- Analyzing a Twitter/X account's recent activity and engagement
- Tracking what people are saying about a topic or hashtag
- Monitoring brand mentions or competitor activity
- Researching audience sentiment and trends
- Getting real-time social data for market research
- Finding influencers or key voices on a topic

## What This Skill Does

1. **Account Analysis** (`@username`): Analyzes recent posts, engagement patterns, content style, and audience interactions
2. **Topic Tracking** (`#topic`): Monitors trending discussions, popular posts, and sentiment around hashtags
3. **Keyword Monitoring** (`"keyword"`): Tracks brand mentions, competitor activity, and industry discussions
4. **Engagement Insights**: Provides metrics on likes, replies, and viral potential

## How to Use

### Basic Usage

```
/twitter-intel @elonmusk
```

```
/twitter-intel #AI
```

```
/twitter-intel "artificial intelligence startups"
```

### Natural Language

You can also use natural language:

```
What's @blockrunai posting about lately?
```

```
What's trending about AI agents on X?
```

```
Check Twitter for mentions of "Claude Code"
```

### Advanced Usage

Combine multiple analyses:

```
/twitter-intel @competitor1 @competitor2 - compare their content strategies
```

```
/twitter-intel #Web3 - focus on posts from the last 24 hours with high engagement
```

## Instructions

When a user requests Twitter/X intelligence, follow these steps:

### 1. Install Dependencies (First Time Only)

If the BlockRun SDK is not installed, install it:

```bash
pip install blockrun-llm
```

### 2. Initialize the Client

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()
```

If this is the first time, the client will display a QR code for funding the wallet. The user needs to add USDC on Base network ($1-5 is enough for many queries).

### 3. Execute the Query

**For Account Analysis (@username):**

```python
response = client.chat(
    "xai/grok-3",
    f"Analyze @{username}'s recent X/Twitter activity. Include: recent posts, engagement patterns, content themes, posting frequency, and notable interactions.",
    search_parameters={
        "mode": "on",
        "sources": [
            {
                "type": "x",
                "included_x_handles": [username],
                "post_favorite_count": 5
            }
        ],
        "max_search_results": 15,
        "return_citations": True
    }
)
```

**For Topic/Hashtag Tracking (#topic):**

```python
response = client.chat(
    "xai/grok-3",
    f"What are people saying about #{topic} on X/Twitter right now? Include: trending discussions, popular posts, key voices, and overall sentiment.",
    search_parameters={
        "mode": "on",
        "sources": [{"type": "x", "post_favorite_count": 50}],
        "max_search_results": 20,
        "return_citations": True
    }
)
```

**For Keyword Monitoring ("keyword"):**

```python
response = client.chat(
    "xai/grok-3",
    f"Search X/Twitter for mentions of '{keyword}'. Include: recent discussions, sentiment, key influencers mentioning this, and notable posts.",
    search_parameters={
        "mode": "on",
        "sources": [{"type": "x", "post_favorite_count": 10}],
        "max_search_results": 15,
        "return_citations": True
    }
)
```

### 4. Format the Output

Present results in a clear, actionable format:

```markdown
# Twitter Intel: @username

## Overview
- **Account**: @username
- **Recent Activity**: [Summary of posting frequency]
- **Primary Topics**: [Main themes they discuss]

## Recent Highlights
1. **[Post summary]** - [engagement metrics]
   > Quote or key excerpt

2. **[Post summary]** - [engagement metrics]
   > Quote or key excerpt

## Content Analysis
- **Tone**: [Professional/Casual/Technical/etc.]
- **Engagement Rate**: [High/Medium/Low based on follower count]
- **Best Performing Content**: [What type of posts get most engagement]

## Key Insights
- [Insight 1]
- [Insight 2]
- [Insight 3]

## Sources
[Links to referenced posts]
```

### 5. Report Costs

After each query, show the cost:

```python
spending = client.get_spending()
print(f"Query cost: ${spending['total_usd']:.4f}")
```

## Pricing

- **Per source retrieved**: $0.025
- **Typical query (10-20 sources)**: $0.25-0.50
- **Account analysis**: ~$0.38 (15 sources)
- **Topic tracking**: ~$0.50 (20 sources)

## Examples

### Example 1: Account Analysis

**User**: `/twitter-intel @pmarca`

**Output**:
```
# Twitter Intel: @pmarca

## Overview
- **Account**: @pmarca (Marc Andreessen)
- **Recent Activity**: Very active, 5-10 posts daily
- **Primary Topics**: AI, startups, tech policy, venture capital

## Recent Highlights
1. **Thread on AI regulation** - 2.5K likes, 400 replies
   > "The AI moment is different because..."

2. **Startup advice post** - 1.8K likes
   > "The best founders I've met..."

## Content Analysis
- **Tone**: Intellectual, contrarian, long-form threads
- **Engagement Rate**: Extremely high (10K+ avg likes)
- **Best Performing**: Controversial takes and founder advice

## Key Insights
- Consistently bullish on AI despite regulatory concerns
- Engages heavily with tech policy debates
- High influence on VC/startup community sentiment

Query cost: $0.38
```

### Example 2: Topic Tracking

**User**: `/twitter-intel #AIAgents`

**Output**:
```
# Twitter Intel: #AIAgents

## Trending Now
- Discussions around autonomous coding assistants
- Debate on agent safety and sandboxing
- New tool launches getting attention

## Top Posts (Last 24h)
1. @developer: "Just built an agent that..." - 500 likes
2. @researcher: "The problem with current agents..." - 320 likes

## Sentiment Analysis
- **Overall**: Excited but cautious
- **Main concerns**: Safety, costs, reliability
- **Main enthusiasm**: Productivity gains, automation

## Key Voices
- @karpathy - Technical deep dives
- @swyx - Developer tooling focus
- @anthropic - Safety-focused takes

Query cost: $0.50
```

### Example 3: Keyword Monitoring

**User**: `/twitter-intel "Claude Code"`

**Output**:
```
# Twitter Intel: "Claude Code"

## Mention Summary
- **Volume**: Moderate, growing steadily
- **Sentiment**: Very positive
- **Context**: Mostly developer reviews and tips

## Notable Mentions
1. @dev_influencer: "Claude Code just saved me 3 hours..." - 200 likes
2. @techreview: "Comparing Cursor vs Claude Code..." - 150 likes

## Common Themes
- Praise for code understanding
- Questions about pricing
- Comparisons to Cursor, Copilot

## Recommendations
- Engage with comparison discussions
- Address pricing questions proactively
- Amplify positive developer testimonials

Query cost: $0.38
```

## Tips

- **Reduce costs**: Use `max_search_results: 5` for quick checks
- **Increase depth**: Use `max_search_results: 30` for comprehensive analysis
- **Filter by engagement**: Increase `post_favorite_count` to focus on viral content
- **Date filtering**: Add `from_date` and `to_date` for time-specific analysis

## Requirements

- **BlockRun SDK**: `pip install blockrun-llm`
- **Wallet**: Auto-created on first use, fund with USDC on Base
- **Minimum balance**: $0.50 recommended for a few queries

## Related Use Cases

- Competitive intelligence gathering
- Influencer identification for marketing campaigns
- Real-time crisis monitoring
- Product launch sentiment tracking
- Industry trend analysis
