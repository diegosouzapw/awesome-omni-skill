---
name: add-driver
description: Scaffold a new LLM provider driver for Prompture. Creates sync + async driver classes, registers them in the driver registry, adds settings, env template, setup.py extras, package exports, discovery integration, and models.dev pricing. Use when adding support for a new LLM provider.
metadata:
  author: prompture
  version: "2.0"
---

# Add a New LLM Driver

Scaffolds all files needed to integrate a new LLM provider into Prompture.

## Before Starting

Ask the user for:
- **Provider name** (lowercase, used as registry key and `provider/model` prefix)
- **SDK package name** on PyPI and minimum version (or `requests`/`httpx` for raw HTTP)
- **Default model ID**
- **Authentication** — API key env var name, endpoint URL, or both
- **API compatibility** — OpenAI-compatible (`/v1/chat/completions`), custom SDK, or proprietary HTTP
- **Lazy or eager import** — lazy if SDK is optional, eager if it's in `install_requires`

Also look up the provider on [models.dev](https://models.dev) to determine:
- **models.dev provider name** (e.g., `"anthropic"` for Claude, `"xai"` for Grok, `"moonshotai"` for Moonshot)
- **Whether models.dev has entries** — if yes, pricing comes from models.dev live data (set `MODEL_PRICING = {}`). If no, add hardcoded pricing.

## Files to Create or Modify (11 total)

### 1. NEW: `prompture/drivers/{provider}_driver.py` (sync driver)

See [references/driver-template.md](references/driver-template.md) for the full skeleton.

Key rules:
- Subclass `CostMixin, Driver` (NOT just `Driver`)
- Set class-level capability flags: `supports_json_mode`, `supports_json_schema`, `supports_tool_use`, `supports_streaming`, `supports_vision`, `supports_messages`
- Use `self._get_model_config(provider, model)` to get per-model `tokens_param` and `supports_temperature` from models.dev
- Use `self._calculate_cost(provider, model, prompt_tokens, completion_tokens)` — do NOT manually compute costs
- Use `self._validate_model_capabilities(provider, model, ...)` before API calls to warn about unsupported features
- If models.dev has this provider's data, set `MODEL_PRICING = {}` (empty — pricing comes live from models.dev)
- `generate()` returns `{"text": str, "meta": dict}`
- `meta` MUST contain: `prompt_tokens`, `completion_tokens`, `total_tokens`, `cost`, `raw_response`, `model_name`
- Implement `generate_messages()`, `generate_messages_with_tools()`, and `generate_messages_stream()` for full feature support
- Optional SDK: wrap import in try/except, raise `ImportError` pointing to `pip install prompture[{provider}]`

### 2. NEW: `prompture/drivers/async_{provider}_driver.py` (async driver)

Mirror of the sync driver using `AsyncDriver` base class:
- Subclass `CostMixin, AsyncDriver`
- Same capability flags as the sync driver
- Share `MODEL_PRICING` from the sync driver: `MODEL_PRICING = {Provider}Driver.MODEL_PRICING`
- Use `httpx.AsyncClient` for HTTP calls (or async SDK methods)
- All generate methods are `async def`
- Streaming returns `AsyncIterator[dict[str, Any]]`

### 3. `prompture/drivers/__init__.py`

- Add sync import: `from .{provider}_driver import {Provider}Driver`
- Add async import: `from .async_{provider}_driver import Async{Provider}Driver`
- Register sync driver with `register_driver()`:
  ```python
  register_driver(
      "{provider}",
      lambda model=None: {Provider}Driver(
          api_key=settings.{provider}_api_key,
          model=model or settings.{provider}_model,
      ),
      overwrite=True,
  )
  ```
- Add `"{Provider}Driver"` and `"Async{Provider}Driver"` to `__all__`

### 4. `prompture/__init__.py`

- Add `{Provider}Driver` to the `.drivers` import line
- Add `"{Provider}Driver"` to `__all__` under `# Drivers`

### 5. `prompture/settings.py`

Add inside `Settings` class:
```python
# {Provider}
{provider}_api_key: Optional[str] = None
{provider}_model: str = "default-model"
# Add endpoint if the provider supports custom endpoints:
# {provider}_endpoint: str = "https://api.example.com/v1"
```

### 6. `prompture/discovery.py`

Two changes required:

**a) Add to `provider_classes` dict and configuration check:**
- Import the driver class at the top of the file
- Add to `provider_classes`: `"{provider}": {Provider}Driver`
- Add configuration check in the `is_configured` block:
  ```python
  elif provider == "{provider}":
      if settings.{provider}_api_key or os.getenv("{PROVIDER}_API_KEY"):
          is_configured = True
  ```
  For local/endpoint-only providers (like ollama), use endpoint presence instead.

