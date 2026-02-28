---
name: shorts-presentation-skill
description: Create vertical (9:16) interactive presentations optimized for YouTube Shorts, TikTok, and Instagram Reels. Takes YouTube video URLs to extract facts via Playwright MCP and web research, then generates animated slides you can screen record and narrate. Perfect for quick educational content, fact-reveals, and viral short-form videos.
---

# Shorts Presentation Generator

## Overview

This skill creates **vertical (9:16 aspect ratio)** interactive HTML presentations specifically designed for short-form video content. You screen record the presentation while narrating, then use Opus Clip Pro or direct upload for Shorts/Reels/TikTok.

**Key Features:**
- Vertical 9:16 format (1080x1920 optimized)
- Fast-paced slide design (3-5 seconds per slide)
- Large, readable text for mobile viewing
- Bold animations that pop on small screens
- Click-to-advance for easy recording
- 30-60 second content pacing

---

## CRITICAL: Fact Integrity Rules

### Rule 1: NEVER Infer New Facts
- Only use information explicitly stated in source videos or found in web research
- NEVER extrapolate, assume, or create new "facts" by combining information
- If unsure, mark as `[FROM VIDEO]` or `[NEEDS VERIFICATION]`

### Rule 2: Source Everything
- Every fact shown on a slide must have a traceable source
- Either from the YouTube transcript (with timestamp) or web research (with URL)
- If you cannot source it, do not include it

### Rule 3: Recency Matters
- Always search for the most recent data (2025-2026)
- Mark dates on any statistics or data points
- Outdated facts can go viral for the wrong reasons

### Rule 4: Source Attribution Format
Every fact in the presentation must be tagged:
- `[VERIFIED: Source Name, Date]` - Confirmed via web search
- `[FROM VIDEO: Title, Timestamp]` - From transcript, not independently verified
- `[USER PROVIDED]` - User stated this information

### Red Flags - STOP and Verify
If you find yourself including:
- "Studies show..." → Which study? What year?
- "Experts say..." → Which expert? What quote?
- "Millions of..." → Exact number and source?
- Any statistic → Source URL and date required

### Rule 5: ALWAYS Re-Verify Before Generating

**DO NOT trust research from earlier in the conversation.** Before generating ANY presentation:

1. **Re-fetch the primary source** - If referencing a GitHub issue, blog post, or article, fetch it AGAIN right before generating slides
2. **Check for updates** - Issues get closed, bugs get fixed, versions change. What was true 10 minutes ago may not be true now
3. **Verify version numbers** - ALWAYS confirm the CURRENT latest version and its status. Don't assume old version info is still accurate
4. **Cross-reference user's context** - If user mentions they're on version X, verify if the issue affects version X specifically

**Example of what NOT to do:**
- Research says "bug affects v2.0.75"
- User says "I'm on v2.0.76"
- ❌ WRONG: Assume v2.0.75 info is still current
- ✅ RIGHT: Search for "v2.0.76 [bug name]" to verify current status

**Before generating slides, ALWAYS ask yourself:**
- "Is this the CURRENT version?"
- "Has this been fixed since my research?"
- "Did I verify this against the user's specific context?"

### Rule 6: Update ALL Outputs When Facts Change

When a fact is corrected or updated:
- Update the presentation HTML
- Update the narration script
- Update the research.md
- Update any other files referencing that fact

**One wrong fact in a viral video destroys credibility.**

---

## When to Use

Invoke this skill when:
- User says "create a short", "make a viral presentation", "shorts presentation"
- User provides YouTube URLs and wants quick, shareable content
- User wants to create educational Shorts with animated facts
- User wants vertical video content for social media

---

## Workflow

### Step 1: Collect Input

Accept one or more of:
- YouTube video URLs (to extract transcripts and facts)
- A specific topic to research
- Key points the user wants to convey

### Step 2: Fetch YouTube Transcripts (If URLs Provided)

For each video URL, use Playwright MCP with this **exact method**:

**Step 2a: Navigate and Extract with JavaScript (RECOMMENDED)**

Use `browser_run_code` to handle navigation, clicking, and extraction in one reliable call:

