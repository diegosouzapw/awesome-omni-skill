---
name: laiye-doc-processing
description: Enterprise-grade agentic document processing API. Accurately extracts key fields and line items from invoices, receipts, orders and more across 10+ file formats, with confidence scoring. Zero-configuration, fast integration. Professionally optimized on massive enterprise documents.
License: Commercial license required. New users receive 100 free credits monthly to offset usage.
---

# Agentic Document Processing

Agentic Document Processing API — convert 10+ file formats（.jpeg,.jpg,.png,.bmp,.tiff,.pdf,.doc,.docx,.xls,.xlsx） to structured JSON/Excel with per-field confidence scores using VLM and LLM.

> **Base URL:** `https://adp-global.laiye.com/?utm_source=clawhub`

## Quick Start

```bash
curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "$ADP_APP_KEY",
    "app_secret": "$ADP_APP_SECRET",
    "file_url": "https://example.com/invoice.pdf"
  }'
```

Response:
```json
{
  "status": "success",
  "extraction_result": [
    {
      "field_key": "invoice_number",
      "field_value": "INV-2024-001",
      "field_type": "text",
      "confidence": 0.95,
      "source_pages": [1]
    },
    {
      "field_key": "total_amount",
      "field_value": "1000.00",
      "field_type": "number",
      "confidence": 0.98,
      "source_pages": [1]
    }
  ]
}
```

## Setup

### 1. Get Your API Credentials

```bash
# Contact ADP service provider to obtain:
# - app_key: Application access key
# - app_secret: Application secret key
# - X-Access-Key: Tenant-level access key
```

Save your credentials:
```bash
export ADP_ACCESS_KEY="your_access_key_here"
export ADP_APP_KEY="your_app_key_here"
export ADP_APP_SECRET="your_app_secret_here"
```

### 2. Configuration (Optional)

**Recommended: Use environment variables** (most secure):
```json5
{
  skills: {
    entries: {
      "adp-doc-extraction": {
        enabled: true,
        // API credentials loaded from environment variables
      },
    },
  },
}
```

**Security Note:**
- Set file permissions: `chmod 600 ~/.openclaw/openclaw.json`
- Never commit this file to version control
- Prefer environment variables or secret stores
- Rotate credentials regularly

## Common Tasks

### Extract from File URL

```bash
curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf"
  }'
```

### Extract from Base64

```bash
# Convert file to base64
file_base64=$(base64 -i document.pdf | tr -d '\n')

curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d "{
    \"app_key\": \"$ADP_APP_KEY\",
    \"app_secret\": \"$ADP_APP_SECRET\",
    \"file_base64\": \"$file_base64\",
    \"file_name\": \"document.pdf\"
  }"
```

### Extract with VLM Results

```bash
curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf",
    "with_rec_result": true
  }'
```

Access VLM results: `response["doc_recognize_result"]`

### Async Extraction (Large Documents)

**Create extraction task:**
```bash
curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract/create/task" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/large-document.pdf"
  }'

# Returns: {"task_id": "task_id_value", "metadata": {...}}
```

**Poll for results:**
```bash
curl -X GET "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract/query/task/{task_id}" \
  -H "X-Access-Key: $ADP_ACCESS_KEY"
```

## Advanced Features

### Custom Scale Parameter

Enhance VLM quality with higher resolution:
```bash
# model_params: { "scale": 2.0 }
```

### Specify Config Version

Use a specific extraction configuration:
```bash
# model_params: { "version_id": "config_version_id" }
```

### Document Recognition Only

Get VLM results without extraction:
```bash
curl -X POST "https://adp-global.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/recognize" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf"
  }'
```

## When to Use

### Use ADP For:
- Invoice processing
- Order processing
- Receipt processing
- Financial document processing
- Logistics document processing
- Multi-table document data extraction

### Don't Use For:
- Video transcription
- audio transcription

## Best Practices

| Document Size | Endpoint | Notes |
|---------------|----------|-------|
| Small files | `/doc/extract` (sync) | Immediate response |
| Large files | `/doc/extract/create/task` (async) | Poll for results |

**File Input:**
- `file_url`: Prefer for large files (already hosted)
- `file_base64`: Use for direct upload (max 20MB)

