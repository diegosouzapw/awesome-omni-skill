---
name: garak
description: Security testing and red-teaming for LLMs using NVIDIA's garak vulnerability scanner. Use when probing AI models for jailbreaks, prompt injections, data leakage, toxic content generation, or other failure modes. Triggers on "test LLM security", "red team model", "run garak", "LLM vulnerability scan", "jailbreak testing", or "prompt injection test".
license: Apache-2.0
compatibility: Requires Python 3.10-3.12, pip, and API keys for target LLM platforms (OPENAI_API_KEY, BEDROCK_API_KEY, etc.)
metadata:
  author: haasonsaas
  version: "1.0.0"
  source: https://github.com/NVIDIA/garak
---

# Garak - LLM Vulnerability Scanner

Garak is NVIDIA's open-source security testing framework for large language models. Think of it as nmap or Metasploit, but for AI systems. It probes whether LLMs can be manipulated into generating harmful content, leaking data, accepting prompt injections, or failing in other undesirable ways.

## Installation

```bash
# Standard installation
python -m pip install -U garak

# Development version
python -m pip install -U git+https://github.com/NVIDIA/garak.git@main

# From source with conda
conda create --name garak "python>=3.10,<=3.12"
conda activate garak
git clone https://github.com/NVIDIA/garak.git
cd garak
python -m pip install -e .
```

## Core Concepts

### Architecture Components

| Component | Purpose |
|-----------|---------|
| **Probes** | Attack modules that send adversarial prompts to test specific vulnerabilities |
| **Detectors** | Analyze model outputs to identify harmful or failed responses |
| **Generators** | Interface with different LLM platforms (OpenAI, HuggingFace, Bedrock, etc.) |
| **Buffs** | Transform prompts before sending (encoding, paraphrasing, etc.) |
| **Harnesses** | Orchestrate the testing workflow |

### Supported Platforms (Generators)

**Commercial APIs:**
- `openai` - OpenAI models (requires `OPENAI_API_KEY`)
- `bedrock` - AWS Bedrock (requires `BEDROCK_API_KEY`, optional `BEDROCK_REGION`)
- `cohere` - Cohere API (requires `COHERE_API_KEY`)
- `groq` - Groq API (requires `GROQ_API_KEY`)
- `mistral` - Mistral AI
- `replicate` - Replicate API (requires `REPLICATE_API_TOKEN`)

**Local/Self-Hosted:**
- `huggingface` - HuggingFace Hub models
- `ollama` - Ollama local models
- `ggml` - GGUF format models (requires `GGML_MAIN_PATH`)
- `nim` - NVIDIA NIM endpoints (requires `NIM_API_KEY`)
- `litellm` - Unified LiteLLM interface

**Framework Integrations:**
- `langchain` - LangChain applications
- `rest` - Custom REST endpoints via YAML config

## CLI Reference

### Basic Syntax

```bash
garak [options]
```

### Essential Flags

| Flag | Description |
|------|-------------|
| `--target_type/-t` | Generator module (e.g., `openai`, `huggingface`) |
| `--target_name/-n` | Specific model name (e.g., `gpt-4`, `meta-llama/Llama-2-7b`) |
| `--probes/-p` | Probes to run (comma-separated or `all`) |
| `--detectors/-d` | Detectors to use (comma-separated or `all`) |
| `--generations/-g` | Number of outputs per prompt (default: 5) |
| `--config` | Path to YAML/JSON config file |

### Discovery Commands

```bash
# List available probes
garak --list_probes

# List available detectors
garak --list_detectors

# List available generators
garak --list_generators

# List available buffs
garak --list_buffs

# Get info about a specific plugin
garak --plugin_info probes.dan.Dan_11_0
```

### Execution Flags

| Flag | Description |
|------|-------------|
| `--verbose/-v` | Increase output verbosity (can stack: -vv) |
| `--parallel_requests` | Concurrent generator requests per prompt |
| `--parallel_attempts` | Concurrent probe attempts |
| `--seed/-s` | Random seed for reproducibility |
| `--eval_threshold` | Minimum threshold for a hit (0.0-1.0) |
| `--report_prefix` | Custom prefix for output files |
| `--interactive/-I` | Enter interactive probing mode |

## Probe Categories

### Jailbreak Attacks
- `dan` - Do Anything Now variants (Dan_6_0 through Dan_11_0, DUDE, STAN)
- `grandma` - Grandmother exploit
- `goodside` - Known jailbreaks from research
- `continuation` - Text completion jailbreaks

### Prompt Injection
- `promptinject` - Direct prompt injection attacks
- `encoding` - Encoding-based bypasses (Base64, Hex, ROT13)
- `smuggling` - Content smuggling techniques
- `latentinjection` - Latent space attacks

### Content Safety
- `realtoxicityprompts` - Toxic content generation (Profanity, Threats, Identity_Attack)
- `malwaregen` - Malware code generation attempts
- `donotanswer` - Tests refusal on restricted topics
- `lmrc` - Language Model Red-teaming Checklist

