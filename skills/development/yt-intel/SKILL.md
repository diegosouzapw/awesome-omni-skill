---
name: yt-intel
description: YouTube Content Intelligence Pipeline (UNATTENDED MODE). Extracts metadata + transcript from a YouTube video, applies the Fabric extract_wisdom analysis pattern, scores relevance to Delivery Factory, and (if relevant) generates a branded DOCX report, uploads to Google Drive, and posts a summary to Discord #ad-intel.
---

# YouTube Content Intelligence Pipeline — UNATTENDED MODE

**This is an automated invocation.** There is no interactive user. All decisions must be made autonomously based on the thresholds below.

You are executing the `/yt-intel` pipeline. This is a 6-step process that extracts intelligence from a YouTube video, gates it for relevance to Agentic Delivery / Delivery Factory, and (if relevant) produces a branded report with Drive archival and Discord notification.

**Input:** A YouTube URL (e.g., `https://www.youtube.com/watch?v=VIDEO_ID`)

---

## Step 1: EXTRACT — Video Metadata + Transcript

### 1a. Metadata extraction

Run via Bash:

```bash
yt-dlp --dump-json "<URL>" > /tmp/yt-intel/metadata.json
```

Parse from the JSON:

| Field | JSON key |
|-------|----------|
| Title | `title` |
| Channel | `channel` (or `uploader`) |
| Published date | `upload_date` (format: YYYYMMDD -> reformat to YYYY-MM-DD) |
| Duration | `duration` (seconds -> format as MM:SS or HH:MM:SS) |
| Views | `view_count` |
| Likes | `like_count` |
| Tags | `tags` (array -> comma-separated) |
| Chapters | `chapters` (array of {title, start_time, end_time}) |
| Description | `description` (first 500 chars) |

### 1b. Transcript extraction

```bash
# Download auto-generated subtitles
yt-dlp --write-auto-subs --sub-lang en --skip-download -o "/tmp/yt-intel/transcript" "<URL>"
```

This produces a `.vtt` file. Clean it:

```bash
# Parse VTT -> clean text (deduplicate, remove timestamps)
python3 -c "
import re, sys

with open('/tmp/yt-intel/transcript.en.vtt', 'r') as f:
    content = f.read()

# Remove VTT header and metadata
lines = content.split('\n')
text_lines = []
seen = set()
for line in lines:
    line = line.strip()
    # Skip timestamps, empty lines, VTT headers, and style tags
    if not line or '-->' in line or line.startswith('WEBVTT') or line.startswith('Kind:') or line.startswith('Language:') or line[0:1].isdigit() and len(line) < 10:
        continue
    # Remove HTML tags
    clean = re.sub(r'<[^>]+>', '', line)
    if clean and clean not in seen:
        seen.add(clean)
        text_lines.append(clean)

with open('/tmp/yt-intel/transcript_clean.txt', 'w') as f:
    f.write(' '.join(text_lines))
print(f'Transcript: {len(text_lines)} unique lines, {len(\" \".join(text_lines))} chars')
"
```

Read the clean transcript for analysis. If transcript extraction fails (no captions available), continue with metadata-only analysis.

---

## Step 2: ANALYZE — Extract Wisdom

Apply the Fabric `extract_wisdom` structured analysis to the transcript. Produce these sections:

### Output Structure

```markdown
# YouTube Content Intelligence Report

> **Generated:** YYYY-MM-DD | **Pipeline:** /yt-intel (extract_wisdom)
> **Source:** [Title](URL)

---

## Video Metadata

| Field | Value |
|-------|-------|
| **Title** | ... |
| **Channel** | ... |
| **Published** | ... |
| **Duration** | ... |
| **Views** | ... |
| **Likes** | ... |
| **Tags** | ... |

---

## Summary
[3-5 sentence overview of the video's core argument and conclusions]

---

## Ideas
[10-15 numbered key concepts/ideas, each 1-2 sentences with **bold lead**]

---

## Insights
[5-8 numbered deeper analyses — non-obvious takeaways, each 2-3 sentences with **bold lead**]

---

## Quotes
[8-12 notable verbatim quotes from the speaker, attributed]

---

## Facts
[10-15 numbered factual claims with specific numbers, studies, or data points]

---

## Habits & Practices
[5-10 numbered actionable habits or practices mentioned]

---

## References
[Table: Type | Name | Context — for all tools, people, companies, studies, products mentioned]

---

## Recommendations
[8-12 numbered recommendations from the speaker]

---

## One-Sentence Takeaway
> **[Single most important point in bold]**
```

