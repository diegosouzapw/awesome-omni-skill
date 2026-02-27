---
name: telegram-bot-creator
description: "Complete Telegram bot creation for AI agents and pipelines. Build end-to-end bots that bridge backend agents/logic to Telegram chat. Supports multiple frameworks (aiogram, python-telegram-bot), LLM integration (Claude/OpenRouter), multi-step workflows, and various deployment options (polling, webhooks). Use when: (1) Creating a new Telegram bot from scratch, (2) Adding Telegram interface to existing backend agents/pipelines, (3) Building conversational AI bots, (4) Designing customer support/routing bots, (5) Implementing multi-step automated workflows, or (6) The agent needs a Telegram bot created."
---

# Telegram Bot Creator

## When to use this skill
- Use when the user request matches this skill's domain and capabilities.
- Use when this workflow or toolchain is explicitly requested.

## When not to use this skill
- Do not use when another skill is a better direct match for the task.
- Do not use when the request is outside this skill's scope.

## Overview

Build production-ready Telegram bots that bridge backend agents and pipelines to Telegram chat. This skill handles complete bot creation: from scaffolding, to agent integration, to deployment.

**Key architectural pattern:** Backend agents do the business logic; Telegram bot provides the chat interface and state management.

## Quick Start (5 minutes)

### 1. Choose Your Framework

```bash
# Option A: Modern async (recommended for complex agents)
pip install aiogram python-dotenv

# Option B: Simpler, more established
pip install python-telegram-bot python-dotenv
```

**Framework comparison:** See [frameworks.md](references/frameworks.md) for detailed comparison.

### 2. Get Telegram Bot Token

1. Message @BotFather on Telegram
2. Create new bot: `/newbot`
3. Copy token: `123456:ABC-DEF...`

### 3. Scaffold Bot Project

```bash
python3 scripts/bot_initializer.py --name my_bot --framework aiogram
```

This creates:
- `main.py` - Bot entry point with example handlers
- `.env.example` - Config template
- `requirements.txt` - Dependencies

### 4. Implement Your Agent

Copy `scripts/agent_adapter.py` to your project, then subclass it:

```python
from agent_adapter import TelegramAgentAdapter

class MyAgent(TelegramAgentAdapter):
    async def process_agent_request(self, message: str, user_id: str) -> str:
        # Your agent logic here
        # Can call LLMs, external services, databases, etc.
        return f"Agent response: {message}"
```

### 5. Run Locally

```bash
export TELEGRAM_BOT_TOKEN="your_token_here"
python3 main.py
```

Find your bot on Telegram and send a message!

---

## Use Cases & Patterns

### Pattern 1: Simple Routing Agent

Route messages to different handlers based on content.

**When to use:** Customer support, FAQ bots, request classification

**See:** Agent integration guide → Pattern 1

```python
class SupportRouter(TelegramAgentAdapter):
    async def process_agent_request(self, message: str, user_id: str) -> str:
        if "refund" in message.lower():
            return "📧 Routing to refunds team..."
        elif "technical" in message.lower():
            return "🔧 Routing to support..."
        return "ℹ️ How can I help?"
```

### Pattern 2: LLM-Based Conversational Bot

AI-powered chat that understands natural language.

**When to use:** Virtual assistants, customer support, Q&A

**See:** Agent integration guide → Pattern 2

**Easy setup with OpenRouter (free Claude access):**

```python
import httpx

class ChatBot(TelegramAgentAdapter):
    async def process_agent_request(self, message: str, user_id: str) -> str:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "https://openrouter.io/api/v1/chat/completions",
                headers={"Authorization": f"Bearer {OPENROUTER_KEY}"},
                json={
                    "model": "claude-3-5-sonnet:free",
                    "messages": [{"role": "user", "content": message}],
                    "max_tokens": 1024
                }
            )
        data = response.json()
        return data['choices'][0]['message']['content']
```

### Pattern 3: Multi-Step Workflow

Guide users through sequential steps (onboarding, forms, processes).

**When to use:** Onboarding, multi-step forms, guided workflows

**See:** Agent integration guide → Pattern 3, also `assets/example_bot_comprehensive.py`

```python
class OnboardingBot(TelegramAgentAdapter):
    async def process_agent_request(self, message: str, user_id: str) -> str:
        state = self.get_user_state(user_id)
        step = state.get("step", 0)

        if step == 0:
            self.update_user_state(user_id, "name", message)
            self.update_user_state(user_id, "step", 1)
            return "Thanks! Now what's your email?"

        elif step == 1:
            self.update_user_state(user_id, "email", message)
            self.update_user_state(user_id, "step", 2)
            return "🎉 Onboarding complete!"
```

