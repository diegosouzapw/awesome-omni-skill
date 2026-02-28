---
name: nvidia-nim
description: NVIDIA NIM (NVIDIA Inference Microservices) for deploying and managing AI models. Use for NIM microservices, model inference, API integration, and building AI applications with NVIDIA's inference infrastructure.
---

# NVIDIA NIM Skill

Comprehensive guide for deploying GPU-accelerated AI inference microservices with NVIDIA NIM™. NIM provides containers to self-host pretrained and customized AI models across clouds, data centers, and RTX™ AI PCs with industry-standard APIs.

## When to Use This Skill

This skill should be triggered when you:

**Deployment & Infrastructure:**
- Need to deploy AI models with GPU acceleration (TensorRT, vLLM, SGLang, TensorRT-LLM)
- Want to self-host inference microservices on NVIDIA GPUs
- Are setting up AI infrastructure on clouds, data centers, or RTX workstations
- Need to containerize AI models for production deployment

**Model Integration:**
- Working with LLMs, vision models, or other foundation models
- Integrating pretrained or fine-tuned models into applications
- Need industry-standard API endpoints (OpenAI-compatible, REST)
- Building RAG pipelines, agentic AI workflows, or chatbots

**Performance Optimization:**
- Optimizing inference latency and throughput
- Need high-performance model serving on NVIDIA GPUs
- Working with model quantization or optimization
- Scaling AI workloads on Kubernetes

**Specific Use Cases:**
- "Deploy Llama 3 on my GPU cluster"
- "Set up inference endpoints for custom models"
- "Build AI agents with NVIDIA infrastructure"
- "Optimize model inference performance"
- "Create self-hosted AI applications with observability"

## Key Concepts

### NVIDIA NIM Architecture
**NIM (NVIDIA Inference Microservices)** are containerized microservices that provide:
- **Pre-optimized Models**: Accelerated with TensorRT, vLLM, SGLang, TensorRT-LLM
- **Industry-Standard APIs**: OpenAI-compatible REST endpoints
- **GPU Optimization**: Tailored for specific NVIDIA GPU architectures
- **Self-Hosting**: Deploy anywhere with NVIDIA acceleration

### Core Components
- **NIM Container**: Docker/Kubernetes-ready inference service
- **Inference Engines**: TensorRT-LLM, vLLM, SGLang for different model types
- **API Layer**: REST/gRPC endpoints with OpenAI compatibility
- **Observability**: Built-in metrics for monitoring and dashboards

### Deployment Targets
- **Cloud**: AWS, Azure, GCP with NVIDIA GPUs
- **Data Center**: On-premise GPU clusters
- **Edge**: RTX AI PCs and workstations
- **Kubernetes**: Helm charts for orchestration

### Supported Models
- **LLMs**: Llama, Mistral, Gemma, Falcon, and community fine-tunes
- **Vision Models**: Image generation, classification, detection
- **Custom Models**: Fine-tuned models on your data
- **Model Catalog**: Access thousands of models from NVIDIA and partners

## Quick Reference

### 1. Deploy NIM Container with Single Command

```bash
# Deploy a Llama 3 inference microservice
docker run --gpus all \
  -e NGC_API_KEY=$NGC_API_KEY \
  -p 8000:8000 \
  nvcr.io/nvidia/nim/llama-3-8b-instruct:latest
```
*Launches a GPU-accelerated Llama 3 inference service on port 8000*

### 2. Call NIM API Endpoint (OpenAI-Compatible)

```python
# Use NIM API like OpenAI
import openai

client = openai.OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="not-used"  # NIM handles auth differently
)

response = client.chat.completions.create(
    model="llama-3-8b-instruct",
    messages=[
        {"role": "user", "content": "Explain quantum computing"}
    ],
    temperature=0.7,
    max_tokens=512
)

print(response.choices[0].message.content)
```
*Standard OpenAI SDK works seamlessly with NIM endpoints*

### 3. Deploy NIM on Kubernetes with Helm

```bash
# Add NVIDIA Helm repository
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia

# Deploy NIM microservice
helm install my-nim nvidia/nim \
  --set image.repository=nvcr.io/nvidia/nim/llama-3-8b-instruct \
  --set image.tag=latest \
  --set replicaCount=3 \
  --set resources.limits.nvidia.com/gpu=1
```
*Scale NIM inference across Kubernetes cluster with GPU allocation*

### 4. Access NIM via REST API

```bash
# Direct REST API call to NIM
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama-3-8b-instruct",
    "messages": [{"role": "user", "content": "Hello, world!"}],
    "temperature": 0.5,
    "max_tokens": 100
  }'
```
*REST endpoint for language-agnostic integration*

### 5. Deploy Custom Fine-Tuned Model