**Quality bar:** Each section must have substantive content. If the video doesn't support a section (e.g., no specific quotes), write "None identified" rather than fabricating content. Facts must be verifiable claims from the video, not your own additions.

---

## Step 3: RELEVANCE GATE

Score the video's relevance to **Agentic Delivery / Delivery Factory** on a weighted 1-10 scale.

### Scoring Criteria

| Criterion | Weight | What counts as relevant |
|-----------|--------|------------------------|
| **Dark factory / autonomous delivery** | 3x | Software factories, lights-out development, autonomous agents building software, zero-human-code workflows |
| **AI coding tools & practices** | 2x | Claude Code, Codex, Cursor, Copilot, agentic coding, vibe coding, spec-driven development, AI-assisted dev workflows |
| **DevOps / infrastructure automation** | 2x | CI/CD, containers, monitoring, deployment automation, GitOps, infrastructure-as-code |
| **Enterprise software delivery** | 1x | Engineering management, team structures, developer productivity, delivery metrics, technical leadership |
| **Business model / pricing / GTM** | 1x | SaaS pricing, product-led growth, AI-native business models, developer tools market |
| **Irrelevant content** | -5x | Gaming, lifestyle, politics, entertainment, cooking, sports, or other unrelated topics |

### Calculation

1. Rate each criterion 0-10
2. Multiply by weight
3. Sum weighted scores, divide by total weight (10)
4. Clamp to 1-10 range

### Decision Thresholds (UNATTENDED MODE)

| Score | Action |
|-------|--------|
| **>= 6** | **PROCEED** — Continue to Steps 4-6 (DOCX, Drive, Discord) |
| **3 - 5** | **PROCEED WITH REDUCED PRIORITY** — Continue to Steps 4-6 but prefix Discord title with `[MARGINAL]`. The pre-filter already scored this above threshold, so complete the analysis. |
| **< 3** | **SKIP** — Output the summary and score to stdout. Print: "Low relevance to Delivery Factory (score: X/10). Skipping Drive upload and Discord notification." Stop here. |

### Relevance Section (added to report if proceeding)

After the One-Sentence Takeaway, add:

```markdown
---

## Relevance to Agentic Delivery

**Score: X/10**

[Brief justification of the score — 2-3 sentences]

| Video Concept | Agentic Delivery Equivalent |
|--------------|---------------------------|
| [concept from video] | [how it maps to our product/positioning] |
| ... | ... |

### Sales Talking Points Extracted

1. [talking point derived from video insights]
2. ...
```

---

## Step 4: GENERATE DOCX (only if relevance gate passes)

### 4a. Write Markdown report

Save the complete report (Steps 2 + 3 output) to:

```
/tmp/yt-intel/YYYY-MM-DD_<video-title-slug>_<channel-slug>_extract-wisdom.md
```

**Slug rules:** lowercase, hyphens for spaces, strip special chars, max 50 chars per slug.

### 4b. Convert to branded DOCX

Use the Pandoc MCP with the Agentic Delivery reference template:

```
mcp__pandoc__convert-contents
  input_file: "/tmp/yt-intel/YYYY-MM-DD_<slug>.md"
  output_format: "docx"
  reference_doc: "/home/burhan/github/dev-delivery-factory-deployment/.claude/templates/AgenticDelivery_Document-Template_v2.docx"
  output_file: "/tmp/yt-intel/YYYY-MM-DD_<video-title-slug>_<channel-slug>_extract-wisdom.docx"
```

