---
name: genviral
description: Complete genviral Partner API automation. Create and schedule posts (video + slideshow) across TikTok, Instagram, and any supported platform. Includes slideshow generation, file uploads, template/pack management, analytics, and full content pipeline automation.
homepage: https://github.com/fdarkaou/genviral-skill
metadata:
  openclaw:
    emoji: "🎬"
    requires:
      bins: ["curl", "jq", "bash"]
---

# genviral Partner API Skill

> **TL;DR for agents:** This skill wraps genviral's Partner API into 42+ bash commands covering all documented endpoints. Core workflow: `generate` (create slideshow from prompt) > `render` (produce images) > `review` (check quality) > `create-post` (publish). Auth via `GENVIRAL_API_KEY` env var. Config in `config.md`. New: full analytics support (summary, posts, targets, refresh). Product context in `context/`. Hook library in `hooks/`. Track results in `performance/`. The skill self-improves: post > track > analyze > adapt strategy > post better content.

Complete automation for genviral's Partner API. Create video posts, AI-generated slideshows, manage templates and image packs, track analytics, and schedule content across any platform genviral supports (TikTok, Instagram, etc.).

This skill provides a full CLI wrapper around the Partner API with commands for every endpoint, plus higher-level workflows for content creation, performance tracking, and strategy optimization.

## What This Skill Does

- **Multi-Platform Posting:** Create posts for TikTok, Instagram, or any connected account (video OR slideshow, multiple accounts per post)
- **File Management:** Upload videos/images to genviral's CDN with presigned URL flow
- **AI Slideshow Generation:** Generate photo carousels from prompts, render them to images
- **Template System:** Create reusable slideshow templates, convert winning slideshows to templates
- **Pack Management:** Manage image packs (backgrounds for slideshows)
- **Analytics:** Get summary KPIs, post-level metrics, manage tracked accounts, trigger refreshes
- **Content Pipeline:** Full automation from prompt to posted draft
- **Performance Tracking:** Log posts, track metrics, learn from results
- **Hook Library:** Maintain and evolve a library of proven content hooks

## How It Works

The core workflow is:

1. **Generate or upload media** (slideshow from prompt, or upload your own video/images)
2. **Create a post** targeting one or more accounts
3. **Schedule or publish** (immediately or at a specific time)
4. **Track performance** via analytics endpoints
5. **Learn and optimize** (promote winning hooks, retire underperformers)

The skill handles the full automation. For TikTok slideshow posts, it can optionally save as drafts so you add trending audio before publishing (music selection requires human judgment for best results).

## First-Time Setup

If this is a fresh install, read `setup.md` and walk your human through onboarding conversationally:

1. Set API key and verify it works
2. List accounts and pick which ones to post to
3. Discuss image strategy (existing packs, create new ones, generate per post, or mix)
4. Optionally set up product context and brand voice together

No hardcoded defaults needed. The agent should ask the user what they prefer and adapt. Everything done through this skill shows up in the Genviral dashboard, so the user always has full visibility and control.

All configuration lives in `config.md`. Secrets are loaded from environment variables.

## File Structure

```
genviral/
  SKILL.md                  # This file (comprehensive API reference + strategy)
  setup.md                  # Quick setup guide (3 steps)
  config.md               # API config, defaults, schedule settings

  context/
    product.md              # Product description, value props, target audience
    brand-voice.md          # Tone, style, do's and don'ts
    niche-research.md       # Platform research for the niche

  hooks/
    library.json            # Hook instances (grows over time, tracks performance)
    formulas.md             # Hook formula patterns and psychology

  content/
    scratchpad.md           # Working content plan, ideas, drafts in progress
    calendar.json           # Content calendar (upcoming planned posts)

  performance/
    log.json                # Post performance tracking (views, likes, shares)
    insights.md             # Agent's learnings from performance data
    weekly-review.md        # Weekly review template and process

  scripts/
    genviral.sh             # Main API wrapper script (all commands)

  prompts/
    slideshow.md            # Prompt templates for slideshow generation
    hooks.md                # Prompt templates for hook brainstorming
```

