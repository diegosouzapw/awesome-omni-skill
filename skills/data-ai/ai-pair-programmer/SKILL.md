---
name: ai-pair-programmer
description: Get second opinions from AI providers (Grok, ChatGPT, Gemini) on implementation plans, code, or architecture decisions. Use when the user asks to "review with [AI name]", "get [AI]'s opinion", "pair program with [AI]", or wants a second perspective on their solution. Supports multiple providers simultaneously for comparative feedback. (Triggers: review with grok, review with gemini, review with chatgpt, pair program, second opinion, ai review)
allowed-tools: ["Bash(python3 {{SKILL_DIR}}/scripts/pair_review.py:*)"]
---

# AI Pair Programmer

You are the lead architect. AI pair programmers (Grok, ChatGPT, Gemini) provide second opinions.

## Supported Providers

| Provider | Aliases | API Key Env Var | Model Override Env Var | Default Model |
|----------|---------|-----------------|------------------------|---------------|
| Grok (xAI) | `grok`, `xai` | `XAI_API_KEY` | `GROK_MODEL` | grok-4-1-fast-reasoning |
| ChatGPT (OpenAI) | `chatgpt`, `openai`, `gpt` | `OPENAI_API_KEY` | `OPENAI_MODEL` | gpt-5.1 |
| Gemini (Google) | `gemini`, `google` | `GEMINI_API_KEY` | `GEMINI_MODEL` | gemini-3-pro-preview |

## Choosing a Provider

When the user mentions a specific AI, use that provider:
- "review with Grok" → `--provider grok`
- "get ChatGPT's opinion" → `--provider chatgpt`
- "ask Gemini" → `--provider gemini`
- "review with Grok and Gemini" → `--provider grok,gemini`
- "get multiple opinions" / "ask all AIs" → `--provider all`

If no specific AI is mentioned, prefer using multiple providers for better coverage:
- `--provider all` uses all configured providers in parallel

## CRITICAL: Content Requirement

**Every call MUST have content to review.** The script will fail with "No content to review" if you don't provide one of these:

| Content Source | When to Use |
|----------------|-------------|
| `--files file1.py file2.py` | Code review with multiple files |
| `--file path/to/file.py` | Code review with single file |
| `--proposal "Your approach..."` | Architecture/plan reviews WITHOUT code files |
| `"Content here"` (positional) | Quick reviews with inline content |
| stdin (piped) | Git diffs via `git diff \| python3 ...` |

**Common mistake:** Calling the script with only `--context` and `--app-context` but no content source. This will fail!

```bash
# WRONG - No content source, will fail!
python3 pair_review.py --provider grok \
  --app-context "React app" \
  --context "User wants dark mode"

# CORRECT - Use --proposal for architecture reviews without files
python3 pair_review.py --provider grok \
  --app-context "React app" \
  --context "User wants dark mode" \
  --proposal "Add ThemeContext provider, store preference in localStorage, use CSS variables"
```

## REQUIRED: Always Provide Full Context

When calling any AI pair programmer, you MUST provide:

