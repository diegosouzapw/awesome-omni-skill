---
name: browser-automation
description: "Browser automation for ChatGPT Plus and Gemini Advanced web interfaces. Uses Playwright MCP to interact with your paid subscriptions without API costs. Supports both models for writing comparison."
---

# Browser Automation for AI Web Interfaces

Use your **ChatGPT Plus** and **Gemini Advanced** subscriptions through browser automation. No API costs - just your monthly subscription.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    BROWSER AUTOMATION FLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Your Prompt ──► Playwright MCP ──► Browser Instance            │
│                                           │                      │
│                              ┌────────────┴────────────┐        │
│                              ▼                         ▼        │
│                      ┌─────────────┐           ┌─────────────┐  │
│                      │  ChatGPT    │           │   Gemini    │  │
│                      │  chat.openai│           │   gemini.   │  │
│                      │  .com       │           │   google.com│  │
│                      └──────┬──────┘           └──────┬──────┘  │
│                             │                         │         │
│                             ▼                         ▼         │
│                      Response captured & returned to you        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

### 1. Playwright MCP Must Be Active

You have Playwright MCP configured. Verify it's working:

```
Use browser_snapshot to check if browser is available
```

### 2. Login Sessions

The browser automation uses saved sessions. You need to log in once:

**First-time setup:**
1. Navigate to ChatGPT/Gemini
2. Log in with your credentials
3. Session is saved for future use

---

## ChatGPT Automation

### Step-by-Step Workflow

**Step 1: Navigate to ChatGPT**
```
browser_navigate → https://chat.openai.com
```

**Step 2: Check if logged in**
```
browser_snapshot → Look for chat input or login button
```

**Step 3: If not logged in, authenticate**
```
browser_click → "Log in" button
browser_type → Enter email
browser_click → Continue
browser_type → Enter password
browser_click → Log in
```

**Step 4: Start new chat**
```
browser_click → "New chat" button (or navigate to chat.openai.com)
```

**Step 5: Type your prompt**
```
browser_type → Your prompt text in the message input
```

**Step 6: Submit and wait**
```
browser_click → Send button
browser_wait_for → Wait for response to complete
```

**Step 7: Capture response**
```
browser_snapshot → Get the response text
```

### Example: ChatGPT Writing Task

```
I will now use browser automation to get ChatGPT's response:

1. browser_navigate to https://chat.openai.com
2. browser_snapshot to see current state
3. browser_type to enter prompt in textarea
4. browser_click to send
5. browser_wait_for response
6. browser_snapshot to capture output
```

---

## Gemini Automation

### Step-by-Step Workflow

**Step 1: Navigate to Gemini**
```
browser_navigate → https://gemini.google.com
```

**Step 2: Check if logged in**
```
browser_snapshot → Look for chat input
```

**Step 3: Type your prompt**
```
browser_type → Your prompt in the input area
```

**Step 4: Submit**
```
browser_press_key → Enter (or click send button)
```

**Step 5: Wait and capture**
```
browser_wait_for → Response generation
browser_snapshot → Get response
```

---

## Practical Commands

### For Claude Code Session

When you want me to use browser automation, say:

```
"Use browser automation to ask ChatGPT: [your prompt]"
"Get Gemini's take on: [your prompt]"
"Compare browser outputs for: [your prompt]"
```

I will then:
1. Use Playwright MCP tools
2. Navigate to the appropriate site
3. Enter your prompt
4. Capture and return the response

---

## Handling Authentication

### Session Persistence

Browser automation works best with persistent sessions:

```python
# The Playwright MCP maintains browser state
# Once logged in, sessions typically persist
```

### If Session Expires

If you see a login screen:

1. **ChatGPT**: Look for "Log in" button, click it
2. **Gemini**: Look for "Sign in" button, click it
3. Complete authentication flow
4. Resume automation

### Two-Factor Authentication

If 2FA is required:
1. Automation will pause at 2FA screen
2. You manually complete 2FA
3. Automation continues

---

## Limitations

### Browser Automation Caveats

| Limitation | Workaround |
|------------|------------|
| Slower than API | Use for comparison, not bulk |
| Can break if UI changes | Report issues, I'll adapt |
| Requires active session | Keep browser open |
| Rate limits still apply | Don't spam requests |
| CAPTCHAs possible | May need manual intervention |

### When NOT to Use Browser Automation

- Bulk content generation (use GLM-4.7 API instead)
- Time-critical tasks (APIs are faster)
- Fully automated pipelines (APIs more reliable)

### When TO Use Browser Automation

- Comparing writing styles
- Using features only in Plus/Advanced
- Testing latest model versions
- When APIs are down

---

## Comparison Workflow

### Get Same Prompt from Multiple Sources

```
Step 1: Write with Claude (default, in this conversation)
Step 2: browser_navigate to ChatGPT, get response
Step 3: browser_navigate to Gemini, get response
Step 4: Compare all three side-by-side
```

### Example Request

```
"Compare how you, ChatGPT, and Gemini would write a tweet about
the cardiovascular benefits of SGLT2 inhibitors"
```

I will:
1. Write my version (Claude)
2. Use browser automation to get ChatGPT's version
3. Use browser automation to get Gemini's version
4. Present all three for comparison

---

## Troubleshooting

### Browser Not Responding

```
browser_close → Close current browser
Then start fresh with browser_navigate
```

### Wrong Page Loaded

```
browser_snapshot → Check current state
browser_navigate → Go to correct URL
```

### Element Not Found

```
browser_snapshot → Get fresh page state
Look for correct element reference
Retry with updated reference
```

### Session Logged Out

```
browser_navigate → Go to login page
Complete login flow
Resume automation
```

---

## Integration with Multi-Model Writer

This skill works with `multi-model-writer`:

```
API Models:
- /write-glm → Z.AI API
- /write-gpt → OpenAI API
- /write-gemini → Google AI Studio API

Browser Models:
- /browser-chatgpt → ChatGPT Plus web
- /browser-gemini → Gemini Advanced web
```

Use APIs for speed and reliability.
Use browser for subscription-only features or comparison.

---

## Example Session

```
User: "Use browser to compare how ChatGPT writes about statins"

Claude: I'll get ChatGPT's perspective using browser automation.

[Uses browser_navigate to https://chat.openai.com]
[Uses browser_snapshot to verify page state]
[Uses browser_type to enter: "Write a patient-friendly explanation of how statins work"]
[Uses browser_click to send]
[Uses browser_wait_for to wait for response]
[Uses browser_snapshot to capture response]

Here's what ChatGPT wrote:
[Response text]

Compared to my approach:
[Claude's version]

Key differences:
- ChatGPT emphasized X while I focused on Y
- Tone: ChatGPT more conversational, mine more clinical
- Length: Similar word count
```

---

*Browser automation gives you access to your paid subscriptions programmatically, complementing the API-based models in your arsenal.*
