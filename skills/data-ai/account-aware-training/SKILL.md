---
name: account-aware-training
description: "Add account state (P&L, win rate, drawdown) to RL observations + drawdown penalty in rewards. Trigger when: (1) model needs account awareness, (2) training should penalize drawdowns, (3) upgrading obs_dim 5300→5600."
author: Claude Code
date: 2024-12-26
---

# Account-Aware RL Training (v2.4)

## Experiment Overview
| Item | Details |
|------|---------|
| **Date** | 2024-12-26 |
| **Goal** | Make RL model learn from account state (P&L, win rate, drawdown) |
| **Environment** | vectorized_env.py, inference_obs_builder.py, training notebook |
| **Status** | Success |

## Context

Prior to v2.4, the RL model was "blind" to account performance. It received:
- 53 features: price action, technicals, regime probabilities, calendar effects
- No information about cumulative P&L, win rate, or drawdown

**Problem**: The model could generate signals that were individually good but led to excessive drawdowns at the account level. It had no incentive to trade conservatively after losses.

**Solution**: Add 3 account-level features + drawdown penalty in rewards.

## Verified Workflow

### 1. Config Parameters (GPUEnvConfig)

```python
# In vectorized_env.py GPUEnvConfig dataclass (~line 405)
# Account-aware training (v2.4)
drawdown_penalty_threshold: float = 0.15  # Penalize when drawdown > 15%
drawdown_penalty_weight: float = 0.10     # Weight in reward function
```

### 2. Equity Tracking Tensors

```python
# In _init_state_tensors() after line 712
# Account-level equity tracking (v2.4)
self.initial_equity = torch.ones(self.n_envs, dtype=self.dtype, device=self.device)
self.peak_equity = torch.ones(self.n_envs, dtype=self.dtype, device=self.device)
self.current_equity = torch.ones(self.n_envs, dtype=self.dtype, device=self.device)
```

### 3. Reset Equity Tensors

```python
# In reset() after line 850
# Reset account-level equity tracking
self.initial_equity[env_ids] = 1.0
self.peak_equity[env_ids] = 1.0
self.current_equity[env_ids] = 1.0
```

### 4. Update Equity in step()

```python
# In step() after line 926
# Update account-level equity tracking (v2.4)
self.current_equity = self.initial_equity + self.total_pnl / (current_prices + 1e-8)
self.peak_equity = torch.maximum(self.peak_equity, self.current_equity)
```

### 5. Feature Count Update

```python
# In _calculate_obs_features() line 682
# Add account features
account = 3  # total_pnl_pct, rolling_win_rate, current_drawdown_pct
return base + technical + intraday + temporal + markov + extended + multi_window + account
# Result: 53 + 3 = 56 features
```

### 6. Account Features in Observations

```python
# In _get_observations() after line 1258, before sanitization

# === ACCOUNT-LEVEL FEATURES (3) - v2.4 ===

# Feature 1: Total P&L % (normalized to [-1, 1])
total_pnl_pct = self.total_pnl / (self.initial_equity + 1e-8)
total_pnl_pct_norm = torch.tanh(total_pnl_pct * 10)
obs[:, :, feat_idx] = total_pnl_pct_norm[env_ids].unsqueeze(1).expand(-1, self.config.window)
feat_idx += 1

# Feature 2: Rolling win rate (0.5 if no trades)
win_rate = torch.where(
    self.n_trades[env_ids] > 0,
    self.n_wins[env_ids].float() / self.n_trades[env_ids].float(),
    torch.full((n_envs,), 0.5, dtype=self.dtype, device=self.device)
)
obs[:, :, feat_idx] = win_rate.unsqueeze(1).expand(-1, self.config.window)
feat_idx += 1

# Feature 3: Current drawdown % [0, 1]
drawdown = (self.peak_equity[env_ids] - self.current_equity[env_ids]) / (self.peak_equity[env_ids] + 1e-8)
drawdown = torch.clamp(drawdown, 0.0, 1.0)
obs[:, :, feat_idx] = drawdown.unsqueeze(1).expand(-1, self.config.window)
feat_idx += 1
```

### 7. Drawdown Penalty in Rewards

