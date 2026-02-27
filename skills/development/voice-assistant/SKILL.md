---
name: voice-assistant
description: "AI-powered voice assistant combining ElevenLabs Conversational AI with Twilio telephony. Build two-way voice agents for any scenario: birthday bots, sales assistants, customer support, appointment booking, surveys, and more. Supports agent versioning, A/B testing, knowledge bases, custom tools, and 30+ languages."
homepage: https://elevenlabs.io/docs/agents-platform/overview
tags: ["voice", "calling", "elevenlabs", "twilio", "conversation", "sales", "support", "ai-agent", "convai", "hackathon"]
metadata: {"openclaw":{"emoji":"🎙️","requires":{"env":["ELEVENLABS_API_KEY","TWILIO_ACCOUNT_SID","TWILIO_AUTH_TOKEN","TWILIO_PHONE_NUMBER"],"bins":["curl","jq"]},"primaryEnv":"ELEVENLABS_API_KEY"}}
---

# Voice Assistant

**Build production-ready AI voice agents for any calling scenario.** ElevenLabs handles the conversation intelligence, Twilio handles the phone infrastructure.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ElevenLabs ConvAI                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────────┐   │
│  │   ASR   │→│   LLM   │→│   TTS   │→│ Turn-Taking  │   │
│  │ (Speech │  │(Reason) │  │ (Voice) │  │   Model      │   │
│  │ to Text)│  │         │  │         │  │              │   │
│  └─────────┘  └─────────┘  └─────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↕ WebSocket/WebRTC
┌─────────────────────────────────────────────────────────────┐
│                    Twilio Telephony                         │
│         Phone calls, PSTN connectivity, call routing        │
└─────────────────────────────────────────────────────────────┘
```

## Setup

### Environment Variables

```bash
# ElevenLabs
export ELEVENLABS_API_KEY="sk_..."
export ELEVENLABS_AGENT_ID="agent_..."  # After creating agent

# Twilio
export TWILIO_ACCOUNT_SID="AC..."
export TWILIO_AUTH_TOKEN="..."
export TWILIO_PHONE_NUMBER="+1234567890"
```

### Package Installation

```bash
# React SDK (recommended for web)
npm install @elevenlabs/react@0.12.3 zod

# JavaScript SDK (browser + server)
npm install @elevenlabs/client@0.12.2

# Server-only SDK (Node.js - full API access)
npm install @elevenlabs/elevenlabs-js@2.30.0

# CLI for "Agents as Code"
npm install -g @elevenlabs/agents-cli@0.6.1
```

---

## Quick Start: Create Your First Agent

### Option 1: API (Programmatic)

```bash
# Create agent
curl -X POST "https://api.elevenlabs.io/v1/convai/agents" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Birthday Bot",
    "conversation_config": {
      "agent": {
        "prompt": {
          "prompt": "You are calling to wish someone a happy birthday. Be warm, enthusiastic, and genuine. Ask about their plans. Keep it under 3 minutes.",
          "llm": "gpt-4o",
          "temperature": 0.7
        },
        "first_message": "Happy birthday! I wanted to call and celebrate with you!",
        "language": "en"
      },
      "tts": {
        "model_id": "eleven_turbo_v2_5",
        "voice_id": "IKne3meq5aSn9XLyUdCD"
      },
      "turn": {
        "mode": "normal"
      }
    }
  }'
```

### Option 2: CLI ("Agents as Code")

```bash
elevenlabs auth login
elevenlabs agents init
elevenlabs agents add "Birthday Bot" --template customer-service
# Edit agent_configs/birthday-bot.json
elevenlabs agents push --env prod
```

### Option 3: JavaScript SDK

```typescript
import { ElevenLabsClient } from '@elevenlabs/elevenlabs-js';

const client = new ElevenLabsClient({ apiKey: process.env.ELEVENLABS_API_KEY });

