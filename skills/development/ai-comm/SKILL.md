---
name: ai-comm
description: Use when collaborating with other AI assistants (Codex, Gemini, Aider, Cursor, OpenCode), delegating tasks, or requesting code review.
allowed-tools: Bash, Read
---

# ai-comm

Cross-AI CLI communication tool for Kitty terminal. Enables AI assistants running in separate Kitty windows to communicate with each other.

## Workflow

1. `ai-comm list-ai-windows` — Find available AI windows
2. `ai-comm send <MESSAGE> -w <ID>` — Send message and get response

## Important Notes

1. **Replying to ai-comm messages.** Just output your response as normal text—the sender automatically captures your terminal output. Do NOT use ai-comm to reply (especially not to the sender's window ID shown in the message header)—this causes deadlock. If you need the sender to act, include the request in your response.

2. **For long responses, request file output.** Bash tool has a 30000-character limit. If you expect a long response, ask the AI to write to a markdown file in the project directory (`/tmp` and other external paths require manual approval on target AI — avoid them).

3. **Only use documented parameters.** Never use parameters not listed in this SKILL or `ai-comm --help`. Hidden/internal parameters exist for debugging only.

## When to Use

- Delegate code review to Codex or Gemini
- Get second opinions on architecture decisions
- Request specialized analysis from another AI
- Verify implementations with alternative models

## Resources

- For command reference, see [reference.md](reference.md)
- For workflow examples, see [examples.md](examples.md)