## Script Reference

All commands use the wrapper script:

```bash
/path/to/genviral/scripts/genviral.sh <command> [options]
```

The script requires `GENVIRAL_API_KEY` as an environment variable. It loads defaults from `config.md`.

---

## Account & File Commands

### accounts
List connected BYO and hosted accounts in your scope. Use this to discover account IDs for posting.

```bash
genviral.sh accounts
genviral.sh accounts --json
```

Returns:
- Account ID (use in --accounts for create-post)
- Platform (tiktok, instagram, etc.)
- Type (byo or hosted)
- Username, display name, status

### upload
Upload a file to genviral's CDN using the presigned URL flow. Returns a CDN URL you can use in posts.

```bash
genviral.sh upload --file video.mp4 --content-type video/mp4
genviral.sh upload --file slide1.jpg --content-type image/jpeg --filename "slide1.jpg"
```

Supported content types:
- Videos: `video/mp4`, `video/quicktime`, `video/x-msvideo`, `video/webm`, `video/x-m4v`
- Images: `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`

Returns the CDN URL (use in create-post).

### list-files
List files uploaded via the Partner API.

```bash
genviral.sh list-files
genviral.sh list-files --type video --limit 20 --offset 0
genviral.sh list-files --type image --context ai-studio,media-upload
genviral.sh list-files --context all  # include all contexts
genviral.sh list-files --json
```

`--type` accepts: `image` or `video`.

---

## Post Commands

### create-post
Create a post (video OR slideshow) targeting one or more accounts. This is the core posting command.

**Video post:**

```bash
genviral.sh create-post \
  --caption "Your caption with #hashtags" \
  --media-type video \
  --media-url "https://cdn.genviral.com/your-video.mp4" \
  --accounts "account_id_1,account_id_2" \
  --scheduled-at "2025-03-01T15:00:00Z"
```

**Slideshow post:**

```bash
genviral.sh create-post \
  --caption "Your caption" \
  --media-type slideshow \
  --media-urls "url1,url2,url3,url4,url5,url6" \
  --accounts "account_id_1" \
  --music-url "https://www.tiktok.com/@user/video/1234567890"
```

**TikTok-specific settings** (only when ALL accounts are TikTok BYO):

```bash
genviral.sh create-post \
  --caption "Caption" \
  --media-type slideshow \
  --media-urls "url1,url2,url3,url4,url5,url6" \
  --accounts "tiktok_account_id" \
  --tiktok-title "Optional title" \
  --tiktok-description "Optional description" \
  --tiktok-post-mode "MEDIA_UPLOAD" \
  --tiktok-privacy "SELF_ONLY" \
  --tiktok-disable-comment \
  --tiktok-disable-duet \
  --tiktok-disable-stitch \
  --auto-add-music true \
  --is-commercial false \
  --is-branded-content false \
  --user-consent true \
  --is-your-brand false
```

Boolean TikTok toggles support both forms:
- `--tiktok-disable-comment` (sets `true`)
- `--tiktok-disable-comment false` (explicit false)

Same behavior applies to: `--tiktok-disable-duet`, `--tiktok-disable-stitch`, `--auto-add-music`, `--is-commercial`, `--is-branded-content`, `--user-consent`, `--is-your-brand`.

TikTok `post_mode` options:
- `DIRECT_POST` - publish immediately (default)
- `MEDIA_UPLOAD` - save to TikTok drafts inbox (only supported for slideshow media)

TikTok `privacy_level` options:
- `PUBLIC_TO_EVERYONE` (default)
- `MUTUAL_FOLLOW_FRIENDS`
- `FOLLOWER_OF_CREATOR`
- `SELF_ONLY` (draft mode)

**Scheduling:**

