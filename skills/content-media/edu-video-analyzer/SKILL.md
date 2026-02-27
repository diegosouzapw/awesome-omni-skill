---
name: edu-video-analyzer
description: Analyze educational YouTube channels for classroom adoption potential, curriculum alignment, and pedagogical effectiveness. Use when comparing educational video content (like MRU vs Crash Course), evaluating teaching methodologies, identifying content gaps for course design, or developing educational video strategy focused on student learning outcomes rather than monetization.
---

# Educational Video Channel Analyzer

Analyze educational YouTube channels to optimize for classroom adoption, student learning outcomes, and curriculum integration.

## Inputs Required

1. **Primary channel database** - SQLite file with videos, playlists, and subtitles tables (e.g., MRU)
2. **Comparison channel data** - SQLite file or playlist URL (e.g., Crash Course Economics)
3. **Subject area** - e.g., "Principles of Economics", "Microeconomics", "Macroeconomics"
4. **Target adoption context** - e.g., "AP Economics", "Undergraduate Intro", "Graduate supplement"

## Database Setup

Use the included `build_channel_db.py` script to create channel databases:

### Installation

```bash
pip install yt-dlp
```

### Building a Database

```bash
# Basic metadata only (fast - minutes)
python scripts/build_channel_db.py "https://www.youtube.com/@MRUniversity" mru.db

# Full metadata with descriptions and tags (slower - may take an hour+)
python scripts/build_channel_db.py "https://www.youtube.com/@MRUniversity" mru.db --details

# Complete with subtitles/transcripts (slowest but enables text search)
python scripts/build_channel_db.py "https://www.youtube.com/@MRUniversity" mru.db --details --subtitles

# Resume interrupted download
python scripts/build_channel_db.py "https://www.youtube.com/@MRUniversity" mru.db --details --subtitles --resume
```

### Script Options

| Option | Description |
|--------|-------------|
| `--details` | Fetch full metadata per video (description, tags, categories, like/comment counts) |
| `--subtitles` | Download and parse English transcripts into searchable segments |
| `--resume` | Skip videos already in database (for interrupted downloads) |
| `--rate-limit N` | Seconds to wait between API calls (default: 1, increase if throttled) |

### Database Schema

```sql
CREATE TABLE videos (
    id TEXT PRIMARY KEY,
    title TEXT,
    description TEXT,
    upload_date TEXT,
    duration_seconds INTEGER,
    view_count INTEGER,
    like_count INTEGER,
    comment_count INTEGER,
    playlist_id TEXT,
    playlist_title TEXT,
    channel_id TEXT,
    channel_title TEXT,
    tags TEXT,           -- JSON array
    categories TEXT,     -- JSON array
    has_subtitles INTEGER DEFAULT 0
);

CREATE TABLE playlists (
    id TEXT PRIMARY KEY,
    title TEXT,
    description TEXT,
    video_count INTEGER,
    channel_id TEXT
);

CREATE TABLE subtitle_segments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    video_id TEXT,
    start_time REAL,
    end_time REAL,
    text TEXT,
    FOREIGN KEY (video_id) REFERENCES videos(id)
);

-- Indexes for performance
CREATE INDEX idx_videos_playlist ON videos(playlist_id);
CREATE INDEX idx_subtitles_video ON subtitle_segments(video_id);
```

### Data Availability by Mode

| Field | Basic | --details | --subtitles |
|-------|-------|-----------|-------------|
| id, title | Yes | Yes | Yes |
| duration_seconds, view_count | Yes | Yes | Yes |
| playlist_id, playlist_title | Yes | Yes | Yes |
| description | No | Yes | Yes |
| tags, categories | No | Yes | Yes |
| like_count, comment_count | No | Yes | Yes |
| upload_date | Partial | Yes | Yes |
| subtitle_segments | No | No | Yes |

## Analysis Process

Execute these 5 stages sequentially.

### Stage 1: Educational Content Inventory

