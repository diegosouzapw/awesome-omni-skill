---
name: SkipClass
description: This skill should be used when the user asks to "convert lecture recordings into notes", "transcribe class videos", "match transcripts to slides", "summarize a lecture from recording and slides", or "generate lecture notes with notifications and TODOs".
version: 0.1.0
---

# SkipClass Skill

## Purpose

Convert course recordings into structured English lecture notes by running the provided scripts (video -> audio -> transcript), matching transcripts to slide PDFs, and summarizing the content into a fixed note template that highlights key points, notifications, TODOs, and attendance checks.

Use this skill to operationalize a repeatable workflow for course note generation in this repository.

## Core Workflow (High Level)

1. Verify the workspace layout and required data folders.
2. Place lecture videos and slide PDFs in the correct folders.
3. Run the scripts to produce audio and transcript files.
4. Match transcripts to slides for context.
5. Generate structured notes using transcript + slide context.
6. Record artifacts in `data/notes/` for traceability.

## Repository Layout and Data Folders

Maintain the following folder structure under `data/`:

- `data/video/` - Raw lecture recordings (mp4/mov/etc.)
- `data/audio/` - Extracted audio files (wav)
- `data/transcripts/` - Text transcripts from audio
- `data/class_slides/` - Lecture slide PDFs
- `data/notes/` - Final lecture notes and slide matches

Create missing folders before running scripts. Keep file names consistent across stages (video -> audio -> transcript) to simplify matching.

## Scripts and Their Roles

- `scripts/video2audio.py` - Convert videos to audio with optional silence removal.
- `scripts/audio2text.py` - Transcribe audio using faster-whisper; supports chunking.
- `scripts/find_matched_slides.py` - Retrieve top-matching slide PDFs for a transcript.

## Step-by-Step Workflow

### 1) Inspect and Prepare the Workspace

Check that `data/` exists and has required subfolders. Create any missing folders.

Confirm that dependencies are available:
- `ffmpeg` and `ffprobe` on PATH.
- Python packages: `faster-whisper`, `torch` (if using Silero VAD), `pymupdf`, `sentence-transformers`.

### 2) Place Input Files

- Put lecture videos in `data/video/`.
- Put lecture slides (PDFs) in `data/class_slides/`.
- If using a pre-existing transcript, place it in `data/transcripts/` (or `data/rawtexts/` for legacy data).

Prefer consistent naming, e.g. `week2_lecture.mp4` -> `week2_lecture.wav` -> `week2_lecture.txt`.

### 3) Run Video -> Audio

Convert lecture videos to audio files:

```bash
uv run scripts/video2audio.py \
  --input-dir data/video \
  --output-dir data/audio \
  --silence-backend silero
```

### 4) Run Audio -> Text

Transcribe audio into text:

```bash
uv run scripts/audio2text.py \
  --input-dir data/audio \
  --output-dir data/transcripts \
  --model base \
  --chunk-minutes 30 \
  --chunk-overlap-minutes 0.2 \
  --parallel-workers 2 \
  --language en \
  --device cpu \
  --compute-type int8
```

### 5) Verify Transcript Quality

Open the generated transcript in `data/transcripts/` and scan for:
- Missing segments (check for long gaps or repeated phrases).
- Poor language detection (wrong language).
- Excessive silence removal (too short transcript).

If needed, re-run with adjusted parameters:
- Increase `--min-silence-duration` or lower `--silence-threshold-db`.
- Change `--model` to a larger Whisper model.
- Increase `--chunk-minutes` for long lectures.

### 6) Match Slides to Transcript

Use the slide matching script:

```bash
uv run scripts/find_matched_slides.py \
  --mode hybrid \
  --hybrid-weight 0.7 \
  --transcript data/transcripts/week2_lecture.txt \
  --slides-dir data/class_slides \
  --top-k 5
```

Record the results in `data/notes/` for reference and use the top results as context when summarizing.

### 7) Generate Lecture Notes

Combine the transcript and the most relevant slides into a concise English note that follows the structure and tone of `example_note.md`. Keep the output short, factual, and actionable. Use markdown headings and bullet points where appropriate.

#### Required Output Structure

1. **Lesson Recap**
   - Primarily summarize the **slides** content.
   - Organize by sections/topics as in the slides (use numbered sections or subheadings if helpful).
   - Include key points, definitions, examples, and important formulas/processes **from slides**.

2. **Key Takeaways By Teacher**
   - Synthesize **Lesson Recap** with the **teacher’s spoken emphasis** from the transcript.
   - Extract higher-level insights, rationale, or cautions the teacher stressed.
   - Avoid repeating the Lesson Recap verbatim; focus on distilled, teacher-driven emphasis.

3. **Important Notification**
   - Exams/quiz arrangements.
   - Assignment details and deadlines (include explicit dates).
   - Project milestones or schedule changes.

4. **TODO**
   - Direct instructions for a student who skipped class.
   - Required readings, problem sets, or code tasks.
   - Mention how to verify understanding (e.g. "work through slide X").

5. **Attendance Check**
   - State whether attendance was checked.
   - If unknown, state "Not mentioned in recording."

### 8) Save the Final Output

Write the final notes to `data/notes/<lecture_name>_notes.md`.
Optionally store slide matches to `data/notes/<lecture_name>_slides.txt`.

## Notes Quality Rules

- Keep each section concise and scannable.
- Prefer concrete terms over generic summaries.
- Avoid fabricating deadlines or announcements; if not found, state "Not mentioned."
- Convert relative time phrases into absolute dates when possible.

## Troubleshooting

- **ffmpeg/ffprobe not found**: Install ffmpeg and ensure it is on PATH.
- **faster-whisper import error**: Install `faster-whisper` and its dependencies.
- **Slide matching slow**: Reduce PDF count or use `--mode keyword`.
- **Slide matching inaccurate**: Use `--mode hybrid` and tune `--hybrid-weight`.

## Additional Resources

### Scripts
- `scripts/video2audio.py` - Video to audio conversion and silence removal.
- `scripts/audio2text.py` - Faster-whisper transcription with chunking.
- `scripts/find_matched_slides.py` - Slide matching with vector/keyword/hybrid retrieval.
