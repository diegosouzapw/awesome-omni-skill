---
name: fetching-youtube-transcripts
description: "Fetch transcripts and subtitles from YouTube videos using youtube-transcript-api. Use when extracting video transcripts, listing available languages, translating captions, or processing YouTube content for summarization or analysis."
---

<objective>
Fetch YouTube video transcripts using the youtube-transcript-api Python library. Supports listing available transcripts, fetching by language preference, translating to other languages, and bulk retrieval.
</objective>

<quick_start>
**Install the library:**
```bash
pip install youtube-transcript-api
```

**Fetch a transcript (simplest approach):**
```python
from youtube_transcript_api import YouTubeTranscriptApi

ytt_api = YouTubeTranscriptApi()
transcript = ytt_api.fetch(video_id)  # Returns list of transcript snippets

for snippet in transcript:
    print(f"{snippet.start:.1f}s: {snippet.text}")
```

**Extract video ID from URL:**
- `https://www.youtube.com/watch?v=dQw4w9WgXcQ` → `dQw4w9WgXcQ`
- `https://youtu.be/dQw4w9WgXcQ` → `dQw4w9WgXcQ`
</quick_start>

<core_operations>

**1. Fetch transcript with language preference:**
```python
from youtube_transcript_api import YouTubeTranscriptApi

ytt_api = YouTubeTranscriptApi()

# Fetch with language priority (tries 'en' first, then 'de')
transcript = ytt_api.fetch(video_id, languages=['en', 'de'])

for snippet in transcript:
    print(f"{snippet.start:.1f}s - {snippet.duration:.1f}s: {snippet.text}")
```

**2. List available transcripts:**
```python
transcript_list = ytt_api.list(video_id)

for transcript in transcript_list:
    print(f"Language: {transcript.language} ({transcript.language_code})")
    print(f"  Auto-generated: {transcript.is_generated}")
    print(f"  Translatable: {transcript.is_translatable}")
```

**3. Find specific transcript types:**
```python
transcript_list = ytt_api.list(video_id)

# Find by language preference
transcript = transcript_list.find_transcript(['en', 'de', 'es'])

# Find only manually created (human) transcripts
manual = transcript_list.find_manually_created_transcript(['en'])

# Find only auto-generated transcripts
auto = transcript_list.find_generated_transcript(['en'])

# Fetch the data
data = transcript.fetch()
```

**4. Translate a transcript:**
```python
transcript_list = ytt_api.list(video_id)
en_transcript = transcript_list.find_transcript(['en'])

if en_transcript.is_translatable:
    # Translate to German
    de_transcript = en_transcript.translate('de')
    translated_data = de_transcript.fetch()

    # List available translation languages
    for lang in en_transcript.translation_languages:
        print(f"{lang.language_code}: {lang.language}")
```

**5. Fetch multiple videos:**
```python
video_ids = ['video_id_1', 'video_id_2', 'video_id_3']

for vid in video_ids:
    try:
        transcript = ytt_api.fetch(vid, languages=['en'])
        # Process transcript...
    except Exception as e:
        print(f"Failed for {vid}: {e}")
```

</core_operations>

<cli_usage>
The library includes a CLI tool:

```bash
# Fetch transcript (outputs JSON by default)
youtube_transcript_api VIDEO_ID

# Fetch multiple videos
youtube_transcript_api VIDEO_ID_1 VIDEO_ID_2

# Specify language preference
youtube_transcript_api VIDEO_ID --languages en de es

# Translate to another language
youtube_transcript_api VIDEO_ID --languages en --translate de

# List available transcripts for a video
youtube_transcript_api --list-transcripts VIDEO_ID

# Output formats
youtube_transcript_api VIDEO_ID --format json
youtube_transcript_api VIDEO_ID --format text
youtube_transcript_api VIDEO_ID --format srt
youtube_transcript_api VIDEO_ID --format vtt
```
</cli_usage>

<transcript_data_format>
Each snippet in a transcript contains:
```python
{
    'text': 'The actual text content',
    'start': 0.0,      # Start time in seconds
    'duration': 2.5    # Duration in seconds
}
```

**Combine into plain text:**
```python
transcript = ytt_api.fetch(video_id)
full_text = ' '.join(snippet.text for snippet in transcript)
```

**Format with timestamps:**
```python
def format_timestamp(seconds):
    mins, secs = divmod(int(seconds), 60)
    hours, mins = divmod(mins, 60)
    return f"{hours:02d}:{mins:02d}:{secs:02d}"

transcript = ytt_api.fetch(video_id)
for snippet in transcript:
    print(f"[{format_timestamp(snippet.start)}] {snippet.text}")
```
</transcript_data_format>

<error_handling>
```python
from youtube_transcript_api import YouTubeTranscriptApi
from youtube_transcript_api._errors import (
    TranscriptsDisabled,
    NoTranscriptFound,
    VideoUnavailable,
    NoTranscriptAvailable
)

ytt_api = YouTubeTranscriptApi()

try:
    transcript = ytt_api.fetch(video_id, languages=['en'])
except TranscriptsDisabled:
    print("Transcripts are disabled for this video")
except NoTranscriptFound:
    print("No transcript found for requested languages")
except NoTranscriptAvailable:
    print("No transcripts available at all")
except VideoUnavailable:
    print("Video is unavailable (private, deleted, or doesn't exist)")
except Exception as e:
    print(f"Unexpected error: {e}")
```
</error_handling>

<common_patterns>

**Extract video ID from URL:**
```python
import re

def extract_video_id(url):
    patterns = [
        r'(?:v=|\/)([0-9A-Za-z_-]{11}).*',
        r'(?:embed\/)([0-9A-Za-z_-]{11})',
        r'(?:youtu\.be\/)([0-9A-Za-z_-]{11})',
    ]
    for pattern in patterns:
        match = re.search(pattern, url)
        if match:
            return match.group(1)
    return None

# Usage
video_id = extract_video_id("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

**Get transcript for summarization:**
```python
def get_transcript_text(video_id, language='en'):
    """Fetch transcript as plain text for LLM summarization."""
    ytt_api = YouTubeTranscriptApi()

    try:
        transcript = ytt_api.fetch(video_id, languages=[language, 'en'])
        return ' '.join(snippet.text for snippet in transcript)
    except Exception as e:
        return None
```

</common_patterns>

<success_criteria>
- Transcript data retrieved as list of snippets with text, start time, and duration
- Correct language selected based on preference or translation applied
- Errors handled gracefully with informative messages
</success_criteria>
