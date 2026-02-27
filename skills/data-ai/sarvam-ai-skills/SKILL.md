---
name: sarvam-ai-skills
description: Guide for building AI applications with Sarvam AI APIs for Indian languages. Use when working with speech-to-text transcription, text-to-speech synthesis, text translation, chat completion, or document intelligence. Covers models saarika:v2.5, saaras:v2.5/v3, bulbul:v3, mayura:v1, sarvam-translate:v1, sarvam-m, and sarvam-vision for 11-23 Indian languages. Trigger when user asks about Indian language AI, STT, TTS, translation, multilingual chatbots, voice assistants, or document processing.
license: MIT
---

# Sarvam AI Skills

Build AI applications with Sarvam AI APIs for Indian languages.

## Overview

Sarvam AI provides specialized models for Indian language processing:

- **Speech-to-Text (saarika:v2.5, saaras:v3)** - Transcribe audio in 11 languages (saaras:v3 has 5 modes)
- **Speech-to-Text-Translate (saaras:v2.5)** - Transcribe and auto-translate to English
- **Text-to-Speech (bulbul:v3)** - Natural speech with 45 voice options, temperature/pace control
- **Text Translation (mayura:v1, sarvam-translate:v1)** - Translate between 11-22 Indian languages
- **Chat Completion (sarvam-m)** - 24B parameter multilingual model
- **Document Intelligence (sarvam-vision)** - 3B parameter VLM for document processing in 23 languages

## Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
sarvam-ai-skills/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: sarvam-ai-skills
│   │   └── description: Guide for building AI applications...
│   └── Markdown instructions (required)
│       ├── API Overview
│       ├── Setup instructions
│       ├── Quick Start examples
│       └── Best practices
│
└── Bundled Resources (optional)
    ├── examples/          - Working Python code (STT, TTS, Translation, Chat)
    ├── templates/         - Documentation guides for skill creation
    └── assets/            - Configuration files (.env, requirements.txt)
```

### SKILL.md (required)

The entrypoint file containing:

**Frontmatter (YAML):**
- `name` - Skill identifier
- `description` - When to use this skill
- `license` - MIT license

**Body (Markdown):**
- API overview and endpoints
- Setup instructions
- Quick start code examples
- Best practices and patterns

### Bundled Resources (optional)

**examples/** - Executable Python scripts demonstrating each API:
- `speech_to_text.py` - Transcription examples
- `text_to_speech.py` - Speech generation examples
- `text_translation.py` - Translation examples
- `chat_completion.py` - Conversational AI examples
- `end_to_end_example.py` - Multi-API workflow examples

**templates/** - Reference documentation for skill creation:
- `speech-to-text-template.md` - Complete STT guide
- `text-to-speech-template.md` - Complete TTS guide
- `text-translation-template.md` - Complete translation guide
- `chat-completion-template.md` - Complete chat guide
- `skill-template.md` - General skill framework

**assets/** - Configuration and dependencies:
- `.env` - API key storage
- `.env.example` - Environment template
- `requirements.txt` - Python dependencies

## Setup

```bash
pip install sarvamai
export SARVAM_API_KEY="your_key"
```

```python
from sarvamai import SarvamAI
import os
client = SarvamAI(api_subscription_key=os.getenv("SARVAM_API_KEY"))
```

## Quick Start

### Speech to Text
```python
# File: examples/speech_to_text.py
with open("audio.wav", "rb") as f:
    response = client.speech_to_text.transcribe(
        file=f, language_code="hi-IN"
    )
print(response.transcript)
```

### Speech to Text Translate
```python
# File: examples/speech_to_text_translate.py
with open("audio.wav", "rb") as f:
    response = client.speech_to_text.translate(file=f, )
print(response.translation)  # English output
```

### Text to Speech
```python
# File: examples/text_to_speech.py
# bulbul:v3 (default) - 45 speakers: aditya, shubh, ritu, priya, neha, rahul, pooja, and more
response = client.text_to_speech.convert(
    text="नमस्ते", target_language_code="hi-IN",
    speaker="shubh", pace=1.0, temperature=0.6
)
# Decode: base64.b64decode(response.audios[0])
```

### Text Translation
```python
# File: examples/text_translation.py
response = client.text.translate(
    input="Hello", source_language_code="en-IN", target_language_code="hi-IN"
)
print(response.translated_text)
```

### Chat Completion
```python
# File: examples/chat_completion.py
response = client.chat.completions(
    messages=[{"role": "user", "content": "What is AI?"}],
    temperature=0.7
)
print(response.choices[0].message.content)
```

### Document Intelligence
```python
# File: examples/document_intelligence.py
# Process documents in 23 languages (22 Indian + English)
# Supports PDF, ZIP; outputs HTML or Markdown only (as ZIP)
from sarvamai import SarvamAI