- Omit `--scheduled-at` or set it within 30 seconds of now: post is queued for immediate publish (status: `pending`)
- Provide future ISO timestamp: post is scheduled (status: `scheduled`)
- `--scheduled-at` must be ISO 8601 with timezone offset (example: `2026-02-14T19:47:00Z`)

`--music-url` must point to a TikTok URL.

**Multi-account posting:**

You can target up to 10 accounts per post. Mix TikTok, Instagram, etc. Music is only supported when ALL accounts support it (currently TikTok only). TikTok-specific settings only work when ALL accounts are TikTok BYO.

### update-post
Update an existing post (only editable if status is draft, pending, scheduled, retry, or failed).

```bash
genviral.sh update-post \
  --id POST_ID \
  --caption "Updated caption" \
  --media-type video \
  --media-url "https://new-video.mp4" \
  --accounts "new_account_id_1,new_account_id_2" \
  --scheduled-at "2025-03-15T18:00:00Z"
```

Clear operations:
- Remove music: `--music-url null`
- Clear scheduled time: `--clear-scheduled-at`
- Clear all TikTok settings: `--clear-tiktok`

Validation notes:
- `--scheduled-at` must be ISO 8601 with timezone offset (example: `2026-02-14T19:47:00Z`)
- `--music-url` must be a TikTok URL (unless using `null` to clear)
- TikTok boolean toggles support both flag form (`--auto-add-music`) and explicit values (`--auto-add-music false`)

### retry-posts
Retry failed or partial posts.

```bash
genviral.sh retry-posts --post-ids "post_id_1,post_id_2"
genviral.sh retry-posts --post-ids "post_id_1" --account-ids "account_id_1"
```

Limits:
- `post_ids`: 1-20 IDs
- `account_ids`: 1-10 IDs

### list-posts
List posts with optional filters.

```bash
genviral.sh list-posts
genviral.sh list-posts --status scheduled --limit 20
genviral.sh list-posts --since "2025-02-01T00:00:00Z" --until "2025-02-28T23:59:59Z"
genviral.sh list-posts --json
```

`--since` and `--until` must be ISO 8601 datetimes with timezone offset.

Status filters: `draft`, `pending`, `scheduled`, `posted`, `failed`, `partial`, `retry`

### get-post
Get details for a specific post.

```bash
genviral.sh get-post --id POST_ID
```

### delete-posts (alias: `delete-post`)
Bulk delete posts by IDs.

```bash
genviral.sh delete-posts --ids "post_id_1,post_id_2,post_id_3"
# equivalent option name
genviral.sh delete-posts --post-ids "post_id_1,post_id_2,post_id_3"
# command alias
genviral.sh delete-post --ids "post_id_1,post_id_2"
```

Limit: up to 50 IDs per request.