1. **App Context** (`--app-context`): Summarize the tech stack
   - Language/runtime (TypeScript, Python, Go, C#, etc.)
   - Framework (React, Next.js, Django, Express, .NET, etc.)
   - Architecture patterns (REST API, GraphQL, microservices, monolith, MVVM, etc.)
   - Key libraries or infrastructure (Redis, PostgreSQL, Docker, etc.)

2. **Problem Context** (`--context`): What is the user trying to accomplish?
   - The user's original request or the problem being solved
   - Any constraints or requirements mentioned

3. **Your Proposal** (`--proposal`): What is YOUR proposed solution?
   - Your approach to solving the problem
   - Key decisions you've made and why
   - This is what the AI(s) will evaluate
   - **IMPORTANT:** If not providing `--files` or `--file`, `--proposal` becomes the content to review!

4. **Already Considered** (`--considered`): What you already tried or rejected (optional)
   - Approaches that didn't work and why
   - Ideas you ruled out so reviewers don't suggest them again
   - Example: "Tried using localStorage but it's not persistent enough; considered Redux but overkill for this use case"

5. **Files/Code** (`--files` or `--file`): The relevant code
   - Only include files directly relevant to the review
   - For large changes, use `--summary` flag instead
   - **If omitted:** You MUST provide `--proposal` which will be used as the content

## Workflow

1. **Understand the request** - Clarify what the user wants
2. **Analyze the codebase** - Read relevant files, understand patterns
3. **Formulate YOUR solution** - Decide on an approach as lead architect
4. **Call AI pair programmer(s)** - Provide full context
5. **Synthesize feedback** - Evaluate points critically, note agreements/disagreements
6. **Present to user** - Share your final recommendation

## When NOT to Call AI Pair Programmers

Skip to save API costs:
- Trivial syntax fixes or one-liners
- Pure UI/styling tweaks
- When you're 100% confident
- User says "no second opinion" or "just implement"
- Questions better answered by docs/search

Reserve for substantive decisions: plans, refactors, architecture trade-offs.

## Script Usage

```bash
# Single provider - Plan review (with files)
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider grok \
  --app-context "React/TypeScript frontend with Redux, Node.js/Express backend, PostgreSQL" \
  --context "User wants to add real-time notifications" \
  --proposal "Use WebSockets with Socket.io, store notification state in Redux, persist to DB" \
  --type plan \
  --files src/services/NotificationService.ts src/store/notificationSlice.ts

# Architecture decision WITHOUT files (--proposal becomes the content)
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider grok \
  --app-context ".NET 9 MvvmCross app for iOS/Android" \
  --context "Auto-scroll to first validation error field after form submit fails" \
  --proposal "Add ScrollToField observable property to ViewModel. iOS: TableViewSource calls ScrollToRow(). Android: Find View Y position and scroll NestedScrollView." \
  --type architecture

# Multiple providers - Architecture decision (with already-considered alternatives)
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider grok,gemini \
  --app-context "Python FastAPI microservices, Docker/Kubernetes, Redis for caching" \
  --context "Need to decide on inter-service communication pattern" \
  --proposal "Use async message queue (RabbitMQ) instead of synchronous HTTP calls" \
  --considered "Tried gRPC but adds complexity; considered Redis pub/sub but need persistence" \
  --type architecture

# All configured providers - Code review
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider all \
  --app-context "Go REST API with Chi router, PostgreSQL, clean architecture" \
  --context "Refactoring authentication to support OAuth2" \
  --proposal "Add OAuth2 middleware, separate auth logic into domain service" \
  --type code \
  --file internal/auth/service.go

# ChatGPT only - API design
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider chatgpt \
  --app-context "Django REST Framework backend, React frontend" \
  --context "Designing new API endpoints for user management" \
  --proposal "RESTful endpoints with versioning, pagination, and rate limiting" \
  --type architecture

# Gemini only - Performance review
python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider gemini \
  --app-context "Next.js app with server components, Prisma ORM, Vercel deployment" \
  --context "Page load times are slow on the dashboard" \
  --proposal "Add Redis caching layer, optimize database queries, use React Suspense" \
  --type code \
  --files src/app/dashboard/page.tsx src/lib/queries.ts

# Git diff review with multiple providers
git diff | python3 {{SKILL_DIR}}/scripts/pair_review.py \
  --provider grok,chatgpt \
  --app-context "Ruby on Rails monolith, Sidekiq for background jobs" \
  --context "Bug fix for race condition in order processing" \
  --proposal "Added database-level locking and idempotency checks" \
  --diff

# List available providers and their status
python3 {{SKILL_DIR}}/scripts/pair_review.py --list-providers
```

## Review Types

| Type | Use For |
|------|---------|
| `--type plan` | Implementation plans, step-by-step approaches |
| `--type code` | Code review, bug fixes, refactoring |
| `--type architecture` | System design, technology choices, patterns |
| `--type general` | Anything else |
| `--summary` | High-level descriptions of large changes |
| `--diff` | Git diffs, focused on what changed |
| `--files` | Multiple related files as a unit |

## Multi-Provider Output

When using multiple providers, the output includes:
- Each provider's feedback with clear attribution
- A synthesis guidance section highlighting:
  - Points of agreement (high confidence)
  - Points of disagreement (needs your judgment)
  - Unique insights from each AI

## After Receiving Feedback

As lead architect, YOU make the final decision:

1. **Evaluate each point** - Is the concern valid for this specific context?
2. **Note agreements** - Where AIs agree, confidence increases
3. **Consider adjustments** - If valid issues raised, incorporate them
4. **Disagree when appropriate** - If you have good reasons, explain to user

### Response Format (Single Provider)

```
I consulted with [AI Name] on this approach. Here's the synthesis:

**The Problem:** [what we're solving]

**My Approach:** [your proposed solution]

**[AI Name]'s Feedback:**
- [Key point 1 - whether you agree/disagree and why]
- [Key point 2]

**Final Recommendation:** [your decision, incorporating valid feedback]
```

### Response Format (Multiple Providers)

```
I consulted with [AI 1] and [AI 2] on this approach. Here's the synthesis:

**The Problem:** [what we're solving]

**My Approach:** [your proposed solution]

**Points of Agreement:**
- [Both/All AIs agreed on X]
- [This gives high confidence in Y]

**Differing Perspectives:**
- [AI 1] suggested Z, while [AI 2] preferred W
- My take: [your evaluation of these perspectives]

**Final Recommendation:** [your decision, synthesizing the best insights]
```

## Environment Setup

API keys can be configured in two ways (environment variables take priority):

### Option 1: Environment Variables

```bash
# Grok (xAI)
export XAI_API_KEY="xai-your-api-key-here"

# ChatGPT (OpenAI)
export OPENAI_API_KEY="sk-your-api-key-here"

# Gemini (Google)
export GEMINI_API_KEY="your-api-key-here"
```

### Option 2: config.json (Persistent)

Add `api_key` to each provider in `config.json`:

```json
{
  "providers": {
    "chatgpt": {
      "model": "gpt-5.1",
      "api_key": "sk-your-api-key-here"
    }
  }
}
```

Only configure the providers you plan to use. The skill will automatically detect which are available.

## Model Configuration

Model selection priority (highest to lowest):
1. CLI `--model` flag
2. Environment variable (`GROK_MODEL`, `OPENAI_MODEL`, `GEMINI_MODEL`)
3. `config.json` file
4. Built-in defaults

### Using Environment Variables (Easiest)

Set model override via environment variable:

```bash
# Override Grok model
export GROK_MODEL="grok-3-mini"

# Override ChatGPT model
export OPENAI_MODEL="gpt-4-turbo"

# Override Gemini model
export GEMINI_MODEL="gemini-2.0-flash"
```

Run `--list-providers` to see current model sources:

```bash
python3 {{SKILL_DIR}}/scripts/pair_review.py --list-providers
```

### Using config.json

Models and API keys can be configured in `config.json` in the skill directory:

```json
{
  "providers": {
    "grok": {
      "model": "grok-4-1-fast-reasoning",
      "api_key": "",
      "description": "Grok (xAI) - Set api_key here or XAI_API_KEY env var"
    },
    "chatgpt": {
      "model": "gpt-5.1",
      "api_key": "",
      "description": "ChatGPT (OpenAI) - Set api_key here or OPENAI_API_KEY env var"
    },
    "gemini": {
      "model": "gemini-3-pro-preview",
      "api_key": "",
      "description": "Gemini (Google) - Set api_key here or GEMINI_API_KEY env var"
    }
  },
  "defaults": {
    "provider": "grok",
    "temperature": 0.7,
    "max_tokens": 4096
  }
}
```

**Priority order:** Environment variables > config.json api_key

You can still override any model at runtime with `--model`.

## Debugging

Use `--debug` to see the full prompt sent to providers:
```bash
python3 {{SKILL_DIR}}/scripts/pair_review.py --debug --provider grok ...
```

Use `--list-providers` to check which providers are configured:
```bash
python3 {{SKILL_DIR}}/scripts/pair_review.py --list-providers
```