```bash
# Deploy your custom fine-tuned model with NIM
docker run --gpus all \
  -e NGC_API_KEY=$NGC_API_KEY \
  -e MODEL_PATH=/models/my-custom-model \
  -v /path/to/models:/models \
  -p 8000:8000 \
  nvcr.io/nvidia/nim/base-llm:latest
```
*NIM supports custom models fine-tuned on your data*

### 6. RAG Pipeline with NIM

```python
# Building retrieval-augmented generation with NIM
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA
from langchain.vectorstores import FAISS

# Point LangChain to NIM endpoint
llm = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="not-used",
    model="llama-3-8b-instruct"
)

# Create RAG chain with NIM-powered LLM
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectorstore.as_retriever(),
    chain_type="stuff"
)

answer = qa_chain.run("What are the key features of NIM?")
```
*Integrate NIM into RAG workflows with frameworks like LangChain*

### 7. Configure NIM for Multi-GPU Setup

```yaml
# docker-compose.yml for multi-GPU NIM deployment
version: '3'
services:
  nim-service:
    image: nvcr.io/nvidia/nim/llama-3-70b-instruct:latest
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 4  # Use 4 GPUs
              capabilities: [gpu]
    environment:
      - NGC_API_KEY=${NGC_API_KEY}
      - TENSOR_PARALLEL_SIZE=4
    ports:
      - "8000:8000"
```
*Leverage multiple GPUs for large model inference*

### 8. Monitor NIM with Observability Metrics

```python
# Access built-in metrics from NIM
import requests

metrics = requests.get("http://localhost:8000/metrics")
print(metrics.text)

# Prometheus-compatible metrics include:
# - nim_inference_requests_total
# - nim_inference_duration_seconds
# - nim_gpu_utilization_percent
# - nim_throughput_tokens_per_second
```
*Built-in Prometheus metrics for dashboarding and monitoring*

### 9. Build AI Agent with NVIDIA Blueprints

```python
# Use NVIDIA AI Blueprints with NIM
from nvidia_blueprints import AgentBlueprint

# Initialize blueprint with NIM endpoint
agent = AgentBlueprint(
    nim_endpoint="http://localhost:8000/v1",
    model="llama-3-8b-instruct",
    tools=["web_search", "calculator", "code_executor"]
)

# Execute agentic workflow
result = agent.run(
    task="Research and summarize recent AI developments"
)
```
*Predefined AI workflows using NIM as inference backend*

### 10. Deploy NIM on Hugging Face Dedicated Endpoints

```python
# Alternative: Use Hugging Face dedicated endpoints
from huggingface_hub import InferenceClient

client = InferenceClient(
    model="nvidia/llama-3-8b-nim",
    token=hf_token
)

response = client.text_generation(
    "Explain NVIDIA NIM",
    max_new_tokens=200
)
```
*Managed NIM deployment via Hugging Face cloud infrastructure*

## Reference Files

This skill includes comprehensive documentation in `references/`:

### microservices.md
**Primary documentation for NVIDIA NIM architecture and capabilities:**
- Complete overview of NIM inference microservices
- How NIM works: architecture, engines, and optimization
- Deployment guides for clouds, data centers, and RTX systems
- API documentation and integration examples
- Performance optimization with TensorRT, vLLM, SGLang
- Model catalog and customization options
- Kubernetes scaling and Helm charts
- Observability and monitoring setup

**Best for:**
- Understanding NIM fundamentals
- Deployment planning and architecture
- Performance tuning strategies
- API integration patterns

### other.md
**Additional resources and references:**
- NVIDIA Build catalog (build.nvidia.com)
- Model browser and filters
- AI Blueprints and workflow templates
- Agent blueprints for agentic AI

**Best for:**
- Discovering available models
- Finding pre-built AI workflows
- Exploring agent architectures

## Working with This Skill

### For Beginners

**Start here:**
1. Read `references/microservices.md` "How It Works" section
2. Try the Quick Reference examples #1-2 (basic deployment and API calls)
3. Experiment with different models from the NVIDIA catalog
4. Learn about NIM container structure and APIs

**First Steps:**
- Get NGC API key from NVIDIA
- Install Docker with NVIDIA Container Toolkit
- Deploy your first NIM container locally
- Test with simple API calls

### For Application Developers

**Focus on:**
- Quick Reference examples #2, #4, #6 (API integration patterns)
- OpenAI compatibility for seamless migration
- RAG pipeline integration with LangChain/LlamaIndex
- Building agents and chatbots with NIM endpoints

**Key Skills:**
- REST API integration
- Handling streaming responses
- Error handling and retry logic
- Authentication and security

### For MLOps/Infrastructure Engineers