client = SarvamAI(api_subscription_key=os.getenv("SARVAM_API_KEY"))

# Step 1: Create a document intelligence job
job = client.document_intelligence.create_job(
    language="hi-IN",
    output_format="md"
)
print(f"Job created: {job.job_id}")

# Step 2: Upload document
job.upload_file("document.pdf")
print("File uploaded")

# Step 3: Start processing
job.start()
print("Job started")

# Step 4: Wait for completion
status = job.wait_until_complete()
print(f"Job completed with state: {status.job_state}")

# Step 5: Get processing metrics
metrics = job.get_page_metrics()
print(f"Page metrics: {metrics}")

# Step 6: Download output (ZIP file containing the processed document)
job.download_output("./output.zip")
print("Output saved to ./output.zip")
```

### Document Intelligence - Batch Processing (Large PDFs)
```python
# File: examples/document_intelligence_batch.py
# Process large PDFs by splitting into chunks and merging results
# Automatically handles PDFs of any size (small, medium, large)
from document_intelligence_batch import process_large_pdf

# Automatically chooses best strategy:
# ≤5 pages: Direct processing (no splitting)
# >5 pages: Split into 5-page chunks, process, and merge
output = process_large_pdf(
    input_pdf="large_document.pdf",  # 25 pages → 5 chunks
    language="hi-IN",
    output_format="md",  # or "html"
    pages_per_chunk=5,
    cleanup=True
)
# Output: large_document_merged.md (all chunks merged in order)
```

### Vision
```python
# File: examples/vision.py
# Image analysis: captioning, OCR, markdown extraction
# Supports 23 languages (22 Indian + English)
import requests

# Option 1: Generate caption in Hindi
files = {"file": ("image.jpg", open("image.jpg", "rb"), "image/jpeg")}
data = {"prompt_type": "caption_in", "language": "hi-IN"}
response = requests.post(
    "https://api.sarvam.ai/vision",
    headers={"API-Subscription-Key": os.getenv("SARVAM_API_KEY")},
    files=files,
    data=data
)
print(response.json()['content'])  # "एक सुंदर पहाड़ी दृश्य"

# Option 2: Extract text (OCR)
files = {"file": ("document.jpg", open("document.jpg", "rb"), "image/jpeg")}
data = {"prompt_type": "default_ocr"}
response = requests.post(
    "https://api.sarvam.ai/vision",
    headers={"API-Subscription-Key": os.getenv("SARVAM_API_KEY")},
    files=files,
    data=data
)
print(response.json()['content'])

# Option 3: Convert to markdown
files = {"file": ("slide.jpg", open("slide.jpg", "rb"), "image/jpeg")}
data = {"prompt_type": "extract_as_markdown"}
response = requests.post(
    "https://api.sarvam.ai/vision",
    headers={"API-Subscription-Key": os.getenv("SARVAM_API_KEY")},
    files=files,
    data=data
)
print(response.json()['content'])
```

## Supported Languages

**Core 11 languages** (all models): hi-IN, en-IN, bn-IN, gu-IN, kn-IN, ml-IN, mr-IN, od-IN, pa-IN, ta-IN, te-IN

**Extended 22 languages** (sarvam-translate:v1): + as-IN, brx-IN, doi-IN, kok-IN, ks-IN, mai-IN, mni-IN, ne-IN, sa-IN, sat-IN, sd-IN, ur-IN

**Document Intelligence & Vision (23 languages)** - sarvam-vision model supports all 22 Indian languages + English with their native scripts:

| Language | Code | Script | Language | Code | Script |
|----------|------|--------|----------|------|--------|
| Hindi | hi-IN | Devanagari | Assamese | as-IN | Assamese |
| Bengali | bn-IN | Bengali | Urdu | ur-IN | Perso-Arabic |
| Tamil | ta-IN | Tamil | Sanskrit | sa-IN | Devanagari |
| Telugu | te-IN | Telugu | Nepali | ne-IN | Devanagari |
| Marathi | mr-IN | Devanagari | Konkani | kok-IN | Devanagari |
| Gujarati | gu-IN | Gujarati | Maithili | mai-IN | Devanagari |
| Kannada | kn-IN | Kannada | Sindhi | sd-IN | Devanagari/Arabic |
| Malayalam | ml-IN | Malayalam | Kashmiri | ks-IN | Perso-Arabic |
| Odia | od-IN | Odia | Dogri | doi-IN | Devanagari |
| Punjabi | pa-IN | Gurmukhi | Manipuri | mni-IN | Meetei Mayek |
| English | en-IN | Latin | Bodo | brx-IN | Devanagari |
| | | | Santali | sat-IN | Ol Chiki |

## Repository Structure

```
sarvam-skills/
├── SKILL.md                          # This file - Skill definition
├── FILE_STRUCTURE.md                 # Detailed file tree
├── README.md                         # Main documentation
├── CONTRIBUTING.md                   # Contribution guidelines
├── requirements.txt                  # Python dependencies
├── .env                             # Your API key (configured)
├── .env.example                     # Environment template
│
├── examples/                        # Working code examples
│   ├── speech_to_text.py            # STT: Transcribe audio (saarika:v2.5, saaras:v3)
│   ├── speech_to_text_translate.py  # STT-Translate: Audio → English (saaras:v2.5)
│   ├── text_to_speech.py            # TTS: Text → Audio (bulbul:v3)
│   ├── text_translation.py          # Translation: 11-22 languages
│   ├── chat_completion.py           # Chat: sarvam-m model
│   ├── document_intelligence.py     # Document: Process small docs (≤5 pages)
│   ├── document_intelligence_batch.py  # Document: Batch processing for large PDFs
│   ├── end_to_end_example.py        # Multi-API workflows
│   └── README.md                    # Examples documentation
│
└── templates/                       # Skill creation guides
    ├── API_VERSIONS.md              # ✅ Cross-verified API endpoints (Feb 2026)
    ├── speech-to-text-template.md   # STT skill creation
    ├── text-to-speech-template.md   # TTS skill creation
    ├── text-translation-template.md # Translation skills
    ├── chat-completion-template.md  # Chat AI skills
    ├── document-intelligence-template.md  # Document processing skills
    ├── skill-template.md            # General skill template
    └── README.md                    # Templates overview
