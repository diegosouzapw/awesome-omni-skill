---
name: attention-please
description: Play an alert sound and speak "Project NAME needs your attention." Always use at the end of every response or whenever Codex needs user input or confirmation; derive the project name from the git remote (origin) with fallback to the repo folder or an override.
---

# Attention Please

## Overview

Play a short audible alert and a spoken prompt indicating which project needs attention. Use this as the final step after completing work on every response, and also right before asking the user for input. This skill is repo-agnostic and can be used in any repository.

## Rules (non-negotiable)

- Always run this script at the end of every response, even if you are not asking for input.
- If you missed it in the previous turn, run it immediately at the start of the next turn before doing anything else, then continue.
- Treat a missed run as a bug; prioritize correcting it as soon as you notice.

## Workflow

1. Run from inside the target repo so the script can read the git remote.
2. Execute `~/.codex/skills/public/attention-please/scripts/attention-please.sh`.
3. Run this immediately before sending your final response to the user.
4. Continue with your response to the user.

### Project name resolution

- Primary: `git remote get-url origin` and extract the repo name.
- Remote override: set `ATTENTION_PLEASE_REMOTE`.
- Fallback: repo folder name.
- Override: set `ATTENTION_PLEASE_PROJECT`.

### Sound and speech

- Sound: macOS `afplay` with `/System/Library/Sounds/Ping.aiff` by default.
- Override sound: set `ATTENTION_PLEASE_SOUND`.
- Disable sound: set `ATTENTION_PLEASE_NO_SOUND=1`.
- Speech: macOS `say`; if unavailable, the message prints to stdout.
- Disable speech: set `ATTENTION_PLEASE_NO_SAY=1`.
- Voice: set `ATTENTION_PLEASE_SAY_VOICE`.
- Rate: set `ATTENTION_PLEASE_SAY_RATE`.

### Message override

- Override the full phrase with `ATTENTION_PLEASE_MESSAGE`.

## Example

```bash
ATTENTION_PLEASE_PROJECT="project-name" ATTENTION_PLEASE_SAY_VOICE="Samantha" ~/.codex/skills/public/attention-please/scripts/attention-please.sh
```
