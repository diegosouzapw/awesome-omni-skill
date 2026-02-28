---
name: adapter-factory
description: Guide for creating new CLI or HTTP adapters to integrate AI models into the AI Counsel deliberation system
---

# Adapter Factory Skill

This skill teaches how to integrate new AI models into the AI Counsel MCP server by creating adapters. There are two types of adapters:

1. **CLI Adapters** - For command-line AI tools (e.g., `claude`, `droid`, `codex`)
2. **HTTP Adapters** - For HTTP API integrations (e.g., Ollama, LM Studio, OpenRouter)

## When to Use This Skill

Use this skill when you need to:
- Add support for a new AI model or service to participate in deliberations
- Integrate a command-line AI tool (CLI adapter)
- Integrate an HTTP-based AI API (HTTP adapter)
- Understand the adapter pattern used in AI Counsel
- Debug or modify existing adapters

## Architecture Overview

### Base Classes

**BaseCLIAdapter** (`adapters/base.py`)
- Handles subprocess execution, timeout management, and error handling
- Provides `invoke()` method that manages the full CLI lifecycle
- Subclasses only implement `parse_output()` for tool-specific parsing
- Optional: Override `_adjust_args_for_context()` for deliberation vs. non-deliberation behavior
- Optional: Implement `validate_prompt_length()` for length limits

**BaseHTTPAdapter** (`adapters/base_http.py`)
- Handles HTTP requests, retry logic with exponential backoff, and timeout management
- Uses `httpx` for async HTTP and `tenacity` for retry logic
- Retries on 5xx/429/network errors, fails fast on 4xx client errors
- Subclasses implement `build_request()` and `parse_response()`
- Supports environment variable substitution for API keys

### Factory Pattern

The `create_adapter()` function in `adapters/__init__.py`:
- Maintains registries for CLI and HTTP adapters
- Creates appropriate adapter instances from config
- Handles backward compatibility with legacy config formats

---

## Creating a CLI Adapter

Follow these 6 steps to add a new command-line AI tool:

### Step 1: Create Adapter File

Create `adapters/your_cli.py`:

```python
"""Your CLI tool adapter."""
from adapters.base import BaseCLIAdapter


class YourCLIAdapter(BaseCLIAdapter):
    """Adapter for your-cli tool."""

    def parse_output(self, raw_output: str) -> str:
        """
        Parse your-cli output to extract model response.

        Args:
            raw_output: Raw stdout from CLI tool

        Returns:
            Parsed model response text
        """
        # Example: Strip headers and extract main response
        lines = raw_output.strip().split("\n")

        # Skip header lines (tool-specific logic)
        start_idx = 0
        for i, line in enumerate(lines):
            if line.strip() and not line.startswith("Loading"):
                start_idx = i
                break

        return "\n".join(lines[start_idx:]).strip()
```

**Optional:** Override `_adjust_args_for_context()` if your CLI needs different args for deliberation vs. regular use:

```python
def _adjust_args_for_context(self, is_deliberation: bool) -> list[str]:
    """
    Adjust CLI arguments based on context.

    Args:
        is_deliberation: True if part of multi-model deliberation

    Returns:
        Adjusted argument list
    """
    args = self.args.copy()

    if is_deliberation:
        # Remove flags that interfere with deliberation
        if "--interactive" in args:
            args.remove("--interactive")
    else:
        # Add flags for regular Claude Code work
        if "--context" not in args:
            args.append("--context")

    return args
```

**Optional:** Implement `validate_prompt_length()` if your API has length limits:

```python
MAX_PROMPT_CHARS = 100000  # Example: 100k char limit

def validate_prompt_length(self, prompt: str) -> bool:
    """
    Validate prompt length against API limits.

    Args:
        prompt: The full prompt to validate

    Returns:
        True if valid, False if too long
    """
    return len(prompt) <= self.MAX_PROMPT_CHARS
```

### Step 2: Update Config

Add your CLI to `config.yaml`:

```yaml
adapters:
  your_cli:
    type: cli
    command: "your-cli"
    args: ["--model", "{model}", "{prompt}"]
    timeout: 60  # Adjust based on your model's speed
```

**Placeholder Variables:**
- `{model}` - Replaced with model identifier
- `{prompt}` - Replaced with full prompt text (including context)

**Timeout Guidelines:**
- Fast models (GPT-3.5): 30-60s
- Reasoning models (Claude Sonnet 4.5, GPT-5): 180-300s
- Local models: Varies, test and adjust

### Step 3: Register Adapter

Update `adapters/__init__.py`:

```python
# Add import at top
from adapters.your_cli import YourCLIAdapter

# Add to cli_adapters dict in create_adapter()
cli_adapters: dict[str, Type[BaseCLIAdapter]] = {
    "claude": ClaudeAdapter,
    "codex": CodexAdapter,
    "droid": DroidAdapter,
    "gemini": GeminiAdapter,
    "llamacpp": LlamaCppAdapter,
    "your_cli": YourCLIAdapter,  # Add this line
}

# Add to __all__ export list
__all__ = [
    "BaseCLIAdapter",
    "BaseHTTPAdapter",
    "ClaudeAdapter",
    "CodexAdapter",
    "DroidAdapter",
    "GeminiAdapter",
    "LlamaCppAdapter",
    "YourCLIAdapter",  # Add this line
    # ... rest of exports
]
```

### Step 4: Update Schema

Update `models/schema.py`:

```python
# Find the Participant class and add your CLI to the Literal type
class Participant(BaseModel):
    cli: Literal[
        "claude",
        "codex",
        "droid",
        "gemini",
        "llamacpp",
        "your_cli"  # Add this
    ]
    model: str
```

Update the MCP tool description in `server.py`:

```python
# Find RECOMMENDED_MODELS and add your models
RECOMMENDED_MODELS = {
    "claude": ["sonnet-4.5", "opus-4"],
    "codex": ["gpt-5-codex", "gpt-4o"],
    # ... other models
    "your_cli": ["your-model-1", "your-model-2"],  # Add this
}
```

### Step 5: Add Recommended Models

Update `server.py::RECOMMENDED_MODELS` with suggested models for your CLI:

```python
RECOMMENDED_MODELS = {
    # ... existing models
    "your_cli": [
        "your-fast-model",      # For quick responses
        "your-reasoning-model",  # For complex analysis
    ],
}
```

### Step 6: Write Tests

Create unit tests in `tests/unit/test_adapters.py`:

```python
import pytest
from adapters.your_cli import YourCLIAdapter


class TestYourCLIAdapter:
    def test_parse_output_basic(self):
        """Test basic output parsing."""
        adapter = YourCLIAdapter(
            command="your-cli",
            args=["--model", "{model}", "{prompt}"],
            timeout=60
        )

        raw_output = "Loading...\n\nActual response text here"
        result = adapter.parse_output(raw_output)

        assert result == "Actual response text here"
        assert "Loading" not in result

    def test_parse_output_multiline(self):
        """Test multiline response parsing."""
        adapter = YourCLIAdapter(
            command="your-cli",
            args=["--model", "{model}", "{prompt}"],
            timeout=60
        )

        raw_output = "Header\n\nLine 1\nLine 2\nLine 3"
        result = adapter.parse_output(raw_output)

        assert "Line 1" in result
        assert "Line 2" in result
        assert "Line 3" in result
```

Add integration tests in `tests/integration/`:

```python
import pytest
from adapters.your_cli import YourCLIAdapter


@pytest.mark.integration
@pytest.mark.asyncio
async def test_your_cli_integration():
    """Test actual CLI invocation (requires your-cli installed)."""
    adapter = YourCLIAdapter(
        command="your-cli",
        args=["--model", "{model}", "{prompt}"],
        timeout=60
    )

    result = await adapter.invoke(
        prompt="What is 2+2?",
        model="your-default-model"
    )

    assert result
    assert len(result) > 0
    # Add assertions specific to your model's response format
```

