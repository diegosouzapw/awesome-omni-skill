---
name: Media Uploader - R2/S3 with video download
description: Upload files or download videos from popular platforms (YouTube, Vimeo, Bilibili, etc.) and upload to Cloudflare R2, AWS S3, or any S3-compatible storage with secure presigned download links.
summary: TypeScript-based MCP skill for uploading files and downloading videos to cloud storage (R2, S3, MinIO) with secure, temporary download links. Features multi-bucket support, interactive onboarding, video download via lux, and 5-minute default expiration.
---

# Media Uploader - R2/S3 with Video Download

Upload files or download videos from popular platforms to Cloudflare R2 or any S3-compatible storage and generate presigned download links.

**What it does:**

- Upload any file to R2/S3 and get a shareable link
- Download videos from 100+ sites using [lux](https://github.com/iawia002/lux):
  - **Chinese platforms:** Xiaohongshu (小红书/xhslink.com), Douyin, Bilibili, iXigua
  - **International:** YouTube, Vimeo, Instagram, Twitter/X, TikTok, Facebook
- Extract audio-only from videos
- All uploads get secure, expiring presigned URLs (default: 5 minutes)

## Features

- Upload files to R2/S3 buckets
- Download videos from popular platforms and auto-upload to R2/S3
- Generate presigned download URLs (configurable expiration)
- Support for any S3-compatible storage (R2, AWS S3, MinIO, etc.)
- Multiple bucket configurations
- Automatic content-type detection
- Audio-only download option for videos

## Configuration

Create `~/.r2-upload.yml` (or set `R2_UPLOAD_CONFIG` env var):

```yaml
# Default bucket (used when no bucket specified)
default: my-bucket

# Bucket configurations
buckets:
  my-bucket:
    endpoint: https://abc123.r2.cloudflarestorage.com
    access_key_id: your_access_key
    secret_access_key: your_secret_key
    bucket_name: my-bucket
    public_url: https://files.example.com # Optional: custom domain
    region: auto # For R2, use "auto"

  # Additional buckets
  personal:
    endpoint: https://xyz789.r2.cloudflarestorage.com
    access_key_id: ...
    secret_access_key: ...
    bucket_name: personal-files
    region: auto
```

### Cloudflare R2 Setup

1. Go to Cloudflare Dashboard → R2
2. Create a bucket
3. Go to R2 API Tokens: `https://dash.cloudflare.com/<ACCOUNT_ID>/r2/api-tokens`
4. Create a new API token
   - **Important:** Apply to specific bucket (select your bucket)
   - Permissions: Object Read & Write
5. Copy the Access Key ID and Secret Access Key
6. Use endpoint format: `https://<account_id>.r2.cloudflarestorage.com`
7. Set `region: auto`

### AWS S3 Setup

```yaml
aws-bucket:
  endpoint: https://s3.us-east-1.amazonaws.com
  access_key_id: ...
  secret_access_key: ...
  bucket_name: my-aws-bucket
  region: us-east-1
```

## Usage

### Upload a file

```bash
r2-upload /path/to/file.pdf
# Returns: https://files.example.com/abc123/file.pdf?signature=...
```

### Upload with custom path

```bash
r2-upload /path/to/file.pdf --key uploads/2026/file.pdf
```

### Upload to specific bucket

```bash
r2-upload /path/to/file.pdf --bucket personal
```

### Custom expiration (default: 5 minutes)

```bash
r2-upload /path/to/file.pdf --expires 24h
r2-upload /path/to/file.pdf --expires 1d
r2-upload /path/to/file.pdf --expires 300  # seconds
```

### Public URL (no signature)

```bash
r2-upload /path/to/file.pdf --public
```

### Video Download and Upload

Download videos from YouTube, Vimeo, Bilibili, Xiaohongshu (小红书), and 100+ other platforms:

```bash
# YouTube
video_download_upload "https://www.youtube.com/watch?v=dQw4w9WgXcQ"

# Xiaohongshu (小红书) share link
video_download_upload "http://xhslink.com/o/9y5ucJlQUJE"

# Bilibili
video_download_upload "https://www.bilibili.com/video/BV1xx411c7mD"

# Download specific quality
video_download_upload "URL" --stream "best"

# Audio only
video_download_upload "URL" --audio-only

# Upload to specific bucket
video_download_upload "URL" --bucket my-videos

# Custom expiration
video_download_upload "URL" --expires 1h
```

**Video Options:**

- `url` (required): Video URL to download
- `bucket`: Target bucket (uses default if not specified)
- `expires`: URL expiration time (default: 5m)
- `stream`: Video quality/format selector (e.g., "best", "137", "248")
- `audio_only`: Download audio only instead of video
- `cookie`: Cookie string for authenticated downloads
- `key`: Custom S3 key path (default: auto-generated with video title)

**Prerequisites for video download:**

- [lux](https://github.com/iawia002/lux) must be installed
- [FFmpeg](https://ffmpeg.org/) is required for video processing

Install lux:

```bash
# macOS
brew install lux

# Linux
go install github.com/iawia002/lux@latest
```

## Tools

- `r2_upload` - Upload file and get presigned URL
- `r2_list` - List files in bucket
- `r2_delete` - Delete a file
- `r2_generate_url` - Generate presigned URL for existing file
- `video_download_upload` - Download video using lux and upload to R2/S3

## Environment Variables

- `R2_UPLOAD_CONFIG` - Path to config file (default: `~/.r2-upload.yml`)
- `R2_DEFAULT_BUCKET` - Override default bucket
- `R2_DEFAULT_EXPIRES` - Default expiration in seconds (default: 300 = 5 minutes)

## Notes

- Uploaded files are stored with their original filename unless `--key` is specified
- Automatic UUID prefix added to prevent collisions (e.g., `abc123/file.pdf`)
- Content-Type automatically detected from file extension
- Presigned URLs expire after the configured duration