**b) This ensures `get_available_models()` returns the provider's models** from both:
- Static detection: `MODEL_PRICING` keys (or empty if pricing is from models.dev)
- models.dev enrichment: via `PROVIDER_MAP` in `model_rates.py` (see step 7)

### 7. `prompture/model_rates.py` — `PROVIDER_MAP`

If models.dev has this provider's data, add the mapping:
```python
PROVIDER_MAP: dict[str, str] = {
    ...
    "{provider}": "{models_dev_name}",  # e.g., "moonshot": "moonshotai"
}
```

This enables:
- **Live pricing** via `get_model_rates()` — used by `CostMixin._calculate_cost()`
- **Capability metadata** via `get_model_capabilities()` — used by `_get_model_config()` and `_validate_model_capabilities()`
- **Model discovery** via `get_all_provider_models()` — called by `discovery.py` to list all available models

To find the correct models.dev name, check: `https://models.dev/{models_dev_name}`

If models.dev does NOT have this provider, skip this step. The driver will use hardcoded `MODEL_PRICING` for costs and return `None` for capabilities.

### 8. `setup.py` / `pyproject.toml`

If optional: add `"{provider}": ["{sdk}>={version}"]` to `extras_require`.
If required: add to `install_requires`.

### 9. `.env.copy`

Add section:
```
# {Provider} Configuration
{PROVIDER}_API_KEY=your-api-key-here
{PROVIDER}_MODEL=default-model
```

### 10. `CLAUDE.md`

Add `{provider}` to the driver list in the Module Layout bullet.

### 11. OPTIONAL: `examples/{provider}_example.py`

Follow the existing example pattern (see `grok_example.py` or `groq_example.py`):
- Two extraction examples: default instruction + custom instruction
- Show different models if available
- Print JSON output and token usage statistics

## Important: Reasoning Model Handling

If the provider has reasoning models (models with `reasoning: true` on models.dev):
- Check `caps.is_reasoning` before sending `response_format` — reasoning models often don't support it
- Handle `reasoning_content` field in responses (both regular and streaming)
- Some reasoning models don't support `temperature` — respect `supports_temperature` from `_get_model_config()`

Example pattern (see `moonshot_driver.py`):
```python
if options.get("json_mode"):
    from ..model_rates import get_model_capabilities

    caps = get_model_capabilities("{provider}", model)
    is_reasoning = caps is not None and caps.is_reasoning is True
    model_supports_structured = (
        caps is None or caps.supports_structured_output is not False
    ) and not is_reasoning

    if model_supports_structured:
        # Send response_format
        ...
```

## How models.dev Integration Works

```
User calls extract_and_jsonify("moonshot/kimi-k2.5", ...)
    │
    ├─► core.py checks driver.supports_json_mode → decides json_mode
    │
    ├─► driver._get_model_config("moonshot", "kimi-k2.5")
    │       └─► model_rates.get_model_capabilities("moonshot", "kimi-k2.5")
    │               └─► PROVIDER_MAP["moonshot"] → "moonshotai"
    │               └─► models.dev data["moonshotai"]["models"]["kimi-k2.5"]
    │               └─► Returns: supports_temperature, is_reasoning, context_window, etc.
    │
    ├─► driver._calculate_cost("moonshot", "kimi-k2.5", tokens...)
    │       └─► model_rates.get_model_rates("moonshot", "kimi-k2.5")
    │               └─► Same lookup → returns {input: 0.6, output: 3.0} per 1M tokens
    │
    └─► discovery.get_available_models()
            └─► Iterates PROVIDER_MAP → get_all_provider_models("moonshotai")
            └─► Returns all model IDs under the provider
```

## Model Name Resolution

Model names are **always provider-scoped**. The format is `"provider/model_id"`.

- `get_driver_for_model("openrouter/qwen-2.5")` → looks up `"openrouter"` in the driver registry
- `get_model_capabilities("openrouter", "qwen-2.5")` → looks in models.dev under `data["openrouter"]["models"]["qwen-2.5"]`
- `get_model_capabilities("modelscope", "qwen-2.5")` → looks in models.dev under `data["modelscope"]["models"]["qwen-2.5"]`

The same model ID under different providers is **not ambiguous** — each provider has its own namespace in both the driver registry and models.dev data.

## Verification

```bash
# Import check
python -c "from prompture import {Provider}Driver; print('OK')"
python -c "from prompture.drivers import Async{Provider}Driver; print('OK')"

# Registry check
python -c "from prompture.drivers import get_driver_for_model; d = get_driver_for_model('{provider}/test'); print(type(d).__name__, d.model)"

# Discovery check
python -c "from prompture import get_available_models; ms = [m for m in get_available_models() if m.startswith('{provider}/')]; print(f'Found {{len(ms)}} models'); print(ms[:5])"

# Run tests
pytest tests/ -x -q
```