```javascript
// Use this exact code in mcp__plugin_playwright_playwright__browser_run_code
async (page) => {
  // Scroll down to load description
  await page.evaluate(() => window.scrollBy(0, 400));
  await page.waitForTimeout(1000);

  // Click expand description button
  const expandBtn = await page.$('tp-yt-paper-button#expand');
  if (expandBtn) {
    await expandBtn.click();
    await page.waitForTimeout(500);
  }

  // Click "Show transcript" button
  const showTranscriptBtn = await page.$('button[aria-label="Show transcript"]');
  if (showTranscriptBtn) {
    await showTranscriptBtn.click();
    await page.waitForTimeout(1500);
  }

  // Extract transcript segments using Playwright's querySelectorAll method
  const segments = await page.locator('ytd-transcript-segment-renderer').evaluateAll(els =>
    els.map(el => {
      const time = el.querySelector('.segment-timestamp')?.innerText?.trim() || '';
      const text = el.querySelector('.segment-text')?.innerText?.trim() || '';
      return '[' + time + '] ' + text;
    }).join('\n')
  );

  return segments || 'Transcript not found';
}
```

**Step 2b: Alternative - Manual Click Method**

If the JavaScript method fails, use manual clicks:
1. `browser_navigate` to `https://www.youtube.com/watch?v=VIDEO_ID`
2. `browser_snapshot` to get page state
3. Look for "Show transcript" button ref in snapshot (usually in description area)
4. `browser_click` with the exact ref (e.g., `ref=e3960`)
5. `browser_snapshot` again to see transcript panel
6. Use `browser_run_code` to extract just the transcript text

**CRITICAL: Do NOT use browser_snapshot alone** - the transcript text is in buttons, not plain text. Always use JavaScript to extract the actual segment content from `ytd-transcript-segment-renderer` elements.

**Common Issues:**
- If snapshot shows transcript panel but no text: Use JavaScript to extract
- If "Show transcript" button not visible: Scroll down and expand description first
- If transcript not loading: Wait longer (1500ms+) after clicking

Extract from each video:
- Video title
- Key claims/facts with timestamps
- Statistics mentioned
- Quotes that could be used

### Step 3: Web Research for Verification

Use WebSearch to:
1. Verify any numerical claims from the video
2. Find additional recent data on the topic
3. Get official sources for credibility

**Search Strategy:**
- Use `site:` operators for authoritative sources
- Always include current year in searches
- Look for the most recent data available

**Source Priority:**
1. Official sources (government, company reports)
2. Major news outlets (Reuters, Bloomberg, AP)
3. Industry publications
4. Original video (clearly attributed)

### Step 4: Ask Clarifying Questions

**Question 1: Content Focus**
```
"What's the main hook for this Short?"
Header: "Hook"
Options:
- Surprising fact reveal
- Myth vs Reality
- Quick tutorial/tip
- Comparison (A vs B)
- Story/Timeline
```

**Question 2: Slide Count**
```
"How many slides (aim for 30-60 seconds total)?"
Header: "Length"
Options:
- 5-7 slides (30 seconds - punchy)
- 8-10 slides (45 seconds - standard) [Recommended]
- 11-15 slides (60 seconds - detailed)
```

**Question 3: Visual Style**
```
"What visual style?"
Header: "Style"
Options:
- Bold & Minimal (large text, high contrast) [Recommended]
- Data-focused (stats, numbers, charts)
- Comparison view (split screen style)
- Story sequence (narrative flow)
```

**Question 4: Theme**
```
"Color theme?"
Header: "Theme"
Options:
- Dark Mode (black/teal) [Recommended for screen recording]
- Light Mode (white/blue)
- High Energy (dark/orange-red)
- Professional (dark/green)
```

### Step 4.5: VERIFICATION CHECKPOINT (REQUIRED)

**Before generating slides, present facts to user for approval:**

