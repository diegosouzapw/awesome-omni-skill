---
name: External KI Integration
description: Skill for accessing external AI services (ChatGPT, Claude, Hugging Face, etc.) via browser automation (Chrome Relay) and APIs to assist with tasks.
---

# External KI Integration

Use external AI services via browser automation (ChatGPT, Claude, web‑based LLMs) and APIs (Hugging Face Inference, OpenAI‑compatible endpoints) to augment your capabilities.

## When to use this skill

- You need to consult an external AI model (ChatGPT, Claude, Gemini, etc.) for reasoning, analysis, or generation tasks.
- The user has granted access to their chat interfaces (e.g., via Chrome Relay attached tab).
- You want to use Hugging Face Inference API (if token provided) for model inference.
- You need to interact with a free AI demo or Space via browser automation.
- The task benefits from a second opinion or specialized model (coding, creative writing, summarization).

## Requirements

1. **Browser automation** – the `browser` tool with `profile="chrome"` (user must have attached a tab to OpenClaw Browser Relay).
2. **External AI accounts** – user must be logged into the target service (ChatGPT, Claude, etc.) in the attached Chrome tab.
3. **Hugging Face token** (optional) – for Inference API access, stored in `~/.openclaw/openclaw.json` or provided as environment variable.
4. **Other API keys** (optional) – e.g., OpenAI, Anthropic, if user provides them.

## Setup

### Chrome Relay Attachment
The user must click the OpenClaw Browser Relay toolbar icon on the desired tab (badge ON). Verify attachment:

```bash
openclaw browser status
```

Or via `browser` tool: `browser(action=status, profile="chrome")`.

### Hugging Face Token
If token already stored in config, it will be used automatically. Otherwise, ask user to provide it.

### Environment Variables (optional)
For API‑based access, you may set:

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export HF_TOKEN="hf_..."
```

## Browser Automation for Web UIs

### General Pattern

1. **Navigate** to the service URL (e.g., `https://chat.openai.com`, `https://claude.ai`, `https://gemini.google.com`).
2. **Wait for page load**, snapshot with `refs="aria"` to locate UI elements.
3. **Find input area** (role="textbox", role="textbox" with name "Message", etc.).
4. **Type** your query using `act` with `ref` or `selector`.
5. **Click send/submit** button (role="button", name="Send").
6. **Wait for response** (poll for new text elements, detect loading indicator disappearance).
7. **Extract response** from the output container (role="article", class "markdown", etc.).
8. **Return** the extracted text.

### Example: ChatGPT via Chrome Relay

```javascript
// 1. Navigate
browser(action="open", profile="chrome", targetUrl="https://chat.openai.com");

// 2. Snapshot after load
const snap = browser(action="snapshot", profile="chrome", refs="aria", interactive=true);

// 3. Find textbox (adapt ref based on snapshot)
browser(action="act", profile="chrome", request={ kind: "type", ref: "textbox:Message", text: "Your query here" });

// 4. Click send button
browser(action="act", profile="chrome", request={ kind: "click", ref: "button:Send" });

// 5. Wait for response (poll until new text appears)
// 6. Extract response
```

### Adaptation Notes

- **UI changes frequently**: Use `refs="aria"` for stable references (aria‑role, aria‑name). Fall back to `selector` with CSS classes if needed.
- **Rate limiting**: Be gentle; wait 2–5 seconds between interactions.
- **Session persistence**: The attached tab retains login state; you can continue conversation in same chat.

## API Integration

### Hugging Face Inference API
See the dedicated [Hugging Face skill](../huggingface/SKILL.md) for detailed usage.

### OpenAI‑compatible endpoints
If user provides an API key, you can call models via `curl` or `exec`:

```bash
curl -s -X POST https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Anthropic Claude
```bash
curl -s -X POST https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-opus-20240229",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Cost & Safety

### Browser Automation (free)
- No direct cost, but uses user's existing subscription (if any).
- Respect rate limits; do not spam requests.
- Do not expose user credentials; rely on attached logged‑in session.

### API Usage (paid)
- **Hugging Face Inference**: Track estimated costs via `system/logs/hf-costs.log`. Stay within monthly budget (e.g., 33€). Notify user at 50% threshold.
- **OpenAI/Anthropic**: If user provides API key, assume they accept associated costs. Still estimate token usage and log if possible.
- **General rule**: Prefer browser automation for free services; use paid APIs only when explicitly authorized and task justifies cost.

### Safety
- **No sensitive data**: Avoid sending personal, financial, or confidential information to external services unless user explicitly approves.
- **Compliance**: Follow external service terms of service.
- **Fallback**: If external service fails, continue with internal reasoning; do not block task completion.

## Integration with OpenClaw Skills

This skill complements:
- **Hugging Face skill** – for dedicated Hugging Face API/Spaces.
- **Browser automation patterns** – for generic web interaction.
- **Multi‑model orchestration** – for delegating sub‑tasks to external models.

Add this skill to `skills/index.md`:

```
| External KI Integration | skills/external‑ki‑integration/SKILL.md |
```

## Example Workflow

1. **Task**: Need to generate a complex code snippet.
2. **Check**: User has ChatGPT tab attached via Chrome Relay.
3. **Open** ChatGPT, snapshot, locate input.
4. **Type**: "Write a Python function that validates email addresses with regex and DNS MX check."
5. **Click** Send.
6. **Wait** for response, extract code.
7. **Return** code to user, optionally refine via follow‑up.
8. **Log** the interaction in memory (pattern learned).

## Troubleshooting

- **Tab not attached**: Ask user to click Browser Relay icon on the target tab.
- **UI changes**: Update refs/selectors based on snapshot.
- **Rate limits**: Wait longer between requests.
- **API errors**: Check token permissions, budget, network.

## References

- [OpenClaw Browser Relay docs](https://docs.openclaw.ai/browser-relay)
- [Hugging Face skill](../huggingface/SKILL.md)
- [Browser automation playbook](../../memory/patterns/playbooks.md)
- [Using free AI models online pattern](../../memory/2026-02-18.md)