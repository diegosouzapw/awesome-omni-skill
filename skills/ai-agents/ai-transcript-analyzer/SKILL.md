---
name: ai-transcript-analyzer
description: Analyze transcript files using OpenAI API (gpt-5-mini) to extract insights, summaries, key topics, quotes, and action items. This skill should be used when users have transcript files (from WhisperKit, YouTube, podcasts, meetings, etc.) and want AI-powered analysis, summaries, or custom insights extracted from the content. Supports both default comprehensive analysis and custom prompts for specific information extraction.
---

# AI Transcript Analyzer

## Overview

This skill provides AI-powered analysis of transcript files using OpenAI's gpt-5-mini model. Process transcripts from any source (WhisperKit, YouTube, meetings, podcasts, interviews) to extract comprehensive insights, summaries, key topics, notable quotes, and actionable recommendations. Supports custom prompts for targeted analysis.

## When to Use This Skill

Activate this skill when users:
- Have transcript files they want analyzed or summarized
- Request insights from video/audio transcripts
- Ask "what are the key points from this transcript?"
- Want to extract specific information (e.g., "what skills were discussed?", "what are the action items?")
- Need summaries of long-form content
- Want to analyze meeting notes, interview transcripts, or podcast content

## Quick Start

**Default comprehensive analysis:**
```bash
python3 scripts/analyze_transcript.py path/to/transcript.md
```

**Custom analysis prompt:**
```bash
python3 scripts/analyze_transcript.py path/to/transcript.md --prompt "List all technical tools and technologies mentioned"
```

## Core Workflow

### 1. Receive Analysis Request

When a user provides a transcript file for analysis:

1. Verify the transcript file exists and is readable
2. Determine if default analysis or custom prompt is needed
3. Confirm the output location if user wants specific placement
4. Execute the analysis script

### 2. Execute Analysis

Use the bundled `scripts/analyze_transcript.py` script for all analysis tasks.

**Default comprehensive analysis:**
```bash
python3 scripts/analyze_transcript.py transcript.md
```

This provides:
- Executive summary (2-3 paragraphs)
- Key insights (5-7 bullet points)
- Topics discussed (with summaries and key points)
- Notable quotes (3-5 memorable quotes)
- Action items (concrete next steps)
- Additional notes (context and observations)

**Custom prompt analysis:**
```bash
python3 scripts/analyze_transcript.py transcript.md --prompt "Your custom question or analysis request"
```

Examples of custom prompts:
- "List all the specific tools and products mentioned"
- "What are the technical implementation details discussed?"
- "Extract all statistics and data points mentioned"
- "Summarize the main argument in 3 paragraphs"
- "What questions were asked and how were they answered?"

**Specify output location:**
```bash
python3 scripts/analyze_transcript.py transcript.md --output path/to/analysis.md
```

**Print to stdout instead of saving:**
```bash
python3 scripts/analyze_transcript.py transcript.md --print
```

**Use different OpenAI model:**
```bash
python3 scripts/analyze_transcript.py transcript.md --model gpt-4o-mini
```

### 3. Handle Output

The script automatically:
- Reads the transcript file
- Sends to OpenAI API with appropriate prompt
- Formats response as structured markdown
- Saves to `transcript_name_analysis.md` or custom path
- Displays token usage statistics
- Returns formatted analysis with metadata header

Output format:
```markdown
# Transcript Analysis

**Source Transcript:** path/to/transcript.md
**Analysis Model:** gpt-5-mini
**Tokens Used:** 33,763

---

[Analysis content here]
```

## Configuration

### Environment Requirements

**OPENAI_API_KEY** (required):
- Must be set as environment variable
- Script will error with helpful message if not found
- Set with: `export OPENAI_API_KEY='your-key-here'`

### Model Selection

Default model: `gpt-5-mini` (fast, cost-effective, good quality)

Use `--model` flag to specify a different OpenAI model if needed.

Note: gpt-5-mini does not support temperature parameter (uses default)

### Python Dependencies

Required packages:
```bash
pip install openai
```

The script uses the official OpenAI Python client.

## Common Usage Patterns

### Pattern 1: Quick Summary

User request: "Give me a summary of this transcript"

Execute:
```bash
python3 scripts/analyze_transcript.py whisper-transcriptions/meeting.md
```

Explain to user:
- Analysis saved to `whisper-transcriptions/meeting_analysis.md`
- Includes executive summary, key insights, topics, quotes, and action items
- Token usage displayed for cost tracking

### Pattern 2: Targeted Information Extraction

User request: "What specific technologies were mentioned in the video?"

Execute:
```bash
python3 scripts/analyze_transcript.py transcript.md --prompt "List and describe each specific technology, tool, or product mentioned in this transcript. For each item include: 1) The name, 2) What it does, 3) How it was discussed, and 4) Any technical details mentioned."
```

This approach works for:
- Extracting people/companies mentioned
- Finding technical details
- Listing recommendations
- Identifying questions and answers
- Pulling out data points and statistics

### Pattern 3: Custom Output Location