const agent = await client.conversationalAi.agents.create({
  name: 'Birthday Bot',
  conversationConfig: {
    agent: {
      prompt: {
        prompt: 'You are calling to wish someone a happy birthday...',
        llm: 'gpt-4o',
        temperature: 0.7
      },
      firstMessage: 'Happy birthday!',
      language: 'en'
    },
    tts: {
      modelId: 'eleven_turbo_v2_5',
      voiceId: 'IKne3meq5aSn9XLyUdCD'
    }
  }
});

console.log('Agent ID:', agent.agentId);
```

---

## Making Outbound Calls

### Full Two-Way Conversation (ConvAI + Twilio)

```bash
# Step 1: Get signed URL from ElevenLabs
SIGNED_URL=$(curl -s -X POST "https://api.elevenlabs.io/v1/convai/twilio/outbound" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "'$ELEVENLABS_AGENT_ID'",
    "agent_phone_number": "'$TWILIO_PHONE_NUMBER'",
    "to_number": "+15551234567"
  }' | jq -r '.signed_url')

# Step 2: Initiate call via Twilio
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Calls.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  --data-urlencode "To=+15551234567" \
  --data-urlencode "From=$TWILIO_PHONE_NUMBER" \
  --data-urlencode "Url=$SIGNED_URL"
```

### One-Way AI Voice Message (TTS Only)

For announcements where you don't need conversation:

```bash
# Step 1: Generate audio with ElevenLabs
curl -X POST "https://api.elevenlabs.io/v1/text-to-speech/IKne3meq5aSn9XLyUdCD" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Happy birthday! I wanted to be one of the first to wish you an amazing day!",
    "model_id": "eleven_turbo_v2_5",
    "voice_settings": { "stability": 0.5, "similarity_boost": 0.75 }
  }' \
  --output /tmp/message.mp3

# Step 2: Upload to temporary hosting
UPLOAD=$(curl -s -X POST -F "file=@/tmp/message.mp3" "https://tmpfiles.org/api/v1/upload")
AUDIO_URL=$(echo $UPLOAD | jq -r '.data.url' | sed 's|tmpfiles.org/|tmpfiles.org/dl/|')

# Step 3: Make call with Twilio
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Calls.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  --data-urlencode "To=+15551234567" \
  --data-urlencode "From=$TWILIO_PHONE_NUMBER" \
  --data-urlencode "Twiml=<Response><Play>$AUDIO_URL</Play></Response>"
```

---

## Agent API Reference

### Agents CRUD

```bash
# List all agents
curl -X GET "https://api.elevenlabs.io/v1/convai/agents" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Get specific agent
curl -X GET "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Update agent
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_config": {
      "agent": {
        "prompt": { "prompt": "Updated system prompt..." }
      }
    }
  }'

# Delete agent
curl -X DELETE "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Archive agent (soft delete)
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "archived": true }'
```

### Conversations

```bash
# List conversations
curl -X GET "https://api.elevenlabs.io/v1/convai/conversations?agent_id=$AGENT_ID&limit=20" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Get conversation with transcript
curl -X GET "https://api.elevenlabs.io/v1/convai/conversations/$CONVERSATION_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Get conversation audio
curl -X GET "https://api.elevenlabs.io/v1/convai/conversations/$CONVERSATION_ID/audio" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  --output conversation.mp3

# Start conversation with dynamic variables
curl -X POST "https://api.elevenlabs.io/v1/convai/conversations" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "'$AGENT_ID'",
    "dynamic_variables": {
      "user_name": "John",
      "birthday_age": "30",
      "favorite_memory": "the trip to Hawaii last year"
    }
  }'
```

---

## Agent Versioning & A/B Testing

ElevenLabs supports git-like version control for agents. Perfect for testing new prompts safely.

### Enable Versioning

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "enable_versioning_if_not_enabled": true }'
```

### Branch Management

```bash
# Create branch for experimentation
curl -X POST "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/branches" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "parent_version_id": "agtvrsn_xxxx",
    "name": "experiment-warmer-tone"
  }'

# List branches
curl -X GET "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/branches" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Delete branch
curl -X DELETE "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/branches/$BRANCH_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"
```

### Traffic Deployment (A/B Testing)

