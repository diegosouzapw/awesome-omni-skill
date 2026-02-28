---
name: convex-ai
description: Convex AI Integration - OpenAI, actions, streaming, and AI patterns with database integration
globs:
  - "convex/**/*.ts"
  - "**/*ai*.ts"
  - "**/*openai*.ts"
triggers:
  - openai
  - gpt
  - ai
  - llm
  - chat completion
  - generate
  - "use node"
  - action
  - OPENAI_API_KEY
  - ctx.runAction
---

# Convex AI Integration Guide

Complete guide for integrating AI capabilities (OpenAI, Google, etc.) with Convex, including actions, streaming, and best practices.

---

# OpenAI Integration

## Basic Setup

Install the OpenAI package:
```bash
npm install openai
```

## Using OpenAI in Actions

Actions are the right place for AI calls because they can run for up to 10 minutes and make external API calls.

```typescript
// convex/ai.ts
"use node";

import { action } from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const generateText = action({
  args: {
    prompt: v.string(),
  },
  returns: v.string(),
  handler: async (ctx, args) => {
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: args.prompt }],
    });
    return response.choices[0].message.content ?? "";
  },
});
```

## Chat Completion with Context

```typescript
// convex/ai.ts
"use node";

import { action, internalQuery } from "./_generated/server";
import { internal } from "./_generated/api";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const generateResponse = action({
  args: {
    conversationId: v.id("conversations"),
  },
  returns: v.string(),
  handler: async (ctx, args) => {
    // Load context from the database
    const messages = await ctx.runQuery(internal.ai.loadMessages, {
      conversationId: args.conversationId,
    });

    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: messages,
    });

    const content = response.choices[0].message.content;
    if (!content) {
      throw new Error("No content in response");
    }

    // Save the response to the database
    await ctx.runMutation(internal.ai.saveResponse, {
      conversationId: args.conversationId,
      content,
    });

    return content;
  },
});

export const loadMessages = internalQuery({
  args: {
    conversationId: v.id("conversations"),
  },
  returns: v.array(
    v.object({
      role: v.union(v.literal("user"), v.literal("assistant"), v.literal("system")),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query("messages")
      .withIndex("by_conversation", (q) => q.eq("conversationId", args.conversationId))
      .order("asc")
      .take(50);

    return messages.map((msg) => ({
      role: msg.role as "user" | "assistant" | "system",
      content: msg.content,
    }));
  },
});
```

---

# Scheduling AI Responses

Use the scheduler to generate AI responses asynchronously:

```typescript
// convex/messages.ts
import { mutation, internalMutation, internalAction } from "./_generated/server";
import { internal } from "./_generated/api";
import { v } from "convex/values";

export const sendMessage = mutation({
  args: {
    conversationId: v.id("conversations"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Save user message
    await ctx.db.insert("messages", {
      conversationId: args.conversationId,
      role: "user",
      content: args.content,
    });

    // Schedule AI response (runs immediately but async)
    await ctx.scheduler.runAfter(0, internal.ai.generateResponse, {
      conversationId: args.conversationId,
    });

    return null;
  },
});
```

---

# Pattern: AI with Database Updates

When an AI action needs to update the database:

```typescript
// convex/ai.ts
"use node";

import { internalAction, internalMutation } from "./_generated/server";
import { internal } from "./_generated/api";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const processWithAI = internalAction({
  args: {
    documentId: v.id("documents"),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    // 1. Load data from database
    const document = await ctx.runQuery(internal.documents.get, {
      id: args.documentId,
    });

    if (!document) {
      throw new Error("Document not found");
    }

    // 2. Call AI
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [
        { role: "system", content: "Summarize the following document." },
        { role: "user", content: document.content },
      ],
    });

    const summary = response.choices[0].message.content ?? "";

    // 3. Save result to database
    await ctx.runMutation(internal.documents.updateSummary, {
      id: args.documentId,
      summary,
    });

    return null;
  },
});

export const updateSummary = internalMutation({
  args: {
    id: v.id("documents"),
    summary: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.patch(args.id, { summary: args.summary });
    return null;
  },
});
```

---

# Bundled OpenAI (Chef Environment)

If you're using Chef's WebContainer environment, you have access to bundled OpenAI tokens:

```typescript
// convex/ai.ts
import { action } from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";

// Use Chef's bundled OpenAI
const openai = new OpenAI({
  baseURL: process.env.CONVEX_OPENAI_BASE_URL,
  apiKey: process.env.CONVEX_OPENAI_API_KEY,
});

export const generateText = action({
  args: {
    prompt: v.string(),
  },
  returns: v.string(),
  handler: async (ctx, args) => {
    const resp = await openai.chat.completions.create({
      model: "gpt-4.1-nano", // or "gpt-4o-mini"
      messages: [{ role: "user", content: args.prompt }],
    });
    return resp.choices[0].message.content ?? "";
  },
});
```

