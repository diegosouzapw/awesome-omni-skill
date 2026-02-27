---
name: jetbrains-marketplace-reviews
description: Fetch and visualize reviews for any JetBrains Marketplace plugin. Use when (1) analyzing plugin review trends, (2) getting review statistics for a time period, (3) visualizing rating distributions, (4) monitoring user feedback. Triggers on requests like "get JetBrains reviews", "copilot plugin feedback", "JetBrains marketplace reviews", "visualize plugin ratings", "analyze JetBrains plugin reviews".
---

# JetBrains Marketplace Reviews Skill

Fetch, analyze, and visualize reviews for any plugin on the JetBrains Marketplace.

## Well-Known Plugin IDs

| Plugin | ID | Slug |
|--------|----|------|
| GitHub Copilot | `17718` | `github-copilot--your-ai-pair-programmer` |
| JetBrains AI Assistant | `22282` | `ai-assistant` |

To find the ID of any plugin, open its JetBrains Marketplace page — the numeric ID is in the URL:
`https://plugins.jetbrains.com/plugin/{ID}-{slug}`

## Workflow

1. **Identify the plugin**: Determine the plugin ID from user input (default: 17718 for GitHub Copilot)
2. **Determine time scope**: Parse user input for the desired time range (e.g. "last 2 weeks", "all reviews", "since 2025-06-01")
3. **Fetch reviews**: Run the fetch script with the appropriate flags
4. **Visualize**: Run the visualize script against the fetched JSON
5. **Report**: Present summary statistics and the generated chart to the user

### Email Reports

When the user requests an email report or wants charts embedded in HTML:

1. **Fetch reviews** as usual
2. **Render email charts**: Use `render_email_charts.py` to generate an HTML report with base64-encoded PNG chart images. Email clients strip `<script>` tags, so JavaScript-based charting (Chart.js, Plotly) cannot work. This script uses matplotlib to render charts server-side and embeds them as `<img src="data:image/png;base64,...">` — universally supported by email clients.
3. **Send**: Pass the HTML output as the email body via the `send-email` skill

Alternatively, use `visualize_reviews.py --base64` to get individual chart images as a JSON dict for custom HTML assembly.

## Reference Scripts

The `scripts/` directory contains **reference implementations** that the AI agent should invoke directly. The scripts accept CLI arguments so the agent can adapt them to any user request without modifying the source.

### fetch_reviews.py

Fetches reviews from the JetBrains Marketplace API with full pagination support. Works with **any** plugin ID.

```bash
# Default: GitHub Copilot, last 3 weeks
py .github/skills/jetbrains-marketplace-reviews/scripts/fetch_reviews.py \
  --output /path/to/reviews.json

# Specific plugin, last N weeks
py .github/skills/jetbrains-marketplace-reviews/scripts/fetch_reviews.py \
  --plugin-id 22282 --plugin-slug ai-assistant \
  --weeks 8 --output /path/to/reviews.json

# All reviews from the beginning
py .github/skills/jetbrains-marketplace-reviews/scripts/fetch_reviews.py \
  --plugin-id 17718 --all --output /path/to/reviews.json

# Date range with full content
py .github/skills/jetbrains-marketplace-reviews/scripts/fetch_reviews.py \
  --since 2025-01-01 --until 2025-06-30 --full-content \
  --output /path/to/reviews.json
```

**CLI options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--plugin-id ID` | JetBrains plugin numeric ID | `17718` |
| `--plugin-slug SLUG` | Plugin URL slug (for review links) | `github-copilot--your-ai-pair-programmer` |
| `--all` | Fetch every review (paginated) | off |
| `--weeks N` | Fetch last N weeks | 3 (when no scope given) |
| `--since YYYY-MM-DD` | Start date filter | none |
| `--until YYYY-MM-DD` | End date filter | none |
| `--full-content` | Include author name and comment text | off |
| `--output PATH` | Output JSON file path | `scripts/reviews.json` |

### visualize_reviews.py

Generates visualization charts from the JSON data. **Auto-adapts** layout and granularity based on the time span of the data:

| Time Span | Granularity | Layout |
|-----------|-------------|--------|
| ≤ 30 days | Daily | 4-panel (2×2) |
| ≤ 365 days | Weekly | 4-panel (2×2) |
| > 365 days | Monthly | 6-panel (3×2) |

```bash
py .github/skills/jetbrains-marketplace-reviews/scripts/visualize_reviews.py \
  --input /path/to/reviews.json --output /path/to/chart.png \
  --title "My Plugin" --no-show
