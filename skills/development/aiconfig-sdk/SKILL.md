---
name: aiconfig-sdk
description: Use LaunchDarkly AI Configs in your Python application with the Python AI SDK. Consume AI Configs, use tools and custom parameters, handle fallbacks, and track metrics for both agent and completion modes.
compatibility: Python 3.8+ with launchdarkly-server-sdk and launchdarkly-server-sdk-ai packages.
metadata:
  author: launchdarkly
  version: "0.2.0"
---

# AI Config Python SDK

Use LaunchDarkly AI Configs in your Python application to dynamically control AI behavior without code changes. This skill covers the **Python AI SDK** specifically.

## Other Language SDKs

LaunchDarkly provides AI SDKs for multiple languages:

- **Python** (this guide) - [Documentation](https://docs.launchdarkly.com/sdk/ai/python)
- **Node.js** - [Documentation](https://docs.launchdarkly.com/sdk/ai/node-js)
- **.NET** - [Documentation](https://docs.launchdarkly.com/sdk/ai/dotnet)
- **Go** - [Documentation](https://docs.launchdarkly.com/sdk/ai/go)
- **Ruby** - [Documentation](https://docs.launchdarkly.com/sdk/ai/ruby)

For other languages, check the [SDK documentation](https://docs.launchdarkly.com/sdk).

## Prerequisites

- LaunchDarkly SDK key (from project settings or via API)
- Python 3.8+
- AI Config created in LaunchDarkly (see `aiconfig-create`)

## Getting the SDK Key

You can retrieve the SDK key via the LaunchDarkly API:

```python
import requests

def get_sdk_key(api_token: str, project_key: str, environment: str = "production"):
    """Retrieve SDK key for a project environment via API."""
    url = f"https://app.launchdarkly.com/api/v2/projects/{project_key}/environments"
    headers = {"Authorization": api_token}

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        for env in response.json().get("items", []):
            if env["key"] == environment:
                return env.get("apiKey")  # This is the SDK key
    return None

# Example usage
API_TOKEN = "api-xxx-your-api-token"  # From ~/.claude/config.json or environment
sdk_key = get_sdk_key(API_TOKEN, "your-project", "production")
print(f"SDK Key: {sdk_key}")
```

**API endpoint:** `GET /api/v2/projects/{projectKey}/environments`

Each environment returns:
- `apiKey` - Server-side SDK key
- `mobileKey` - Mobile/client-side SDK key

## Installation

```bash
pip install launchdarkly-server-sdk launchdarkly-server-sdk-ai
```

## Core Concepts

### SDK vs API Usage
- **SDK (Preferred)**: Use for consuming configs in your application at runtime
- **API**: Use for administrative tasks (creating, updating configs)

### Configuration Modes
- **Agent Mode**: For LangGraph, CrewAI, or custom agent workflows with instructions
- **Completion Mode**: For LLM calls with message arrays

## Python Implementation

### Basic SDK Setup

```python
import os
import ldclient
from ldclient import Context
from ldclient.config import Config
from ldai.client import LDAIClient

def initialize_launchdarkly(sdk_key: str):
    """Initialize LaunchDarkly SDK."""
    config = Config(sdk_key)
    ldclient.set_config(config)

    ld_client = ldclient.get()
    ai_client = LDAIClient(ld_client)

    if not ld_client.is_initialized():
        raise Exception("LaunchDarkly client failed to initialize")

    print(f"[OK] SDK initialized")
    return ld_client, ai_client

SDK_KEY = os.environ.get("LAUNCHDARKLY_SDK_KEY")
ld_client, ai_client = initialize_launchdarkly(SDK_KEY)
```

### Building User Context

```python
def build_context(user_id: str, **attributes):
    """Build LaunchDarkly context for targeting."""
    builder = Context.builder(user_id)
    for key, value in attributes.items():
        builder.set(key, value)
    return builder.build()

# Basic context
context = build_context("user-123")

# Context with attributes for targeting
context = build_context(
    "user-123",
    subscription_tier="premium",
    region="us-west"
)
```

### Consuming Completion Configs

```python
from typing import Dict
from ldai.client import AICompletionConfigDefault, ModelConfig, LDMessage, ProviderConfig

# Register fallback configs for specific config keys that need fallbacks
fallback_configs: Dict[str, AICompletionConfigDefault] = {}

def get_completion_config(ai_client, config_key: str, context, variables: dict = None):
    """Get completion-mode AI Config with optional fallback."""
    fallback = fallback_configs.get(config_key, AICompletionConfigDefault(enabled=False))
    config = ai_client.completion_config(config_key, context, fallback, variables or {})
    return config

# Usage
context = build_context("user-123", tier="premium")
config = get_completion_config(ai_client, "chatbot-config", context)

if config.enabled:
    model = config.model.name
    messages = config.messages
    tracker = config.tracker
    # Use tracker.track_success() or tracker.track_error() after AI call
else:
    print(f"[WARNING] Config 'chatbot-config' is disabled or not found")
```

### Consuming Agent Configs

```python
from typing import Dict
from ldai.client import AIAgentConfigDefault

# Register fallback configs for specific agent config keys that need fallbacks
agent_fallback_configs: Dict[str, AIAgentConfigDefault] = {}

def get_agent_config(ai_client, config_key: str, context, variables: dict = None):
    """Get agent-mode AI Config with optional fallback."""
    fallback = agent_fallback_configs.get(config_key, AIAgentConfigDefault(enabled=False))
    config = ai_client.agent_config(config_key, context, fallback, variables or {})
    return config

# Usage
context = build_context("user-123")
config = get_agent_config(ai_client, "support-agent", context)

if config.enabled:
    instructions = config.instructions
    model_name = config.model.name
    tracker = config.tracker
else:
    print(f"[WARNING] Agent config 'support-agent' is disabled or not found")
```

### Fresh Configs Per Request

Always fetch fresh configs per request to ensure targeting works correctly:

```python
class DynamicAIClient:
    """Fetch fresh config for every request - never cache configs."""
    def __init__(self, ai_client, config_key: str):
        self.ai_client = ai_client
        self.config_key = config_key

    def generate(self, prompt: str, user_id: str, **user_attributes):
        """Get fresh config for each request."""
        context = build_context(user_id, **user_attributes)
        config = get_completion_config(self.ai_client, self.config_key, context)
        return config  # Process with this config

# Each user gets their own config based on targeting rules
client = DynamicAIClient(ai_client, "chatbot-config")
config1 = client.generate("Hello", "user-123", tier="free")
config2 = client.generate("Hello", "user-456", tier="premium")
```

### Handling Multiple Configs

```python
def generate_summary(text: str, config) -> str:
    """Generate summary using the config's model and messages."""
    # Use config.model.name, config.messages, config.tracker
    # Call your LLM provider and return summary
    pass

def translate_text(text: str, config) -> str:
    """Translate text using the config's model and messages."""
    # Use config.model.name, config.messages, config.tracker
    # Call your LLM provider and return translation
    pass

def get_multiple_configs(ai_client, user_id: str):
    """Get multiple AI Configs for different purposes."""
    context = build_context(user_id)

    configs = {
        "summarizer": get_completion_config(ai_client, "summary-config", context),
        "translator": get_completion_config(ai_client, "translation-config", context),
        "analyzer": get_agent_config(ai_client, "analysis-agent", context)
    }
    return configs

# Use configs in sequence - summarize then translate the summary
configs = get_multiple_configs(ai_client, "user-123")

if configs["summarizer"].enabled:
    summary = generate_summary(text, configs["summarizer"])

    # Pass summary to translator
    if configs["translator"].enabled:
        translation = translate_text(summary, configs["translator"])
```

### Variable Substitution

```python
# In LaunchDarkly, your instruction might be:
# "You are a {{role}} assistant for {{company}}. Focus on {{focus_area}}."

context = build_context("user-123")

# Provide variable values at runtime
config = get_agent_config(
    ai_client,
    "dynamic-agent",
    context,
    variables={
        "role": "customer support",
        "company": "TechCorp",
        "focus_area": "billing issues"
    }
)

if config.enabled:
    # Instructions are populated with variable values
    print(config.instructions)
    # Output: "You are a customer support assistant for TechCorp.
    #          Focus on billing issues."
```

### Error Handling

```python
import logging

logger = logging.getLogger(__name__)

def process_with_config(ai_client, config_key: str, user_id: str, prompt: str):
    """Process request with AI Config and proper error handling."""
    context = build_context(user_id)
    config = get_completion_config(ai_client, config_key, context)

    if not config.enabled:
        logger.warning(f"Config '{config_key}' is disabled or not found")
        return None

    try:
        # Use the config to make your LLM call
        result = call_llm(config.model.name, config.messages, prompt)
        config.tracker.track_success()
        return result

    except Exception as e:
        logger.error(f"LLM call failed: {e}")
        config.tracker.track_error()
        return None
```

### Lambda/Serverless Considerations (CRITICAL)

```python
import json
import ldclient
import os
from ldclient.config import Config
from ldai.client import LDAIClient

def lambda_handler(event, lambda_context):
    """
    AWS Lambda handler with LaunchDarkly

    CRITICAL FOR SERVERLESS:
    1. Initialize SDK once and cache between invocations
    2. ALWAYS flush() before function terminates
    """

    # Initialize SDK (cached between invocations for warm starts)
    if not hasattr(lambda_handler, 'ai_client'):
        sdk_key = os.environ['LAUNCHDARKLY_SDK_KEY']

        # Configure with shorter flush interval for Lambda
        ld_config = Config(
            sdk_key,
            events_max_pending=100,
            flush_interval=1
        )

        ldclient.set_config(ld_config)
        lambda_handler.ld_client = ldclient.get()
        lambda_handler.ai_client = LDAIClient(lambda_handler.ld_client)

        # Wait for initialization (important for cold starts)
        if not lambda_handler.ld_client.is_initialized():
            lambda_handler.ld_client.wait_for_initialization(5)

    try:
        user_id = event.get('user_id', 'anonymous')
        context = build_context(user_id)
        config = get_completion_config(lambda_handler.ai_client, "lambda-config", context)

        if not config.enabled:
            return {'statusCode': 503, 'body': 'Config not available'}

        # Process with config
        result = call_llm(config.model.name, config.messages, event['prompt'])
        config.tracker.track_success()

        return {
            'statusCode': 200,
            'body': json.dumps({'response': result})
        }

    except Exception as e:
        if 'config' in locals() and config.enabled:
            config.tracker.track_error()
        raise

    finally:
        # CRITICAL: ALWAYS flush before Lambda terminates!
        lambda_handler.ld_client.flush()
```

## Best Practices

1. **Always Use Fresh Configs**
   ```python
   # DON'T cache configs across users
   cached_config = get_config(user1_context)
   for user in users:
       process(cached_config)  # Wrong!

   # DO fetch fresh config per request
   for user in users:
       config = get_config(user.context)
       process(config)  # Correct!
   ```

2. **Check config.enabled**
   - Always check `config.enabled` before using the config
   - Register fallbacks for critical config keys if needed

3. **Track Metrics**
   - Use the tracker object from configs
   - Essential for cost management and optimization

4. **Handle PII Carefully**
   ```python
   # DON'T send PII
   context = Context.builder(user.email).build()  # Bad

   # DO use opaque identifiers
   context = Context.builder(user.id).build()  # Good
   ```

5. **Flush in Serverless**
   - Call `ld_client.flush()` before Lambda/Function terminates
   - Ensures metrics are delivered

## Common Patterns

### Per-Request Configuration
```python
@app.route('/chat', methods=['POST'])
def chat_endpoint():
    """API endpoint with per-request config."""
    user_id = request.headers.get('X-User-ID', 'anonymous')

    # Fresh config for this request
    context = build_context(user_id)
    config = get_completion_config(ai_client, "chat-config", context)

    if not config.enabled:
        return jsonify({'error': 'Service unavailable'}), 503

    # Process with current config
    response = generate_response(request.json['message'], config)
    return jsonify({'response': response})
```

## Next Steps

- Use `aiconfig-create` to create new AI Configs via API
- Use `aiconfig-targeting` to set up targeting rules
- Use `aiconfig-ai-metrics` to track performance
- Use `aiconfig-list` to manage existing configs

## Related Skills

### Getting Started
- `aiconfig-create` - Create AI Configs programmatically
- `aiconfig-projects` - Create projects to organize configs
- `aiconfig-api` - API reference for managing configs

### Configuration
- `aiconfig-targeting` - Configure targeting rules
- `aiconfig-variations` - Manage multiple variations
- `aiconfig-context-basic` - Basic context patterns
- `aiconfig-context-advanced` - Advanced multi-context patterns

### Frameworks & Integration
- `aiconfig-frameworks` - Integration with LangGraph, CrewAI, and other frameworks
- `aiconfig-tools` - Manage tools for function calling
- `aiconfig-experiments` - Run A/B experiments

### Monitoring
- `aiconfig-ai-metrics` - Track automatic AI metrics
- `aiconfig-custom-metrics` - Track business metrics
- `aiconfig-online-evals` - Quality monitoring with judges

## References

- [Python AI SDK Documentation](https://docs.launchdarkly.com/sdk/ai/python)
- [AI Configs Quickstart](https://docs.launchdarkly.com/home/ai-configs/quickstart)
- [AI Config Best Practices Tutorial](https://docs.launchdarkly.com/tutorials/ai-configs-best-practices)
- [SDK Features - AI Configs](https://docs.launchdarkly.com/sdk/features/ai-config)