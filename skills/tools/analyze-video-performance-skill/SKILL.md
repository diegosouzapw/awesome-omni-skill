---
name: analyze-video-performance-skill
description: Analyze your uploaded YouTube videos' performance against competitor benchmarks using Playwright MCP. Use when asked to review video analytics, compare performance, check how a video is doing, or reflect on a video's success.
---

# Video Performance Analysis Skill

## Overview

This skill uses Playwright MCP to navigate to YouTube Studio and competitor videos, extract performance metrics, and generate actionable insights. It compares your uploaded videos against similar competitor content to identify what's working and what to improve.

---

## DATA ACCURACY RULES

### Rule 1: Only Report What You Can See
- Only report metrics you actually extracted from YouTube Studio
- NEVER guess or estimate metrics you couldn't read
- If a metric is unclear or partially visible, say "Could not extract [metric]"

### Rule 2: Mark Extraction Confidence
When reporting metrics, use these markers:
- `[EXACT]` - Read directly from the UI
- `[APPROXIMATE]` - Number was visible but might have rounding
- `[COULD NOT EXTRACT]` - Unable to read this metric

### Rule 3: No Interpretation Without Data
- Don't say "your retention is dropping at the hook" unless you SAW the retention curve
- Don't recommend changes without having the actual metrics
- If extraction fails, ask user to manually provide numbers

### Rule 4: Competitor Data is Public Only
- For competitor videos, you can only see public metrics (views, likes, comments)
- You CANNOT see competitor CTR, retention, or analytics
- Be clear about this limitation in comparisons

---

## When This Skill is Invoked

Triggers:
- "How is my video doing?"
- "Analyze [video name] performance"
- "Compare my video to competitors"
- "Let's reflect on [video]"
- "Check my analytics"
- "Why isn't my video performing?"

## Workflow

### Step 1: Identify the Video to Analyze

Ask if not provided:
- "Which video do you want to analyze? (Provide title or YouTube URL)"
- Check if video exists in `~/YT/Uploaded/` or `~/YT/In Progress/`

### Step 2: Navigate to YouTube Studio Analytics

Use Playwright MCP to access your video's analytics:

#### 2a. Navigate to YouTube Studio
```
mcp__plugin_playwright_playwright__browser_navigate to https://studio.youtube.com
```

#### 2b. Take a snapshot to verify login state
```
mcp__plugin_playwright_playwright__browser_snapshot
```

If not logged in, prompt user:
"Please log into YouTube Studio in the browser window, then tell me when you're ready."

#### 2c. Navigate to the specific video's analytics
Use the search or content library to find the video, then click into Analytics.

#### 2d. Extract Key Metrics

Use `browser_snapshot` and `browser_evaluate` to extract:

| Metric | Where to Find | Target |
|--------|---------------|--------|
| Views (first 48h) | Overview tab | Context-dependent |
| CTR (Click-through rate) | Reach tab | 4%+ is good, 6%+ is excellent |
| Average view duration | Engagement tab | Higher is better |
| Average % viewed | Engagement tab | 50%+ is top tier |
| First 30 sec retention | Retention graph | 70%+ |
| Traffic sources | Reach tab | Understand where views come from |
| Impressions | Reach tab | How often thumbnail shown |

**JavaScript to extract retention data (example):**
```javascript
() => {
  // Extract visible metrics from the analytics page
  const metrics = {};
  const cards = document.querySelectorAll('[class*="metric"]');
  cards.forEach(card => {
    const label = card.querySelector('[class*="label"]')?.textContent;
    const value = card.querySelector('[class*="value"]')?.textContent;
    if (label && value) metrics[label] = value;
  });
  return JSON.stringify(metrics);
}
```

### Step 3: Capture Retention Curve Shape

Take a screenshot of the retention graph:
```
mcp__plugin_playwright_playwright__browser_take_screenshot with filename "retention-curve.png"
```

Analyze the curve shape:
- **Flat/gradual decline** = Good pacing, engaged audience
- **Sharp early drop** = Hook isn't working
- **Mid-video drops** = Specific sections losing interest
- **Spikes** = Rewatch points (good!)
- **End spike** = End screen working

### Step 4: Get Competitor Benchmarks (Optional)

If user wants comparison:

#### 4a. Ask for competitor videos
"Provide 1-3 competitor videos covering similar topics for comparison"

#### 4b. Navigate to each competitor video
```
mcp__plugin_playwright_playwright__browser_navigate to https://www.youtube.com/watch?v=VIDEO_ID
```