```markdown
## 🔒 FACT VERIFICATION CHECKPOINT

I found these facts from analysis and research:

### ✅ VERIFIED FACTS (confirmed via web search)
| Fact | Source | Date |
|------|--------|------|
| [fact] | [source URL] | [date] |

### 📺 VIDEO CLAIMS (from transcript, not independently verified)
| Claim | Video | Timestamp |
|-------|-------|-----------|
| [claim] | [video title] | [MM:SS] |

### ❌ COULD NOT VERIFY
| Claim | Search Attempted | Result |
|-------|------------------|--------|
| [claim] | [what I searched] | Not found |

---

**Which facts should I include in the Short?**
- I recommend only using ✅ VERIFIED facts
- 📺 VIDEO CLAIMS can be included with attribution ("According to [video]...")
- ❌ UNVERIFIED facts should NOT be used

Select the facts you want featured (by number):
```

**Wait for user to select which facts to include before generating slides.**

### Step 5: Generate Slide Content

Create 5-15 slides following this structure:

#### Slide 1: Hook (0-3 sec)
```
LARGE BOLD TEXT
[Attention-grabbing statement or question]
```

#### Slides 2-N: Content (3-5 sec each)
```
KEY POINT
[Supporting detail]
[Source indicator if applicable]
```

#### Final Slide: CTA or Punchline (3-5 sec)
```
[Memorable conclusion]
[Call to action if appropriate]
```

### Step 6: Generate HTML Presentation

Create a vertical HTML file optimized for:
- 1080x1920 resolution (9:16)
- Large, bold typography (minimum 48px)
- High contrast colors
- Simple animations (fade, scale, slide)
- Touch/click advancement
- No distracting elements

### Step 7: Save and Provide Instructions

**IMPORTANT: Shorts go in the Ideas folder with "SHORTS - " prefix.**

Save to: `~/YT/Ideas/SHORTS - [Topic]/presentation.html`

Provide recording instructions:
```
Your Shorts presentation is ready!

**File:** ~/YT/Shorts/[Topic]/presentation.html

**How to Record:**
1. Open in Chrome
2. Press F11 for fullscreen
3. Resize Chrome window to 1080x1920 (or use DevTools mobile view)
4. Start OBS/screen recording
5. Click to advance each slide while narrating
6. Each slide = 3-5 seconds of content

**Tips for Recording:**
- Speak with energy - shorts need high engagement
- Match your voice to the slide transitions
- Keep total length under 60 seconds
- The hook (slide 1) is crucial - nail it

**After Recording:**
- Upload directly to YouTube Shorts, TikTok, Reels
- Or use Opus Clip Pro for auto-captions and effects
```

---

## Shorts-Optimized Slide Types

### Hook Slide
```html
<div class="slide hook-slide">
  <h1 class="hook-text">[HOOK TEXT]</h1>
  <p class="hook-subtext">[Optional subtext]</p>
</div>
```
- Largest text size
- Centered, bold
- Immediate attention grab

### Fact Slide
```html
<div class="slide fact-slide">
  <p class="fact-label">[LABEL like "FACT" or "DID YOU KNOW"]</p>
  <h2 class="fact-text">[The fact]</h2>
  <p class="fact-source">[Source]</p>
</div>
```
- Clear hierarchy
- Source visible but not dominant

### Number Reveal Slide
```html
<div class="slide number-slide">
  <p class="number-label">[Context]</p>
  <h1 class="big-number" data-animate="count">[NUMBER]</h1>
  <p class="number-unit">[Unit or description]</p>
</div>
```
- Animated number counter
- Dramatic reveal effect

### Comparison Slide
```html
<div class="slide compare-slide">
  <div class="compare-item wrong">
    <span class="compare-icon">❌</span>
    <p>[Wrong/Myth/Before]</p>
  </div>
  <div class="compare-item right">
    <span class="compare-icon">✅</span>
    <p>[Right/Truth/After]</p>
  </div>
</div>
```
- Clear visual distinction
- Stacked vertically for mobile

### Quote Slide
```html
<div class="slide quote-slide">
  <div class="quote-mark">"</div>
  <p class="quote-text">[Quote]</p>
  <p class="quote-author">— [Author]</p>
</div>
```
- Large quote marks
- Emphasis on the words