**Confidence Scores:**
- Range: 0-1 per field
- Review fields with confidence <0.8 manually

**Response Structure:**
- `extraction_result`: Array of extracted fields
- `doc_recognize_result`: VLM results (when `with_rec_result=true`)
- `metadata`: Processing info (pages, time, model)

## Response Schema

### Success Response
```json
{
  "status": "success",
  "message": "string",
  "extraction_result": [
    {
      "field_key": "string",
      "field_value": "string",
      "field_type": "text|number|date|table",
      "confidence": 0.95,
      "source_pages": [1],
      "table_data": [...]  // for field_type="table"
    }
  ],
  "doc_recognize_result": [...],  // when with_rec_result=true
  "extract_config_version": "string",
  "metadata": {
    "total_pages": 5,
    "processing_time": 8.2,
    "model_used": "gpt-4o"
  }
}
```

### Error Response
```json
{
  "detail": "Error message description"
}
```

## Common Use Cases

### Invoice Extraction
Extracts: invoice_number, date, vendor, total_amount, line_items

### ID Card Extraction
Extracts: name, id_number, address, issue_date, expiry_date

### Contract Extraction
Extracts: parties, dates, clauses, amounts, signatures

## Security & Privacy

### Data Handling

**Important:** Documents uploaded to ADP are transmitted to `https://adp-global.laiye.com/?utm_source=clawhub` and processed on external servers.

**Before uploading sensitive documents:**
- Review ADP privacy policy and data retention policies
- Verify encryption in transit (HTTPS) and at rest
- Confirm data deletion/retention timelines
- Test with non-sensitive sample documents first

**Best practices:**
- Do not upload highly sensitive PII until you've confirmed security posture
- Use credentials with limited permissions if available
- Rotate credentials regularly (every 90 days recommended)
- Monitor API usage logs for unauthorized access
- Never log or commit credentials to repositories

### File Size Limits

- **Max file size:** 20MB
- **Supported formats:** PDF, PNG, JPG, JPEG
- **Concurrency limit:** 10 requests per tenant
- **Timeout:** 10 minutes for sync requests

### Operational Safeguards

- Always use environment variables or secure secret stores for credentials
- Never include real credentials in code examples or documentation
- Use placeholder values like `"your_access_key_here"` in examples
- Set appropriate file permissions on configuration files (600)
- Enable credential rotation and monitor usage

## Billing

| Processing Stage | Cost |
|-----------------|------|
| Document Recognition (VLM) | 0.5 credits/page |
| Document Extraction (LLM) | 1 credit/page |
| **Total** | **1.5 credits/page** |

## Troubleshooting

**400 Bad Request:**
- Provide `app_key` and `app_secret`
- Provide exactly one input: `file_url` or `file_base64`
- Verify application has published extraction config

**401 Unauthorized:**
- Verify `X-Access-Key` is valid
- Check timestamp format (Unix timestamp)
- Verify signature format (UUID)

**404 Not Found:**
- Application does not exist
- No published extraction config found

**500 Internal Server Error:**
- Document conversion failed
- VLM recognition timeout
- LLM extraction failure

**Sync Timeout:**
- Use async endpoint for large files
- Poll `/query/task/{task_id}` for results

## Pre-Publish Security Checklist

Before publishing or updating this skill, verify:

- [ ] `package.json` declares `requiredEnv` and `primaryEnv` for credentials
- [ ] `package.json` lists API endpoints in `endpoints` array
- [ ] All code examples use placeholder values not real credentials
- [ ] No credentials or secrets are embedded in `SKILL.md` or `package.json`
- [ ] Security & Privacy section documents data handling and risks
- [ ] Configuration examples include security warnings for plaintext storage
- [ ] File permission guidance is included for config files

## References

- **API Base URL:** https://adp-global.laiye.com/
- **Flow Documentation:** [ADP Flow Index](../../refers/backend/flows/ADP-Flow-Index.md)
- **Extraction Flow:** [Document Extraction Flow](../../refers/backend/flows/09-Doc-Extraction-Flow.md)
- **Recognition Flow:** [Document Recognition Flow](../../refers/backend/flows/08-Doc-Recognition-Flow.md)
