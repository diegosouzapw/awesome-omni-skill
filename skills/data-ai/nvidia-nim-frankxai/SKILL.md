---
name: nvidia-nim
description: NVIDIA NIM inference microservices for deploying AI models with OpenAI-compatible APIs, self-hosted or cloud
version: 1.1.0
last_updated: 2026-01-06
external_version: "NVIDIA NIM 1.x"
---

# NVIDIA NIM Expert Skill

You are an expert in NVIDIA NIM (NVIDIA Inference Microservices) - a set of accelerated inference microservices for deploying foundation models on any cloud, data center, workstation, or PC.

## Overview

NVIDIA NIM provides:
- **OpenAI-compatible APIs** for seamless integration with existing tools
- **Optimized inference** using TensorRT-LLM, vLLM, and Triton Inference Server
- **Flexible deployment** - self-hosted containers or NVIDIA's cloud API
- **Enterprise-ready** - part of NVIDIA AI Enterprise with security updates

## Quick Start

### Cloud API (integrate.api.nvidia.com)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://integrate.api.nvidia.com/v1",
    api_key="nvapi-YOUR_API_KEY"  # Get from build.nvidia.com
)

response = client.chat.completions.create(
    model="meta/llama-3.1-70b-instruct",
    messages=[{"role": "user", "content": "Hello, world!"}],
    temperature=0.7,
    max_tokens=1024
)
print(response.choices[0].message.content)
```

### Self-Hosted Container

```bash
# Pull and run NIM container
docker run -d --gpus all \
  -e NGC_API_KEY=$NGC_API_KEY \
  -p 8000:8000 \
  nvcr.io/nim/meta/llama-3.1-8b-instruct:latest

# Use with OpenAI client
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="not-used")
```

## Supported Models

### LLM Models
| Model | Size | Best For |
|-------|------|----------|
| meta/llama-3.1-405b-instruct | 405B | Complex reasoning, enterprise |
| meta/llama-3.1-70b-instruct | 70B | General purpose, balanced |
| meta/llama-3.1-8b-instruct | 8B | Fast inference, cost-effective |
| mistralai/mixtral-8x22b-instruct-v0.1 | 141B | Multi-expert reasoning |
| nvidia/nemotron-4-340b-instruct | 340B | Enterprise, high accuracy |
| google/gemma-2-27b-it | 27B | Efficient, open weights |

### Vision Models (VLM)
| Model | Capabilities |
|-------|-------------|
| microsoft/phi-3-vision-128k-instruct | Image understanding |
| nvidia/vila-1.5-40b | Video/image analysis |
| google/paligemma-3b-mix-224 | Multimodal tasks |

### Embedding Models
| Model | Dimensions | Use Case |
|-------|------------|----------|
| nvidia/nv-embedqa-e5-v5 | 1024 | RAG, semantic search |
| nvidia/nv-embed-v2 | 4096 | High-quality embeddings |
| nvidia/llama-3.2-nv-embedqa-1b-v2 | 2048 | Balanced performance |

### Reranking Models
| Model | Use Case |
|-------|----------|
| nvidia/nv-rerankqa-mistral-4b-v3 | Document reranking |
| nvidia/llama-3.2-nv-rerankqa-1b-v2 | Fast reranking |

## API Reference

### Chat Completions

```python
response = client.chat.completions.create(
    model="meta/llama-3.1-70b-instruct",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing."}
    ],
    temperature=0.7,
    top_p=0.95,
    max_tokens=2048,
    stream=True,  # Enable streaming
    frequency_penalty=0.0,
    presence_penalty=0.0
)

# Handle streaming
for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Embeddings

```python
response = client.embeddings.create(
    model="nvidia/nv-embedqa-e5-v5",
    input=["Your text to embed"],
    encoding_format="float"  # or "base64"
)
embeddings = response.data[0].embedding
```

### Tool Calling / Function Calling

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "City name"}
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="meta/llama-3.1-70b-instruct",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)
```

## LangChain Integration

```python
from langchain_nvidia_ai_endpoints import ChatNVIDIA, NVIDIAEmbeddings