```bash
# Deploy 90/10 traffic split
curl -X POST "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/deployments" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "deployments": [
      { "branch_id": "agtbrch_main", "percentage": 90 },
      { "branch_id": "agtbrch_experiment", "percentage": 10 }
    ]
  }'

# Get deployment status
curl -X GET "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/deployments" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"
```

### Merge Successful Experiment

```bash
curl -X POST "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID/branches/merge" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source_branch_id": "agtbrch_experiment",
    "target_branch_id": "agtbrch_main",
    "archive_source_branch": true
  }'
```

---

## Testing Framework

9 API endpoints for automated agent testing.

```bash
# Create test
curl -X POST "https://api.elevenlabs.io/v1/convai/tests" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Birthday Flow Test",
    "agent_id": "'$AGENT_ID'",
    "scenario": "User receives birthday call",
    "success_criteria": [
      "Agent wishes happy birthday enthusiastically",
      "Agent asks about their plans",
      "Call ends warmly"
    ],
    "user_messages": ["Thank you!", "I am having a party tonight", "Thanks for calling!"]
  }'

# Execute test
curl -X POST "https://api.elevenlabs.io/v1/convai/tests/$TEST_ID/execute" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Get test results
curl -X GET "https://api.elevenlabs.io/v1/convai/test-results/$RESULT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Get detailed debug info
curl -X GET "https://api.elevenlabs.io/v1/convai/test-results/$RESULT_ID/debug" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# List test invocations
curl -X GET "https://api.elevenlabs.io/v1/convai/test-invocations?agent_id=$AGENT_ID&page_size=30" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Resubmit failed test
curl -X POST "https://api.elevenlabs.io/v1/convai/test-invocations/$INVOCATION_ID/resubmit" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"
```

---

## Knowledge Base (RAG)

Give your agent access to documents without loading everything into context.

```bash
# Upload document
curl -X POST "https://api.elevenlabs.io/v1/convai/knowledge-base" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -F "file=@product_catalog.pdf" \
  -F "name=Product Catalog"

# Compute RAG index (required before use)
curl -X POST "https://api.elevenlabs.io/v1/convai/knowledge-base/$DOC_ID/compute-rag-index" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "embedding_model": "e5_mistral_7b" }'

# Check index status
curl -X GET "https://api.elevenlabs.io/v1/convai/knowledge-base/$DOC_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY"

# Attach to agent
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_config": {
      "agent": {
        "prompt": {
          "knowledge_base": ["'$DOC_ID'"]
        }
      }
    }
  }'
```

---

## Tools Configuration

### Server Tools (Webhooks)

```bash
# Create webhook tool
curl -X POST "https://api.elevenlabs.io/v1/convai/tools" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "schedule_callback",
    "description": "Schedule a callback at a specific time",
    "type": "webhook",
    "webhook_config": {
      "url": "https://your-api.com/schedule",
      "method": "POST",
      "headers": {
        "Authorization": "Bearer {{secret__api_key}}"
      }
    },
    "parameters": {
      "type": "object",
      "properties": {
        "phone": { "type": "string", "description": "Phone number to call back" },
        "time": { "type": "string", "description": "ISO timestamp for callback" }
      },
      "required": ["phone", "time"]
    }
  }'

# Attach tool to agent
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_config": {
      "agent": {
        "prompt": {
          "tool_ids": ["tool_abc123"],
          "built_in_tools": ["end_call"]
        }
      }
    }
  }'
```

### Client Tools (Browser-Side)

```typescript
import { useConversation } from '@elevenlabs/react';

const { startConversation } = useConversation({
  agentId: 'your-agent-id',
  clientTools: {
    showConfetti: {
      description: 'Display birthday confetti animation on screen',
      parameters: z.object({
        color: z.string().describe('Confetti color'),
        duration: z.number().describe('Duration in seconds')
      }),
      handler: async ({ color, duration }) => {
        triggerConfetti({ color, duration });
        return { success: true };
      }
    },
    playBirthdaySong: {
      description: 'Play happy birthday song',
      parameters: z.object({}),
      handler: async () => {
        audioPlayer.play('/birthday-song.mp3');
        return { playing: true };
      }
    }
  }
});
```

