---
name: basilica-cli-helper
description: This skill should be used when users need to rent GPUs, run ML training jobs, or manage compute resources on Basilica's decentralized GPU marketplace. Use it for PyTorch/TensorFlow training, distributed training setup, GPU rental management, cost monitoring, or any Basilica CLI workflows. Includes workaround for non-TTY environments like Claude Code.
---

# Basilica CLI Helper

**Rent GPUs and run ML training jobs on Basilica's decentralized compute platform.**

This skill helps access high-performance GPUs through Basilica's CLI. Use this when needing to:
- Rent GPUs for machine learning training
- Run distributed training jobs
- Manage compute resources and costs
- Execute code on remote GPU instances

## Claude Code TTY Limitation

**IMPORTANT**: The `basilica up` command requires interactive TTY selection, which fails in Claude Code's non-terminal environment.

### Workaround: Use the bundled script

```bash
# List available GPUs (non-interactive)
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py list

# Filter by GPU type and cloud
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py list --gpu-type h100 --compute secure-cloud

# Rent by selection number (from list output)
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent --select 1

# Rent by offering ID directly
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent hyperstack-165
```

This script uses the Basilica REST API directly, bypassing the interactive selection.

## Quick Reference

### Essential Commands

| Command | Description |
|---------|-------------|
| `basilica login` | Authenticate with Basilica |
| `basilica ls` | List available GPUs with pricing |
| `basilica ps` | List your active rentals |
| `basilica status <uid>` | Check rental status |
| `basilica exec "cmd" --target <uid>` | Execute command on rental |
| `basilica ssh <uid>` | SSH into instance |
| `basilica cp <src> <dst>` | Copy files to/from instance |
| `basilica down <uid>` | Terminate rental |
| `basilica balance` | Check account balance |

### Cloud Types

Basilica offers two compute sources:

| Cloud | Flag | Description |
|-------|------|-------------|
| Secure Cloud | `--compute secure-cloud` | Datacenter GPUs (Hyperstack, DataCrunch, Lambda) |
| Community Cloud | `--compute community-cloud` | Decentralized miner GPUs (Bittensor network) |

## Command Reference

### Authentication

```bash
# Standard login (opens browser)
basilica login

# Device code flow (for WSL, SSH, containers)
basilica login --device-code

# Logout
basilica logout

# Check balance
basilica balance
```

### Listing GPUs

```bash
# List all available GPUs
basilica ls

# Filter by GPU type
basilica ls h100
basilica ls a100

# Filter by cloud type
basilica ls --compute secure-cloud
basilica ls --compute community-cloud

# Additional filters
basilica ls --gpu-min 2 --gpu-max 8
basilica ls --price-max 5.00
basilica ls --memory-min 80
basilica ls --country US

# JSON output
basilica ls --json
```

### Starting Rentals

**Note**: `basilica up` requires interactive TTY. In Claude Code, use the workaround script above.

In a terminal:
```bash
# Interactive GPU selection
basilica up

# Filter to GPU type (still requires selection)
basilica up h100
basilica up a100

# Specify cloud type
basilica up --compute secure-cloud
basilica up --compute community-cloud

# Detached mode (don't auto-connect)
basilica up -d

# Additional options
basilica up --gpu-count 4
basilica up --country US
basilica up --no-ssh  # Faster startup without SSH
```

### Managing Rentals

```bash
# List active rentals
basilica ps

# Show rental history
basilica ps --history

# Filter by cloud type
basilica ps --compute secure-cloud

# Check specific rental
basilica status <rental-id>

# Terminate rental
basilica down <rental-id>

# Terminate all rentals
basilica down --all

# Restart container
basilica restart <rental-id>
```

### Executing Code

```bash
# Execute command on rental
basilica exec "python train.py" --target <rental-id>

# If only one active rental, --target is optional
basilica exec "nvidia-smi"
basilica exec "pip install -r requirements.txt"
```

### File Transfer

```bash
# Copy file to rental
basilica cp train.py <rental-id>:/workspace/

# Copy directory to rental
basilica cp ./project/ <rental-id>:/workspace/project/

# Download file from rental
basilica cp <rental-id>:/workspace/model.pth ./

# Download directory
basilica cp <rental-id>:/workspace/checkpoints/ ./checkpoints/
```

### SSH Access

