---
name: windows
description: Note: Qwen 3 coder is a 30B parameter model requiring at least 24GB of VRAM to run smoothly. More is required for longer context lengths.
metadata:
  source: llms.txt
  source_url: https://docs.ollama.com/llms-full.txt
  generated: 2026-02-22T13:35:54.604Z
---

# Windows

> Note: Qwen 3 coder is a 30B parameter model requiring at least 24GB of VRAM to run smoothly. More is required for longer context lengths.

Note: Qwen 3 coder is a 30B parameter model requiring at least 24GB of VRAM to run smoothly. More is required for longer context lengths.

## Available Resources

### Usage

### Using with Claude Code

### Endpoints

### Models

### Differences from the Anthropic API

### Signing in

### API keys

### Status codes

### Error messages

### Errors that occur while streaming

### Get started

### Base URL

### Example request

### Libraries

- **Python**
  - URL: https://github.com/ollama/ollama-python

- **JavaScript**
  - URL: https://github.com/ollama/ollama-js

### Versioning

### Usage

### Endpoints

### Models

### Disabling streaming

### When to use streaming vs non-streaming

### Example response

### Recommended models

- **embeddinggemma**
  - URL: https://ollama.com/library/embeddinggemma

- **qwen3-embedding**
  - URL: https://ollama.com/library/qwen3-embedding

- **all-minilm**
  - URL: https://ollama.com/library/all-minilm

### Generate embeddings

### Generate a batch of embeddings

### Tips

### Key streaming concepts

### Handling streamed chunks

### Generating structured JSON

### Generating structured JSON with a schema

### Example: Extract structured data

### Example: Vision with structured outputs

### Tips for reliable structured outputs

### Supported models

- **Qwen 3**
  - URL: https://ollama.com/library/qwen3

- **GPT-OSS**
  - URL: https://ollama.com/library/gpt-oss

- **DeepSeek-v3.1**
  - URL: https://ollama.com/library/deepseek-v3.1

- **DeepSeek R1**
  - URL: https://ollama.com/library/deepseek-r1

- **thinking models**
  - URL: https://ollama.com/search?c=thinking

### Enable thinking in API calls

### Stream the reasoning trace

### CLI quick reference

### Calling a single tool

### Parallel tool calling

### Multi-turn tool calling (Agent loop)

### Tool calling with streaming

### Using functions as tools with Ollama Python SDK

### Quick start

### Usage with Ollama's API

### Authentication

### Web search API

### Web fetch API

### Building a search agent

### MCP Server

### Cloud Models

### Cloud API access

### Local only

### Setting context length

### CPU only

### Nvidia GPU

- **Link**: Configure the repository

```shell theme={"system"}
curl -fsSL  \
    | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
    | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
    | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
```
  - URL: https://nvidia.github.io/libnvidia-container/gpgkey

- **Link**: Configure the repository

```shell theme={"system"}
curl -fsSL  \
    | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
```
  - URL: https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo

### AMD GPU

### Vulkan Support

### Run model locally

### Try different models

### How can I upgrade Ollama?

### How can I view the logs?

### Is my GPU compatible with Ollama?

### How can I specify the context window size?

### How can I tell if my model was loaded onto the GPU?

### How do I configure Ollama server?

### How do I use Ollama behind a proxy?

### Does Ollama send my prompts and answers back to ollama.com?

### How do I disable Ollama's cloud features?

### How can I expose Ollama on my network?

### How can I use Ollama with a proxy server?

### How can I use Ollama with ngrok?

### How can I use Ollama with Cloudflare Tunnel?

### How can I allow additional web origins to access Ollama?

### Where are models stored?

### How can I use Ollama in Visual Studio Code?

### How do I use Ollama with GPU acceleration in Docker?

### Why is networking slow in WSL2 on Windows 10?

### How can I preload a model into Ollama to get faster response times?

### How do I keep a model loaded in memory or make it unload immediately?

### How do I manage the maximum number of requests the Ollama server can queue?

### How does Ollama handle concurrent requests?

### How does Ollama load models on multiple GPUs?

### How can I enable Flash Attention?

### How can I set the quantization type for the K/V cache?

### Where can I find my Ollama Public Key?

- **Ollama Cloud**
  - URL: https://ollama.com/cloud

- **https://ollama.com/settings/keys**
  - URL: https://ollama.com/settings/keys

### How can I stop Ollama from starting when I login to my computer?

### Nvidia

### AMD Radeon

### Metal (Apple GPUs)

### Vulkan GPU Support

- **https://dgpu-docs.intel.com/driver/client/overview.html**
  - URL: https://dgpu-docs.intel.com/driver/client/overview.html

- **https://amdgpu-install.readthedocs.io/en/latest/install-script.html#specifying-a-vulkan-implementation**
  - URL: https://amdgpu-install.readthedocs.io/en/latest/install-script.html#specifying-a-vulkan-implementation

### Table of Contents

- **Importing a Safetensors adapter**
  - URL: #Importing-a-fine-tuned-adapter-from-Safetensors-weights

- **Importing a Safetensors model**
  - URL: #Importing-a-model-from-Safetensors-weights

- **Importing a GGUF file**
  - URL: #Importing-a-GGUF-based-model-or-adapter

- **Sharing models on ollama.com**
  - URL: #Sharing-your-model-on-ollamacom

### Importing a fine tuned adapter from Safetensors weights

- **fine tuning framework**
  - URL: https://huggingface.co/docs/transformers/en/training