### System Tools

Built-in tools that don't require external APIs:

- `end_call` - Gracefully end the conversation
- `transfer_to_number` - Transfer to another phone number
- `transfer_agent` - Hand off to another agent
- `detect_language` - Identify caller's language
- `dtmf_playpad` - Play DTMF tones
- `voicemail_detection` - Detect if reached voicemail

---

## Dynamic Variables

Personalize conversations with runtime data.

### System Variables (Auto-Populated)

| Variable | Description |
|----------|-------------|
| `{{system__agent_id}}` | Current agent ID |
| `{{system__conversation_id}}` | Current conversation ID |
| `{{system__caller_id}}` | Caller's phone number |
| `{{system__called_number}}` | Your Twilio number |
| `{{system__call_duration_secs}}` | Call duration |
| `{{system__time_utc}}` | Current UTC time |
| `{{system__call_sid}}` | Twilio call SID |

### Custom Variables

```bash
# Pass at conversation start
curl -X POST "https://api.elevenlabs.io/v1/convai/conversations" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "'$AGENT_ID'",
    "dynamic_variables": {
      "recipient_name": "Sarah",
      "birthday_age": "30",
      "sender_name": "Mom",
      "special_message": "I am so proud of everything you have accomplished"
    }
  }'
```

**Use in prompt:**
```
You are calling {{recipient_name}} on behalf of {{sender_name}} to wish them a happy {{birthday_age}}th birthday.

Include this special message: "{{special_message}}"
```

### Secret Variables

For sensitive data (API keys) - only used in headers, never sent to LLM:

```json
{
  "headers": {
    "Authorization": "Bearer {{secret__api_key}}"
  }
}
```

---

## Webhooks (Post-Call Analytics)

Receive call data after each conversation.

### Configure Webhook

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook_config": {
      "url": "https://your-api.com/elevenlabs-webhook",
      "secret": "your-hmac-secret"
    }
  }'
```

### Verify Webhook (HMAC)

```typescript
import crypto from 'crypto';

function verifyWebhook(payload: string, signature: string, secret: string): boolean {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  return signature === expected;
}

// In your handler:
app.post('/elevenlabs-webhook', (req, res) => {
  const signature = req.headers['elevenlabs-signature']; // Note: NO "X-" prefix!

  if (!verifyWebhook(JSON.stringify(req.body), signature, WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }

  // Process webhook...
  res.status(200).send('OK'); // MUST return 200 or webhook gets disabled after 10 failures
});
```

### Webhook Payload Structure

```json
{
  "conversation_id": "conv_xxx",
  "agent_id": "agent_xxx",
  "status": "completed",
  "transcript": [
    { "role": "agent", "message": "Happy birthday!", "timestamp": 0.0 },
    { "role": "user", "message": "Thank you so much!", "timestamp": 2.5 }
  ],
  "metadata": {
    "call_duration_secs": 45,
    "charging": {
      "llm_price": 0.0036,
      "llm_charge": 18,
      "call_charge": 60
    }
  },
  "analysis": {
    "call_successful": "success",
    "transcript_summary": "Agent wished user happy birthday, user expressed gratitude",
    "data_collection_results": { ... },
    "evaluation_criteria_results": { ... }
  }
}
```

---

## Voices

### Pre-Built Voices

| Voice ID | Name | Accent | Best For |
|----------|------|--------|----------|
| `IKne3meq5aSn9XLyUdCD` | Charlie | Australian | Friendly, casual |
| `JBFqnCBsd6RMkjVDRZzb` | George | British | Professional |
| `Xb7hH8MSUJpSbSDYk0k2` | Alice | British | Warm, friendly |
| `EXAVITQu4vr4xnSDxMaL` | Sarah | American | Clear, neutral |
| `CwhRBWXzGAHq8TQ4Fs17` | Roger | American | Authoritative |

### Clone Your Voice

```bash
# Upload voice sample
curl -X POST "https://api.elevenlabs.io/v1/voices/add" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -F "name=My Voice" \
  -F "files=@voice_sample.mp3" \
  -F "description=My cloned voice for calling"
```

**Best practices for voice cloning:**
- 1-2 minutes of clear audio
- No background noise, music, or pops
- Consistent microphone distance
- Natural speaking pace

### Voice Settings

```json
{
  "tts": {
    "model_id": "eleven_turbo_v2_5",
    "voice_id": "IKne3meq5aSn9XLyUdCD",
    "voice_settings": {
      "stability": 0.5,
      "similarity_boost": 0.75,
      "speed": 1.0
    }
  }
}
```

| Setting | Range | Effect |
|---------|-------|--------|
| `stability` | 0-1 | Higher = more consistent, lower = more expressive |
| `similarity_boost` | 0-1 | Higher = closer to original voice |
| `speed` | 0.7-1.2 | Speaking rate (use 0.9-1.1 for natural sound) |

### Pronunciation Dictionary

```bash
curl -X PATCH "https://api.elevenlabs.io/v1/convai/agents/$AGENT_ID" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_config": {
      "tts": {
        "pronunciation_dictionary": [
          { "word": "GIF", "pronunciation": "jif", "format": "cmu" },
          { "word": "SQL", "substitution": "sequel" }
        ]
      }
    }
  }'