Verify the DOCX was created:

```bash
ls -la /tmp/yt-intel/*.docx
```

---

## Step 5: UPLOAD TO DRIVE (only if relevance gate passes)

### 5a. Copy files to accessible path

The Google Workspace MCP restricts `file://` URLs to `ALLOWED_FILE_DIRS` (`$HOME` and `/tmp`). Files in `/tmp/yt-intel/` are already accessible.

### 5b. Upload DOCX report

```
mcp__google-workspace__create_drive_file
  file_name: "YYYY-MM-DD_<video-title-slug>_<channel-slug>_extract-wisdom.docx"
  fileUrl: "file:///tmp/yt-intel/<filename>.docx"
  folder_id: "1ISaW-3u8Kcrl_sfnGUR8pYfivlmazFzD"
  user_google_email: "burhan@ocut.se"
```

**Folder:** `05-YouTube-Intelligence/Summaries/` (ID: `1ISaW-3u8Kcrl_sfnGUR8pYfivlmazFzD`)

**IMPORTANT:** If `create_drive_file` uploads with wrong MIME type (e.g. `text/plain`), the DOCX will render as garbage on Drive. Verify the upload is `application/vnd.openxmlformats-officedocument.wordprocessingml.document`. If it fails, copy the DOCX to `$HOME` first and retry from there.

### 5c. Upload clean transcript

```
mcp__google-workspace__create_drive_file
  file_name: "YYYY-MM-DD_<video-title-slug>_transcript.txt"
  fileUrl: "file:///tmp/yt-intel/transcript_clean.txt"
  folder_id: "1IZBTofol-NwDEPWRzpc_HOy8PVr2JsXy"
  user_google_email: "burhan@ocut.se"
```

**Folder:** `05-YouTube-Intelligence/Transcripts/` (ID: `1IZBTofol-NwDEPWRzpc_HOy8PVr2JsXy`)

### 5d. Get shareable link

```
mcp__google-workspace__get_drive_shareable_link
  file_id: <uploaded DOCX file ID>
  user_google_email: "burhan@ocut.se"
```

Save this link for the Discord message.

---

## Step 6: POST TO DISCORD (only if relevance gate passes)

### Channel

**#ad-intel** — Channel ID: `1475173738646470826`

### Message Format

```
**INTEL REPORT** — <Clickbait Title>

<2-3 sentence hook with the most provocative or surprising finding from the video. Make it compelling enough that someone scrolling Discord stops to read.>

**Key stats:**
- <stat 1 — most striking number/claim>
- <stat 2>
- <stat 3>
- <stat 4 (optional)>

**Relevance (X/10):** <One sentence on why this matters for Delivery Factory>

Full analysis: <Drive shareable link>
Source: <YouTube URL>
```

### Constraints

- **Max 2000 characters** (Discord message limit)
- The clickbait title should be your own reframing, not just the video title
- The hook should make someone want to read the full report
- Stats should be specific numbers, not vague claims
- Do NOT use `>>>` block quotes (renders poorly in Discord)

### Send via Discord MCP

Use `discord_discover` to find the send message action, then `discord_execute` to post. The channel ID is `1475173738646470826`.

---

## Error Handling

| Error | Recovery |
|-------|---------|
| `yt-dlp` not found | Pipeline error — yt-dlp must be pre-installed |
| No subtitles available | Continue with metadata-only analysis |
| Pandoc conversion fails | Check `pandoc` is installed. Fall back to uploading the .md file |
| Drive upload fails | Check Google auth. Retry once. |
| Discord post fails | Log error and continue — report is still on Drive |
| Video is age-restricted/private | Log error and skip |

---

## Working Directory

All intermediate files go to `/tmp/yt-intel/`. Create it if it doesn't exist:

```bash
mkdir -p /tmp/yt-intel
```

Clean up previous run files at the start of each invocation, but **do not delete** files from other videos (check filenames).
