---
name: openai-api
description: Use the OpenAI API effectively in a NextJS project. Trigger when the user wants to integrate OpenAI, GPT, or ChatGPT functionality into a NextJS application, including chat completions, streaming responses, or any OpenAI SDK usage. Also trigger when existing project code imports from the "openai" package and the user wants to add or modify that integration.
---

# OpenAI API in NextJS

## Required: Study Documentation First

**BEFORE writing any code that interfaces with the OpenAI API, ALWAYS fetch and study both of the following documents:**

1. **TypeScript SDK**: https://github.com/openai/openai-node — Fetch this to understand the SDK's installation, client initialization, and available methods.
2. **Responses API Streaming**: https://developers.openai.com/api/reference/resources/chat/subresources/completions/streaming-events — Fetch this to understand streaming event types and how to handle streamed responses.

Use `WebFetch` on both URLs and study the returned content before writing or modifying any OpenAI integration code. Do not rely on prior knowledge of the API — the documentation is the source of truth.