**Available models:**
- `gpt-4.1-nano` (preferred for speed/cost)
- `gpt-4o-mini`

**Limitations:**
- Only chat completions API is available
- If you need different APIs or models, set up your own OpenAI API key

---

# Error Handling for AI Calls

```typescript
"use node";

import { action } from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const safeGenerate = action({
  args: {
    prompt: v.string(),
  },
  returns: v.union(
    v.object({ success: v.literal(true), content: v.string() }),
    v.object({ success: v.literal(false), error: v.string() })
  ),
  handler: async (ctx, args) => {
    try {
      const response = await openai.chat.completions.create({
        model: "gpt-4o-mini",
        messages: [{ role: "user", content: args.prompt }],
      });

      const content = response.choices[0].message.content;
      if (!content) {
        return { success: false as const, error: "No content in response" };
      }

      return { success: true as const, content };
    } catch (error) {
      const message = error instanceof Error ? error.message : "Unknown error";
      return { success: false as const, error: message };
    }
  },
});
```

---

# React Integration for AI Actions

```tsx
import { useAction } from "convex/react";
import { api } from "../convex/_generated/api";
import { useState } from "react";

function AIChat() {
  const generateResponse = useAction(api.ai.generateText);
  const [prompt, setPrompt] = useState("");
  const [response, setResponse] = useState("");
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!prompt.trim()) return;

    setIsLoading(true);
    setError(null);

    try {
      const result = await generateResponse({ prompt });
      setResponse(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : "An error occurred");
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <textarea
          value={prompt}
          onChange={(e) => setPrompt(e.target.value)}
          placeholder="Enter your prompt..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading || !prompt.trim()}>
          {isLoading ? "Generating..." : "Generate"}
        </button>
      </form>

      {error && <div className="error">{error}</div>}

      {response && (
        <div className="response">
          <h3>Response:</h3>
          <p>{response}</p>
        </div>
      )}
    </div>
  );
}
```

---

# Secrets and Environment Variables

## Best Practices

1. **Never hardcode secrets** - Always use environment variables
2. **Use Convex dashboard** to set environment variables
3. **Different values per environment** - Dev vs Production

## Reading Environment Variables

Environment variables are available via `process.env` in all Convex functions:

```typescript
// Works in queries, mutations, actions, and HTTP actions
const apiKey = process.env.MY_API_KEY;
const baseUrl = process.env.MY_SERVICE_URL;
```

## Common Environment Variables

```
OPENAI_API_KEY=sk-...
RESEND_API_KEY=re_...
RESEND_DOMAIN=yourdomain.com
RESEND_WEBHOOK_SECRET=whsec_...
```

---

# Rate Limiting AI Calls

To prevent abuse and control costs:

```typescript
// convex/ai.ts
import { action, query } from "./_generated/server";
import { v } from "convex/values";
import { getAuthUserId } from "@convex-dev/auth/server";

const RATE_LIMIT_WINDOW = 60 * 1000; // 1 minute
const MAX_REQUESTS = 10;

export const generateWithRateLimit = action({
  args: {
    prompt: v.string(),
  },
  returns: v.string(),
  handler: async (ctx, args) => {
    // Check rate limit
    const canProceed = await ctx.runQuery(internal.ai.checkRateLimit);
    if (!canProceed) {
      throw new Error("Rate limit exceeded. Please wait before trying again.");
    }

    // Record this request
    await ctx.runMutation(internal.ai.recordRequest);

    // Make AI call
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: args.prompt }],
    });

    return response.choices[0].message.content ?? "";
  },
});

export const checkRateLimit = internalQuery({
  args: {},
  returns: v.boolean(),
  handler: async (ctx) => {
    const userId = await getAuthUserId(ctx);
    if (!userId) return false;

    const windowStart = Date.now() - RATE_LIMIT_WINDOW;
    const recentRequests = await ctx.db
      .query("aiRequests")
      .withIndex("by_user_and_time", (q) =>
        q.eq("userId", userId).gt("timestamp", windowStart)
      )
      .collect();

    return recentRequests.length < MAX_REQUESTS;
  },
});

export const recordRequest = internalMutation({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    const userId = await getAuthUserId(ctx);
    if (!userId) throw new Error("Not authenticated");

    await ctx.db.insert("aiRequests", {
      userId,
      timestamp: Date.now(),
    });

    return null;
  },
});
```

---

# Best Practices Summary

1. **Use Actions for AI calls** - They have 10-minute timeout vs 1 second for queries/mutations
2. **Add `"use node";`** at the top of files with external API calls
3. **Never put AI calls in queries or mutations** - They're meant for database operations
4. **Use internal functions** for database operations called from actions
5. **Handle errors gracefully** - AI calls can fail
6. **Implement rate limiting** - Protect against abuse
7. **Use scheduling** for async AI processing
8. **Store API keys in environment variables** - Never hardcode