### Pattern 4: External Agent Integration

Your agent logic lives on a separate service; bot just forwards requests.

**When to use:** Large agents, existing backends, microservices

**See:** Agent integration guide → Pattern 4

```python
class RemoteAgent(TelegramAgentAdapter):
    async def process_agent_request(self, message: str, user_id: str) -> str:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "https://your-api.com/agent/process",
                json={"user_id": user_id, "message": message},
                timeout=30.0
            )
        return response.json()["response"]
```

### Pattern 5: Agentskills.io Protocol

Integrate agents following the agentskills.io specification.

**See:** Agent integration guide → Pattern 5

---

## Deployment

### Development: Polling (Simplest)

```bash
# Set token
export TELEGRAM_BOT_TOKEN="your_token"

# Run locally
python3 main.py
```

**Advantages:** No setup, works on localhost, perfect for testing
**Limitations:** Higher latency, not ideal for high volume

### Production: Polling (Scaled)

Run on cloud platform (keep bot running):
- DigitalOcean App Platform
- Google Cloud Run (persistent container)
- Heroku
- AWS ECS

### Production: Webhooks (High Scale)

Telegram sends updates directly to your server. Lower latency, more scalable.

**Requirements:**
- HTTPS endpoint with SSL certificate
- Public domain

**Setup:** See [deployment.md](references/deployment.md) for full webhook guide and Cloud Run/Lambda examples.

**Quick webhook with aiogram:**

```python
from aiohttp import web

WEBHOOK_URL = "https://yourdomain.com/webhook/telegram"

async def on_startup():
    await bot.set_webhook(url=WEBHOOK_URL, secret_token="random-secret")

# Run webhook server...
```

---

## State Management

### Simple: In-Memory (Development)

Built into `agent_adapter.py`. Uses `get_user_state()` and `update_user_state()` methods.

```python
async def handle_message(self, user_id: str, message: str) -> str:
    state = self.get_user_state(user_id)
    # Process with state
    self.update_user_state(user_id, "key", "value")
    return response
```

**Limitation:** Data lost on restart. Only for development.

### Production: Redis

Fast, distributed, recommended for most bots.

```bash
# Quick Redis (Docker)
docker run -d -p 6379:6379 redis:7-alpine

# Or managed cloud: Upstash, Redis Labs, AWS ElastiCache
```

**See:** [state_management.md](references/state_management.md) for full Redis integration code

### Production: PostgreSQL

For structured data, complex queries, compliance.

**See:** [state_management.md](references/state_management.md) for schema and integration

### Docker Compose (All-in-One)

```bash
# Includes bot, Redis, PostgreSQL
docker-compose up
```

**See:** `assets/docker-compose.yml` for configuration

---

## Framework-Specific Features

### Aiogram: FSM (Finite State Machine)

For multi-step conversations with explicit state transitions:

```python
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.redis import RedisStorage

class MyStates(StatesGroup):
    step1 = State()
    step2 = State()

# Use Redis for distributed state
storage = RedisStorage.from_url("redis://localhost")
dp = Dispatcher(storage=storage)

@dp.message(MyStates.step1)
async def handle_step1(message: types.Message, state: FSMContext):
    await state.set_state(MyStates.step2)
    # ...
```

### Aiogram: Middleware

Route messages intelligently before handlers:

```python
@dp.middleware()
async def agent_selector(handler, event, data):
    user_id = event.from_user.id
    data["agent"] = get_agent_for_user(user_id)
    return await handler(event, data)
```

### Python-telegram-bot: Handlers

Simple handler registration:

```python
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, message_handler))
```

---

## Common Tasks

### Task: Add Conversation History to LLM

Keep messages in memory or database for context-aware responses:

```python
state = self.get_user_state(user_id)
messages = state.get("conversation", [])
messages.append({"role": "user", "content": message})

# Call LLM with full history
response = await llm_call(messages)

messages.append({"role": "assistant", "content": response})
self.update_user_state(user_id, "conversation", messages)
```

### Task: Handle Long-Running Operations

Async task without blocking user:

```python
@dp.message()
async def handle_long_operation(message: types.Message):
    await message.answer("Processing... 🔄")

    # Run async operation
    result = await long_operation()

    # Send result
    await message.answer(f"Done! {result}")
```

