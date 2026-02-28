---
name: content-publisher
description: Autonomous content creation, review, and multi-platform publishing for ACT ecosystem. Includes Empathy Ledger Content Hub for photography, videography, and storytelling with AI-powered tagging and ecosystem syndication.
---

# Content Publisher - ACT Communications Automation

## Empathy Ledger Content Hub (NEW)

Empathy Ledger serves as the **central content hub** for the ACT ecosystem - photography, videography, writing, and storytelling with intelligent tagging and API-driven syndication.

### Content Hub Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONTENT SOURCES                                   │
│  • Your photography and videography                                  │
│  • Storyteller contributions                                         │
│  • ACT project milestones                                           │
│  • Ralph-generated content                                          │
└────────────────────────────┬────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────────┐
│              EMPATHY LEDGER CONTENT HUB                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ MEDIA INTELLIGENCE (Consent-Gated)                           │    │
│  │ • Auto-tag objects/scenes in photos (Claude Vision)          │    │
│  │ • Face recognition → link to storyteller profiles            │    │
│  │ • Theme extraction → narrative connections                   │    │
│  │ • ALMA integration (interventions, evidence)                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ THREE-TIER VISIBILITY                                        │    │
│  │ • Private: Storyteller only                                  │    │
│  │ • Community: ACT ecosystem + approved partners               │    │
│  │ • Public: Anyone with attribution                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CONTENT SYNDICATION API                                      │    │
│  │ • GET /api/v1/content-hub/stories                           │    │
│  │ • GET /api/v1/content-hub/articles                          │    │
│  │ • GET /api/v1/content-hub/media/library                     │    │
│  │ • GET /api/v1/content-hub/themes                            │    │
│  │ • POST /api/v1/content-hub/syndicate                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
└────────────────────────────┬────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    DISTRIBUTION                                      │
│                                                                      │
│  ACT Projects (via API)          External Users           Social    │
│  • JusticeHub                    • Partner orgs           • GHL     │
│  • ACT Farm                      • Researchers            • Social  │
│  • The Harvest                   • Media                  • Posts   │
│  • Goods on Country                                                 │
│  • ACT Placemat                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Content Hub Database Tables

| Table | Purpose |
|-------|---------|
| `media_ai_analysis` | Smart photo/video tagging (consent-gated) |
| `media_person_recognition` | Face recognition with two-party consent |
| `media_narrative_themes` | Theme extraction and story linking |
| `articles` | Blog/article content for syndication |
| `content_syndication` | Track content distribution |

### Media Intelligence Pipeline

```typescript
// Photo analysis with consent gate
import { createPhotoAnalyzer } from './lib/media-intelligence/photo-analyzer';

const analyzer = createPhotoAnalyzer();

// 1. Grant consent (REQUIRED before any AI analysis)
await analyzer.grantConsent(mediaAssetId, storytellerId, grantedBy);

// 2. Analyze photo
const result = await analyzer.analyzePhoto({
  mediaAssetId,
  storytellerId,
  imageUrl,
  consentGranted: true
});

// Result includes:
// - detectedObjects: [{label, confidence}]
// - sceneClassification: 'portrait' | 'ceremony' | etc.
// - autoTags: ['community', 'gathering', ...]
// - culturalReviewRequired: boolean
// - emotionalTone: 'hopeful' | 'reflective' | etc.
```

### Face Recognition (Two-Party Consent)

```typescript
import { createFaceRecognition } from './lib/media-intelligence/face-recognition';

const faces = createFaceRecognition();

// Step 1: Uploader grants consent
await faces.grantUploaderConsent(recognitionId, uploaderId);

// Step 2: Person in photo grants consent
await faces.grantPersonConsent(recognitionId, storytellerId, personId, canBePublic);

// Only after BOTH consents can identity be linked
```

### Content Hub API Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/content-hub/stories` | GET | List published stories |
| `/api/v1/content-hub/stories/:id` | GET | Single story with full content |
| `/api/v1/content-hub/articles` | GET | List published articles |
| `/api/v1/content-hub/articles/:slug` | GET | Single article by slug |
| `/api/v1/content-hub/media/library` | GET | Browse media library |
| `/api/v1/content-hub/themes` | GET | Discover content by themes |
| `/api/v1/content-hub/search` | GET | Full-text search |
| `/api/v1/content-hub/syndicate` | POST | Register syndication |

### Content Hub File References

| Need | Reference |
|------|-----------|
| Database migration | `empathy-ledger-v2/supabase/migrations/20260109000001_content_hub_schema.sql` |
| Photo analyzer | `references/photo-analyzer.ts` |
| Face recognition | `references/face-recognition.ts` |
| Theme extractor | `references/theme-extractor.ts` |
| Syndication API | `references/content-syndication-api.ts` |

---

## When to Use
- Creating social media content from project milestones
- Generating posts from Ralph PRD completions
- Publishing approved content to social platforms
- Checking content sync status
- Generating content ideas from ecosystem activity

## Quick Commands

