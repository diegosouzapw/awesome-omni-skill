---
name: chat-sdk
description: Expert on Vercel's Chat SDK for building production-ready AI chatbots. Use when user wants to build chatbot apps, add generative UI, create artifacts, or deploy AI chat interfaces. Triggers on mentions of chat-sdk, ai-chatbot, chatgpt clone, vercel chat, generative ui.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch
---

# Chat SDK Skill

Expert at building AI chatbot applications with Vercel's Chat SDK.

## Overview

**Chat SDK** is a free, open-source template for building production-ready chatbots:
- Built on Next.js App Router with AI SDK integration
- Generative UI for dynamic, interactive interfaces
- Custom artifacts for specialized workspaces
- In-browser code execution via WASM/pyodide
- Auth, persistence, multimodal support, and shareable chats
- One-click Vercel deployment

## Quick Start

```bash
# Clone and setup
npx create-next-app --example https://github.com/vercel/ai-chatbot my-chatbot
cd my-chatbot

# Install and configure
pnpm install
cp .env.example .env.local
# Add your API keys to .env.local

# Database setup
pnpm db:migrate

# Run development server
pnpm dev
```

## Core Concepts

### AI SDK Integration
Uses unified API for text generation, structured objects, and tool calls:
```typescript
import { generateText } from 'ai';
import { xai } from '@ai-sdk/xai';

const { text } = await generateText({
  model: xai('grok-2-vision-1212'),
  prompt: 'Hello!',
});
```

### Model Providers
Default: AI SDK Gateway with xAI (grok-2-vision-1212). Switch providers easily:
- OpenAI, Anthropic, Google, xAI, and more via unified gateway
- Configure in `lib/ai/models.ts`

### Generative UI
Dynamic interfaces that adapt to conversation context beyond text.

## Documentation

For detailed information, see the reference documentation:

- **[Overview](docs/overview.md)** - Feature summary and doc structure
- **[Setup](docs/setup.md)** - Installation and configuration
- **[Architecture](docs/architecture.md)** - Project structure and components
- **[Models and Providers](docs/models-and-providers.md)** - AI providers and configuration
- **[Artifacts](docs/artifacts.md)** - Custom workspaces and tools
- **[Theming](docs/theming.md)** - Customizing appearance
- **[Deployment](docs/deployment.md)** - Vercel and self-hosting
- **[Upstream README](docs/readme-upstream.md)** - Full original documentation

## Common Workflows

### Add New AI Provider
```typescript
// lib/ai/models.ts
import { anthropic } from '@ai-sdk/anthropic';

export const models = {
  'claude-3': anthropic('claude-3-5-sonnet-20241022'),
};
```

### Create Custom Artifact
See `docs/artifacts.md` for creating interactive workspaces.

### Deploy to Vercel
```bash
vercel deploy
```

## Upstream Sources

- **Repository**: https://github.com/vercel/ai-chatbot
- **Documentation**: https://chat-sdk.dev/

## Sync & Update

When user runs `sync`: fetch latest from upstream sources, update docs/ files.
When user runs `diff`: compare current vs upstream, report changes.