- **Unsloth**
  - URL: https://github.com/unslothai/unsloth

- **MLX**
  - URL: https://github.com/ml-explore/mlx

### Importing a model from Safetensors weights

### Importing a GGUF based model or adapter

### Quantizing a Model

### Sharing your model on ollama.com

### Libraries

### Community

### Install

### Usage with Ollama

### Recommended Models

### Install

### Usage with Ollama

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Click on `Use custom base URL` and set it to `
  - URL: https://ollama.com`

### Install

### Usage with Ollama

### Connecting to ollama.com

### Install

### Usage with Ollama

### Cloud Models

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Add the cloud configuration block to `~/.factory/config.json`:

```json theme={"system"}
{
  "custom_models": [
    {
      "model_display_name": "qwen3-coder [Ollama Cloud]",
      "model": "qwen3-coder:480b",
      "base_url": "
      "api_key": "OLLAMA_API_KEY",
      "provider": "generic-chat-completion-api",
      "max_tokens": 128000
    }
  ]
}
```
  - URL: https://ollama.com/v1/",

### Goose Desktop

- **Link**: Confirm **API Host** is ` and click Submit
  - URL: http://localhost:11434`

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: In Goose, set **API Host** to `
  - URL: https://ollama.com`

### Goose CLI

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Update **OLLAMA\_HOST** to `
  - URL: https://ollama.com`

### Coding Agents

- **Claude Code**
  - URL: /integrations/claude-code

- **Codex**
  - URL: /integrations/codex

- **OpenCode**
  - URL: /integrations/opencode

- **Droid**
  - URL: /integrations/droid

- **Goose**
  - URL: /integrations/goose

- **Pi**
  - URL: /integrations/pi

### Assistants

- **OpenClaw**
  - URL: /integrations/openclaw

### IDEs & Editors

- **VS Code**
  - URL: /integrations/vscode

- **Cline**
  - URL: /integrations/cline

- **Roo Code**
  - URL: /integrations/roo-code

- **JetBrains**
  - URL: /integrations/jetbrains

- **Xcode**
  - URL: /integrations/xcode

- **Zed**
  - URL: /integrations/zed

### Chat & RAG

- **Onyx**
  - URL: /integrations/onyx

### Automation

- **n8n**
  - URL: /integrations/n8n

### Notebooks

- **marimo**
  - URL: /integrations/marimo

### Install

### Usage with Ollama

- **Link**: Confirm the **Host URL** is ` then click **Ok**
  - URL: http://localhost:11434`,

### Install

### Usage with Ollama

- **Link**: In marimo, go to the user settings and go to the AI tab. From here
you can find and configure Ollama as an AI provider. For local use you
would typically point the base url to `
  - URL: http://localhost:11434/v1`.

### Connecting to ollama.com

### Install

### Using Ollama Locally

- **Link**: Confirm Base URL is set to ` if running locally or `http://host.docker.internal:11434` if running through docker and click **Save**
  - URL: http://localhost:11434`

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Set the **API URL** to `
  - URL: https://ollama.com`

### Overview

### Install Onyx

### Usage with Ollama

- **Link**: Provide your **Ollama API URL** and select your models.
<Note>If you're running Onyx in Docker, to access your computer's local network use ` instead of `http://127.0.0.1`.</Note>
  - URL: http://host.docker.internal`

### Send your first query

### Quick start

### Configure without launching

### Recommended models

### Connect messaging apps

### Stopping the gateway

### Install

### Usage with Ollama

### Cloud Models

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

### Install

### Usage with Ollama

### Install

### Usage with Ollama

- **Link**: (Optional) Update `Base URL` if your Ollama instance is running remotely. The default is `
  - URL: http://localhost:11434`

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Enable `Use custom base URL` and set it to `
  - URL: https://ollama.com`

### Install

### Usage with Ollama

### Install

### Usage with Ollama

### Connecting to ollama.com directly

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Select **Internet Hosted** and enter URL as `
  - URL: https://ollama.com`

### Install

### Usage with Ollama

- **Link**: Confirm the **Host URL** is ` then click **Connect**
  - URL: http://localhost:11434`,

### Connecting to ollama.com

- **API key**
  - URL: https://ollama.com/settings/keys

- **Link**: Set the **API URL** to `
  - URL: https://ollama.com`

### Install

### Manual install

### Customizing

### Updating

### Installing specific versions

### Viewing logs

### Uninstall

### System Requirements

### Filesystem Requirements

### Troubleshooting

### Uninstall

### Table of Contents

- **Format**
  - URL: #format

- **Examples**
  - URL: #examples

- **Instructions**
  - URL: #instructions

- **Notes**
  - URL: #notes

### Format

### Examples

### Instructions

### Notes

### Get Started

### Assistants

### Coding

### API

### LLM libraries

### Installing older or pre-release versions on Linux

### Linux tmp noexec

### Linux docker

### NVIDIA GPU Discovery

### AMD GPU Discovery

### Multiple AMD GPUs

- **https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native\_linux/mgpu.html#mgpu-known-issues-and-limitations**
  - URL: https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/mgpu.html#mgpu-known-issues-and-limitations

### Windows Terminal Errors

### System Requirements

- **https://www.amd.com/en/support**
  - URL: https://www.amd.com/en/support

### Filesystem Requirements

### API Access

### Troubleshooting

### Uninstall

### Standalone CLI

## How to Use This Skill

Reference these resources when working with Windows.