#### 4c. Extract visible metrics
- View count
- Like/dislike ratio (via engagement)
- Comment count
- Video length
- Published date
- Channel subscriber count

#### 4d. Calculate relative performance
Compare your video's metrics against competitors, adjusting for:
- Channel size difference
- Time since publish
- Topic overlap

### Step 4.5: Compare to Personal Baseline

**Read the channel baseline file:** `~/YT/_Analytics/channel-baseline.md`

Compare this video's metrics against YOUR OWN averages:

| Metric | This Video | Your Avg | vs Baseline |
|--------|------------|----------|-------------|
| Views (48h) | [extracted] | [from file] | Above/Below |
| CTR | [extracted] | [from file] | Above/Below |
| Retention | [extracted] | [from file] | Above/Below |
| First 30s | [extracted] | [from file] | Above/Below |

**Interpretation:**
- **Above baseline:** This approach is working - identify what's different
- **Below baseline:** Something underperformed - identify what changed
- **Matching baseline:** Consistent performance

**Why this matters:**
- Competitor comparison tells you how you compare externally
- Baseline comparison tells you if YOU'RE improving
- Both are valuable but baseline shows your growth trajectory

### Step 5: Shorts Performance (If Applicable)

Navigate to YouTube Studio Shorts analytics:
- Which shorts came from this video?
- Individual short view counts
- Best performing short and why

### Step 6: Generate Performance Report

Create a report in the video's `todo.md` or a new `analytics-report.md`:

```markdown
# Performance Analysis: [Video Title]

**Analyzed:** [Date]
**Video URL:** [URL]
**Days Since Publish:** [X]

## Key Metrics

| Metric | Your Video | Target | Status |
|--------|------------|--------|--------|
| Views (first 48h) | | | |
| CTR | | 4%+ | |
| Avg View Duration | | | |
| Retention | | 50%+ | |
| First 30s Retention | | 70%+ | |

## Retention Curve Analysis

[Description of curve shape and what it means]

## Traffic Sources Breakdown

| Source | % of Views | Notes |
|--------|-----------|-------|
| Browse | | |
| Search | | |
| Suggested | | |
| External | | |

## Baseline Comparison (vs YOUR channel averages)

| Metric | This Video | Your Avg | vs Baseline | Interpretation |
|--------|------------|----------|-------------|----------------|
| Views (48h) | | | | |
| CTR | | | | |
| Retention | | | | |
| First 30s | | | | |

## Competitor Comparison (if applicable)

| Metric | Your Video | Competitor A | Competitor B |
|--------|------------|--------------|--------------|
| Views | | | |
| CTR | | | |
| Engagement | | | |

## What Worked Well
- [Based on metrics and retention]

## Areas for Improvement
- [Based on drops, CTR, etc.]

## Recommendations for Next Video
1. [Specific actionable item]
2. [Specific actionable item]
3. [Specific actionable item]

## Shorts Analysis
| Short | Views | Why It Worked/Didn't |
|-------|-------|---------------------|
| | | |
```

### Step 7: Update todo.md Reflection Section

If the video has a `todo.md`, update the reflection section with:
- Analytics snapshot table
- What went well
- What could be improved
- Key learnings

```markdown
## Reflection (After 1 Week)
- [x] Review analytics completed
- [x] Reflection questions answered

### Analytics Snapshot
| Metric | Result | Target | Notes |
|--------|--------|--------|-------|
| Views (48h) | X | | |
| CTR | X% | 4%+ | |
| Avg View Duration | X:XX | | |
| Retention | X% | 50%+ | |
| First 30s Retention | X% | 70%+ | |
| Best Performing Short | [Title] | | |

### What Went Well
- [Finding 1]
- [Finding 2]

### What Could Be Improved
- [Finding 1]
- [Finding 2]

### Key Learnings for Next Video
- [Learning 1]
- [Learning 2]
```

### Step 8: Update Channel Baseline

**After completing the reflection, update `~/YT/_Analytics/channel-baseline.md`:**

1. **Add to Individual Video Performance table:**
   - Video title, publish date, all key metrics

2. **Recalculate Running Averages:**
   - Sum all videos' metrics, divide by count
   - Update the average row

3. **Check Top/Bottom 5:**
   - If this video outperformed, add to Top 5
   - If this video underperformed, add to Bottom 5
   - Include "Why It Worked" or "What Went Wrong"

4. **Update Monthly Totals:**
   - Add to the current month in Trend Over Time

**Prompt user:**
```
I've completed the reflection. Would you like me to update your channel baseline file with these metrics?

This helps track your growth over time and gives context for future videos.
```

## Metric Interpretation Guide

