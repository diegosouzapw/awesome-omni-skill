---
name: social-media-scheduler
description: Generate a full week of social media content for any topic. Outputs platform-optimized posts for Twitter/X, LinkedIn, and Instagram with hashtags and posting times.
triggers:
  - generate social media posts
  - create content calendar
  - weekly social media plan
---

# Social Media Scheduler 📅

Generate a full week of social media content for any topic or niche. Outputs platform-optimized posts for **Twitter/X**, **LinkedIn**, and **Instagram**.

## Usage

```bash
# Generate a week of posts for a topic
./generate.sh "artificial intelligence"

# Specify a niche audience
./generate.sh "sustainable fashion" "eco-conscious millennials"
```

### Arguments
1. **Topic/Niche** (required) — The subject for your content week
2. **Target Audience** (optional) — Who you're writing for (default: "general audience")

### Output
Creates `content-calendar.md` in the current directory with:
- 7 days of posts (Mon–Sun)
- 3 platform variants per day (Twitter, LinkedIn, Instagram)
- Hashtag suggestions per platform
- Best posting times
- Content mix (educational, engagement, promotional, storytelling)

## Content Strategy Built In

The generator follows a proven weekly content mix:
| Day | Theme |
|-----|-------|
| Mon | Motivational / Week opener |
| Tue | Educational / How-to |
| Wed | Engagement / Question |
| Thu | Behind-the-scenes / Story |
| Fri | Tip / Quick win |
| Sat | Curated / Industry news |
| Sun | Reflection / Community |

## Platform Guidelines

- **Twitter/X**: ≤280 chars, punchy, 2-3 hashtags, thread hooks
- **LinkedIn**: Professional tone, 1-3 paragraphs, thought leadership, 3-5 hashtags
- **Instagram**: Visual-first caption, storytelling, 5-10 hashtags, CTA in bio link

## Requirements
- Bash + `cat` (that's it — the script generates a markdown template you can customize)
- For AI-powered personalization, pipe output to your favorite LLM

## Example

```bash
./generate.sh "productivity tips" "remote workers"
# → Creates content-calendar.md with 21 ready-to-post drafts
```

## Author
🦞 Shelly — [@ShellyToMillion](https://twitter.com/ShellyToMillion)