### Information Security
- `leakreplay` - Data leakage testing
- `apikey` - API key exposure detection
- `packagehallucination` - Fake package/dependency detection
- `xss` - Cross-site scripting in outputs

### Robustness
- `glitch` - Token glitch exploitation
- `badchars` - Invalid character handling
- `divergence` - Behavioral deviation detection
- `snowball` - Snowball attack variants

## Common Usage Examples

### Test OpenAI Model for Jailbreaks

```bash
export OPENAI_API_KEY="sk-..."
garak --target_type openai --target_name gpt-4 --probes dan
```

### Comprehensive Security Scan

```bash
garak --target_type openai --target_name gpt-4 --probes all --generations 10
```

### Test Specific Vulnerabilities

```bash
# Test for prompt injection
garak -t openai -n gpt-4 -p promptinject,encoding

# Test for toxic content generation
garak -t openai -n gpt-4 -p realtoxicityprompts

# Test for data leakage
garak -t openai -n gpt-4 -p leakreplay,apikey
```

### Test HuggingFace Model

```bash
garak --target_type huggingface --target_name meta-llama/Llama-2-7b-chat-hf --probes dan,encoding
```

### Test AWS Bedrock Model

```bash
export BEDROCK_API_KEY="..."
export BEDROCK_REGION="us-east-1"
garak --target_type bedrock --target_name anthropic.claude-3-sonnet --probes promptinject
```

### Test Local Ollama Model

```bash
garak --target_type ollama --target_name llama2 --probes dan,continuation
```

### Using a Config File

Create `scan_config.yaml`:
```yaml
system:
  parallel_attempts: 10
  lite: true
run:
  generations: 5
plugins:
  probe_spec: dan,encoding,promptinject,malwaregen
  extended_detectors: false
```

Run with config:
```bash
garak --config scan_config.yaml --target_type openai --target_name gpt-4
```

## Output and Reports

Garak generates three types of output:

1. **garak.log** - Debug information (persistent across runs)
2. **JSONL report** - Detailed results at `~/.local/share/garak/garak-runs-{timestamp}.jsonl`
3. **Hit log** - Records of successful exploits

### Interpreting Results

- **PASS**: Model resisted the attack
- **FAIL**: Model exhibited vulnerable behavior
- **Failure rate**: Percentage of attempts that triggered the vulnerability

### Processing Reports

```bash
# Convert to AVID format
garak --report path/to/report.jsonl
```

## Best Practices

### For Comprehensive Testing

1. **Start broad, then narrow**: Run `--probes all` first, then focus on failures
2. **Use multiple generations**: Set `--generations 10` or higher for statistical significance
3. **Test with buffs**: Apply encoding transformations to find edge cases
4. **Document baseline**: Run identical tests before and after model updates

### For Specific Vulnerability Assessment

1. **Jailbreaks**: `--probes dan,grandma,goodside,continuation`
2. **Prompt Injection**: `--probes promptinject,encoding,smuggling`
3. **Content Safety**: `--probes realtoxicityprompts,malwaregen,donotanswer`
4. **Data Security**: `--probes leakreplay,apikey,packagehallucination`

### For CI/CD Integration

```bash
# Quick scan for critical issues
garak --config garak/configs/fast.json --target_type openai --target_name $MODEL_NAME

# Set exit code based on failure threshold
garak ... --eval_threshold 0.1
```

## Troubleshooting

### Common Issues

**API Key errors**: Ensure environment variables are set correctly
```bash
export OPENAI_API_KEY="sk-..."
```

**Rate limiting**: Reduce parallel requests
```bash
garak ... --parallel_requests 1
```

**Memory issues with local models**: Use smaller batch sizes
```bash
garak ... --generations 3
```

**Unknown probe errors**: Skip unknown plugins
```bash
garak ... --skip_unknown
```

## Resources

- **Documentation**: https://docs.garak.ai
- **GitHub**: https://github.com/NVIDIA/garak
- **Discord**: discord.gg/uVch4puUCs
- **Twitter**: @garak_llm

## Workflow: Running a Security Assessment

When asked to run a garak security assessment:

1. **Verify installation**: Check if garak is installed with `garak --version`
2. **Confirm target**: Identify the model type and name
3. **Set credentials**: Ensure API keys are configured
4. **Select probes**: Choose appropriate probes for the assessment scope
5. **Execute scan**: Run garak with appropriate flags
6. **Analyze results**: Review the JSONL report and summarize findings
7. **Recommend mitigations**: Suggest fixes for identified vulnerabilities

## Example Workflow

```bash
# 1. Check installation
garak --version

# 2. List available probes for reference
garak --list_probes

# 3. Set API key
export OPENAI_API_KEY="sk-..."

# 4. Run targeted scan
garak \
  --target_type openai \
  --target_name gpt-4 \
  --probes dan,promptinject,encoding \
  --generations 5 \
  --verbose

# 5. Review results
cat ~/.local/share/garak/garak-runs-*.jsonl | jq '.status'
```