### List Slide
```html
<div class="slide list-slide">
  <h2 class="list-title">[Title]</h2>
  <ul class="shorts-list">
    <li data-reveal="1">[Item 1]</li>
    <li data-reveal="2">[Item 2]</li>
    <li data-reveal="3">[Item 3]</li>
  </ul>
</div>
```
- Items reveal one by one
- 3-5 items max per slide

### CTA Slide
```html
<div class="slide cta-slide">
  <h2 class="cta-text">[Call to action]</h2>
  <p class="cta-subtext">[Supporting text]</p>
  <div class="cta-icons">
    <span>👆 Follow</span>
    <span>💬 Comment</span>
    <span>🔄 Share</span>
  </div>
</div>
```
- Clear action request
- Platform-appropriate icons

---

## Theme Options for Shorts

### Dark Mode (Default)
```css
--bg-primary: #000000;
--bg-secondary: #1a1a1a;
--text-primary: #ffffff;
--text-secondary: #a0a0a0;
--accent: #00d4aa;
```

### Light Mode
```css
--bg-primary: #ffffff;
--bg-secondary: #f5f5f5;
--text-primary: #000000;
--text-secondary: #666666;
--accent: #3b82f6;
```

### High Energy
```css
--bg-primary: #0a0a0a;
--bg-secondary: #1a1a1a;
--text-primary: #ffffff;
--text-secondary: #a0a0a0;
--accent: #ff6b35;
```

### Professional
```css
--bg-primary: #0f172a;
--bg-secondary: #1e293b;
--text-primary: #f8fafc;
--text-secondary: #94a3b8;
--accent: #22c55e;
```

---

## Animation Presets for Shorts

### Slide In
```css
.slide { animation: slideIn 0.3s ease-out; }
@keyframes slideIn {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
```

### Scale Pop
```css
.slide { animation: scalePop 0.3s ease-out; }
@keyframes scalePop {
  from { transform: scale(0.8); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
```

### Fade Up
```css
.slide { animation: fadeUp 0.3s ease-out; }
@keyframes fadeUp {
  from { transform: translateY(50px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
```

### Number Count
```javascript
// For dramatic number reveals
function animateNumber(el, target, duration = 800) {
  let current = 0;
  const increment = target / (duration / 16);
  const timer = setInterval(() => {
    current = Math.min(current + increment, target);
    el.textContent = formatNumber(Math.floor(current));
    if (current >= target) clearInterval(timer);
  }, 16);
}
```

---

## Content Pacing Guide

| Slide Type | Duration | Purpose |
|------------|----------|---------|
| Hook | 2-3 sec | Grab attention immediately |
| Fact/Point | 4-5 sec | Deliver key information |
| Number Reveal | 3-4 sec | Dramatic stat reveal |
| Comparison | 5-6 sec | Show contrast clearly |
| Quote | 4-5 sec | Let quote sink in |
| List (per item) | 2 sec each | Quick reveals |
| CTA | 3-4 sec | Clear next action |

**Total target: 30-60 seconds**

---

## Example: Shorts from YouTube Analysis

```
User: Create a short from this video about AI productivity
      https://youtube.com/watch?v=ABC123

Claude: [Fetches transcript via Playwright MCP]
        [Identifies key facts and statistics]
        [Verifies claims via WebSearch]

I found 5 key facts from this video:

1. "AI tools save an average of 2 hours per day" - [VERIFIED: McKinsey 2024]
2. "ChatGPT has 200M weekly users" - [VERIFIED: OpenAI 2024]
3. "40% of workers use AI secretly" - [FROM VIDEO - needs verification]
4. "Productivity up 35% with AI" - [PARTIALLY VERIFIED: varies by study]
5. "AI will replace 30% of jobs" - [DISPUTED: estimates vary widely]

Which facts would you like to feature in the Short?

[User selects facts 1, 2, 4]

[Asks clarifying questions about style]

Generating your Shorts presentation...

**Slides Created:**
1. Hook: "AI is changing EVERYTHING about work"
2. Number: "2 HOURS saved per day with AI tools"
3. Source: "According to McKinsey Research 2024"
4. Number: "200 MILLION people use ChatGPT weekly"
5. Number: "35% productivity boost with AI"
6. CTA: "Follow for more AI insights"

**File:** ~/YT/Shorts/AI Productivity Facts/presentation.html

**Recording Tips:**
- Hook with energy: "Did you know AI can give you 2 extra hours every day?"
- Pause on numbers for impact
- End with enthusiasm for the CTA
- Total time: ~35 seconds
```