```

**CLI options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--input PATH` | Input JSON file | `scripts/reviews.json` |
| `--output PATH` | Output PNG file | `scripts/review_analysis.png` |
| `--title TEXT` | Plugin name shown in chart titles | `JetBrains Plugin` |
| `--no-show` | Skip interactive display | off |
| `--base64` | Output each chart as base64 PNG (JSON dict) instead of a single image file. For email embedding. | off |

When `--base64` is used, the output is a JSON file (same path but `.json` extension) containing a dict of `{chart_name: base64_png_string}` pairs. Use these to build custom HTML emails with inline `<img>` tags.

### render_email_charts.py

Renders all review charts as base64-encoded PNG images and outputs a complete HTML report fragment suitable for embedding in email bodies. This is the recommended approach for email reports since email clients strip `<script>` tags (making Chart.js/Plotly unusable).

```bash
# Generate email-ready HTML report
py .github/skills/jetbrains-marketplace-reviews/scripts/render_email_charts.py \
  --input /path/to/reviews.json --output /path/to/report.html \
  --title "GitHub Copilot"

# Also export individual charts as JSON for custom use
py .github/skills/jetbrains-marketplace-reviews/scripts/render_email_charts.py \
  --input /path/to/reviews.json --output /path/to/report.html \
  --title "GitHub Copilot" --json-output /path/to/charts.json
```

**CLI options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--input PATH` | Input JSON file | `scripts/reviews.json` |
| `--output PATH` | Output HTML file | `scripts/email_report.html` |
| `--title TEXT` | Plugin name shown in titles | `JetBrains Plugin` |
| `--json-output PATH` | Also save chart base64 data as JSON | none |

**Charts included:**
- Rating Distribution (bar chart)
- Average Rating by Year (bar chart, long-range only)
- Monthly Review Volume (bar chart)
- Monthly Average Rating Trend (line chart with 5-bucket moving avg + neutral line)
- Monthly Rating Breakdown (stacked bar)
- Rolling Average Rating (area chart, when enough data)
- Yearly Summary table, KPI cards, recent feedback tables

The output HTML uses inline styles and `<img src="data:image/png;base64,...">` tags, which are universally supported by email clients (Outlook, Gmail, Apple Mail, etc.).

## API Details

```
GET https://plugins.jetbrains.com/api/plugins/{plugin_id}/comments?size=100&page={page}
```

- API returns max 100 reviews per page; use `page=1,2,3…` for pagination
- Reviews are sorted newest-first

## JSON Data Schema

Each review object in the output JSON:

```json
{
  "id": 134243,
  "date": "2026-02-10",
  "rating": 5,
  "has_replies": false,
  "link": "https://plugins.jetbrains.com/plugin/17718-.../reviews#review=134243",
  "author": "Enes Oscar",
  "comment": "Overall, it provides a great experience..."
}
```

- `rating`: 0 = no rating given, 1-5 = star rating
- `author` and `comment` are only present when `--full-content` is used

## Usage Examples

- "Get the last 2 weeks of JetBrains Copilot reviews" → `--plugin-id 17718 --weeks 2`
- "Fetch all Copilot JetBrains reviews ever" → `--plugin-id 17718 --all`
- "Show reviews for JetBrains AI Assistant from January 2025" → `--plugin-id 22282 --plugin-slug ai-assistant --since 2025-01-01 --until 2025-01-31`
- "Visualize the rating trend for the last 6 months" → `--weeks 26`
- "Analyze recent feedback with full review text" → `--weeks 3 --full-content`

## Dependencies

- Python 3.x (stdlib only for fetching)
- matplotlib, numpy (for visualization)
- scipy (optional, for trend smoothing)

Install: `py -m pip install matplotlib numpy scipy`

## Notes

- Reviews with `rating: 0` mean the user left a comment without giving a star rating
- The API rate-limits large batch requests; the fetch script paginates responsibly
- For long ranges, the visualize script automatically switches to monthly aggregation and a 6-panel layout with yearly summaries, stacked breakdowns, and rolling averages