# Chat model
llm = ChatNVIDIA(
    model="meta/llama-3.1-70b-instruct",
    api_key="nvapi-YOUR_KEY",  # Or use NVIDIA_API_KEY env var
    temperature=0.7,
    max_tokens=1024
)

# For self-hosted
llm = ChatNVIDIA(
    base_url="http://localhost:8000/v1",
    model="meta/llama-3.1-8b-instruct"
)

# Embeddings
embeddings = NVIDIAEmbeddings(
    model="nvidia/nv-embedqa-e5-v5",
    truncate="END"  # or "NONE", "START"
)
```

## LlamaIndex Integration

```python
from llama_index.llms.nvidia import NVIDIA
from llama_index.embeddings.nvidia import NVIDIAEmbedding

# LLM
llm = NVIDIA(
    model="meta/llama-3.1-70b-instruct",
    api_key="nvapi-YOUR_KEY"
)

# Embeddings
embed_model = NVIDIAEmbedding(
    model="nvidia/nv-embedqa-e5-v5",
    truncate="END"
)
```

## Deployment Options

### 1. Docker Compose (Development)

```yaml
version: '3.8'
services:
  nim-llm:
    image: nvcr.io/nim/meta/llama-3.1-8b-instruct:latest
    ports:
      - "8000:8000"
    environment:
      - NGC_API_KEY=${NGC_API_KEY}
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
              count: 1
```

### 2. Kubernetes with Helm

```bash
# Add NVIDIA Helm repo
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update

# Install NIM
helm install nim-llm nvidia/nim-llm \
  --set model.ngcAPIKey=$NGC_API_KEY \
  --set model.name="meta/llama-3.1-8b-instruct" \
  --set resources.gpu=1
```

### 3. NVIDIA AI Workbench

```bash
# Clone NIM-anywhere template
nvwb clone https://github.com/NVIDIA/nim-anywhere

# Configure and launch
nvwb run
```

## Performance Tuning

### GPU Memory Optimization

```bash
# Environment variables for memory tuning
docker run -d --gpus all \
  -e NGC_API_KEY=$NGC_API_KEY \
  -e NIM_MAX_MODEL_LEN=4096 \
  -e NIM_GPU_MEMORY_UTILIZATION=0.9 \
  -e NIM_TENSOR_PARALLEL_SIZE=2 \
  -p 8000:8000 \
  nvcr.io/nim/meta/llama-3.1-70b-instruct:latest
```

### Key Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| NIM_MAX_MODEL_LEN | Max sequence length | Model default |
| NIM_GPU_MEMORY_UTILIZATION | GPU memory fraction | 0.9 |
| NIM_TENSOR_PARALLEL_SIZE | Multi-GPU parallelism | 1 |
| NIM_MAX_BATCH_SIZE | Max concurrent requests | Auto |
| NIM_ENABLE_KV_CACHE_REUSE | KV cache optimization | true |

## Multi-Cloud Architecture Patterns

### Pattern 1: NVIDIA Cloud + OCI Hybrid

```
User Request → API Gateway (OCI)
                    ↓
            ┌──────────────┐
            │ Route based  │
            │ on workload  │
            └──────────────┘
                    ↓
    ┌───────────────┴───────────────┐
    ↓                               ↓
NVIDIA NIM Cloud              OCI GenAI DAC
(integrate.api.nvidia.com)    (Self-hosted NIM)
- Burst capacity              - Dedicated capacity
- Pay-per-token               - Predictable costs
- Latest models               - Data residency
```

### Pattern 2: Self-Hosted Multi-Region

```
┌─────────────────────────────────────────────┐
│           Global Load Balancer              │
└─────────────────────────────────────────────┘
         ↓              ↓              ↓
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │ US-West │    │ EU-West │    │ AP-East │
   │   NIM   │    │   NIM   │    │   NIM   │
   └─────────┘    └─────────┘    └─────────┘
   A100 x4        H100 x2        A100 x2
```

## Security Best Practices

### API Key Management

```python
import os
from openai import OpenAI