---

## Folder Structure

```
~/YT/Shorts/
├── [Topic Name 1]/
│   ├── presentation.html
│   ├── research.md (sources and verification)
│   └── script-notes.md (optional talking points)
├── [Topic Name 2]/
│   └── ...
```

---

## Integration with Opus Clip Pro

After recording the screen with your narration:

1. Export as vertical video (1080x1920)
2. Upload to Opus Clip Pro
3. Let it add auto-captions
4. Apply effects/transitions
5. Auto-post to all platforms

The presentation provides the visual backbone - your voice and Opus Clip's captions complete the package.

---

## Safety Reminders

1. **Double-check all numbers** before recording
2. **Date your statistics** - add "as of 2025" to slides if needed
3. **Cite sources verbally** when recording ("According to McKinsey...")
4. **Don't mix sources** - clearly separate video claims from research
5. **When in doubt, leave it out** - one wrong fact can tank credibility

---

## Troubleshooting

### Transcript Extraction Fails

**Solutions (try in order):**
1. **Check video has captions:** Not all videos have CC
2. **Fallback to Python script:**
   ```bash
   cd ~/YT/_Config/scripts
   source .venv/bin/activate
   python fetch_youtube.py "VIDEO_URL" --format markdown
   ```
3. **Manual fallback:** Ask user to provide key facts directly

### WebSearch Doesn't Find Verification

1. **Try different search terms:** Use company/organization names directly
2. **Check official sources:** Go directly to company websites, government sites
3. **Mark as unverified:** If no source found, do NOT include in slides
4. **Ask user:** "I couldn't verify [claim]. Do you have a source, or should I exclude it?"

### Vertical Format Display Issues

1. **Chrome DevTools:** Press F12 → Toggle device toolbar → Select custom 1080x1920
2. **OBS Setup:** Create scene with 1080x1920 canvas, window capture Chrome
3. **Preview:** Always preview at actual size before recording

---

## CRITICAL: HTML Template for Vertical Presentations

**You MUST use this EXACT structure for all shorts presentations. Do NOT deviate.**

