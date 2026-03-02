---
name: video-sourcing
version: 1.0.0
description: Run the Video Sourcing Agent with deterministic, concise chat UX for /video_sourcing using a pinned self-bootstrap runtime.
user-invocable: true
metadata:
  openclaw:
    os:
      - macos
      - linux
    homepage: https://github.com/Memories-ai-labs/video-sourcing-agent
    primaryEnv: GOOGLE_API_KEY
    requires:
      bins:
        - git
        - uv
      env:
        - GOOGLE_API_KEY
        - YOUTUBE_API_KEY
---

# Video Sourcing Skill

Use this skill when the user asks to find, compare, or analyze social videos (YouTube, TikTok, Instagram, Twitter/X), or explicitly invokes `/video_sourcing`.

This workflow expects host runtime execution (sandbox mode off).
The runner auto-bootstraps a pinned runtime from `Memories-ai-labs/video-sourcing-agent@v0.2.3` when `VIDEO_SOURCING_AGENT_ROOT` is not set.

## Triggering

Run this workflow when either condition is true:

1. Message starts with `/video_sourcing`.
2. The user asks for video sourcing/trend/creator/brand analysis and wants concrete video links.

If `/video_sourcing` is used with no query body, ask for the missing query.

## Execution contract

1. Resolve query text:
   - `/video_sourcing ...` => strip `/video_sourcing` and use remaining text.
   - Free-form => use user message as query.
2. Default to compact mode:
   - `--event-detail compact`
3. If user asks for debugging/raw payloads:
   - Switch to `--event-detail verbose`

### `/video_sourcing` deterministic path

1. Build command with required args:
   - `<skill_dir>/scripts/run_video_query.sh --query "<query>" --event-detail <compact|verbose> --ux-mode three_message --progress-gate-seconds 10`
2. Start with `exec` using `background: true`.
3. Poll with `process` using `action: "poll"` every 2-4 seconds until process exits.
4. Parse NDJSON output and render only these events:
   - `started` => send: `Starting video sourcing...`
   - `ux_progress` => send concise middle progress updates from `summary` (throttled by runner)
     Send each `ux_progress` as a separate assistant message in Telegram.
   - terminal event (`complete`, `clarification_needed`, `error`) => send final message as-is
5. Do not forward raw `progress`, `tool_call`, or `tool_result` events for `/video_sourcing`.
6. Do not rewrite final answer tone/style; preserve the user's existing OpenClaw personality behavior.

Behavior target for `/video_sourcing`:

1. Fast run (<10s): 2 messages (`started`, terminal).
2. Longer run (>=10s): recurring throttled `ux_progress` updates, then terminal.

### Free-form path (non-strict)

1. Keep existing flexible behavior.
2. Build command without forcing `three_message` mode:
   - `<skill_dir>/scripts/run_video_query.sh --query "<query>" --event-detail <compact|verbose>`
3. Stream useful progress updates and final response naturally.

## Final response format

When terminal event is `complete`:

1. One short paragraph conclusion.
2. Top 3 video references only by default:
   - `title`
   - `url`
   - one-line relevance note
3. `Tools used: ...` with a compact status summary.

If fewer than 3 videos exist, show all available references.

When terminal event is `clarification_needed`:

1. Ask the clarification question directly.
2. Treat this as the final response for the current run.

When terminal event is `error`:

1. Send concise failure reason.
2. Include one actionable next step.

## Safety and fallback

1. If script fails due to missing env/tooling, explain exact missing piece (for example `VIDEO_SOURCING_AGENT_ROOT`, `uv`, or API key env var).
2. If `VIDEO_SOURCING_AGENT_ROOT` is unset, the runner uses managed path:
   - `~/.openclaw/data/video-sourcing-agent/v0.2.3`
3. `VIDEO_SOURCING_AGENT_ROOT` remains an advanced override for local development.
4. Keep response concise and action-oriented.
5. Never fabricate video URLs or metrics.
