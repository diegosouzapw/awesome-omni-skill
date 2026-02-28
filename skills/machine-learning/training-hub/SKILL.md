---
name: training-hub
description: Fine-tune LLMs using Red Hat training-hub library with SFT, LoRA, and OSFT algorithms. Use when preparing JSONL datasets, running training jobs, configuring hardware, scaling to clusters, evaluating models, or deploying with vLLM.
---

# Training Hub

Red Hat's unified library for LLM post-training: SFT, LoRA, and OSFT (continual learning).

## Quick Reference

| Task | Command |
|------|---------|
| Recommend config | `python scripts/recommend_config.py --model <model> --hardware <hw>` |
| Estimate memory | `python scripts/estimate_memory.py --model <model> --method sft --hardware h100` |
| Validate dataset | `python scripts/validate_dataset.py data.jsonl` |
| Full fine-tuning | `from training_hub import sft` |
| LoRA training | `from training_hub import lora_sft` |
| OSFT (continual) | `from training_hub import osft` |

## Installation

```bash
pip install training-hub              # Basic
pip install training-hub[lora]        # LoRA with Unsloth (2x faster)
pip install training-hub[cuda] --no-build-isolation  # CUDA support
```

## Get Started Fast

```bash
# Get optimal config for your hardware
python scripts/recommend_config.py \
  --model meta-llama/Llama-3.1-8B-Instruct \
  --hardware rtx-5090
```

## Data Format

Training data must be JSONL with message structure:

```json
{"messages": [{"role": "user", "content": "Q"}, {"role": "assistant", "content": "A"}]}
```

**Validate before training:**
```bash
python scripts/validate_dataset.py ./training_data.jsonl
```

For data preparation details, see [DATA-FORMATS.md](DATA-FORMATS.md).

## Training Methods

### Supervised Fine-Tuning (SFT)

Full-parameter fine-tuning. Requires significant VRAM.

```python
from training_hub import sft

result = sft(
    model_path="Qwen/Qwen2.5-7B-Instruct",
    data_path="./training_data.jsonl",
    ckpt_output_dir="./checkpoints",
    num_epochs=3,
    effective_batch_size=8,
    learning_rate=2e-5,
    max_seq_len=2048,
    max_tokens_per_gpu=45000,
)
```

### LoRA Fine-Tuning

Memory-efficient adaptation (up to 2x faster, 70% less VRAM):

```python
from training_hub import lora_sft

result = lora_sft(
    model_path="Qwen/Qwen2.5-7B-Instruct",
    data_path="./training_data.jsonl",
    ckpt_output_dir="./outputs",
    lora_r=16,
    lora_alpha=32,
    num_epochs=3,
    learning_rate=2e-4,
)
```

**QLoRA (4-bit):** Add `load_in_4bit=True` for large models on limited VRAM.

### OSFT (Continual Learning)

Adapt without catastrophic forgetting:

```python
from training_hub import osft

result = osft(
    model_path="meta-llama/Llama-3.1-8B-Instruct",
    data_path="./domain_data.jsonl",
    ckpt_output_dir="./checkpoints",
    unfreeze_rank_ratio=0.25,
    effective_batch_size=16,
    learning_rate=2e-5,
)
```

For all parameters, see [ALGORITHMS.md](ALGORITHMS.md).

## Hardware Support

| Hardware | VRAM | Best For |
|----------|------|----------|
| RTX 5090 | 32GB | 8B LoRA, 70B QLoRA |
| DGX Spark | 128GB | 70B SFT |
| H100 | 80GB | 14B SFT, 70B LoRA |
| 8×H100 | 640GB | 70B SFT |

```bash
# Check if your config fits
python scripts/estimate_memory.py \
  --model meta-llama/Llama-3.1-70B-Instruct \
  --method lora \
  --hardware h100 \
  --num-gpus 8
```

For hardware-specific configs, see [HARDWARE.md](HARDWARE.md).

## Scaling

**Multi-GPU:**
```python
result = sft(..., nproc_per_node=8)
```

**Multi-node:**
```python
result = sft(..., nnodes=2, node_rank=0, nproc_per_node=8, rdzv_endpoint="0.0.0.0:29500")
```

For Slurm, Kubernetes, and datacenter deployments, see [SCALE.md](SCALE.md).

## Algorithm Selection

| Scenario | Method |
|----------|--------|
| First-time fine-tuning, large dataset | SFT |
| Memory constrained | LoRA |
| Very large model (70B+), limited VRAM | LoRA + QLoRA |
| Preserve existing capabilities | OSFT |
| Domain adaptation, small dataset | OSFT |

## Documentation

| Topic | File |
|-------|------|
| Hardware profiles & configs | [HARDWARE.md](HARDWARE.md) |
| All algorithm parameters | [ALGORITHMS.md](ALGORITHMS.md) |
| Data formats & conversion | [DATA-FORMATS.md](DATA-FORMATS.md) |
| Datacenter & cluster setup | [SCALE.md](SCALE.md) |
| Model evaluation | [EVALUATION.md](EVALUATION.md) |
| vLLM inference & serving | [INFERENCE.md](INFERENCE.md) |
| Advanced techniques | [ADVANCED.md](ADVANCED.md) |
| Model-specific configs | [MODELS.md](MODELS.md) |
| Troubleshooting | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) |
| Distributed training | [DISTRIBUTED.md](DISTRIBUTED.md) |

## Utility Scripts

| Script | Purpose |
|--------|---------|
| `recommend_config.py` | Generate optimal config for model + hardware |
| `estimate_memory.py` | Estimate GPU memory requirements |
| `validate_dataset.py` | Validate JSONL dataset format |
| `convert_to_jsonl.py` | Convert CSV, Alpaca, ShareGPT to JSONL |

## Troubleshooting

**CUDA OOM:** Reduce `max_tokens_per_gpu`, use LoRA + QLoRA, or add GPUs

**Dataset errors:** Run `python scripts/validate_dataset.py` first

**LoRA multi-GPU:** Requires `torchrun --nproc-per-node=N script.py`

**Training diverges:** Lower `learning_rate` (try 1e-5 for SFT, 1e-4 for LoRA)

For more, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