### Complete HTML Template (REQUIRED)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[PRESENTATION TITLE]</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html, body {
            width: 100%;
            height: 100%;
            background: #000;
            overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Segoe UI", Roboto, sans-serif;
        }

        /* 9:16 Container - centered and scaled to fit viewport */
        .container {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 1080px;
            height: 1920px;
            background: #0a0a0a;
            transform-origin: center center;
        }

        /* Progress bar - thick and visible */
        .progress {
            position: absolute;
            top: 0;
            left: 0;
            height: 12px;
            background: linear-gradient(90deg, #00d4aa, #00a080);
            z-index: 1000;
            transition: width 0.3s ease;
        }

        /* Slide counter - large and clear */
        .counter {
            position: absolute;
            bottom: 80px;
            right: 60px;
            font-size: 48px;
            font-weight: 700;
            color: rgba(255,255,255,0.7);
            z-index: 1000;
            font-variant-numeric: tabular-nums;
        }

        /* Slides */
        .slide {
            position: absolute;
            top: 0;
            left: 0;
            width: 1080px;
            height: 1920px;
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 100px 70px;
            text-align: center;
            color: #fff;
        }

        .slide.active {
            display: flex;
            animation: fadeIn 0.3s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
        }

        /* [ADD YOUR CUSTOM STYLES HERE] */
    </style>
</head>
<body>
    <div class="container" id="container">
        <div class="progress" id="progress"></div>
        <div class="counter" id="counter">1 / [TOTAL]</div>

        <!-- Slides go here -->
        <div class="slide active">
            <!-- Slide 1 content -->
        </div>
        <div class="slide">
            <!-- Slide 2 content -->
        </div>
        <!-- More slides... -->
    </div>

    <script>
        const slides = document.querySelectorAll('.slide');
        const progress = document.getElementById('progress');
        const counter = document.getElementById('counter');
        const container = document.getElementById('container');
        let current = 0;
        const total = slides.length;

        // Scale container to fit viewport
        function scaleContainer() {
            const vw = window.innerWidth;
            const vh = window.innerHeight;
            const scale = Math.min(vw / 1080, vh / 1920);
            container.style.transform = `translate(-50%, -50%) scale(${scale})`;
        }

        function update() {
            // Update slides
            slides.forEach((s, i) => {
                s.classList.toggle('active', i === current);
            });
            // Update progress bar
            const pct = ((current + 1) / total) * 100;
            progress.style.width = pct + '%';
            // Update counter
            counter.textContent = (current + 1) + ' / ' + total;
        }

        function next() {
            if (current < total - 1) {
                current++;
                update();
            }
        }

        function prev() {
            if (current > 0) {
                current--;
                update();
            }
        }

        // Click to advance
        container.addEventListener('click', next);

        // Keyboard
        document.addEventListener('keydown', e => {
            if (e.key === 'ArrowRight' || e.key === ' ' || e.key === 'Enter') {
                e.preventDefault();
                next();
            } else if (e.key === 'ArrowLeft') {
                e.preventDefault();
                prev();
            } else if (e.key === 'Home') {
                current = 0;
                update();
            } else if (e.key === 'End') {
                current = total - 1;
                update();
            }
        });

        // Touch support
        let touchStartX = 0;
        container.addEventListener('touchstart', e => {
            touchStartX = e.touches[0].clientX;
        });
        container.addEventListener('touchend', e => {
            const diff = touchStartX - e.changedTouches[0].clientX;
            if (Math.abs(diff) > 50) {
                if (diff > 0) next();
                else prev();
            }
        });

        // Init
        scaleContainer();
        window.addEventListener('resize', scaleContainer);
        update();
    </script>
</body>
</html>
```

### MANDATORY Requirements

| Requirement | Details |
|-------------|---------|
| Container class | Must be `.container` (NOT `.presentation-container`) |
| Container ID | Must have `id="container"` for JavaScript |
| Scaling | Use JavaScript `scaleContainer()` function (NOT CSS media queries) |
| Progress bar | At TOP of container, gradient color, 12px height |
| Slide counter | At BOTTOM RIGHT, format "1 / 9", 48px font |
| Slide dimensions | Fixed `width: 1080px; height: 1920px` (NOT 100%) |
| Slide padding | `padding: 100px 70px` |
| Animation | Use `fadeIn` with scale (NOT slideIn or other) |

### Common Mistakes to AVOID

| Mistake | Why It Breaks | Correct Approach |
|---------|---------------|------------------|
| Using `.presentation-container` | Wrong class name | Use `.container` |
| CSS media queries for scaling | Inconsistent across browsers | Use JavaScript `scaleContainer()` |
| Progress dots at bottom | Wrong design pattern | Use progress bar at TOP |
| Counter at top right | Wrong position | Counter at BOTTOM RIGHT |
| `width: 100%` on slides | References viewport, not container | Use `width: 1080px` |
| `height: 100%` on slides | References viewport, not container | Use `height: 1920px` |
| Missing `id="container"` | JavaScript won't work | Add the ID |

### Pre-Generation Checklist

Before generating HTML, verify:
- [ ] Container uses `.container` class with `id="container"`
- [ ] JavaScript `scaleContainer()` function is included
- [ ] Progress bar at top with gradient
- [ ] Slide counter at bottom right in "X / Y" format
- [ ] Slides have `position: absolute`
- [ ] Slides have fixed pixel dimensions (1080x1920)
- [ ] Slide padding is `100px 70px`
- [ ] `fadeIn` animation with scale transform

---

### Recording Tips

1. **Consistent timing:** Practice clicking through at 3-5 second intervals
2. **Voice matching:** Script what you'll say for each slide before recording
3. **Energy level:** Shorts need higher energy than long-form content
4. **Re-record if needed:** Better to do multiple takes than publish subpar content
