---
name: signalwire-agents-sdk
description: Expert assistance for building SignalWire AI Agents in Python. Automatically activates when working with AgentBase, SWAIG functions, skills, SWML, voice configuration, DataMap, or any signalwire_agents code. Provides patterns, best practices, and complete working examples.
---

# SignalWire AI Agents SDK Expert

You are an expert in the SignalWire AI Agents SDK for Python. You help developers build production-ready voice AI agents using SWML (SignalWire Markup Language) and SWAIG (SignalWire AI Gateway).

## When This Skill Applies

Activate this skill when the user:
- Imports from `signalwire_agents` or `signalwire_agents.core`
- Creates classes extending `AgentBase`
- Works with SWAIG functions, tools, or handlers
- Configures voice, language, or TTS settings
- Uses DataMap for server-side functions
- Works with agent skills (built-in or custom)
- Asks about SWML, prompts, or call flow
- Deploys agents (serverless, Docker, multi-agent)

## Core SDK Knowledge

### Package Structure

```python
# Main imports
from signalwire_agents import AgentBase
from signalwire_agents.core.function_result import SwaigFunctionResult
from signalwire_agents.core.data_map import DataMap

# For multi-agent deployments
from signalwire_agents import AgentServer

# For custom skills
from signalwire_agents.core.skill_base import SkillBase

# For workflows
from signalwire_agents.core.contexts import Context, Step, ContextBuilder
```

### AgentBase - The Foundation

`AgentBase` is the main class for building agents. It combines multiple mixins:
- **PromptMixin**: Prompt building (`prompt_add_section`, POM)
- **ToolMixin**: SWAIG functions (`define_tool`, `@tool`)
- **SkillMixin**: Skill management (`add_skill`, `remove_skill`)
- **AIConfigMixin**: Voice, language, hints, parameters
- **WebMixin**: HTTP endpoints and routing
- **AuthMixin**: Basic auth, token security
- **StateMixin**: Conversation state
- **ServerlessMixin**: Lambda/Cloud Functions support

**Constructor Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | str | required | Agent identifier |
| `route` | str | "/" | HTTP endpoint path |
| `host` | str | "0.0.0.0" | Server bind address |
| `port` | int | 3000 | Server port |
| `basic_auth` | tuple | None | (username, password) for HTTP auth |
| `auto_answer` | bool | True | Automatically answer calls |
| `record_call` | bool | False | Enable call recording |
| `record_format` | str | "mp4" | Recording format |
| `record_stereo` | bool | True | Stereo recording |

### SWAIG Function Definition

**Method 1: @tool Decorator (Recommended)**

```python
@AgentBase.tool(
    name="function_name",
    description="Clear description for the AI to understand when to call this",
    parameters={
        "param_name": {
            "type": "string",
            "description": "What this parameter represents"
        },
        "optional_param": {
            "type": "integer",
            "description": "Optional parameter",
            "default": 10
        }
    }
)
def function_name(self, args, raw_data):
    param = args.get("param_name")
    return SwaigFunctionResult(f"Result for {param}")
```

**Method 2: define_tool() (Imperative)**

```python
def __init__(self):
    super().__init__(name="my-agent")

    self.define_tool(
        name="lookup_order",
        description="Look up an order by ID",
        parameters={
            "order_id": {
                "type": "string",
                "description": "The order ID to look up"
            }
        },
        handler=self.handle_lookup_order
    )

def handle_lookup_order(self, args, raw_data):
    order_id = args.get("order_id")
    # ... lookup logic
    return SwaigFunctionResult(f"Order {order_id} status: shipped")
```

**Handler Signature:**
```python
def handler(self, args: dict, raw_data: dict) -> SwaigFunctionResult:
    # args: Parameters passed by the AI
    # raw_data: Full request including call_id, metadata, etc.
    pass
```

### SwaigFunctionResult

The return type for all SWAIG function handlers.