### Task: Inline Buttons & Callbacks

```python
from aiogram.types import InlineKeyboardButton, InlineKeyboardMarkup

@dp.message()
async def show_options(message: types.Message):
    buttons = InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="Option 1", callback_data="opt1")],
        [InlineKeyboardButton(text="Option 2", callback_data="opt2")]
    ])
    await message.answer("Choose:", reply_markup=buttons)

@dp.callback_query()
async def handle_callback(query: types.CallbackQuery):
    if query.data == "opt1":
        await query.answer("You chose option 1!")
```

### Task: Error Handling & Logging

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dp.message()
async def safe_handler(message: types.Message):
    try:
        response = await risky_operation()
        await message.answer(response)
    except Exception as e:
        logger.error(f"Error for user {message.from_user.id}: {e}")
        await message.answer("⚠️ Error processing request. Try again later.")
```

### Task: Rate Limiting

Prevent spam:

```python
from collections import defaultdict
from datetime import datetime, timedelta

user_messages = defaultdict(list)

@dp.message()
async def rate_limited_handler(message: types.Message):
    user_id = message.from_user.id
    now = datetime.now()

    # Remove old messages (older than 1 minute)
    user_messages[user_id] = [
        ts for ts in user_messages[user_id]
        if now - ts < timedelta(minutes=1)
    ]

    # Check limit (5 messages per minute)
    if len(user_messages[user_id]) >= 5:
        await message.answer("⏱️ Too many requests. Wait a minute.")
        return

    user_messages[user_id].append(now)
    # Process request...
```

---

## Reference Material

**Framework Setup & Comparison:**
→ [frameworks.md](references/frameworks.md)

**Agent Integration Patterns:**
→ [agent_integration.md](references/agent_integration.md)

**Polling vs Webhooks & Deployment:**
→ [deployment.md](references/deployment.md)

**State Management (Redis, PostgreSQL, etc):**
→ [state_management.md](references/state_management.md)

---

## Scripts & Templates

**Agent Base Class:**
`scripts/agent_adapter.py` - Subclass this to create your agent. Handles state, typing indicators, error wrapping.

**Bot Initializer:**
`scripts/bot_initializer.py` - Scaffold new bot project with framework of choice.

**Example Bot:**
`assets/example_bot_comprehensive.py` - Full example with support routing, LLM chat, and multi-step onboarding.

**Docker:**
`assets/Dockerfile` - Container image for bot
`assets/docker-compose.yml` - Full stack (bot + Redis + PostgreSQL)

**Config:**
`assets/.env.example` - Environment variables template

---

## Quick Reference: Create a New Bot

```bash
# 1. Initialize project
python3 scripts/bot_initializer.py --name my_bot --framework aiogram

# 2. Copy adapter
cp scripts/agent_adapter.py my_bot/

# 3. Update main.py: Replace YourAgent class with your logic

# 4. Set token
export TELEGRAM_BOT_TOKEN="token_from_botfather"

# 5. Run
cd my_bot
python3 main.py

# 6. Find bot on Telegram and send message
```

---

## Troubleshooting

**Bot doesn't respond?**
- Check TELEGRAM_BOT_TOKEN is set correctly
- Verify bot is running (no errors in logs)
- Search for bot in Telegram by username from BotFather

**Agent returning errors?**
- Check agent logic (add print statements, see logs)
- Use agent_adapter error handling: `_error_response()`
- Test agent separately before integrating

**Slow responses?**
- Polling adds latency (polling_timeout). Use webhooks for faster response
- Check agent logic takes too long. Consider async operations
- Redis is much faster than polling each time

**State not persisting?**
- In-memory state (default) is lost on restart. Use Redis/PostgreSQL
- Docker Compose includes Redis/PostgreSQL setup

**Webhook not receiving messages?**
- Check HTTPS certificate is valid (not self-signed in production)
- Verify webhook URL is correct and accessible
- See deployment.md for webhook setup details

---

## Next Steps

1. **Choose framework:** aiogram (modern) or python-telegram-bot (simple)
2. **Scaffold project:** Use `bot_initializer.py`
3. **Implement agent:** Subclass `TelegramAgentAdapter`, implement `process_agent_request()`
4. **Add handlers:** Message routing, commands, callbacks
5. **Test locally:** Use polling (simplest)
6. **Scale:** Add state management (Redis), then webhooks if needed
7. **Deploy:** Docker + DigitalOcean/AWS/GCP

For specific patterns, agent types, or deployment scenarios, see reference material linked above.
