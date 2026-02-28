---
name: templar-miner-claude-skill
description: This skill should be used when setting up, optimizing, or managing Templar AI miners on Bittensor Subnet 3 (netuid 3). Use it for tasks involving miner configuration, performance optimization, troubleshooting gradient scoring issues, managing Bittensor wallets with btcli, monitoring miner metrics, renting GPUs via Basilica for mining operations, or strategizing to achieve top miner ranking in the Templar decentralized training network. Integrates seamlessly with the basilica-cli-helper skill for GPU rentals.
---

# Templar Miner Claude Skill

## Overview

Set up, optimize, and manage Templar AI miners to achieve top performance in the Templar decentralized training network (Bittensor Subnet 3, netuid 3). This skill provides expert guidance on miner operations, from initial wallet setup through advanced optimization strategies.

## Core Capabilities

### 1. MINER SETUP & CONFIGURATION

**Initial Setup Workflow**:

1. **Create Bittensor Wallet** (see `references/bittensor_btcli.md` for complete commands):
   ```bash
   btcli wallet new-coldkey --wallet.name default --n-words 12
   btcli wallet new-hotkey --wallet.name default --wallet.hotkey miner --n-words 12
   ```

2. **Register to Templar Subnet**:
   ```bash
   btcli subnet register --wallet.name default --wallet.hotkey miner \
     --netuid 3 --subtensor.network finney
   ```

3. **Configure Environment** (use `scripts/setup_miner_env.sh`):
   ```bash
   ./scripts/setup_miner_env.sh
   # Then edit .env file with credentials
   ```

4. **Verify Setup** (use `scripts/check_miner_health.py`):
   ```bash
   python scripts/check_miner_health.py
   ```

5. **Install Templar**:
   ```bash
   git clone https://github.com/one-covenant/templar
   cd templar
   pip install -e .
   ```

6. **Launch Miner**:
   ```bash
   python neurons/miner.py \
     --wallet.name default \
     --wallet.hotkey miner \
     --netuid 3 \
     --subtensor.network finney \
     --device cuda
   ```

**Hardware Requirements**:
- **Minimum for Competitive Mining**: 8x H200 GPUs (141GB VRAM each)
- CPU: 64+ cores
- RAM: 512+ GB
- Storage: 1TB+ NVMe SSD
- Network: 10Gbps+ bandwidth

**Note**: Smaller configurations (4x H100, etc.) are unlikely to be competitive for rewards. For GPU rentals, see the **GPU Rental via Basilica** section below.

**GPU Rental via Basilica**:

For those without access to 8x H200 GPUs, rent them through Basilica's decentralized compute marketplace. The `basilica-cli-helper` Claude skill (https://github.com/synapz-org/basilica-cli-claude-skill) integrates seamlessly with this Templar skill.

**Quick Basilica Workflow**:

1. **Check Available GPUs**:
   ```bash
   basilica ls
   basilica price --gpu h200  # Check H200 pricing
   ```

2. **Rent GPUs**:
   ```bash
   basilica up h200 --gpu-count 8
   ```

3. **Check Active Rentals**:
   ```bash
   basilica ps  # Note the rental UID
   ```

4. **Setup Miner on Rental**:
   ```bash
   # Copy setup script to rental
   basilica cp scripts/setup_miner_env.sh [rental-id]:/root/

   # Execute setup on rental
   basilica exec --target [rental-id] "bash /root/setup_miner_env.sh"

   # Copy environment file after editing
   basilica cp .env [rental-id]:/root/templar/

   # Install Templar
   basilica exec --target [rental-id] "git clone https://github.com/one-covenant/templar && cd templar && pip install -e ."
   ```

5. **Launch Miner on Rental**:
   ```bash
   basilica exec --target [rental-id] "cd /root/templar && python neurons/miner.py --wallet.name default --wallet.hotkey miner --netuid 3 --device cuda"
   ```