---

## Creating an HTTP Adapter

Follow these 6 steps to add a new HTTP API integration:

### Step 1: Create Adapter File

Create `adapters/your_adapter.py`:

```python
"""Your API adapter."""
from typing import Tuple
from adapters.base_http import BaseHTTPAdapter


class YourAdapter(BaseHTTPAdapter):
    """
    Adapter for Your AI API.

    API reference: https://docs.yourapi.com
    Default endpoint: https://api.yourservice.com

    Example:
        adapter = YourAdapter(
            base_url="https://api.yourservice.com",
            api_key="your-key",
            timeout=120
        )
        result = await adapter.invoke(prompt="Hello", model="your-model")
    """

    def build_request(
        self, model: str, prompt: str
    ) -> Tuple[str, dict[str, str], dict]:
        """
        Build API request components.

        Args:
            model: Model identifier (e.g., "your-model-v1")
            prompt: The prompt to send

        Returns:
            Tuple of (endpoint, headers, body):
            - endpoint: URL path (e.g., "/v1/chat/completions")
            - headers: Request headers dict
            - body: Request body dict (will be JSON-encoded)
        """
        endpoint = "/v1/chat/completions"

        headers = {
            "Content-Type": "application/json",
        }

        # Add authentication if API key provided
        if self.api_key:
            headers["Authorization"] = f"Bearer {self.api_key}"

        # Build request body (adapt to your API format)
        body = {
            "model": model,
            "messages": [
                {"role": "user", "content": prompt}
            ],
            "temperature": 0.7,
            "stream": False,
        }

        return (endpoint, headers, body)

    def parse_response(self, response_json: dict) -> str:
        """
        Parse API response to extract model output.

        Your API response format:
        {
          "id": "resp-123",
          "model": "your-model",
          "choices": [
            {
              "message": {
                "role": "assistant",
                "content": "The model's response text"
              }
            }
          ]
        }

        Args:
            response_json: Parsed JSON response from API

        Returns:
            Extracted model response text

        Raises:
            KeyError: If response format is unexpected
        """
        try:
            return response_json["choices"][0]["message"]["content"]
        except (KeyError, IndexError) as e:
            raise KeyError(
                f"Unexpected API response format. "
                f"Expected 'choices[0].message.content', "
                f"got keys: {list(response_json.keys())}"
            ) from e
```

### Step 2: Update Config

Add your HTTP adapter to `config.yaml`:

```yaml
adapters:
  your_adapter:
    type: http
    base_url: "https://api.yourservice.com"
    api_key: "${YOUR_API_KEY}"  # Environment variable substitution
    timeout: 120
    max_retries: 3
```

**Configuration Options:**
- `base_url`: Base URL for API (no trailing slash)
- `api_key`: API key (use `${ENV_VAR}` for environment variables)
- `timeout`: Request timeout in seconds
- `max_retries`: Max retry attempts for 5xx/429/network errors (default: 3)
- `headers`: Optional default headers dict

**Environment Variable Substitution:**
- Pattern: `${VAR_NAME}` in config
- Substituted at runtime from environment
- Secure: Keeps secrets out of config files

### Step 3: Register Adapter

Update `adapters/__init__.py`:

```python
# Add import at top
from adapters.your_adapter import YourAdapter

# Add to http_adapters dict in create_adapter()
http_adapters: dict[str, Type[BaseHTTPAdapter]] = {
    "ollama": OllamaAdapter,
    "lmstudio": LMStudioAdapter,
    "openrouter": OpenRouterAdapter,
    "your_adapter": YourAdapter,  # Add this line
}

# Add to __all__ export list
__all__ = [
    "BaseCLIAdapter",
    "BaseHTTPAdapter",
    # ... CLI adapters
    "OllamaAdapter",
    "LMStudioAdapter",
    "OpenRouterAdapter",
    "YourAdapter",  # Add this line
    "create_adapter",
]
```

### Step 4: Set Environment Variables