**Advanced topics:**
- Quick Reference examples #3, #7, #8 (Kubernetes, multi-GPU, monitoring)
- Helm chart customization for production
- Multi-GPU tensor parallelism configuration
- Prometheus metrics and observability
- Autoscaling strategies on Kubernetes
- Model versioning and deployment pipelines

**Production Considerations:**
- GPU resource allocation and limits
- High-availability deployment patterns
- Load balancing across NIM instances
- Monitoring and alerting setup

### For Model Developers

**Custom Models:**
- Quick Reference example #5 (custom model deployment)
- Fine-tuning workflows with NVIDIA tools
- Model optimization with TensorRT
- Quantization for inference efficiency
- Benchmarking and performance validation

**Integration Path:**
1. Fine-tune model on your data
2. Export to compatible format (GGUF, safetensors)
3. Package with NIM base container
4. Deploy and validate performance
5. Optimize with TensorRT-LLM if needed

## Common Workflows

### Workflow 1: Deploy Production-Ready LLM Service
```
1. Choose model from NVIDIA catalog → references/microservices.md
2. Deploy with Kubernetes Helm chart → Quick Reference #3
3. Configure multi-GPU if needed → Quick Reference #7
4. Set up monitoring → Quick Reference #8
5. Test with OpenAI-compatible client → Quick Reference #2
6. Scale based on metrics
```

### Workflow 2: Build RAG Application
```
1. Deploy NIM inference endpoint → Quick Reference #1
2. Integrate with vector database (Milvus, Pinecone)
3. Connect via LangChain → Quick Reference #6
4. Implement retrieval pipeline
5. Add observability and error handling
```

### Workflow 3: Create AI Agent System
```
1. Use NVIDIA AI Blueprint → Quick Reference #9
2. Deploy NIM for reasoning engine
3. Configure tool integrations (search, APIs)
4. Implement agentic workflow
5. Monitor agent performance
```

## Performance Optimization Tips

### Inference Speed
- Use TensorRT-LLM for NVIDIA GPUs (up to 8x faster)
- Enable tensor parallelism for large models (>70B)
- Use quantization (INT8, FP8) for memory efficiency
- Batch requests for higher throughput

### Resource Utilization
- Monitor GPU memory with NIM metrics
- Adjust `max_batch_size` for your workload
- Use appropriate GPU SKU (H100, A100, L40S, RTX)
- Enable KV cache optimization for repeated queries

### Scaling Strategy
- Horizontal scaling with Kubernetes for high QPS
- Vertical scaling (more GPUs) for larger models
- Load balancing across NIM instances
- Auto-scaling based on queue depth and latency

## Troubleshooting

### Common Issues

**Container won't start:**
- Verify NGC_API_KEY is set correctly
- Check GPU availability: `nvidia-smi`
- Ensure NVIDIA Container Toolkit is installed

**Out of memory errors:**
- Reduce model size or use quantized version
- Increase GPU memory or use multi-GPU
- Adjust batch size and context length limits

**Slow inference:**
- Check GPU utilization in metrics
- Verify using optimized engine (TensorRT-LLM)
- Enable tensor parallelism for large models
- Reduce precision (FP16, INT8)

**API compatibility issues:**
- Verify OpenAI SDK version compatibility
- Check endpoint URL format (`/v1/chat/completions`)
- Review NIM logs for error details

## Additional Resources

### NVIDIA Documentation
- **NIM Documentation**: Official guides and reference
- **NGC Catalog**: Browse available NIM containers
- **TensorRT-LLM**: Advanced optimization engine
- **AI Blueprints**: Pre-built workflow templates

### Community & Support
- NVIDIA Developer Forums
- GitHub examples and integrations
- Deployment guides for major cloud providers
- Performance benchmarking results

### Related Technologies
- **TensorRT**: NVIDIA inference optimization SDK
- **Triton Inference Server**: Production deployment platform
- **CUDA**: GPU computing foundation
- **cuDNN**: Deep learning primitives

## Notes

- NIM provides OpenAI-compatible APIs for easy migration
- Pre-optimized for specific GPU architectures (Hopper, Ada, Ampere)
- Supports thousands of models: LLMs, vision, speech, multimodal
- Enterprise support available through NVIDIA AI Enterprise
- Regular updates with latest model releases and optimizations

## Getting Started Checklist

- [ ] Obtain NGC API key from NVIDIA
- [ ] Install NVIDIA Container Toolkit
- [ ] Deploy first NIM container locally (Quick Reference #1)
- [ ] Test API endpoint (Quick Reference #2 or #4)
- [ ] Explore model catalog at build.nvidia.com
- [ ] Review observability metrics (Quick Reference #8)
- [ ] Plan production deployment (Kubernetes/Helm)
- [ ] Implement monitoring and alerting
- [ ] Optimize for your workload (GPU selection, batching)
- [ ] Build your AI application!