```python
from signalwire_agents.core.function_result import SwaigFunctionResult

# Simple response
return SwaigFunctionResult("The weather is sunny and 72°F")

# Response with action
return SwaigFunctionResult("Transferring you now").add_action(
    "transfer", {"dest": "tel:+15551234567"}
)

# Multiple actions (method chaining)
return (SwaigFunctionResult("Let me play some music while I transfer you")
    .add_action("play", {"url": "https://example.com/hold.mp3"})
    .add_action("transfer", {"dest": "sip:support@company.com"}))

# Post-process (AI responds before actions execute)
return SwaigFunctionResult("I'll transfer you to support", post_process=True).add_action(
    "transfer", {"dest": "tel:+15559876543"}
)
```

**Common Actions:**

| Action | Parameters | Description |
|--------|------------|-------------|
| `transfer` | `dest` | Transfer call to destination |
| `hangup` | `reason` | End the call |
| `play` | `url`, `urls` | Play audio file(s) |
| `set_global_data` | key-value pairs | Update conversation data |
| `toggle_functions` | `active`, `inactive` | Enable/disable functions |
| `playback_bg` | `file`, `wait` | Background audio |
| `stop_playback_bg` | - | Stop background audio |

### Voice and Language Configuration

```python
# Add a language with voice
self.add_language("English", "en-US", "rime.spore")

# Multiple languages
self.add_language("English", "en-US", "rime.spore")
self.add_language("Spanish", "es-MX", "rime.spore")

# Available TTS engines and example voices:
# - ElevenLabs: "elevenlabs.josh", "elevenlabs.rachel"
# - Google: "gcloud.en-US-Neural2-A"
# - Azure: "azure.en-US-JennyNeural"
# - Amazon: "polly.Matthew"
# - Cartesia: "cartesia.default"
# - Deepgram: "deepgram.aura-asteria-en"
# - OpenAI: "openai.nova"
# - Rime (default): "rime.spore", "rime.marsh"
```

### Prompt Building

**Method 1: prompt_add_section()**

```python
# Simple section
self.prompt_add_section("Role", "You are a helpful customer service agent.")

# Section with bullets
self.prompt_add_section(
    "Guidelines",
    body="Follow these rules:",
    bullets=[
        "Be friendly and professional",
        "Keep responses concise",
        "Ask clarifying questions when needed"
    ]
)

# Subsection
self.prompt_add_subsection(
    "Guidelines",
    "Escalation",
    body="Transfer to a human if the customer asks."
)
```

**Method 2: Declarative PROMPT_SECTIONS**

```python
class MyAgent(AgentBase):
    PROMPT_SECTIONS = {
        "Role": "You are a helpful assistant.",
        "Guidelines": [
            "Be concise",
            "Be accurate",
            "Be helpful"
        ],
        "Personality": {
            "body": "You have a friendly demeanor.",
            "bullets": ["Use casual language", "Add appropriate humor"]
        }
    }
```

### AI Parameters

```python
self.set_params({
    # Speech detection
    "end_of_speech_timeout": 1000,      # ms of silence to end turn
    "attention_timeout": 10000,          # ms before "are you there?"
    "inactivity_timeout": 300000,        # ms before hanging up

    # Interruption handling
    "barge_match_string": "stop|cancel|help",
    "barge_min_words": 2,

    # AI behavior
    "ai_volume": 0,                       # -50 to 50 dB adjustment
    "local_tz": "America/New_York",

    # Energy detection
    "energy_threshold": 0.05             # 0.01-1.0, lower = more sensitive
})
```

### Speech Recognition Hints

```python
# Add hints for better recognition
self.add_hints(["SignalWire", "SWML", "SWAIG", "API"])

# Industry-specific hints
self.add_hints([
    "account number",
    "routing number",
    "checking",
    "savings"
])
```

### Call Flow Customization

Control what happens before/after the AI conversation:

```python
# Pre-answer: Play ringback while call rings
self.add_pre_answer_verb("play", {
    "urls": ["ring:us"],
    "auto_answer": False  # Required for pre-answer
})

# Post-answer: Welcome message before AI
self.add_post_answer_verb("play", {
    "url": "say:Thank you for calling. This call may be recorded."
})
self.add_post_answer_verb("sleep", {"time": 500})

# Post-AI: Cleanup after conversation ends
self.add_post_ai_verb("request", {
    "url": "https://api.example.com/call-complete",
    "method": "POST"
})
self.add_post_ai_verb("hangup", {})
```

**Pre-answer safe verbs:** transfer, execute, return, label, goto, request, switch, cond, if, eval, set, unset, hangup, send_sms, sleep

### Skills System

**Adding Built-in Skills:**

```python
# Web search
self.add_skill("web_search", {
    "api_key": "your-google-api-key",
    "search_engine_id": "your-cse-id"
})

# Weather
self.add_skill("weather_api", {
    "provider": "openweathermap",
    "api_key": "your-api-key",
    "units": "imperial"
})

# Date/time
self.add_skill("datetime", {"timezone": "America/New_York"})

# Math operations
self.add_skill("math")
```

**Available Built-in Skills:**
- `web_search` - Google Custom Search
- `wikipedia_search` - Wikipedia lookups
- `weather_api` - Weather data
- `math` - Mathematical operations
- `datetime` - Date/time functions
- `native_vector_search` - Local document search
- `swml_transfer` - Call transfers
- `datasphere` - Data integration

### DataMap (Server-Side Functions)

For functions that don't need local handlers:

```python
from signalwire_agents.core.data_map import DataMap

weather_func = (DataMap("get_weather")
    .purpose("Get current weather for a location")
    .parameter("city", "string", "City name", required=True)
    .webhook("GET", "https://api.weather.com/v1/current?q=${args.city}&key=KEY")
    .output(SwaigFunctionResult(
        "The weather in ${args.city} is ${response.condition} "
        "and ${response.temp_f}°F"
    ))
)

self.register_swaig_function(weather_func.to_swaig_function())
```

### Multi-Agent Deployment

```python
from signalwire_agents import AgentServer

server = AgentServer(host="0.0.0.0", port=3000)

server.register(SupportAgent(), "/support")
server.register(SalesAgent(), "/sales")
server.register(FAQAgent(), "/faq")

# Optionally serve static files (web UI)
server.serve_static_files("./web")

server.run()
```

### Environment Variables

Common environment variables for configuration:

```bash
# SignalWire credentials (required for Fabric API)
export SIGNALWIRE_SPACE_NAME="myspace"
export SIGNALWIRE_PROJECT_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export SIGNALWIRE_TOKEN="PTxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Proxy URL for SWML callbacks
export SWML_PROXY_URL_BASE="https://your-domain.com"

# Basic auth
export SWML_BASIC_AUTH_USER="agent"
export SWML_BASIC_AUTH_PASSWORD="secret"

# Debug webhooks
export DEBUG_WEBHOOK_URL="https://webhook.site/your-id"
export DEBUG_WEBHOOK_LEVEL="1"  # 0=off, 1=basic, 2=verbose

# Logging
export SWML_LOG_LEVEL="DEBUG"  # DEBUG, INFO, WARNING, ERROR
```

### Debug Webhooks

Send real-time events to external URL for monitoring:

```python
self.set_params({
    "debug_webhook_url": "https://webhook.site/your-id",
    "debug_webhook_level": 1  # 1=basic, 2=verbose
})
```

Debug levels:
- **0**: Off (default)
- **1**: Function calls, errors, state changes
- **2**: Full transcripts and payloads

### Post-Prompt and Summaries

Get structured data when conversations end:

```python
# Text summary
self.set_post_prompt("Summarize this conversation in 2-3 sentences.")

# Structured JSON
self.set_post_prompt(json_schema={
    "type": "object",
    "properties": {
        "resolved": {"type": "boolean"},
        "category": {"type": "string"},
        "summary": {"type": "string"}
    }
})

# Handle summary in your agent
def on_summary(self, summary=None, raw_data=None):
    if summary:
        self.log.info("call_complete", summary=summary)
```

### SignalWire Platform Integration

Register your agent with SignalWire Fabric for phone/WebRTC access:

1. **Create External SWML Handler** - Points SignalWire to your agent URL
2. **Create Subscriber** - Gets an address like `/public/my-agent`
3. **Create Guest Token** - For browser WebRTC connections

```python
# Agent with basic auth for Fabric
class MyAgent(AgentBase):
    def __init__(self):
        super().__init__(
            name="my-agent",
            basic_auth=("user", "password")  # Embedded in Fabric URL
        )
```

See `reference/signalwire-integration.md` for complete Fabric API examples.

### Dynamic Configuration

Override agent behavior per-request:

```python
def on_swml_request(self, request_data=None, callback_path=None, request=None):
    """Called before SWML is generated for each request."""
    call_data = (request_data or {}).get("call", {})
    caller = call_data.get("from", "")

    # Customize based on caller
    if caller.startswith("+1555"):
        self.prompt_add_section("VIP", "This is a VIP customer.")
```

## Code Generation Guidelines

### Always Include

1. Shebang and imports
2. Class docstring
3. `super().__init__()` call with `name`
4. Language configuration
5. At least one prompt section
6. `if __name__ == "__main__"` block

### Standard Template

```python
#!/usr/bin/env python3
from signalwire_agents import AgentBase
from signalwire_agents.core.function_result import SwaigFunctionResult


class MyAgent(AgentBase):
    """Description of what this agent does."""

    def __init__(self):
        super().__init__(name="my-agent", port=3000)

        # Configure voice
        self.add_language("English", "en-US", "rime.spore")

        # Build prompt
        self.prompt_add_section("Role", "You are a helpful assistant.")
        self.prompt_add_section(
            "Guidelines",
            bullets=[
                "Be concise and helpful",
                "Ask clarifying questions when needed"
            ]
        )

        # Configure AI behavior
        self.set_params({
            "end_of_speech_timeout": 1000,
            "attention_timeout": 10000
        })

    @AgentBase.tool(
        name="example_function",
        description="Describe what this function does",
        parameters={
            "input": {
                "type": "string",
                "description": "The input to process"
            }
        }
    )
    def example_function(self, args, raw_data):
        """Handle the example_function call."""
        input_value = args.get("input", "")
        return SwaigFunctionResult(f"Processed: {input_value}")


if __name__ == "__main__":
    agent = MyAgent()
    agent.run()
```

### Serverless Template (AWS Lambda / Google Cloud / Azure)

```python
#!/usr/bin/env python3
"""Serverless handler for SignalWire agent."""

import os
from signalwire_agents import AgentBase, SwaigFunctionResult


class MyAgent(AgentBase):
    """Description of what this agent does."""

    def __init__(self):
        super().__init__(name="my-serverless-agent")

        # Configure voice
        self.add_language("English", "en-US", "rime.spore")

        # Build prompt
        self.prompt_add_section("Role", "You are a helpful assistant.")
        self._setup_functions()

    def _setup_functions(self):
        @self.tool(
            description="Describe what this function does",
            parameters={
                "type": "object",
                "properties": {
                    "input": {
                        "type": "string",
                        "description": "The input to process"
                    }
                },
                "required": ["input"]
            }
        )
        def example_function(args, raw_data):
            input_value = args.get("input", "")
            return SwaigFunctionResult(f"Processed: {input_value}")


# CRITICAL: Create agent instance OUTSIDE handler for cold start optimization
agent = MyAgent()


# AWS Lambda
def lambda_handler(event, context):
    return agent.run(event, context)


# Google Cloud Functions
def main(request):
    return agent.run(request)


# Azure Functions (requires: import azure.functions as func)
# def main(req: func.HttpRequest) -> func.HttpResponse:
#     return agent.run(req)
```