```

---

## Scenario Templates

### Birthday Wishes

```json
{
  "name": "Birthday Bot",
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "PERSONALITY:\nYou are a warm, enthusiastic friend calling to wish someone a happy birthday.\n\nBEHAVIOR:\n- Open with genuine excitement\n- Ask about their birthday plans\n- Share something you appreciate about them (use {{special_message}} if provided)\n- Keep energy positive throughout\n- End warmly when conversation naturally concludes\n\nGUARDRAILS:\n- Keep under 3 minutes unless they want to chat longer\n- If they seem busy, offer warm wishes and let them go\n- Never be pushy or overstay welcome",
        "llm": "gpt-4o",
        "temperature": 0.8
      },
      "first_message": "Happy birthday! I just had to call and celebrate with you!",
      "language": "en"
    },
    "tts": {
      "model_id": "eleven_turbo_v2_5",
      "voice_id": "IKne3meq5aSn9XLyUdCD"
    },
    "turn": { "mode": "normal" }
  }
}
```

### Sales Assistant

```json
{
  "name": "Sales Assistant",
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "PERSONALITY:\nYou are Alex, a consultative sales rep. Your goal is to understand needs, NOT to push products.\n\nBEHAVIOR:\n- Ask permission: 'Do you have 2 minutes?'\n- Focus on THEIR challenges first\n- Only mention solutions that genuinely fit stated needs\n- Respect 'no' immediately and gracefully\n- Offer to send info via email if they prefer\n\nQUALIFICATION (weave naturally):\n- Budget: 'Are you funded for this initiative?'\n- Authority: 'Who else would be involved?'\n- Need: 'What problem are you solving?'\n- Timeline: 'When do you need this by?'\n\nTOOLS:\n- schedule_demo: Book a product demo\n- send_info: Email product information\n\nGUARDRAILS:\n- Never be pushy or desperate\n- Max 2 minutes for cold calls\n- Be honest if not a good fit",
        "llm": "gpt-4o",
        "temperature": 0.7,
        "tool_ids": ["tool_schedule_demo", "tool_send_info"],
        "built_in_tools": ["end_call"]
      },
      "first_message": "Hi, this is Alex from {{company_name}}. Is this a good time for a quick chat?",
      "language": "en"
    },
    "turn": { "mode": "eager" }
  }
}
```

### Customer Support

```json
{
  "name": "Support Agent",
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "PERSONALITY:\nYou are a patient, empathetic support agent. Your goal is to solve problems while making customers feel heard.\n\nBEHAVIOR:\n- Acknowledge their frustration with empathy\n- Ask clarifying questions\n- Explain solutions clearly\n- Escalate to human if you cannot resolve\n- Follow up on whether solution worked\n\nTOOLS:\n- lookup_order: Check order status and details\n- process_refund: Issue refund (requires confirmation)\n- create_ticket: Escalate to human support\n\nGUARDRAILS:\n- Never be defensive\n- Never rush them off\n- Always provide clear next steps",
        "llm": "gpt-4o",
        "temperature": 0.5,
        "tool_ids": ["tool_lookup_order", "tool_process_refund", "tool_create_ticket"],
        "built_in_tools": ["end_call", "transfer_to_number"],
        "knowledge_base": ["doc_faq", "doc_policies"]
      },
      "first_message": "Hi! Thanks for calling support. How can I help you today?",
      "language": "en"
    },
    "turn": { "mode": "patient" }
  }
}
```

### Appointment Booking

```json
{
  "name": "Appointment Scheduler",
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "PERSONALITY:\nYou are a friendly, efficient appointment scheduler.\n\nBEHAVIOR:\n- State purpose clearly\n- Offer 2-3 specific time slots\n- If none work, ask for their preference\n- Confirm all details: time, location/link, what to prepare\n- Send calendar invite via tool\n\nTOOLS:\n- check_availability: Get available time slots\n- book_appointment: Create the appointment\n- send_confirmation: Email confirmation with details\n\nGUARDRAILS:\n- Always confirm before booking\n- Provide clear next steps\n- Keep under 5 minutes",
        "llm": "gpt-4o",
        "temperature": 0.6,
        "tool_ids": ["tool_check_availability", "tool_book_appointment", "tool_send_confirmation"],
        "built_in_tools": ["end_call"]
      },
      "first_message": "Hi {{customer_name}}! I'm calling to help schedule your {{appointment_type}}.",
      "language": "en"
    },
    "turn": { "mode": "normal" }
  }
}
```

### Survey/Feedback

```json
{
  "name": "Survey Caller",
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "PERSONALITY:\nYou are conducting a brief feedback survey. Be respectful of their time.\n\nBEHAVIOR:\n- Ask permission: 'Do you have 2 minutes for quick feedback?'\n- Keep questions simple and direct\n- Listen without defending\n- Thank them sincerely\n- Offer to escalate any issues mentioned\n\nQUESTIONS:\n1. 'On a scale of 1-10, how satisfied were you with {{service_type}}?'\n2. 'What is one thing we could improve?'\n3. 'Would you recommend us to a friend?'\n\nGUARDRAILS:\n- Never exceed 2 minutes\n- Never argue with negative feedback\n- Accept all feedback gracefully",
        "llm": "gpt-4o",
        "temperature": 0.5,
        "built_in_tools": ["end_call"]
      },
      "first_message": "Hi! I'm calling from {{company_name}}. Do you have 2 minutes for quick feedback on your recent {{service_type}}?",
      "language": "en"
    },
    "turn": { "mode": "eager" }
  }
}
```

---

## Turn-Taking Modes

| Mode | Behavior | Best For |
|------|----------|----------|
| `eager` | Responds quickly, minimal pauses | Fast support, quick transactions |
| `normal` | Balanced timing (default) | General conversations |
| `patient` | Waits longer for user to finish | Complex topics, emotional calls |

---

## Multi-Language Support

32+ languages with automatic detection.

```json
{
  "conversation_config": {
    "agent": {
      "language": "en",
      "language_presets": [
        { "language": "en", "voice_id": "EXAVITQu4vr4xnSDxMaL", "first_message": "Hi! How can I help?" },
        { "language": "es", "voice_id": "spanish_voice_id", "first_message": "Hola! Como puedo ayudarte?" },
        { "language": "fr", "voice_id": "french_voice_id", "first_message": "Bonjour! Comment puis-je vous aider?" }
      ]
    }
  }
}
```

---

## React Integration

```typescript
'use client';
import { useConversation } from '@elevenlabs/react';
import { useCallback, useState } from 'react';