| Command | Description |
|---------|-------------|
| `generate <topic>` | Create draft post from ecosystem knowledge |
| `sync` | Sync all sources (Notion + Empathy Ledger) to GHL |
| `sync --notion-only` | Sync only Notion content to GHL |
| `sync --empathy-only` | Sync only Empathy Ledger content to GHL |
| `status` | Check publishing pipeline status |
| `schedule <date>` | Schedule content for specific date |
| `ideas` | Generate post ideas from recent activity |

## Content Flow

```
┌─────────────────────────────────────────────────────┐
│  1. CONTENT SOURCES                                  │
│  • Ralph PRD completions → auto-generate posts       │
│  • Sprint milestones → project updates               │
│  • Codebase commits → technical insights             │
│  • Brand alignment skill → voice/tone check          │
│  • Empathy Ledger articles & stories (NEW)          │
└─────────────────────┬───────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  2A. NOTION CONTENT HUB                              │
│  Database: e400e93e-fd9d-4a21-810c-58d67ed9fe97     │
│                                                      │
│  Status Flow:                                        │
│  Story in Development → Ready to Connect →           │
│  Conversation Scheduled → Connected & Delighted      │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────┐
│  2B. EMPATHY LEDGER CONTENT HUB                      │
│  URL: EMPATHY_LEDGER_URL (default: localhost:3000)  │
│                                                      │
│  Status Flow:                                        │
│  Draft → In Review → Elder Review → Published        │
│                                                      │
│  Article Types:                                      │
│  • story_feature, program_spotlight                  │
│  • research_summary, community_news                  │
│  • editorial, impact_report                          │
│                                                      │
│  Stories with syndication_enabled = true             │
└─────────────────────┬───────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  3. GHL PUBLISHING                                   │
│  Script: scripts/sync-content-to-ghl.mjs            │
│                                                      │
│  Syncs from BOTH sources automatically:              │
│  • Notion "Ready to Connect" items                   │
│  • Empathy Ledger published articles/stories         │
│                                                      │
│  Connected Accounts:                                 │
│  • LinkedIn (Company) - A Curious Tractor           │
│  • LinkedIn (Personal) - Benjamin Knight            │
│  • YouTube - A Curious Tractor                      │
│  • Google Business Profile                          │
│  • Bluesky - A Curious Tractor                      │
│                                                      │
│  View engagement: GHL Dashboard → Social Planner    │
└─────────────────────────────────────────────────────┘
```

## Target Accounts

| Account | Env Variable | Best For |
|---------|--------------|----------|
| LinkedIn (Company) | `GHL_LINKEDIN_ACCOUNT_ID` | Professional updates, milestones |
| LinkedIn (Personal) | `GHL_LINKEDIN_PERSONAL_ID` | Personal stories, reflections |
| YouTube | `GHL_YOUTUBE_ACCOUNT_ID` | Video announcements |
| Google Business | `GHL_GBP_ACCOUNT_ID` | Local updates, events |
| Bluesky | `GHL_BLUESKY_ACCOUNT_ID` | Tech community, open source |

## Content Generation

### From Ralph Completions
When Ralph completes a PRD feature, auto-generate announcement:

```javascript
// Read ralph/progress.txt for recent completions
// Generate post using act-brand-alignment voice
// Create in Notion with "Story in Development" status
```

### From Sprint Milestones
Query sprint tracking for completed items, generate weekly digest.

### From Ecosystem Activity
Scan all 7 project codebases for:
- New releases/tags
- Significant commits
- Documentation updates
- Test coverage improvements

## Publishing Commands

### Sync All Sources (Notion + Empathy Ledger)
```bash
node scripts/sync-content-to-ghl.mjs
```

### Sync Only Notion
```bash
node scripts/sync-content-to-ghl.mjs --notion-only
```

### Sync Only Empathy Ledger
```bash
node scripts/sync-content-to-ghl.mjs --empathy-only
```

### Dry Run (Preview)
```bash
node scripts/sync-content-to-ghl.mjs --dry-run
```

### Check Status
```bash
node scripts/sync-content-to-ghl.mjs --status
```

### Check GHL Posts
```bash
node scripts/check-ghl-engagement.mjs
```

## Brand Alignment Integration

Always invoke `act-brand-alignment` skill before publishing to ensure:
- Farm metaphors (seeds, harvest, cultivating)
- Community ownership language
- No savior framing
- Proper voice/tone

## Notion API Notes

- **API Version:** 2025-09-03
- **Endpoint:** `/data_sources/{id}/query` (not `/databases/`)
- **Create pages:** Use database ID `7005d0d1-41d3-436c-9f86-526d275c2f10`
- **Query content:** Use data source ID `e400e93e-fd9d-4a21-810c-58d67ed9fe97`

## Automated Content Pipeline