6. **Monitor and Manage**:
   ```bash
   # Copy logs back
   basilica cp [rental-id]:/root/templar/logs/ ./local-logs/

   # Terminate when done
   basilica down [rental-id]
   ```

**Cost Optimization Tips**:
- Monitor Basilica pricing fluctuations with `basilica price`
- Use `basilica ps` to track runtime and costs
- Set up automated cost alerts
- Test on smaller GPU counts first before scaling to 8x H200

**Integration Note**: When using both skills together, Claude can automatically coordinate between Templar mining operations and Basilica GPU rentals. Simply ask: "Help me set up a Templar miner on rented Basilica GPUs."

### 2. PERFORMANCE OPTIMIZATION

**Memory Optimization Tactics**:

1. **Parameter Offloading** (neurons/miner.py:493-496):
   - Offload model parameters to CPU before inner_steps
   - Restore to GPU after training
   - Reduces peak GPU memory by ~20%

2. **Error Feedback Management**:
   - Keep error_feedback on GPU during training
   - Move to pinned CPU memory after gradient computation
   - Prefetch back before next window

3. **Mixed Precision**:
   - Use bfloat16 for better stability than fp16
   - Configure in hparams.json: `"mixed_precision_param": "bfloat16"`

**Gradient Quality Optimization**:

1. **Error Feedback Momentum** (Critical for 64x compression):
   - Preserves gradient information across compressions
   - Configured via `momentum_decay` (default: 0.95)
   - Reference: src/tplr/neurons.py:47-222

2. **Compression Hyperparameters**:
   ```json
   {
     "topk_compression": 64,      // Top 64 coefficients
     "target_chunk": 64,          // DCT chunk size
     "momentum_decay": 0.95       // Momentum factor
   }
   ```

3. **Data Assignment Synchronization**:
   - Ensure deterministic UID-based seeding matches validators
   - Verify sample_digest in gradient metadata
   - Rotates every 455 outer steps

**Communication Optimization**:

1. **Peer Selection Strategy**:
   ```json
   {
     "gather_peer_count": 20,     // Peers to gather from
     "gather_share": 0.75,        // 75% success target
     "gather_top_ratio": 2.0      // Bias toward high-stake peers
   }
   ```

2. **Async Transfers**:
   - Use non_blocking=True for GPU↔CPU transfers
   - Pinned memory for host buffers
   - Batch synchronization at end

### 3. MONITORING & METRICS

**Key Metrics to Track**:

**WandB Dashboard** (https://wandb.ai/tplr/templar):
- `miner/window_entry_loss`: Loss at window start
- `miner/tokens_per_sec`: Training throughput
- `miner/gather_success_rate`: % of peers responded
- `miner/gpu_memory_allocated`: VRAM usage
- `miner/global_grad_norm`: Gradient magnitude

**Grafana Dashboard** (https://grafana.tplr.ai):
- View all miner scores and weights
- Compare performance vs other miners
- Track synchronization status

**Critical Alert Thresholds**:
- Gather Success Rate < 50%: Poor peer connectivity
- Sync Score < 0.5: Falling behind validators
- Gradient Score Negative: Producing harmful gradients
- GPU Memory > 95%: Risk of OOM crashes
- Window Time > 30 min: Too slow, missing windows

### 4. UNDERSTANDING THE INCENTIVE MECHANISM

For complete details, see `references/incentive_mechanism.md`.

**Core Scoring Formula**:
```
s_i = L_before - L_after
```

Where validators:
1. Compute loss BEFORE applying miner's gradient
2. Apply miner's gradient to model
3. Compute loss AFTER application
4. Score = improvement (positive is good)

**Moving Average**:
```
s̄_i = α * s_i + (1 - α) * s̄_i  (α = 0.05)
```

**Penalties**:
- Missing Gradient: -99.0 score, 25% slash per window
- Consecutive Negatives: >3 → exclusion from peer selection
- Desynchronization: Exponential penalty if >5 steps behind
- Index Overlap: >40% with others → plagiarism flag

**Target Metrics for Top Miners**:
- Gather Success Rate: >75% (elite: >85%)
- Sync Score: >0.9 (stay within 1 step)
- Gradient Score: Consistently positive (>0.01 MA)
- Tokens/sec: >1000 for 70B on 4x H100
- Uptime: >99%

### 5. TROUBLESHOOTING COMMON ISSUES

**Issue: Missing Gradients**

Symptoms:
- Penalty score of -99.0
- Validators report gradient not found

Solutions:
1. Check R2 bucket write access:
   ```bash
   aws s3 ls s3://$R2_GRADIENTS_BUCKET_NAME \
     --endpoint-url https://$R2_GRADIENTS_ACCOUNT_ID.r2.cloudflarestorage.com
   ```
2. Verify miner logs show "Successfully uploaded gradient"
3. Check network connectivity
4. Verify R2 credentials in environment

**Issue: Low Gather Success Rate**

Symptoms:
- gather_success_rate < 50%
- Timeouts in gather_with_reserve logs

Solutions:
1. Increase gather_peer_count in hparams.json
2. Check network bandwidth (need 1Gbps+)
3. Verify R2 read access
4. Consider adjusting peer_replacement_frequency

**Issue: Negative Gradient Scores**

Symptoms:
- gradient_score consistently negative
- Binary indicator shows -1
- Moving average declining

Solutions:
1. Reload checkpoint from highest-stake validator
2. Verify hparams.json matches current network version
3. Check data assignment synchronization (sample_digest)
4. Review error_feedback buffer state
5. Ensure no bugs in local modifications

**Issue: Desynchronization (Low Sync Score)**

Symptoms:
- sync_score < 0.5
- Validator reports "steps behind"
- Gradient fingerprint mismatches

Solutions:
1. Force checkpoint reload from top validator
2. Verify window synchronization with blockchain
3. Check for missed windows (uptime issues)
4. Ensure scheduler replay is correct

**Issue: OOM (Out of Memory) Errors**

Symptoms:
- CUDA out of memory errors
- Miner crashes during training
- GPU memory at 100%

Solutions:
1. Enable parameter offloading (already default)
2. Enable optimizer state offloading
3. Reduce micro_batch_size in hparams
4. Enable activation checkpointing
5. Use gradient checkpointing for large models

**Issue: Slow Training (Missing Windows)**

Symptoms:
- Window time > 25 minutes
- Missing window boundaries
- Low tokens/sec

Solutions:
1. Reduce batch_size or inner_steps
2. Enable torch.compile if not already enabled
3. Optimize DataLoader (increase num_workers)
4. Check for CPU bottlenecks (use profilers)
5. Verify network not rate-limiting R2 access

### 6. ADVANCED OPTIMIZATION STRATEGIES

**Strategy 1: Maximize Uptime**
- Use systemd service for auto-restart
- Implement health checks and alerting
- Monitor for blockchain forks/reorgs
- Ensure redundant network connectivity

**Strategy 2: Optimize Gradient Quality**
- Fine-tune error feedback momentum (0.9-0.99 range)
- Monitor validator evaluation patterns
- Adjust compression ratio if bandwidth allows
- Perfect data assignment synchronization

**Strategy 3: Maximize Peer Connectivity**
- High-bandwidth network connection (10Gbps ideal)
- Optimize R2 bucket configuration (Cloudflare zones)
- Monitor peer success rates per UID
- Maintain quality reserve peers

**Strategy 4: Stay Synchronized**
- Check sync_score every window
- Maintain <1 step lag from validators
- Quick checkpoint reloads when needed
- Monitor gradient fingerprint matching

**Strategy 5: Continuous Monitoring**
- Set up Grafana alerts for critical thresholds
- Monitor WandB dashboard in real-time
- Track InfluxDB metrics for trends
- Compare performance vs top 10 miners

**Strategy 6: Network Participation**
- Monitor Discord/GitHub for updates
- Track hparams.json version changes
- Quickly adopt new optimizations
- Report issues and contribute fixes

**Strategy 7: Hardware Investment**
- More GPUs → higher throughput → more windows completed
- Better network → higher gather success
- More RAM → larger batch sizes possible
- Faster storage → quicker checkpoint loads

### 7. HYPERPARAMETER TUNING

**Learning Rate Optimization**:
```json
{
  "adamw": {
    "learning_rate": 1.17e-4,    // Default starting point
    "warmup_steps": 1500,        // Gradual ramp-up
    "scheduler": {
      "t_max": 140000,           // Cosine annealing period
      "eta_min_factor": 0.1      // Min LR = 10% of max
    }
  }
}
```

**Momentum Decay Tuning**:
- Higher (0.95-0.99): Better gradient preservation across compression
- Lower (0.85-0.9): Faster adaptation to changes
- Default 0.95 works well for most cases

**Compression Ratio Tuning**:
- topk_compression: 64 (standard)
- Options: 32 (higher quality), 128 (more compression)
- Trade-off: Gradient quality vs bandwidth/storage

## Resources

### Scripts

- **check_miner_health.py**: Comprehensive health check for miner setup
  - Verifies environment variables
  - Checks GPU availability
  - Tests wallet registration
  - Validates R2 connectivity
  - Monitors disk space and memory

- **setup_miner_env.sh**: Generate template .env file with all required variables
  - Creates properly formatted environment file
  - Includes all Templar-specific variables
  - Documents required credentials

### References

- **templar_architecture.md**: Complete codebase structure and file locations
  - Repository organization
  - Core implementations (miner.py, validator.py, trainer.py)
  - Library components (neurons.py, compress.py, comms.py)
  - Configuration parameters

- **incentive_mechanism.md**: Detailed scoring and reward system
  - Loss-based gradient evaluation
  - Moving average smoothing
  - Penalty mechanisms
  - Synchronization scoring
  - OpenSkill rating system

- **bittensor_btcli.md**: Complete btcli command reference
  - Wallet creation and recovery
  - Hotkey management
  - Subnet registration
  - Balance checking
  - Identity management

## Task Execution Guidelines

When helping users with Templar mining:

1. **Always verify wallet setup first** - Use btcli commands from references
2. **Check environment variables** - Run check_miner_health.py script
3. **Review hardware specifications** - Ensure meets minimum requirements
4. **Monitor key metrics** - Track gather success, sync score, gradient score
5. **Optimize systematically** - Memory → gradient quality → communication
6. **Track performance** - Compare against top miners using Grafana
7. **Stay updated** - Monitor for codebase changes and hparam updates
8. **Iterate continuously** - Adjust based on validator feedback

## Code References

When referencing Templar code, use the pattern `file_path:line_number`:

- Miner main loop: `neurons/miner.py:354-942`
- Gradient preparation: `src/tplr/neurons.py:47-222`
- Compression pipeline: `src/tplr/compress.py:51-142`
- Validator evaluation: `neurons/validator.py:1644-1940`
- Error feedback: `src/tplr/neurons.py:152`
- Peer gathering: `neurons/miner.py:705-726`

See `references/templar_architecture.md` for complete file structure and line references.

## External Resources

- **GitHub**: https://github.com/one-covenant/templar
- **Miner Docs**: https://github.com/one-covenant/templar/blob/main/docs/miner.md
- **Validator Docs**: https://github.com/one-covenant/templar/blob/main/docs/validator.md
- **WandB Dashboard**: https://wandb.ai/tplr/templar
- **Grafana Metrics**: https://grafana.tplr.ai/d/ceia6bwlwn8qof/eval-metrics
- **DeepWiki**: https://deepwiki.com/one-covenant/templar
- **Bittensor Docs**: https://docs.learnbittensor.org

The goal is to help users become top-performing miners in the Templar network through systematic setup, optimization, monitoring, and iterative improvement.
