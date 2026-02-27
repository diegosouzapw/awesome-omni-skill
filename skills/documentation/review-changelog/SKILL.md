---
name: review-changelog
description: Review and complete new changelog entries after automated PR creation. Use when there's a new changelog PR with entries that have placeholder URLs or empty fields that need review. Triggers on phrases like "review changelog", "complete changelog entries", "fix changelog PR", or when user mentions a PR with changelog updates.
---

# Review Changelog Entries

This skill helps complete new changelog entries after our automated release workflow creates a PR. The automation generates entries with placeholder values that need human review and completion.

## Context

- **Preview URL**: https://zenml-announcements-preview.vercel.app/ - Paste the final JSON here to preview how it will look
- **Schema docs**: See `changelog_schema/README.md` for field documentation
- **S3 images**: Feature images are uploaded to `public-flavor-logos` S3 bucket in `whats_new/` folder

## Required Fields (never remove)
- `id` - unique sequential number
- `slug` - URL-friendly identifier
- `title` - headline
- `description` - detailed description
- `published_at` - ISO 8601 datetime with Z suffix

## Optional Fields (remove if not provided)
- `feature_image_url` - screenshot/image URL (S3 bucket)
- `video_url` - YouTube embed URL
- `learn_more_url` - blog post or additional info URL
- `docs_url` - ZenML documentation URL
- `published` - boolean (default: true, can usually leave out)
- `highlight_until` - datetime to stop highlighting
- `should_highlight` - boolean, makes announcement pop up for users
- `audience` - "oss", "pro", or "all" (default: "all")
- `labels` - array of: "feature", "improvement", "bugfix", "deprecation"

## Workflow

### Step 1: Identify New Entries

Run these commands to find the current branch and new entries:

```bash
git branch --show-current
git diff main -- changelog.json
```

Parse the diff to identify entries that were added (look for lines starting with `+`). New entries typically have:
- Empty strings for `feature_image_url` and `video_url`
- Placeholder URLs like "https://example.com/REPLACE-ME" or "https://docs.zenml.io/REPLACE-ME"

### Step 2: Review Each Entry

For each new entry, use the `AskUserQuestion` tool to gather information. **IMPORTANT**: Always check recent entries (last ~5) in `changelog.json` for overlapping content before proceeding.

#### 2.0 Should Include (Ask First)

Before asking about details, first ask whether this entry should be included at all:

- **Yes, include this** - Proceed with remaining questions
- **No, duplicate of recent entry** - Remove this entry entirely (content already covered in a recent changelog entry)
- **No, not significant enough** - Remove this entry entirely (too minor for the changelog)
- **Merge with another entry** - Combine with another new entry in this PR

When asking this question, **provide context** by:
1. Showing the entry title and a brief summary
2. Listing any recent entries (last ~5) that might overlap
3. Noting any obvious duplications

If the user chooses to remove or merge, skip all remaining questions for this entry and handle the deletion/merge accordingly.

#### 2.0.1 Update Entry Content (if keeping)

If the user chooses to include the entry but mentions it needs updating (e.g., wrong focus, inaccurate description, or should be based on a specific PR), ask:

- **Keep current title/description** - The generated content is accurate
- **Update based on specific PR** - User provides a PR URL with better context (fetch and read it)
- **Manual update** - User will provide new title/description text

When updating based on a PR:
1. Fetch the PR description using `gh pr view <number> --repo <repo> --json title,body`
2. Extract the key user-facing changes
3. Rewrite the entry title and description to accurately reflect the PR's changes
4. Show the proposed update to the user for approval before applying

#### 2.1 Audience
- **oss** - Only open-source users see this
- **pro** - Only ZenML Pro users see this
- **all** - Everyone sees this (default)

#### 2.2 Labels
Verify the labels are correct. Options:
- `feature` - New functionality
- `improvement` - Enhancement to existing functionality
- `bugfix` - Bug fix
- `deprecation` - Deprecated features