User request: "Analyze this and save it in my Documents folder"

Execute:
```bash
python3 scripts/analyze_transcript.py transcript.md --output ~/Documents/analysis.md
```

### Pattern 4: Multiple Analyses

User has different questions about same transcript:

Execute sequentially with different custom prompts:
```bash
python3 scripts/analyze_transcript.py transcript.md --prompt "What are the main arguments?" --output analysis_arguments.md
python3 scripts/analyze_transcript.py transcript.md --prompt "List all action items" --output analysis_actions.md
python3 scripts/analyze_transcript.py transcript.md --prompt "Extract technical details" --output analysis_technical.md
```

### Pattern 5: Integration with WhisperKit Transcription

User workflow: "Transcribe this video and analyze it"

Execute sequentially:
```bash
# Step 1: Transcribe (using whisperkit-transcriber skill)
python3 whisperkit-transcriber/scripts/transcribe.py "https://youtube.com/watch?v=..."

# Step 2: Analyze the transcription
python3 scripts/analyze_transcript.py whisper-transcriptions/watch.md
```

Result: User gets both transcript and AI analysis.

## Script Parameters

### Required Arguments

- `transcript` - Path to transcript file to analyze

### Optional Arguments

- `--output`, `-o` - Custom output file path (default: auto-generated from input filename)
- `--model`, `-m` - OpenAI model to use (default: gpt-5-mini)
- `--prompt` - Custom analysis prompt (overrides default comprehensive analysis)
- `--print`, `-p` - Print to stdout instead of saving to file

### Examples

```bash
# Minimal usage (default analysis)
python3 scripts/analyze_transcript.py transcript.md

# Custom prompt
python3 scripts/analyze_transcript.py transcript.md --prompt "Summarize in 5 bullet points"

# Different model (if needed)
python3 scripts/analyze_transcript.py transcript.md --model other-model

# Custom output location
python3 scripts/analyze_transcript.py transcript.md --output ~/Documents/analysis.md

# Print to screen
python3 scripts/analyze_transcript.py transcript.md --print

# Combine options
python3 scripts/analyze_transcript.py transcript.md --prompt "List all people mentioned" --output people.md
```

## Token Usage and Costs

The script displays token usage after each analysis:
```
✅ Analysis complete!
   Tokens used: 33,763 (prompt: 31,536, completion: 2,227)
```

This helps track API costs:
- gpt-5-mini is cost-effective for most analyses
- Long transcripts will use more prompt tokens
- Custom prompts typically use similar tokens to default
- gpt-5-mini provides excellent quality for most use cases

## Troubleshooting

### Missing API Key

Error: `OPENAI_API_KEY environment variable not set`

Solution:
```bash
export OPENAI_API_KEY='sk-...'
```

Add to `~/.zshrc` or `~/.bashrc` for persistence.

### File Not Found

Error: `File not found: path/to/transcript.md`

Solution:
- Verify file path is correct
- Use absolute paths if relative paths fail
- Check file permissions

### API Errors

Error: `Error calling OpenAI API: ...`

Common causes:
- Invalid API key
- Rate limiting (too many requests)
- Network connectivity issues
- Model not available
- Invalid parameters (e.g., unsupported temperature value)

Solution:
- Verify API key is valid
- Wait and retry if rate limited
- Check internet connection
- Use default model if custom model fails

### Large Transcripts

For very large transcripts (>100k tokens):
- Consider splitting into sections
- Use more concise custom prompts
- Be aware of context length limits
- Monitor token usage and costs

## Best Practices

1. **Start with default analysis** - Provides comprehensive overview before diving into specifics
2. **Use custom prompts for targeted extraction** - More efficient than reading entire default analysis
3. **Combine with transcription skill** - Create end-to-end video → transcript → insights workflow
4. **Save analyses with descriptive names** - Use `--output` with clear filenames when doing multiple analyses
5. **Monitor token usage** - Track costs, especially for long transcripts or frequent use
6. **Iterate prompts** - Refine custom prompts if initial results don't match expectations
7. **Keep API key secure** - Never commit to git, use environment variables

## Integration Workflows

### Video Analysis Pipeline

1. User provides YouTube URL
2. Use whisperkit-transcriber to generate transcript
3. Use ai-transcript-analyzer for insights
4. Optionally run multiple custom prompts for specific extractions

### Meeting Notes Processing

1. User uploads meeting transcript
2. Run default analysis for overview
3. Extract action items with custom prompt
4. Identify decisions made with another custom prompt
5. Share formatted analyses with team

### Content Research

1. Collect transcripts from multiple sources
2. Run consistent custom prompt across all transcripts
3. Aggregate insights for research synthesis

## Resources

### scripts/

**analyze_transcript.py** - Main analysis script using OpenAI API:
- Reads transcript files
- Supports default comprehensive analysis
- Supports custom prompts for targeted extraction
- Uses gpt-5-mini by default (configurable)
- Outputs structured markdown with metadata
- Displays token usage statistics
- Handles errors with helpful messages

Execute this script directly to perform all transcript analysis tasks.