Returns structured delete results including:
- `deletedIds`
- `blockedStatuses` (posts that can't be deleted due to status)
- `skipped`
- `errors`

---

## Slideshow Commands

### generate | generate-slideshow
Generate a slideshow from a prompt using AI, or create one manually with `--skip-ai`.

```bash
# AI mode (default)
genviral.sh generate \
  --prompt "Your hook and content prompt" \
  --pack-id PACK_ID \
  --slides 5 \
  --type educational \
  --aspect-ratio 4:5 \
  --style tiktok \
  --language en \
  --font-size small \
  --text-width narrow \
  --product-id PRODUCT_ID

# Manual/mixed mode with slide_config
genviral.sh generate \
  --skip-ai \
  --slide-config-file slide-config.json

# Pass slide_config inline
genviral.sh generate \
  --skip-ai \
  --slide-config-json '{"total_slides":2,"slide_types":["image_pack","custom_image"],...}'
```

Options:
- `--prompt` - AI prompt (required unless `--skip-ai` or `--product-id`)
- `--pack-id` - Image pack UUID for backgrounds
- `--slides` - number of slides (1-10, default: 5)
- `--type` - `educational` or `personal`
- `--aspect-ratio` - `9:16` (vertical), `4:5` (default), `1:1` (square)
- `--style` / `--text-preset` - text style preset (e.g., `tiktok`)
- `--language` - language code (e.g., `en`, `es`, `fr`)
- `--font-size` - `default` or `small`
- `--text-width` - `default` or `narrow`
- `--product-id` - optional product context identifier
- `--skip-ai` - skip AI text generation (use with `--slide-config-*`)
- `--slide-config-json` / `--slide-config` - inline JSON slide config
- `--slide-config-file` - path to JSON file with slide config

### render | render-slideshow
Render a slideshow to images via Remotion.

```bash
genviral.sh render --id SLIDESHOW_ID
```

Returns:
- Updated slideshow with rendered image URLs
- Status: `rendered`

### review | get-slideshow
Get full slideshow details for review. Shows slide text, status, rendered URLs.

```bash
genviral.sh review --id SLIDESHOW_ID
genviral.sh review --id SLIDESHOW_ID --json
genviral.sh get-slideshow --id SLIDESHOW_ID  # alias
```

### update | update-slideshow
Update slideshow fields, settings, or slides. Re-render after updating slides.

```bash
# Update title
genviral.sh update --id SLIDESHOW_ID --title "New Title"

# Update status
genviral.sh update --id SLIDESHOW_ID --status draft

# Update settings
genviral.sh update --id SLIDESHOW_ID --settings-json '{"aspect_ratio":"9:16","advanced_settings":{"text_width":"narrow"}}'

# Update slides (full replacement)
genviral.sh update --id SLIDESHOW_ID --slides '[{"image_url":"...","text_elements":[{"content":"..."}]}]'

# Load slides from file
genviral.sh update --id SLIDESHOW_ID --slides-file slides.json

# Update product_id or clear it
genviral.sh update --id SLIDESHOW_ID --product-id NEW_PRODUCT_ID
genviral.sh update --id SLIDESHOW_ID --clear-product-id
```

Options:
- `--title` - Update title
- `--status` - `draft` or `rendered`
- `--slideshow-type` - `educational` or `personal`
- `--product-id` - Link to product
- `--clear-product-id` - Detach product
- `--settings-json` / `--settings-file` - Partial settings patch
- `--slides` / `--slides-file` - Full slides array replacement

### regenerate-slide
Regenerate AI text for a single slide (0-indexed).

```bash
genviral.sh regenerate-slide --id SLIDESHOW_ID --index 2
genviral.sh regenerate-slide --id SLIDESHOW_ID --index 2 --instruction "Make this shorter and more punchy"
```

Constraints:
- `--index` must be a non-negative integer
- `--instruction` max length: 500 characters

### duplicate | duplicate-slideshow
Clone an existing slideshow as a new draft.

```bash
genviral.sh duplicate --id SLIDESHOW_ID
```

### delete | delete-slideshow
Delete a slideshow.

```bash
genviral.sh delete --id SLIDESHOW_ID
```

### list-slideshows
List slideshows with filtering and pagination.

```bash
genviral.sh list-slideshows
genviral.sh list-slideshows --status rendered --search "hook" --limit 20 --offset 0
genviral.sh list-slideshows --json
```

---

## Pack Commands

Packs are collections of background images used in slideshows.

### list-packs
List available image packs.

```bash
genviral.sh list-packs
genviral.sh list-packs --search motivation --include-public false
genviral.sh list-packs --limit 20 --offset 0 --json
```

### get-pack
Get a single pack with all image URLs.

```bash
genviral.sh get-pack --id PACK_ID
```

### create-pack
Create a new pack.

```bash
genviral.sh create-pack --name "My Pack"
genviral.sh create-pack --name "Public Pack" --is-public
# explicit boolean also supported
genviral.sh create-pack --name "Private Pack" --is-public false
```

### update-pack
Update pack name or visibility.

```bash
genviral.sh update-pack --id PACK_ID --name "New Name"
genviral.sh update-pack --id PACK_ID --is-public true
```

### delete-pack
Delete a pack.

```bash
genviral.sh delete-pack --id PACK_ID
```

### add-pack-image
Add an image to a pack.

```bash
genviral.sh add-pack-image --pack-id PACK_ID --image-url "https://cdn.example.com/image.jpg"
genviral.sh add-pack-image --pack-id PACK_ID --image-url "https://cdn.example.com/image.jpg" --file-name "hero-1.jpg"
```

### delete-pack-image
Remove an image from a pack.

```bash
genviral.sh delete-pack-image --pack-id PACK_ID --image-id IMAGE_ID
```

---

## Template Commands

Templates are reusable slideshow structures. Convert winning slideshows into templates for faster iteration.

### list-templates
List templates visible in your scope.

```bash
genviral.sh list-templates
genviral.sh list-templates --search hooks --limit 20 --offset 0
genviral.sh list-templates --json
```

### get-template
Get a single template.

```bash
genviral.sh get-template --id TEMPLATE_ID
```

### create-template
Create a template from a validated template config object.

```bash
# File input
genviral.sh create-template \
  --name "My Template" \
  --description "Description" \
  --visibility private \
  --config-file template-config.json

# Inline JSON input
genviral.sh create-template \
  --name "My Template" \
  --visibility workspace \
  --config-json '{"version":1,"structure":{"slides":[]},"content":{},"visuals":{}}'
```

Config must be valid JSON matching the template config v1 schema.
Use exactly one of:
- `--config-file <path>`
- `--config-json '<json>'`

### update-template
Update template fields.

```bash
genviral.sh update-template --id TEMPLATE_ID --name "New Name"
genviral.sh update-template --id TEMPLATE_ID --visibility workspace
genviral.sh update-template --id TEMPLATE_ID --config-file new-config.json
genviral.sh update-template --id TEMPLATE_ID --config-json '{"version":1,"structure":{"slides":[]},"content":{},"visuals":{}}'
genviral.sh update-template --id TEMPLATE_ID --clear-description
```

Config input: use one of `--config-file` or `--config-json`.

### delete-template
Delete a template.

```bash
genviral.sh delete-template --id TEMPLATE_ID
```

### create-template-from-slideshow
Convert an existing slideshow into a reusable template.

```bash
genviral.sh create-template-from-slideshow \
  --slideshow-id SLIDESHOW_ID \
  --name "Winning Format" \
  --description "Built from high-performing slideshow" \
  --visibility workspace \
  --preserve-text
```

`--preserve-text` supports both forms:
- `--preserve-text` (sets true)
- `--preserve-text true|false`

---

## Analytics Commands

Analytics endpoints provide KPIs, post metrics, and tracked account management.

### analytics-summary (alias: `get-analytics-summary`)
Get analytics summary with KPIs, trends, and content mix.

```bash
genviral.sh analytics-summary
genviral.sh analytics-summary --range 30d
genviral.sh analytics-summary --start 2026-01-01 --end 2026-01-31
genviral.sh analytics-summary --platforms tiktok,instagram
genviral.sh analytics-summary --accounts TARGET_ID_1,TARGET_ID_2
genviral.sh analytics-summary --json
```

Options:
- `--range` - Date preset: `14d`, `30d`, `90d`, `1y`, `all`
- `--start` / `--end` - Custom date range (YYYY-MM-DD), must use both together
- `--platforms` - Comma-separated platform filter
- `--accounts` - Comma-separated analytics target IDs

Returns:
- `kpis` - publishedVideos, activeAccounts, views, likes, comments, shares, saves, engagementRate (with deltas)
- `interactionSeries` - Daily interactions
- `engagementSeries` - Daily engagement rate
- `postingHeatmap` - Daily post counts
- `postingStreak` - Consecutive posting days
- `contentMix` - Posts by platform

### analytics-posts (alias: `list-analytics-posts`)
List post-level analytics with sorting and pagination.

```bash
genviral.sh analytics-posts
genviral.sh analytics-posts --range 90d --sort-by views --sort-order desc --limit 25
genviral.sh analytics-posts --start 2026-01-01 --end 2026-01-31 --platforms tiktok
genviral.sh analytics-posts --json
```

Options:
- `--range` - Date preset: `14d`, `30d`, `90d`, `1y`, `all`
- `--start` / `--end` - Custom date range
- `--platforms` - Platform filter
- `--accounts` - Target ID filter
- `--sort-by` - `published_at`, `views`, `likes`, `comments`, `shares`
- `--sort-order` - `asc` or `desc`
- `--limit` - Page size (max 100)
- `--offset` - Pagination offset

### analytics-targets
List tracked analytics accounts.

```bash
genviral.sh analytics-targets
genviral.sh analytics-targets --json
```

### analytics-target-create
Add a new tracked account.

```bash
genviral.sh analytics-target-create --platform tiktok --identifier @brand
genviral.sh analytics-target-create --platform instagram --identifier @brand --alias "Brand HQ"
```

Options:
- `--platform` - `tiktok`, `instagram`, or `youtube` (required)
- `--identifier` - Account handle (required)
- `--alias` - Display name override

### analytics-target
Get details for a single analytics target.

```bash
genviral.sh analytics-target --id TARGET_ID
```

### analytics-target-update
Update an analytics target.

```bash
genviral.sh analytics-target-update --id TARGET_ID --display-name "New Name"
genviral.sh analytics-target-update --id TARGET_ID --favorite true
genviral.sh analytics-target-update --id TARGET_ID --clear-display-name
genviral.sh analytics-target-update --id TARGET_ID --refresh-policy-json '{"freeDailyRefresh":true}'
genviral.sh analytics-target-update --id TARGET_ID --clear-refresh-policy
```

### analytics-target-delete
Delete an analytics target.

```bash
genviral.sh analytics-target-delete --id TARGET_ID
```

### analytics-target-refresh
Trigger a refresh for an analytics target.

```bash
genviral.sh analytics-target-refresh --id TARGET_ID
```

Returns:
- Refresh ID and status
- `wasFree` - Whether free refresh window was used

### analytics-refresh | get-analytics-refresh
Check refresh status.

```bash
genviral.sh analytics-refresh --id REFRESH_ID
```

Returns:
- `status` - `pending`, `processing`, `completed`, or `failed`
- `credits_used`, `free_refresh_used`
- `started_at`, `completed_at`
- `error` (if failed)

### analytics-workspace-suggestions (alias: `get-analytics-workspace-suggestions`)
List other workspace/personal scopes with tracked accounts.

```bash
genviral.sh analytics-workspace-suggestions
genviral.sh get-analytics-workspace-suggestions
genviral.sh analytics-workspace-suggestions --json
```

---

## Legacy Pipeline Commands

These are TikTok-focused convenience commands from the original skill.

### post-draft
Post a rendered slideshow as a draft (TikTok-focused).

```bash
genviral.sh post-draft \
  --id SLIDESHOW_ID \
  --caption "Your caption with #hashtags" \
  --account-ids "account_id_1"
```

Always forces TikTok draft-safe settings: `privacy_level=SELF_ONLY` and `post_mode=MEDIA_UPLOAD`.

### full-pipeline
End-to-end: generate -> render -> review -> post draft.

```bash
genviral.sh full-pipeline \
  --prompt "Your hook and content prompt" \
  --caption "Caption with #hashtags" \
  --pack-id PACK_ID \
  --slides 5 \
  --type educational \
  --style tiktok \
  --account-ids ACC_ID
```

Use `--skip-post` to stop after rendering (useful for review before posting).

---

## Content Creation Pipeline

This is the recommended workflow for producing posts.

### For Slideshow Posts

1. **Hook Selection:** Read `hooks/library.json` and pick a hook. Rotate through categories.

2. **Prompt Assembly:** Use the selected hook to build a full slideshow prompt. Reference `prompts/slideshow.md`.

3. **Generate Slideshow:** Run `generate` with the assembled prompt.

4. **Review Slide Text:** Check each slide for clarity, readability, flow. Update or regenerate weak slides.

5. **Render:** Run `render` to generate final images.

6. **Visual Review (MANDATORY):** Before posting, visually inspect EVERY rendered slide using an image analysis tool. Check: (a) background images are relevant to the topic and product, (b) text is readable and not obscured by busy backgrounds, (c) no text overflow or clipping, (d) overall quality is something you'd actually want posted. If any slide fails, regenerate it or swap the image. Never post without reviewing.

7. **Post:** Use `create-post` with media-type slideshow, or use legacy `post-draft` for TikTok drafts.

8. **Log the Post (MANDATORY):** Immediately after posting, append an entry to `content/post-log.md` with: date, time (UTC), post ID, type (slideshow/video), hook/caption snippet, status (posted/scheduled/draft), and which pack was used. This is the single source of truth for all content output. If the file doesn't exist, create it with the header format. Never skip this step.

9. **Track Performance:** Use analytics endpoints to monitor metrics. During performance checks (evening cron), update `content/post-log.md` with view/like/comment counts for recent posts.

### For Video Posts

1. **Source Video:** Upload via `upload` command or reference existing CDN URL.

2. **Write Caption:** Follow brand voice, include relevant hashtags.

3. **Create Post:** Run `create-post` with media-type video.

4. **Track Performance:** Check analytics.

---

## Platform Best Practices

### TikTok

**Slide Count:** 5-6 slides is the sweet spot.

**Aspect Ratio:** `9:16` for fullscreen, `4:5` for feed.

**Text Readability:** One idea per slide. 8-16 words max. Avoid text in bottom 20% (UI overlap).

**Narrative Structure (5-slide arc):**
1. Hook (stop the scroll)
2. Problem detail
3. Shift (introduce the solution)
4. Feature/proof
5. CTA

### Instagram

**Slide Count:** 5-10 slides for carousel posts.

**Aspect Ratio:**
- Reels: 9:16
- Feed posts: 4:5 or 1:1

---

## API Error Codes

Common Partner API error patterns:

- `401 unauthorized` - API key missing, malformed, or invalid
- `402 subscription_required` - workspace/account needs an active subscription
- `403 tier_not_allowed` - current plan tier does not include the attempted capability
- `422 invalid_payload` - request shape or enum values are invalid
- `429 rate_limited` - too many requests in a short window

---

## Troubleshooting

**"GENVIRAL_API_KEY is required"**
Export the environment variable: `export GENVIRAL_API_KEY="your_public_id.your_secret"`

**"No rendered image URLs found"**
The slideshow has not been rendered yet. Run `render` first.

**API returns 401, 402, or 403**
- `401`: verify API key format (`public_id.secret`) and token validity
- `402 subscription_required`: activate or upgrade subscription
- `403 tier_not_allowed`: your tier does not permit that feature

**Render takes too long**
Each slide takes 2-5 seconds. For 5 slides, expect up to 25 seconds.

---

## Notes

- **Multi-platform support:** Works with any platform genviral supports (TikTok, Instagram, etc.)
- **Content types:** Supports both video posts and slideshow (photo carousel) posts
- **Hosted vs BYO accounts:** Works with both hosted and BYO accounts
- **Scheduling:** Posts can be scheduled for future publish or queued for immediate posting
- **Draft mode:** For TikTok slideshow posts, use `post_mode: MEDIA_UPLOAD` to save to drafts inbox
- **Template system:** Convert winning slideshows to templates for faster iteration
- **Analytics:** Full analytics support for tracking performance across accounts
- **Never use em-dashes** in any generated content
