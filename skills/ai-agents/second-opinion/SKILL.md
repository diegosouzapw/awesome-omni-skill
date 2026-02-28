---
name: second-opinion
description: This skill queries GPT-5.2 Pro and Gemini 3 Pro in parallel to get alternative perspectives on Claude's outputs. It extracts context automatically from conversation history, sends the same task to both models concurrently via subagents, then presents all responses for Claude to perform objective comparative analysis. The skill should be used when the user types `/second-opinion` to refine or validate Claude's output across any content type: answers, drafts, code, strategies, creative work, or analysis.
---

# Second Opinion

## Overview

This skill enables Claude to get second (and third) opinions on its outputs by querying GPT-5.2 Pro and Gemini 3 Pro in parallel. It automatically extracts the original task and Claude's response from conversation history, submits the same prompt to both alternative models via concurrent async API calls, then presents all three responses for Claude to analyze objectively.

**Claude performs the actual comparative analysis** - the skill just fetches and formats the responses.

## When to Use This Skill

Invoke this skill using `/second-opinion` after Claude has provided a response and you want:
- Validation of accuracy or completeness
- Alternative approaches or perspectives
- Comparison across multiple models
- An objective determination of which answer is best

## Workflow

```
/second-opinion → Validate API Keys
                → Extract Context (auto)
                → Query ChatGPT + Gemini (parallel via asyncio)
                → Format All Responses
                → Claude Analyzes & Compares
```

## Setup Requirements

### API Keys

Both API keys must be set as environment variables:

```bash
export OPENAI_API_KEY="your-openai-key-here"
export GEMINI_API_KEY="your-google-ai-key-here"
```

**Where to get keys:**
- **OpenAI Key**: https://platform.openai.com/api-keys
- **Gemini Key**: https://aistudio.google.com/apikey

### Dependencies

```bash
pip install aiohttp
```

## How It Works

### 1. Context Extraction

When `/second-opinion` is invoked, the skill automatically:
- Parses the conversation history
- Identifies the original user task
- Extracts Claude's current response

### 2. Parallel Model Querying

Both models are queried simultaneously using `asyncio.gather()`:
- GPT-5.2 Pro via OpenAI API
- Gemini 3 Pro via Google Generative AI API

Both receive identical prompts for fair comparison. Includes retry logic with exponential backoff.

### 3. Response Presentation

The skill formats all three responses in markdown:

```markdown
# Second Opinion Results

## Original Task
[The user's question/task]

---

## Claude's Response
[Claude's original response]

---

## GPT-5.2 Pro's Response
[ChatGPT's response]

---

## Gemini 3 Pro's Response
[Gemini's response]

---

*Response times: ChatGPT: 2.3s | Gemini: 1.8s*
```

### 4. Claude's Analysis

Claude then provides an impartial analysis:
- **Accuracy**: Which responses are factually correct?
- **Completeness**: Which best addresses all aspects?
- **Clarity**: Which is most clear and organized?
- **Recommendation**: Which response is best and why?

Claude is explicitly prompted to be impartial and acknowledge if its original response was not the best.

## Error Handling

- **Partial failure**: If one model fails, the comparison continues with available responses
- **Total failure**: Clear error message with troubleshooting guidance

## File Structure

```
second-opinion/
├── SKILL.md
└── scripts/
    ├── main.py              # Async orchestrator
    ├── api_clients.py       # Async OpenAI + Gemini clients
    ├── extract_context.py   # Context extraction
    ├── analyze_responses.py # Basic utilities
    └── format_output.py     # Response formatting
```

## How to Invoke

The script accepts JSON input via stdin. Pipe a JSON object with:
- `conversation_history`: List of message objects with `role` and `content`
- `claude_response`: Claude's response string

Example invocation:

```bash
cd ~/.claude/skills/second-opinion/scripts && echo '{"conversation_history": [{"role": "user", "content": "What is X?"}], "claude_response": "X is..."}' | python main.py
```

## Technical Details

- **Parallel Execution**: Uses `asyncio.gather()` for true concurrent I/O
- **Timeout**: 300 seconds (5 minutes) per model
- **Retry Logic**: 2 retries with exponential backoff
- **Models**: `gpt-5.2-pro` and `gemini-3-pro`

## Limitations

- Both API keys must be configured
- Works best when Claude has already provided a complete response
- Requires `aiohttp` package