```

**Key locations:**
- `examples/` - 6 working Python scripts demonstrating each API
- `templates/` - 6 comprehensive guides (including API_VERSIONS.md)
- `templates/API_VERSIONS.md` - Cross-verified API endpoints and versions
- `.env` - Your API key (already configured)
- `requirements.txt` - Python dependencies

## Common Workflows

### Multilingual Chatbot (STT → Translate → Chat → Translate → TTS)
See: `examples/end_to_end_example.py` (lines 87-130)

### Content Localization (Multi-language + Audio)
See: `examples/text_translation.py` (Example 2, lines 60-90)

### Voice Assistant (Voice → Text → AI → Voice)
See: `examples/end_to_end_example.py` (lines 133-180)

## Model Selection

**Speech-to-Text:** saarika:v2.5 (standard), saaras:v3 (5 modes: transcribe, translate, verbatim, translit, codemix)  
**Text-to-Speech:** bulbul:v3 (45 speakers, temperature/pace control)  
**Translation:** mayura:v1 (11 languages), sarvam-translate:v1 (22 languages)  
**Chat:** sarvam-m only (24B parameters)
**Document Intelligence:** sarvam-vision (3B VLM, 23 languages, PDF/PNG/JPG input)

## Best Practices

**API Key:** Use environment variables (`os.getenv("SARVAM_API_KEY")`)  
**Error Handling:** Wrap API calls in try-except  
**Audio:** Use WAV format, keep under 25MB  
**Translation:** Use auto-detection when source unknown  
**Performance:** Cache translations, batch requests

## Bundled Resources

### Examples (`examples/`)
Working Python scripts for each API endpoint:
- `speech_to_text.py` - Audio transcription
- `speech_to_text_translate.py` - Transcribe + translate
- `text_to_speech.py` - Speech generation
- `text_translation.py` - Text translation
- `chat_completion.py` - Conversational AI
- `document_intelligence.py` - Document processing (small PDFs, ≤5 pages)
- `document_intelligence_batch.py` - Batch processing for large PDFs (any size)
- `end_to_end_example.py` - Multi-API workflows

See [examples/README.md](examples/README.md) for detailed documentation.

### Templates (`templates/`)
Comprehensive guides for building skills:
- `API_VERSIONS.md` - ✅ Cross-verified API endpoints (Feb 2026)
- `speech-to-text-template.md` - STT skill creation
- `text-to-speech-template.md` - TTS skill creation
- `text-translation-template.md` - Translation skills
- `chat-completion-template.md` - Chat AI skills
- `document-intelligence-template.md` - Document processing skills
- `skill-template.md` - General skill template

See [templates/README.md](templates/README.md) and [templates/API_VERSIONS.md](templates/API_VERSIONS.md) for details.

### Structure Reference
See [FILE_STRUCTURE.md](FILE_STRUCTURE.md) for complete file tree with sizes and use cases.

## Resources

- Docs: https://docs.sarvam.ai
- Dashboard: https://dashboard.sarvam.ai
- Discord: https://discord.com/invite/5rAsykttcs
- Python SDK: https://pypi.org/project/sarvamai/