```bash
# SSH into instance
basilica ssh <rental-id>

# Port forwarding (e.g., Jupyter)
basilica ssh <rental-id> -L 8888:localhost:8888

# Remote port forwarding
basilica ssh <rental-id> -R 9999:localhost:9999
```

### Logs

```bash
# View logs
basilica logs <rental-id>

# Follow logs in real-time
basilica logs <rental-id> --follow

# Tail last N lines
basilica logs <rental-id> --tail 100
```

### SSH Key Management

```bash
# Add SSH key
basilica ssh-keys add

# Add with specific file
basilica ssh-keys add --file ~/.ssh/id_rsa.pub

# List registered keys
basilica ssh-keys list

# Delete key
basilica ssh-keys delete
```

### API Token Management

```bash
# Create API token
basilica tokens create <name>

# List tokens
basilica tokens list

# Revoke token
basilica tokens revoke <name>
```

### Funding

```bash
# Show deposit address
basilica fund

# List deposit history
basilica fund list --limit 100
```

## Common Workflows

### PyTorch Training (Claude Code)

```bash
# 1. List available GPUs using workaround script
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py list --compute secure-cloud

# 2. Rent GPU by selection number
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent --select 1
# Note the rental ID from output

# 3. Copy training files
basilica cp train.py <rental-id>:/workspace/
basilica cp requirements.txt <rental-id>:/workspace/

# 4. Install dependencies and run training
basilica exec "pip install -r /workspace/requirements.txt" --target <rental-id>
basilica exec "python /workspace/train.py --epochs 10" --target <rental-id>

# 5. Download results
basilica cp <rental-id>:/workspace/model.pth ./

# 6. Terminate when done
basilica down <rental-id>
```

### PyTorch Training (Terminal with TTY)

```bash
# 1. Start GPU rental (interactive selection)
basilica up h100 --compute secure-cloud -d
# Note the rental ID

# 2. Copy training files
basilica cp train.py <rental-id>:/workspace/

# 3. Run training
basilica exec "python /workspace/train.py" --target <rental-id>

# 4. Download results and cleanup
basilica cp <rental-id>:/workspace/model.pth ./
basilica down <rental-id>
```

### Jupyter Notebook

```bash
# 1. Start rental and SSH with port forward
basilica up h100 -d
basilica ssh <rental-id> -L 8888:localhost:8888

# 2. In SSH session, start Jupyter
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser

# 3. Open http://localhost:8888 in browser
```

### Check Costs

```bash
# Check current balance
basilica balance

# View rental history with costs
basilica ps --history

# Get deposit address if needed
basilica fund
```

## Troubleshooting

### "Selection failed: not a terminal"
This occurs when running `basilica up` in Claude Code. Use the workaround script:
```bash
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py list
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent --select <number>
```

### "Not logged in"
```bash
basilica login
# Or for non-browser environments:
basilica login --device-code
```

### "Insufficient balance"
```bash
basilica balance  # Check current balance
basilica fund     # Get deposit address
```

### "No GPUs available"
```bash
basilica ls  # Check different GPU types
basilica ls --compute community-cloud  # Try community cloud
```

### "SSH key not registered"
```bash
basilica ssh-keys add
```

### "Connection timeout"
```bash
basilica status <rental-id>  # Check if still running
basilica logs <rental-id>    # Check for errors
```

## GPU Selection Guide

| Use Case | Recommended GPU | Typical Price |
|----------|-----------------|---------------|
| Small models, fine-tuning | A100 (1x) | $1-2/hr |
| Medium models | H100 (1x) | $2-3/hr |
| Large models | 4-8x A100/H100 | $5-20/hr |
| Inference testing | Any 1x GPU | $1-3/hr |

## Resources

- **Basilica Homepage**: https://basilica.ai
- **CLI Help**: `basilica help <command>`
- **Version**: Check with `basilica --version`

## Scripts Reference

### basilica_up.py

Non-interactive GPU rental script for Claude Code and other non-TTY environments.

```bash
# Show help
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py --help

# List offerings
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py list [--gpu-type TYPE] [--compute CLOUD]

# Rent by selection
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent --select NUMBER

# Rent by offering ID
python ~/.claude/skills/basilica-cli-helper/scripts/basilica_up.py rent OFFERING_ID
```

The script caches offering data to `/tmp/basilica_offerings.json` for the `--select` option.