If your adapter uses API keys:

```bash
# Add to your shell profile (~/.bashrc, ~/.zshrc, etc.)
export YOUR_API_KEY="your-actual-api-key-here"

# Or set temporarily for testing
export YOUR_API_KEY="test-key" && python server.py
```

### Step 5: Write Tests

Create unit tests with VCR for HTTP response recording in `tests/unit/test_your_adapter.py`:

```python
import pytest
import vcr
from adapters.your_adapter import YourAdapter


# Configure VCR for recording HTTP interactions
vcr_instance = vcr.VCR(
    cassette_library_dir="tests/fixtures/vcr_cassettes/your_adapter/",
    record_mode="once",  # Record once, then replay
    match_on=["method", "scheme", "host", "port", "path", "query"],
    filter_headers=["authorization"],  # Hide API keys in recordings
)


class TestYourAdapter:
    def test_build_request_basic(self):
        """Test request building without API key."""
        adapter = YourAdapter(
            base_url="https://api.example.com",
            timeout=60
        )

        endpoint, headers, body = adapter.build_request(
            model="test-model",
            prompt="Hello"
        )

        assert endpoint == "/v1/chat/completions"
        assert headers["Content-Type"] == "application/json"
        assert body["model"] == "test-model"
        assert body["messages"][0]["content"] == "Hello"

    def test_build_request_with_auth(self):
        """Test request building with API key."""
        adapter = YourAdapter(
            base_url="https://api.example.com",
            api_key="test-key",
            timeout=60
        )

        endpoint, headers, body = adapter.build_request(
            model="test-model",
            prompt="Hello"
        )

        assert "Authorization" in headers
        assert headers["Authorization"] == "Bearer test-key"

    def test_parse_response_success(self):
        """Test parsing successful response."""
        adapter = YourAdapter(
            base_url="https://api.example.com",
            timeout=60
        )

        response = {
            "id": "resp-123",
            "choices": [
                {
                    "message": {
                        "role": "assistant",
                        "content": "Hello! How can I help?"
                    }
                }
            ]
        }

        result = adapter.parse_response(response)
        assert result == "Hello! How can I help?"

    def test_parse_response_missing_field(self):
        """Test error handling for malformed response."""
        adapter = YourAdapter(
            base_url="https://api.example.com",
            timeout=60
        )

        response = {"id": "resp-123"}  # Missing choices

        with pytest.raises(KeyError) as exc_info:
            adapter.parse_response(response)

        assert "Unexpected API response format" in str(exc_info.value)

    @pytest.mark.asyncio
    @vcr_instance.use_cassette("invoke_basic.yaml")
    async def test_invoke_with_vcr(self):
        """Test full invoke() with VCR recording."""
        adapter = YourAdapter(
            base_url="https://api.example.com",
            api_key="test-key",
            timeout=60
        )

        result = await adapter.invoke(
            prompt="What is 2+2?",
            model="test-model"
        )

        assert result
        assert len(result) > 0
```

**Optional:** Add integration tests (requires running service):

```python
@pytest.mark.integration
@pytest.mark.asyncio
async def test_your_adapter_live():
    """Test with live API (requires service running and API key)."""
    import os

    api_key = os.getenv("YOUR_API_KEY")
    if not api_key:
        pytest.skip("YOUR_API_KEY not set")

    adapter = YourAdapter(
        base_url="https://api.yourservice.com",
        api_key=api_key,
        timeout=120
    )

    result = await adapter.invoke(
        prompt="What is the capital of France?",
        model="your-model"
    )

    assert "Paris" in result or "paris" in result
```

### Step 6: Test with Deliberation

Create a simple test script to verify integration:

```python
"""Test your adapter in a deliberation context."""
import asyncio
from adapters.your_adapter import YourAdapter


async def test():
    adapter = YourAdapter(
        base_url="https://api.yourservice.com",
        api_key="your-key",
        timeout=120
    )

    # Test basic invocation
    result = await adapter.invoke(
        prompt="What is 2+2?",
        model="your-model"
    )
    print(f"Response: {result}")

    # Test with context (simulating Round 2+ in deliberation)
    result_with_context = await adapter.invoke(
        prompt="Do you agree with this answer?",
        model="your-model",
        context="Previous participant said: 2+2 equals 4"
    )
    print(f"Response with context: {result_with_context}")


if __name__ == "__main__":
    asyncio.run(test())
```

---

## Key Design Principles

### DRY (Don't Repeat Yourself)
- Common logic in base classes (`BaseCLIAdapter`, `BaseHTTPAdapter`)
- Tool-specific logic in concrete adapters
- Only implement what's unique to your adapter

### YAGNI (You Aren't Gonna Need It)
- Build only what's needed for basic integration
- Don't add features until they're required
- Start simple, extend as needed

### TDD (Test-Driven Development)
- Write tests first (red)
- Implement adapter (green)
- Refactor for clarity (refactor)

### Type Safety
- Use type hints throughout
- Pydantic validation where applicable
- Let mypy catch errors early

### Error Isolation
- Adapter failures don't halt deliberations
- Other participants continue if one fails
- Graceful degradation with informative errors

---

## Common Patterns

### CLI Adapter with Custom Parsing

```python
def parse_output(self, raw_output: str) -> str:
    """Parse CLI output with multiple header formats."""
    lines = raw_output.strip().split("\n")

    # Skip all header lines until we find content
    content_started = False
    result_lines = []

    for line in lines:
        # Detect header patterns
        if any(marker in line.lower() for marker in ["loading", "initializing", "version"]):
            continue

        # Detect content start
        if line.strip():
            content_started = True

        if content_started:
            result_lines.append(line)

    return "\n".join(result_lines).strip()
```

### HTTP Adapter with Streaming Support

```python
def build_request(self, model: str, prompt: str) -> Tuple[str, dict, dict]:
    """Build request with optional streaming."""
    # Note: BaseHTTPAdapter doesn't support streaming yet
    # This is for future extension

    body = {
        "model": model,
        "prompt": prompt,
        "stream": False,  # Keep False for now
    }

    return ("/api/generate", {"Content-Type": "application/json"}, body)
```

### Environment Variable Validation

```python
def __init__(self, base_url: str, api_key: str = None, **kwargs):
    """Initialize with API key validation."""
    if not api_key:
        raise ValueError(
            "API key required. Set YOUR_API_KEY environment variable or "
            "provide api_key parameter."
        )

    super().__init__(base_url=base_url, api_key=api_key, **kwargs)
```

---

## Testing Guidelines

### Unit Tests (Required)
- Test `parse_output()` with various input formats
- Test `build_request()` with different parameters
- Test `parse_response()` with success and error cases
- Mock external dependencies (no actual API calls)
- Fast execution (< 1s total)

### Integration Tests (Optional)
- Test actual CLI/API invocation
- Requires tool installed or service running
- Mark with `@pytest.mark.integration`
- May be slow, use sparingly

### VCR for HTTP Tests
- Record real HTTP interactions once
- Replay from cassettes in CI/CD
- Filter sensitive data (API keys, auth tokens)
- Store in `tests/fixtures/vcr_cassettes/your_adapter/`

### E2E Tests (Optional)
- Full deliberation with your adapter
- Mark with `@pytest.mark.e2e`
- Very slow, expensive (real API calls)
- Use for final validation only

---

## Troubleshooting

### CLI Adapter Issues

**Problem:** Timeout errors
- **Solution:** Increase timeout in config.yaml
- **Reasoning models need 180-300s, not 60s**

**Problem:** Output parsing fails
- **Solution:** Print `raw_output` and examine format
- **Each CLI has unique output format, adjust parsing logic**

**Problem:** Hook interference (Claude CLI)
- **Solution:** Add `--settings '{"disableAllHooks": true}'` to args
- **User hooks can interfere with deliberation invocations**

### HTTP Adapter Issues