```python
# In _calculate_rewards() after line 1618

# COMPONENT 7: Drawdown penalty (v2.4)
current_drawdown = (self.peak_equity - self.current_equity) / (self.peak_equity + 1e-8)
current_drawdown = torch.clamp(current_drawdown, 0.0, 1.0)

# Quadratic penalty when over threshold
drawdown_over_threshold = torch.clamp(current_drawdown - self.config.drawdown_penalty_threshold, min=0.0)
drawdown_penalty = -drawdown_over_threshold ** 2 * 10

# Add to reward combination:
reward = (
    self.config.direction_weight * direction_reward +
    self.config.magnitude_weight * magnitude_reward +
    self.config.pnl_weight * pnl_reward +
    self.config.stop_tp_weight * stop_tp_reward +
    self.config.exploration_weight * exploration_bonus +
    self.config.slippage_weight * slippage_penalty +
    self.config.drawdown_penalty_weight * drawdown_penalty  # NEW
) * risk_adjustment
```

### 8. Inference Observation Builder

```python
# In inference_obs_builder.py get_target_features_from_obs_dim()
if features == 56:
    return 56  # v2.4 with account awareness
elif features == 53:
    return 53  # v2.3
# ... legacy support

# In build_inference_observation() after line 624
# === ACCOUNT-LEVEL FEATURES (3) - v2.4 ===
# Use neutral defaults during inference
if target_features >= 56:
    obs[:, feat_idx] = 0.0   # total_pnl_pct (no prior trades)
    feat_idx += 1
    obs[:, feat_idx] = 0.5   # win_rate (neutral prior)
    feat_idx += 1
    obs[:, feat_idx] = 0.0   # drawdown (no drawdown)
    feat_idx += 1
```

## Failed Attempts (Critical)

| Attempt | Why it Failed | Lesson Learned |
|---------|---------------|----------------|
| Account features with raw P&L values | P&L scale varies by price level | Use P&L percentage normalized with tanh |
| Win rate = 0 when no trades | Invalid input during initial episodes | Default to 0.5 (neutral prior) |
| Peak equity never decreasing | Logical error in update | Use torch.maximum() to track high-water mark |
| Drawdown penalty linear | Too harsh at moderate levels | Quadratic scaling is gentler below threshold |
| Live inference with account state | Would need real account connection | Use neutral defaults (0, 0.5, 0) for inference |

## Final Parameters

```yaml
# GPUEnvConfig (v2.4)
n_features: 56  # Was 53 in v2.3
drawdown_penalty_threshold: 0.15  # 15% drawdown starts penalty
drawdown_penalty_weight: 0.10     # Moderate weight in reward

# Feature breakdown (56 total)
base_features: 7              # price action basics
technical_features: 4         # intraday technicals
temporal_features: 7          # calendar features
markov_features: 12           # 4-chain regime probabilities
extended_features: 14         # extended technicals
multi_window_features: 9      # 20/50/100 bar windows
account_features: 3           # P&L %, win rate, drawdown %

# obs_dim = n_features * window = 56 * 100 = 5600
```

## Key Insights

- **Breaking Change**: obs_dim 5300 → 5600 means v2.3 models CANNOT be used with v2.4 environments
- **Neutral Inference**: Live trading uses neutral defaults (0, 0.5, 0) since account state isn't tracked per-prediction
- **Quadratic Penalty**: The `** 2` makes penalty gentle at 16% drawdown but harsh at 25%+
- **Normalized P&L**: `tanh(pnl * 10)` keeps values in [-1, 1] even for large P&L swings
- **0.5 Win Rate Prior**: Prevents model confusion during initial trades with no history

## Model Behavior Expected

With account awareness, the model should learn:
1. **Reduce position sizing after losses** (sees drawdown feature)
2. **Be more selective after poor win rate** (sees win rate feature)
3. **Avoid compounding losses** (drawdown penalty kicks in at 15%)
4. **Trade more aggressively when profitable** (sees positive P&L)

## References
- `alpaca_trading/gpu/vectorized_env.py`: Lines 405 (config), 712 (tensors), 850 (reset), 926 (step), 1258 (obs)
- `alpaca_trading/gpu/inference_obs_builder.py`: Lines 61-108 (feature detection), 624+ (account features)
- `notebooks/VSCode_Colab_Training_NATIVE.ipynb`: Training notebook with v2.4 settings