### CTR (Click-Through Rate)
| Range | Interpretation | Action |
|-------|---------------|--------|
| 0-2% | Poor | Thumbnail/title needs major work |
| 2-4% | Below average | Test new thumbnails |
| 4-6% | Good | Keep iterating |
| 6-10% | Excellent | Study what worked |
| 10%+ | Viral potential | This is working, do more of this |

### Retention
| Range | Interpretation | Action |
|-------|---------------|--------|
| 0-30% | Poor | Video too long or wrong audience |
| 30-40% | Below average | Hook and pacing need work |
| 40-50% | Average | Room for improvement |
| 50-60% | Good | Top 17% of videos |
| 60%+ | Excellent | Strong content-audience fit |

### First 30 Seconds
| Range | Interpretation | Action |
|-------|---------------|--------|
| <50% | Hook failing | Completely rework intro |
| 50-70% | Needs work | Tighten opening, faster to value |
| 70-80% | Good | Hook is working |
| 80%+ | Excellent | Study and replicate |

## Common Issues & Fixes

| Problem | Likely Cause | Fix |
|---------|--------------|-----|
| Low CTR, high retention | Thumbnail underselling content | Better thumbnail, more curiosity |
| High CTR, low retention | Clickbait or wrong audience | Match thumbnail to content |
| Sharp early drop | Weak hook | Stronger first 30 seconds |
| Mid-video drop | Slow section | Cut filler, add pattern interrupts |
| Low suggested traffic | Not getting algorithmic push | Improve retention, longer watch time |

## Example Interaction

**User:** "How is my Claude Code video doing?"

**Claude:** "Let me pull your analytics using YouTube Studio.

*Uses Playwright to navigate to YouTube Studio*
*Extracts metrics from your video*

Here's your performance analysis:

**Views (first 48h):** 1,247
**CTR:** 5.2% (Good - above 4% target)
**Avg View Duration:** 6:42 / 12:15 (54% - top 17%!)
**First 30s Retention:** 73% (Above target)

**What's working:**
- Your hook is solid (73% first 30s)
- Retention is above average
- CTR shows thumbnail/title combo works

**Opportunity:**
- Traffic mostly from browse (42%), not much from search
- Consider adding more keywords for discoverability

**Next video recommendations:**
1. Keep the same hook style - it's working
2. Optimize title for search keywords
3. Test a more curiosity-driven thumbnail variant

Would you like me to:
1. Compare against competitor videos?
2. Update your todo.md with these findings?
3. Analyze your shorts performance?"

---

## Troubleshooting

### Can't Access YouTube Studio

**Symptom:** Page loads but can't see analytics

**Solutions:**
1. **Login required:** Ask user to log into YouTube Studio manually first
2. **Take snapshot:** Run `browser_snapshot` to see what's on screen
3. **Wait for load:** YouTube Studio can be slow - use `browser_wait_for` with "Analytics" text
4. **Manual fallback:** Ask user to provide metrics directly from their screen

### Metrics Not Extractable

**Symptom:** JavaScript selectors don't find metrics

**Solutions:**
1. **YouTube UI changed:** Take a fresh `browser_snapshot` and look for new element patterns
2. **Page not fully loaded:** Add `browser_wait_for` with longer timeout
3. **Screenshot fallback:** Use `browser_take_screenshot` and ask user to read the numbers
4. **Manual input:** Ask user: "I couldn't extract [metric]. Can you tell me what it shows?"

### Competitor Video Issues

**Symptom:** Can't access competitor video data

**Remember:**
- You can ONLY see public metrics (views, likes, comments, video length)
- You CANNOT see their CTR, retention, or analytics
- Age-restricted videos may not be accessible
- Private/unlisted videos are not accessible

### Common YouTube Studio Selectors

These may need updating if YouTube changes their UI:

| Element | Current Pattern | Notes |
|---------|----------------|-------|
| Analytics tabs | `[data-tab-id="analytics"]` | May change |
| Metric cards | `ytcp-analytics-data-story` | Look for value within |
| Retention graph | Canvas element in engagement tab | Screenshot, can't extract points |

### When Extraction Fails Completely

If you cannot extract any metrics:

1. **Acknowledge the limitation:** "I'm having trouble reading your YouTube Studio analytics"
2. **Ask for manual input:** "Could you provide these key metrics from your screen?"
   - Views (first 48h)
   - CTR percentage
   - Average view duration
   - Average % viewed
3. **Continue with user-provided data:** Generate the same quality report using their input
4. **Log the issue:** Note in the report that data was user-provided, not extracted
