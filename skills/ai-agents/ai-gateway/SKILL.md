---
name: ai-gateway
description: Build AI gateway services for routing and managing LLM requests. Use when implementing API proxies, rate limiting, or multi-provider AI services.
allowed-tools: Read, Write, Grep, Glob
---

# AI Gateway Provider Switching Skill

Multi-provider AI configuration for Cloodle platform.

## Trigger
- AI provider configuration
- Model switching requests
- API key setup

## Supported Providers

### Local (Ollama/LM Studio)
```env
AI_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.2
```

### Anthropic (Sonnet/Haiku)
```env
AI_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-sonnet-4-20250514
```

### HuggingFace (GPT-OSS)
```env
AI_PROVIDER=huggingface
HF_API_KEY=hf_...
HF_MODEL=gpt-oss-20b
```

## Provider Switching Logic
```python
def get_llm():
    provider = os.getenv("AI_PROVIDER", "ollama")

    if provider == "ollama":
        from langchain_ollama import ChatOllama
        return ChatOllama(
            base_url=os.getenv("OLLAMA_BASE_URL"),
            model=os.getenv("OLLAMA_MODEL", "llama3.2")
        )
    elif provider == "anthropic":
        from langchain_anthropic import ChatAnthropic
        return ChatAnthropic(
            model=os.getenv("ANTHROPIC_MODEL")
        )
    elif provider == "huggingface":
        from langchain_huggingface import HuggingFaceEndpoint
        return HuggingFaceEndpoint(
            repo_id=os.getenv("HF_MODEL")
        )
```

## Model Recommendations
| Use Case | Provider | Model |
|----------|----------|-------|
| Development | Ollama | llama3.2 |
| Production Chat | Anthropic | claude-sonnet |
| Cost Sensitive | HuggingFace | gpt-oss-20b |
| High Quality | Anthropic | claude-opus |

## Environment File Location
`/opt/cloodle/tools/ai/multi_agent_rag_system/.env`

## Test Provider
```bash
curl http://localhost:11434/api/tags  # Ollama
```