export function VoiceAssistant() {
  const [isActive, setIsActive] = useState(false);

  const conversation = useConversation({
    agentId: process.env.NEXT_PUBLIC_AGENT_ID,

    onConnect: () => setIsActive(true),
    onDisconnect: () => setIsActive(false),

    onEvent: (event) => {
      if (event.type === 'transcript') {
        console.log('User said:', event.text);
      } else if (event.type === 'agent_response') {
        console.log('Agent said:', event.text);
      }
    },

    clientTools: {
      showConfetti: {
        description: 'Show birthday confetti',
        parameters: z.object({}),
        handler: async () => {
          confetti({ particleCount: 100, spread: 70 });
          return { success: true };
        }
      }
    }
  });

  const startCall = useCallback(async () => {
    await navigator.mediaDevices.getUserMedia({ audio: true });
    await conversation.startSession({
      dynamicVariables: {
        recipient_name: 'Sarah',
        birthday_age: '30'
      }
    });
  }, [conversation]);

  return (
    <div>
      <button onClick={isActive ? conversation.endSession : startCall}>
        {isActive ? 'End Call' : 'Start Call'}
      </button>
      <p>Status: {conversation.status}</p>
      <p>{conversation.isSpeaking ? 'Agent speaking...' : 'Listening...'}</p>
    </div>
  );
}
```

---

## Twilio Call Options

| Option | Description |
|--------|-------------|
| `--timeout 30` | Ring for 30 seconds before giving up |
| `--record` | Record the call |
| `--machine-detection Enable` | Detect answering machines |
| `--send-digits "W1234#"` | Dial extension after connecting |

```bash
# With options
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Calls.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN" \
  --data-urlencode "To=+15551234567" \
  --data-urlencode "From=$TWILIO_PHONE_NUMBER" \
  --data-urlencode "Url=$SIGNED_URL" \
  --data-urlencode "Timeout=30" \
  --data-urlencode "Record=true" \
  --data-urlencode "MachineDetection=Enable"