**Problem:** Connection refused
- **Solution:** Verify base_url and service is running
- **Check with `curl $BASE_URL/health` or similar**

**Problem:** 401 Authentication errors
- **Solution:** Verify API key is set: `echo $YOUR_API_KEY`
- **Check environment variable substitution in config**

**Problem:** 400 Bad Request errors
- **Solution:** Log request body, check API documentation
- **Common issue: Wrong field names or missing required fields**

**Problem:** Retries exhausted
- **Solution:** Check if service is healthy
- **5xx errors trigger retries, 4xx do not (by design)**

### General Issues

**Problem:** Adapter not found
- **Solution:** Verify registration in `adapters/__init__.py`
- **Check both import and dict addition**

**Problem:** Schema validation errors
- **Solution:** Add CLI name to `Participant.cli` Literal in `models/schema.py`
- **MCP won't accept unlisted CLI names**

---

## Reference Files

### Essential Files
- `adapters/base.py` - CLI adapter base class
- `adapters/base_http.py` - HTTP adapter base class
- `adapters/__init__.py` - Adapter factory and registry
- `models/config.py` - Configuration schema
- `models/schema.py` - Data models and validation

### Example Adapters
- `adapters/claude.py` - CLI adapter with context-aware args
- `adapters/gemini.py` - CLI adapter with length validation
- `adapters/ollama.py` - HTTP adapter for local API
- `adapters/lmstudio.py` - HTTP adapter with OpenAI-compatible format

### Test Examples
- `tests/unit/test_adapters.py` - CLI adapter unit tests
- `tests/unit/test_ollama.py` - HTTP adapter unit tests with VCR
- `tests/integration/test_cli_adapters.py` - CLI integration tests

---

## Next Steps After Creating Adapter

1. **Update CLAUDE.md** if you added new patterns or gotchas
2. **Add to RECOMMENDED_MODELS** in `server.py` with usage guidance
3. **Document API quirks** in adapter docstrings for future maintainers
4. **Test in real deliberation** with 2-3 participants
5. **Monitor transcript** for response quality and voting behavior
6. **Share findings** if you discover best practices for your model

---

## Quick Reference

### CLI Adapter Checklist
- [ ] Create `adapters/your_cli.py` with `parse_output()`
- [ ] Add to `config.yaml` with command, args, timeout
- [ ] Register in `adapters/__init__.py` (import + dict + export)
- [ ] Add to `Participant.cli` Literal in `models/schema.py`
- [ ] Add to `RECOMMENDED_MODELS` in `server.py`
- [ ] Write unit tests for `parse_output()`
- [ ] Optional: Write integration test with real CLI

### HTTP Adapter Checklist
- [ ] Create `adapters/your_adapter.py` with `build_request()` and `parse_response()`
- [ ] Add to `config.yaml` with base_url, api_key, timeout
- [ ] Register in `adapters/__init__.py` (import + dict + export)
- [ ] Set environment variables for API keys
- [ ] Write unit tests with VCR cassettes
- [ ] Test with simple script before full deliberation

### Common Commands

```bash
# Run unit tests for new adapter
pytest tests/unit/test_your_adapter.py -v

# Run with coverage
pytest tests/unit/test_your_adapter.py --cov=adapters.your_adapter

# Format and lint
black adapters/your_adapter.py && ruff check adapters/your_adapter.py

# Test integration (requires tool/service)
pytest tests/integration/ -v -m integration

# Record VCR cassette (first run, then commit)
pytest tests/unit/test_your_adapter.py -v
```

---

## Additional Resources

- **CLAUDE.md** - Full project documentation with architecture details
- **MCP Protocol** - https://modelcontextprotocol.io/introduction
- **Pydantic Docs** - https://docs.pydantic.dev/latest/
- **httpx Docs** - https://www.python-httpx.org/
- **VCR.py Docs** - https://vcrpy.readthedocs.io/

---

This skill encodes the institutional knowledge for extending AI Counsel with new model integrations. Follow the patterns, write tests, and maintain backward compatibility.