```sql
-- Total content volume
SELECT COUNT(*) as total_videos,
       SUM(duration_seconds)/3600.0 as total_hours,
       AVG(duration_seconds)/60.0 as avg_minutes,
       MIN(duration_seconds)/60.0 as shortest_min,
       MAX(duration_seconds)/60.0 as longest_min
FROM videos;

-- Playlist summary (curriculum structure)
SELECT p.title as playlist,
       p.video_count as expected,
       COUNT(v.id) as actual,
       SUM(v.duration_seconds)/60.0 as total_minutes
FROM playlists p
LEFT JOIN videos v ON v.playlist_id = p.id
GROUP BY p.id
ORDER BY actual DESC;

-- Content by playlist/series
SELECT playlist_title,
       COUNT(*) as videos,
       SUM(duration_seconds)/60.0 as total_minutes,
       AVG(duration_seconds)/60.0 as avg_minutes
FROM videos
WHERE playlist_title IS NOT NULL
GROUP BY playlist_title
ORDER BY videos DESC;

-- Upload timeline (content development pace)
SELECT strftime('%Y', upload_date) as year,
       COUNT(*) as videos,
       SUM(duration_seconds)/3600.0 as hours_added
FROM videos
WHERE upload_date IS NOT NULL
GROUP BY year
ORDER BY year;

-- Duration distribution (lecture format analysis)
SELECT
  CASE
    WHEN duration_seconds < 180 THEN '0-3 min (micro)'
    WHEN duration_seconds < 420 THEN '3-7 min (short)'
    WHEN duration_seconds < 900 THEN '7-15 min (medium)'
    WHEN duration_seconds < 1800 THEN '15-30 min (lecture)'
    ELSE '30+ min (extended)'
  END as format,
  COUNT(*) as count,
  AVG(view_count) as avg_views
FROM videos
GROUP BY format
ORDER BY count DESC;

-- Subtitle availability (accessibility check)
SELECT
    SUM(has_subtitles) as with_subtitles,
    COUNT(*) - SUM(has_subtitles) as without_subtitles,
    ROUND(100.0 * SUM(has_subtitles) / COUNT(*), 1) as subtitle_percent
FROM videos;
```

### Stage 2: Pedagogical Pattern Analysis

**Series Structure Analysis:**
```sql
-- Identify numbered series (indicates curriculum design)
SELECT title, playlist_title, duration_seconds/60.0 as minutes
FROM videos
WHERE title GLOB '*[0-9]*'
   OR title LIKE '%Part%'
   OR title LIKE '%Episode%'
   OR title LIKE '%Chapter%'
ORDER BY playlist_title, title;

-- Topic coverage by title keywords
SELECT
    SUM(CASE WHEN lower(title) LIKE '%supply%' OR lower(title) LIKE '%demand%' THEN 1 ELSE 0 END) as supply_demand,
    SUM(CASE WHEN lower(title) LIKE '%market%' THEN 1 ELSE 0 END) as markets,
    SUM(CASE WHEN lower(title) LIKE '%gdp%' OR lower(title) LIKE '%growth%' THEN 1 ELSE 0 END) as macro_growth,
    SUM(CASE WHEN lower(title) LIKE '%inflation%' OR lower(title) LIKE '%money%' THEN 1 ELSE 0 END) as monetary,
    SUM(CASE WHEN lower(title) LIKE '%trade%' OR lower(title) LIKE '%tariff%' THEN 1 ELSE 0 END) as trade,
    SUM(CASE WHEN lower(title) LIKE '%monopol%' OR lower(title) LIKE '%competition%' THEN 1 ELSE 0 END) as market_structure,
    SUM(CASE WHEN lower(title) LIKE '%game theory%' OR lower(title) LIKE '%nash%' THEN 1 ELSE 0 END) as game_theory
FROM videos;
```

**Transcript-Based Topic Analysis (requires --subtitles):**
```sql
-- Search transcripts for specific concepts
SELECT v.title, COUNT(*) as mentions
FROM videos v
JOIN subtitle_segments s ON s.video_id = v.id
WHERE lower(s.text) LIKE '%opportunity cost%'
GROUP BY v.id
ORDER BY mentions DESC
LIMIT 10;

-- Find videos discussing specific topics
SELECT DISTINCT v.title, v.playlist_title
FROM videos v
JOIN subtitle_segments s ON s.video_id = v.id
WHERE lower(s.text) LIKE '%elasticity%'
   OR lower(s.text) LIKE '%elastic demand%';

-- Concept density analysis (mentions per minute)
SELECT v.title,
       COUNT(*) as concept_mentions,
       v.duration_seconds/60.0 as minutes,
       ROUND(COUNT(*) / (v.duration_seconds/60.0), 2) as mentions_per_min
FROM videos v
JOIN subtitle_segments s ON s.video_id = v.id
WHERE lower(s.text) LIKE '%price%'
GROUP BY v.id
ORDER BY mentions_per_min DESC
LIMIT 15;
```

**Engagement Signals (proxy for learning effectiveness):**
```sql
-- Views per minute of content (engagement density)
SELECT title,
       view_count,
       duration_seconds/60.0 as minutes,
       view_count/(duration_seconds/60.0) as views_per_minute
FROM videos
WHERE duration_seconds > 0
ORDER BY views_per_minute DESC
LIMIT 20;

-- Engagement by video length (optimal lecture length)
SELECT
  CASE
    WHEN duration_seconds < 300 THEN '0-5 min'
    WHEN duration_seconds < 600 THEN '5-10 min'
    WHEN duration_seconds < 900 THEN '10-15 min'
    ELSE '15+ min'
  END as length_bucket,
  COUNT(*) as videos,
  AVG(view_count) as avg_views,
  SUM(view_count) as total_views
FROM videos
GROUP BY length_bucket;

-- Like ratio analysis (requires --details)
SELECT title,
       view_count,
       like_count,
       ROUND(100.0 * like_count / view_count, 2) as like_percent
FROM videos
WHERE view_count > 1000 AND like_count IS NOT NULL
ORDER BY like_percent DESC
LIMIT 20;
```