```

### Check Call Status

```bash
# List recent calls
curl -X GET "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Calls.json?PageSize=10" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN"

# Get specific call
curl -X GET "https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Calls/$CALL_SID.json" \
  -u "$TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN"
```

---

## Cost Breakdown

| Component | Cost | Notes |
|-----------|------|-------|
| Twilio voice | $0.013/min | Outbound US calls |
| ElevenLabs TTS | ~$0.10/min audio | Turbo v2.5 |
| ElevenLabs ConvAI | ~$0.11-0.13/min | Two-way conversation |
| LLM (GPT-4o) | ~$0.003/call | Varies by conversation length |

**Budget optimization:**
- Use TTS-only for simple announcements
- Use ConvAI for conversations that need interactivity
- Cache common responses
- Enable LLM caching: `{ "caching": { "enabled": true, "ttl_seconds": 3600 } }`

---

## Common Errors & Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| Missing dynamic variables | Variables in prompt not provided | Pass all `{{var}}` in `dynamic_variables` |
| Tool name mismatch | Tool names are case-sensitive | Match exact case in `tool_ids` |
| Webhook disabled | Not returning 200 or 10+ failures | Always return 200, verify HMAC |
| 401 in production | Allowlist or API key issue | Check allowlist settings, verify key |
| Host not supported | Localhost allowlist bug | Use `127.0.0.1:3000` instead of `localhost` |
| WebSocket 1002 error | Network instability | Use WebRTC instead, add reconnection logic |
| RAG not working | Index not ready | Check `index.status === 'ready'` first |
| Wrong voice language | English voice for non-English | Use language-matched voices |

---

## Best Practices

### Do's
- Test on yourself first
- Use appropriate turn-taking mode
- Provide rich dynamic variables
- Handle errors gracefully
- Monitor webhook analytics

### Don'ts
- Don't hide it's AI if asked directly
- Don't be pushy or aggressive
- Don't call at unreasonable hours
- Don't repeatedly call same person
- Don't use for deception

---

## Resources

- [ElevenLabs Platform Docs](https://elevenlabs.io/docs/agents-platform/overview)
- [ElevenLabs API Reference](https://elevenlabs.io/docs/api-reference)
- [Twilio Voice Docs](https://www.twilio.com/docs/voice)
- [ElevenLabs Examples](https://github.com/elevenlabs/elevenlabs-examples)