# Use environment variables
client = OpenAI(
    base_url="https://integrate.api.nvidia.com/v1",
    api_key=os.environ.get("NVIDIA_API_KEY")
)
```

### Network Security for Self-Hosted

```yaml
# Kubernetes NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nim-network-policy
spec:
  podSelector:
    matchLabels:
      app: nim-llm
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: api-gateway
      ports:
        - port: 8000
```

### Guardrails Integration

```python
from nemo_guardrails import RailsConfig, LLMRails

# Configure NeMo Guardrails with NIM
config = RailsConfig.from_path("./guardrails_config")
rails = LLMRails(config)

response = rails.generate(
    messages=[{"role": "user", "content": user_input}]
)
```

## Cost Optimization

### Token-Based Pricing (Cloud API)

| Model | Input (per 1M) | Output (per 1M) |
|-------|----------------|-----------------|
| Llama 3.1 8B | $0.30 | $0.50 |
| Llama 3.1 70B | $0.88 | $1.20 |
| Llama 3.1 405B | $5.00 | $15.00 |

### Self-Hosted Cost Estimation

```
GPU Hours/Month × GPU Cost/Hour = Infrastructure Cost

Example (Llama 3.1 70B on 2x A100):
- 730 hours × $3.50/hour = $2,555/month
- Break-even: ~2M tokens/day vs cloud pricing
```

## Monitoring & Observability

### Prometheus Metrics

```yaml
# scrape_configs in prometheus.yml
- job_name: 'nim'
  static_configs:
    - targets: ['nim-llm:8000']
  metrics_path: /metrics
```

### Key Metrics to Monitor

- `nim_request_latency_seconds` - Request latency
- `nim_tokens_processed_total` - Token throughput
- `nim_gpu_memory_used_bytes` - GPU memory usage
- `nim_active_requests` - Concurrent requests

## NeMo Agent Toolkit Integration

For building agentic applications with NIM:

```python
from nemo_agent_toolkit import AgentConfig, ReactAgent

config = AgentConfig(
    llm_config={
        "_type": "nim",
        "model": "meta/llama-3.1-70b-instruct",
        "base_url": "https://integrate.api.nvidia.com/v1",
        "api_key": os.environ["NVIDIA_API_KEY"]
    }
)

agent = ReactAgent(config)
result = agent.run("Analyze this data and create a report")
```

## MCP Server with NIM

Create an MCP server that uses NIM as the backend:

```python
from mcp import Server
from openai import OpenAI

server = Server("nim-assistant")
nim_client = OpenAI(
    base_url="https://integrate.api.nvidia.com/v1",
    api_key=os.environ["NVIDIA_API_KEY"]
)

@server.tool("generate_text")
async def generate_text(prompt: str, model: str = "meta/llama-3.1-70b-instruct"):
    """Generate text using NVIDIA NIM"""
    response = nim_client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| GPU not detected | Ensure NVIDIA driver 535+ and nvidia-container-toolkit |
| OOM errors | Reduce NIM_MAX_MODEL_LEN or increase tensor parallelism |
| Slow cold start | Pre-warm with dummy requests, use KV cache |
| API key invalid | Verify nvapi- prefix, check NGC account status |

### Health Check

```bash
curl http://localhost:8000/v1/health/ready
# Returns: {"status": "ready"}

curl http://localhost:8000/v1/models
# Returns list of available models
```

## Resources

- [NVIDIA NIM Documentation](https://docs.nvidia.com/nim/index.html)
- [NIM for LLMs Guide](https://docs.nvidia.com/nim/large-language-models/latest/introduction.html)
- [NVIDIA AI Endpoints](https://build.nvidia.com/explore/discover)
- [NeMo Agent Toolkit](https://github.com/NVIDIA/NeMo-Agent-Toolkit)
- [LangChain NVIDIA Integration](https://python.langchain.com/docs/integrations/chat/nvidia_ai_endpoints/)
- [NIM on NGC Catalog](https://catalog.ngc.nvidia.com/containers?filters=nim)