#### 2.3 Feature Image
Ask if there's a feature image/screenshot. Options:
- **No image** - This field will be removed from the entry
- **Already uploaded** - User provides existing S3 URL
- **Local file** - User has a local image that needs processing (see [Processing Local Images](#processing-local-images))

Images are hosted at: `https://public-flavor-logos.s3.eu-central-1.amazonaws.com/whats_new/`

#### 2.4 Video URL
Ask if there's a video demonstration. If yes, get the YouTube embed URL.
- Format: `https://www.youtube-nocookie.com/embed/VIDEO_ID`
- If no video, this field will be removed from the entry

#### 2.5 Learn More URL
Ask if there's a blog post or article. If yes, get the URL.
- Usually a zenml.io/blog post
- If no blog post, this field will be removed from the entry

#### 2.6 Docs URL
Ask if there's relevant documentation. If yes, get the URL.
- Usually https://docs.zenml.io/...
- If no docs URL, this field will be removed from the entry

#### 2.7 Should Highlight
Ask if this announcement should be highlighted (pops up for users).
- Default: false
- Set to true for major features

### Step 3: Update changelog.json

Use the Edit tool to update each entry:
1. Update `audience` if different from default
2. Update or verify `labels`
3. Either set valid URLs or remove placeholder fields entirely
4. Set `should_highlight` if true

**Important**: Remove optional fields with empty/placeholder values rather than leaving them. A cleaner entry looks like:

```json
{
  "id": 10,
  "slug": "enhanced-pipeline-scheduling",
  "title": "Enhanced Pipeline Scheduling",
  "description": "You can now pause and resume schedules...",
  "published_at": "2026-01-14T09:20:00Z",
  "published": true,
  "audience": "all",
  "labels": ["improvement"],
  "docs_url": "https://docs.zenml.io/concepts/schedules"
}
```

### Step 4: Prompt Preview

After updating all entries, tell the user:

> **Next Steps:**
> 1. Copy the updated `changelog.json` content
> 2. Go to https://zenml-announcements-preview.vercel.app/
> 3. Paste the JSON to preview how your entries will appear
> 4. If everything looks good, commit and push the changes

## Example AskUserQuestion Flow

For entry "Enhanced Stack Management with Update Functionality":

```
Questions to ask (can batch related questions):

0. Should we include this entry?
   Context: "Enhanced Stack Management with Update Functionality" - Allows updating stacks from the UI.
   Recent entries that might overlap:
   - ID 10: "Enhanced Pipeline Scheduling and Stack Management" - Already mentions stack update page

   Options:
   - Yes, include this (Recommended if it adds significant new detail)
   - No, duplicate of recent entry (content already in ID 10)
   - No, not significant enough
   - Merge with another entry in this PR

[If user chooses "Yes, include this", continue with remaining questions...]

1. Audience: Is this feature available to...
   - OSS users only
   - Pro users only
   - All users (Recommended)

2. Labels: The current labels are ["feature"]. Is this correct?
   - Yes, keep as feature
   - Change to improvement
   - Change to bugfix
   - Other (specify)

3. Do you have a feature image URL for this entry?
   - No image
   - Yes (will prompt for URL)

4. Do you have a video URL?
   - No video
   - Yes (will prompt for URL)

5. Do you have a blog post URL (learn more)?
   - No blog post
   - Yes (will prompt for URL)

6. Do you have a docs URL?
   - No documentation link
   - Yes (will prompt for URL)

7. Should this entry be highlighted (pop up for users)?
   - No (default)
   - Yes, highlight this
```

## Processing Local Images

When a user has a local image file that needs to be used as a feature image:

### Step 1: Get the Local File Path

Ask the user for the full path to their local image file (PNG, JPG, etc.).

### Step 2: Convert to AVIF

Use the `avif-image-compressor` skill to convert and compress the image:

```bash
~/.claude/skills/avif-image-compressor/scripts/convert_to_avif.sh "/path/to/image.png" --quality 30 --output "/tmp/output-name.avif"
```

- Quality 30 provides good visual fidelity for UI screenshots
- Typical compression: 80-90% size reduction

> **Note**: The AVIF compressor skill is available in the private `zenml-io/skills` repository. Team members who don't have it installed can clone it from there.

### Step 3: Upload to S3

Upload **both** the AVIF and the original PNG to the `public-flavor-logos` S3 bucket:

```bash
# Upload the AVIF version (used by the dashboard widget)
aws s3 cp /tmp/output-name.avif s3://public-flavor-logos/whats_new/output-name.avif --profile default

# Upload the PNG version (needed for email newsletters - many email clients don't support AVIF)
aws s3 cp /path/to/original-image.png s3://public-flavor-logos/whats_new/output-name.png --profile default
```

- Try the `default` AWS profile first
- If that fails, try the `zenml` profile
- All PR reviewers on this repo should have permissions to upload

### Step 4: Get the Final URL

The final URLs will be:
```
https://public-flavor-logos.s3.eu-central-1.amazonaws.com/whats_new/output-name.avif  (dashboard)
https://public-flavor-logos.s3.eu-central-1.amazonaws.com/whats_new/output-name.png   (email)
```

Use the AVIF URL for the `feature_image_url` field in the changelog entry. The PNG version will be used when building email newsletters.

### Naming Convention

Use descriptive, kebab-case names for images:
- ✅ `enhanced-logs.avif`
- ✅ `pipeline-scheduling-ui.avif`
- ❌ `screenshot1.avif`
- ❌ `image.avif`

## Tips

- Batch questions where possible to reduce back-and-forth
- For entries with no supplementary content (no images, videos, blog, docs), just clean up the entry by removing placeholder fields
- The `published: true` field can usually be left in or removed (defaults to true)
- Make sure URLs are valid - schema validation will fail otherwise
