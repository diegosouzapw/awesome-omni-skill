---
name: AI Nervous System - Document Intelligence
version: 1.0.0
description: Vector search and AI-powered document processing skills for OpenClaw integration
author: AI Nervous System
tags:
  - documents
  - vector-search
  - ollama
  - summarization
  - embeddings
requires:
  - fastapi_backend: http://localhost:8000
  - ollama: http://localhost:11434
  - models:
    - llama3.2:3b
    - nomic-embed-text
---

# Document Intelligence Skills

These skills enable OpenClaw (Moltbot) to interact with the AI Nervous System's
document processing pipeline via Telegram, Discord, or other interfaces.

## Skills

### document_upload

**Purpose:** Upload a document for AI processing (summarization + vector embedding)

**Trigger Phrases:**
- "upload document"
- "process file"
- "index document"
- "add to knowledge base"

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file_path | string | Yes | Path to the document file |
| file_url | string | No | URL to download document from |

**API Endpoint:**
```
POST http://localhost:8000/upload
Content-Type: multipart/form-data

file: <binary file data>
```

**Response Schema:**
```json
{
  "message": "Document uploaded successfully",
  "document": {
    "id": 1,
    "filename": "example.md",
    "status": "processing",
    "summary": null,
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

**Instructions:**
1. Accept file from user (attachment or path)
2. POST to /upload endpoint with multipart form data
3. Return document ID and initial status
4. Optionally poll /status/{doc_id} for completion
5. Return AI summary when processing completes

**Example Interaction:**
```
User: Upload this research paper
Bot: Uploading document... Created document #42
Bot: Processing with Ollama (llama3.2:3b)...
Bot: Complete! Summary:
     • Key finding 1
     • Key finding 2
     • Key finding 3
```

---

### document_search

**Purpose:** Semantic vector search across indexed documents

**Trigger Phrases:**
- "search documents for"
- "find files about"
- "what documents mention"
- "search knowledge base"

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| query | string | Yes | - | Natural language search query |
| limit | integer | No | 5 | Max results to return (1-20) |

**API Endpoint:**
```
GET http://localhost:8000/search?q={query}&limit={limit}
```

**Response Schema:**
```json
{
  "query": "machine learning optimization",
  "results": [
    {
      "id": 1,
      "filename": "ml_paper.md",
      "summary": "This document covers...",
      "similarity": 0.89
    }
  ],
  "count": 1
}
```

**Instructions:**
1. Parse user's search intent into query string
2. GET /search with URL-encoded query
3. Format results with similarity scores
4. Highlight high-relevance (>0.7) matches
5. Include document summaries in response

**Example Interaction:**
```
User: Search for documents about Python async patterns
Bot: Found 3 relevant documents:

📄 CLAUDE.md (89% match)
   Summary: Project uses async/await patterns with FastAPI...

📄 api_design.txt (72% match)
   Summary: Guidelines for async endpoint design...

📄 notes.md (58% match)
   Summary: Meeting notes discussing concurrency...
```

---

### document_status

**Purpose:** Check processing status of a specific document

**Trigger Phrases:**
- "status of document"
- "is document ready"
- "check processing"

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| doc_id | integer | Yes | Document ID to check |

**API Endpoint:**
```
GET http://localhost:8000/status/{doc_id}
```

**Response Schema:**
```json
{
  "id": 1,
  "filename": "example.md",
  "status": "completed",
  "summary": "AI-generated summary...",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:01:00Z"
}
```

**Status Values:**
- `pending` - Queued for processing
- `processing` - Ollama is analyzing
- `completed` - Ready with summary and embedding
- `error` - Processing failed

---

## Autonomous Behaviors

### Proactive File Watcher

The Document Intelligence module includes an autonomous file watcher that monitors:
- `~/Downloads`
- `~/Documents/ToProcess`

When new `.txt`, `.md`, or `.pdf` files appear, they are automatically:
1. Uploaded to the FastAPI backend
2. Processed with Ollama for summarization
3. Embedded with nomic-embed-text for vector search
4. Added to the knowledge base

**Notification Pattern:**
```
Bot: 🔔 New document detected: research_paper.pdf
Bot: Processing...
Bot: ✅ Indexed! Summary: [3 bullet points]
```

### Heartbeat Monitor

Every 30 minutes, the system audits the document pipeline:
- Checks for stuck documents (processing > 10 min)
- Retries failed Ollama tasks
- Prepares daily intelligence briefing

---

## Integration Notes

### For Telegram Bot:
```python
@bot.message_handler(content_types=['document'])
async def handle_document(message):
    file_info = await bot.get_file(message.document.file_id)
    # Download and POST to /upload
```

### For Discord Bot:
```python
@bot.event
async def on_message(message):
    if message.attachments:
        for attachment in message.attachments:
            # Download and POST to /upload
```

### Cold War Jazz Aesthetic

All responses should maintain the tactical intelligence aesthetic:
- Use `[INTEL]`, `[CLASSIFIED]`, `[BRIEFING]` prefixes
- Monospace formatting for data
- Teal (#4a9c94) for success, Amber (#d4a56a) for processing