**Key Differences from Server-Based:**
- Agent instantiated **outside** the handler function
- Use `agent.run(event, context)` for Lambda, `agent.run(request)` for GCF
- No port binding needed - HTTP trigger handles routing
- Use `@self.tool` decorator inside `_setup_functions()` method (not `@AgentBase.tool`)

### Common Mistakes to Avoid

1. **Missing language**: Agent won't speak without `add_language()`
2. **Wrong handler signature**: Must be `(self, args, raw_data)`
3. **Returning strings**: Use `SwaigFunctionResult`, not plain strings
4. **Missing name**: Constructor requires `name` parameter
5. **Vague descriptions**: AI needs clear function descriptions to call correctly
6. **Forgetting super().__init__()**: Will break mixin initialization

## Testing

```bash
# Verify SWML output
swaig-test agent.py --dump-swml

# List registered functions
swaig-test agent.py --list-tools

# Execute a function
swaig-test agent.py --exec function_name --param_name "value"

# Test specific class in multi-class file
swaig-test agent.py --agent-class MyAgent
```

## Troubleshooting

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| Agent doesn't speak | No language configured | Add `add_language()` |
| Function never called | Poor description | Make description clearer |
| Parameters missing | Schema format wrong | Use JSON Schema format |
| Import error | Wrong import path | Use `from signalwire_agents import AgentBase` |
| Port in use | Another process | Change port or stop other process |
| Transfer fails | Bad destination | Use `tel:+1...` or `sip:user@domain` |

## Reference Files

For detailed API documentation, see:

### Core Reference
- `reference/agent-base.md` - Complete AgentBase reference
- `reference/swaig-functions.md` - Function definition patterns
- `reference/function-result.md` - SwaigFunctionResult actions
- `reference/agent-server.md` - Multi-agent deployment and static files

### Advanced Features
- `reference/datamap-advanced.md` - DataMap expressions, webhooks, array processing
- `reference/contexts-steps.md` - Workflow system with steps and navigation
- `reference/prefabs.md` - Pre-built agents (InfoGatherer, Survey, Concierge, etc.)
- `reference/voice-configuration.md` - TTS engines, voices, fillers, pronunciation
- `reference/dynamic-configuration.md` - Per-request customization and routing
- `reference/skills-complete.md` - All built-in skills and custom skill development
- `reference/sip-routing.md` - SIP username-based routing
- `reference/bedrock-agent.md` - Amazon Bedrock voice-to-voice integration

### Deployment
- `reference/serverless.md` - AWS Lambda, Google Cloud Functions, Azure, CGI
- `reference/environment-variables.md` - Complete env var reference
- `reference/signalwire-integration.md` - Fabric API, WebRTC, phone numbers
- `reference/webhooks-debugging.md` - Debug webhooks, monitoring, troubleshooting

## Patterns

Best practices and reusable patterns:
- `patterns/common-patterns.md` - Frequently used patterns (lookup, transfer, confirmation)
- `patterns/error-handling.md` - Error handling best practices
- `patterns/testing.md` - Testing with swaig-test and pytest
- `patterns/security.md` - Security best practices

## Example Files

Working code examples:
- `examples/simple-agent.py` - Basic agent template
- `examples/multi-agent-server.py` - Multiple agents with static files
- `examples/webrtc-enabled-agent.py` - Browser-based voice interaction
- `examples/faq-bot.py` - FAQ bot with skills
- `examples/datamap-agent.py` - Server-side functions with DataMap
- `examples/serverless-agent.py` - AWS Lambda, Google Cloud Functions, Azure deployment

## Troubleshooting

See `troubleshooting.md` for common issues and solutions.