### Full Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  1. AGGREGATE ECOSYSTEM KNOWLEDGE                            │
│     node scripts/aggregate-ecosystem-knowledge.mjs           │
│                                                              │
│  Scans: Git activity, Ralph progress, Notion sprints,       │
│         Documentation updates, Deployments                   │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│  2. GENERATE CONTENT                                         │
│     node scripts/generate-content-from-knowledge.mjs         │
│                                                              │
│  Types: weekly-activity, feature-shipped, sprint-milestone,  │
│         technical-insight, project-spotlight                 │
│                                                              │
│  Output: Drafts in Notion with "Story in Development"       │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│  3. HUMAN REVIEW (Notion)                                    │
│     Edit content, add images, approve                        │
│     Change status to "Ready to Connect"                      │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│  4. PUBLISH                                                  │
│     node scripts/sync-content-to-ghl.mjs                    │
│                                                              │
│  Auto-syncs approved content to social platforms             │
└─────────────────────────────────────────────────────────────┘
```

### Quick Commands

```bash
# Generate content drafts from ecosystem knowledge
node scripts/generate-content-from-knowledge.mjs

# Preview without creating (dry run)
node scripts/generate-content-from-knowledge.mjs --dry-run

# Generate specific type
node scripts/generate-content-from-knowledge.mjs --type weekly-activity

# Sync approved content to social
node scripts/sync-content-to-ghl.mjs

# Check status
node scripts/sync-content-to-ghl.mjs --status

# Ralph content runner (combines all)
./ralph/ralph-content.sh
```

### Content Types

| Type | Description | Best Accounts |
|------|-------------|---------------|
| `weekly-activity` | Ecosystem commit summary | LinkedIn (Company) |
| `feature-shipped` | Ralph completion announcement | Both LinkedIn |
| `sprint-milestone` | Sprint progress update | LinkedIn (Company) |
| `technical-insight` | Technical learnings | LinkedIn (Personal) |
| `project-spotlight` | Project deep dive | Both LinkedIn |

### Automated Schedule (GitHub Actions)

| Schedule | Action |
|----------|--------|
| Sunday 10pm UTC (Monday 8am AEST) | Generate weekly content |
| Every 4 hours | Sync approved content to GHL |

Trigger manually: Actions → Content Pipeline → Run workflow

## Media Handling

### Media Service

```javascript
import { createMediaService } from './scripts/lib/media-service.mjs';

const media = createMediaService();

// Get brand asset
const logoUrl = media.getBrandAsset('logo');

// Get project hero image
const heroUrl = media.getProjectAsset('JusticeHub');

// Upload local file
const result = await media.uploadFile('./image.png');

// Suggest image from content
const suggestion = media.suggestImage('Building ecosystem together');
```

### Media Storage

- **Supabase**: `tednluwflfhxyucgwigh.supabase.co/storage/v1/object/public/content-media/`
- **Brand assets**: `brand/` folder
- **Project assets**: `projects/` folder
- **Uploads**: `uploads/` folder

### Image Recommendations

Content generation auto-suggests images:
1. Project mentions → Project hero image
2. Growth/seeds keywords → Pattern asset
3. Community keywords → Cockatoo asset
4. Default → ACT logo

For custom images:
1. Upload to Supabase storage
2. Add URL to Notion "Image" field or "Notes"
3. GHL will use the URL when publishing

## File References

| Need | Reference |
|------|-----------|
| Content generation | `scripts/generate-content-from-knowledge.mjs` |
| Knowledge aggregator | `scripts/aggregate-ecosystem-knowledge.mjs` |
| Media service | `scripts/lib/media-service.mjs` |
| Sync script | `scripts/sync-content-to-ghl.mjs` |
| GHL social service | `scripts/lib/ghl-social-service.mjs` |
| **Empathy Ledger service** | `scripts/lib/empathy-ledger-content.mjs` |
| Content PRD template | `ralph/content-prd-template.json` |
| Ralph content runner | `ralph/ralph-content.sh` |
| GitHub Action | `.github/workflows/content-pipeline.yml` |
| Notion database IDs | `config/notion-database-ids.json` |
| Post creation template | `references/post-template.mjs` |
| Content ideas generator | `references/content-ideas.md` |

## Empathy Ledger Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `EMPATHY_LEDGER_URL` | Empathy Ledger API base URL | `http://localhost:3000` |
| `CONTENT_HUB_API_KEY` | API key for Content Hub access | Auto-generated |

## Empathy Ledger Content Fetcher API

```javascript
import { createEmpathyLedgerService } from './lib/empathy-ledger-content.mjs';

const el = createEmpathyLedgerService({
  baseUrl: 'https://empathyledger.com',
  apiKey: process.env.CONTENT_HUB_API_KEY
});

// Get published articles
const articles = await el.getArticles({ type: 'story_feature', limit: 10 });

// Get public stories
const stories = await el.getStories({ visibility: 'public', limit: 20 });

// Get content ready for GHL sync
const content = await el.getContentForGHLSync();
// Returns: { articles: [...], stories: [...], total: N }

// Transform to GHL post format
const ghlPost = el.articleToGHLPost(article, { accountIds: [...] });

// Register syndication
await el.registerSyndication({
  contentType: 'article',
  contentId: article.id,
  destinationType: 'ghl_social',
  ghlPostId: 'xxx'
});

// Health check
const health = await el.healthCheck();
// Returns: { healthy: true, baseUrl, articlesAvailable }
```