### Stage 3: Comparative Analysis

When comparing two channels (e.g., MRU vs Crash Course):

**Structural Differences:**
- Total content hours
- Average video length
- Series vs standalone ratio
- Topic breadth vs depth

**Pedagogical Approach:**
- Lecture style (talking head, animation, mixed)
- Pacing (concepts per video)
- Use of examples and case studies
- Production value indicators

**Curriculum Alignment:**
Map content to standard economics curriculum:
1. Scarcity and opportunity cost
2. Supply and demand
3. Elasticity
4. Market structures
5. Market failures
6. Macroeconomic measurement
7. Aggregate demand/supply
8. Fiscal and monetary policy
9. International trade
10. Economic growth

### Stage 4: Classroom Adoption Metrics

Evaluate each channel on:

**Modularity Score** (1-5): Can videos be used independently?
- Count standalone vs series-dependent videos
- Average video length (shorter = more modular)
- Clear topic titles

**Curriculum Coverage** (percentage): What standard topics are covered?
- Map titles to curriculum outline
- Identify gaps

**Accessibility Score** (1-5):
- Closed captions availability (check `has_subtitles` column)
- Speaking pace
- Technical vocabulary level
- Visual aids usage

**Assignment Integration** (1-5):
- Natural stopping points for discussion
- Question-prompting content
- Connections to real-world examples

### Stage 5: Strategic Recommendations

Generate recommendations for:

**1. Content Gap Analysis**
- Topics competitor covers that you don't
- Topics where you have advantage
- Underserved curriculum areas

**2. Format Optimization**
- Optimal video length for your audience
- Series structure recommendations
- Playlist organization

**3. Classroom Integration Guide**
- Suggested course pairings
- Assignment ideas
- Discussion prompts per video

**4. Competitive Positioning**
- Unique value proposition
- Target instructor persona
- Differentiation strategy

## Output Format

Write to `analysis/edu-content-strategy.md` and optionally generate a PDF:

```markdown
# Educational Content Strategy: [Channel Name]

Generated: [Date]
Comparison: [Comparison Channel]
Target Context: [AP/Undergraduate/Graduate]

## Executive Summary
[Key findings for educators considering adoption]

## Part 1: Content Inventory

### Your Channel Overview
[Metrics, structure, coverage]

### Comparison Channel Overview
[Same metrics for comparison]

### Head-to-Head Comparison
| Metric | Your Channel | Comparison | Advantage |
|--------|-------------|------------|-----------|
| Total Hours | | | |
| Avg Length | | | |
| Topic Coverage | | | |

## Part 2: Pedagogical Analysis

### Teaching Approach Comparison
[Narrative analysis of methodological differences]

### Curriculum Alignment
[Topic coverage mapped to standard curriculum]

### Classroom Adoption Scorecard
| Factor | Your Channel | Comparison |
|--------|-------------|------------|
| Modularity | /5 | /5 |
| Coverage | % | % |
| Accessibility | /5 | /5 |
| Assignment Fit | /5 | /5 |

## Part 3: Strategic Recommendations

### Content Gaps to Fill
[Specific topics needing coverage]

### Format Recommendations
[Length, structure, pacing]

### Classroom Integration Guide
[How instructors should use content]

### Competitive Positioning
[Differentiation strategy]

## Appendix: Topic Mapping

### Covered Topics
[List with video links]

### Missing Topics
[Standard curriculum topics not covered]
```

## PDF Generation

Convert the analysis markdown to a professionally styled PDF for sharing and printing.

### Using the PDF Converter

```bash
# Generate PDF from markdown analysis
python3 scripts/md_to_pdf.py analysis/edu-content-strategy.md analysis/edu-content-strategy.pdf
```

### PDF Features

The generated PDF includes:
- **Professional styling** - Clean typography with Helvetica/Arial fonts
- **Color-coded headers** - Blue gradient theme for hierarchy
- **Beautiful tables** - Styled headers with alternating row colors
- **Page numbers** - Automatic pagination in header
- **Print-optimized** - A4 format with proper margins
- **Executive-ready** - Suitable for stakeholder presentations

### Dependencies

```bash
# Install required Python packages
pip3 install markdown weasyprint
```

### Custom Styling

The PDF stylesheet can be customized in `scripts/md_to_pdf.py`. Key customization points:
- Page size and margins (`@page` rule)
- Color scheme (header colors, table styling)
- Font family and sizes
- Header/footer content

## Example Invocation

```
Analyze the MRU database at mru.db
Compare with: Crash Course Economics at crashcourse.db
Subject: Principles of Economics
Target: Undergraduate Introductory Economics (Econ 101/102)
```

## Notes

- Educational focus: optimize for learning outcomes, not revenue
- Key metric: Would an instructor assign this to students?
- Consider both flipped classroom and supplementary use cases
- Account for different learning styles and accessibility needs
- Use `--subtitles` mode for deeper content analysis via transcript search
