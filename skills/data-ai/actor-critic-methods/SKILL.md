---
name: actor-critic-methods
description: Master A2C, A3C, SAC, TD3 - actor-critic methods for continuous control
---

# Actor-Critic Methods

## When to Use This Skill

Invoke this skill when you encounter:

- **Algorithm Selection**: "Should I use actor-critic for my continuous control problem?"
- **SAC Implementation**: User implementing SAC and needs guidance on entropy coefficient tuning
- **TD3 Confusion**: "Why does TD3 have twin critics and delayed updates?"
- **Training Instability**: "My actor-critic diverges. How do I stabilize it?"
- **A2C/A3C Questions**: "What's the difference between A2C and A3C?"
- **Continuous Control**: User has continuous action space and needs appropriate algorithm
- **Critic Issues**: "My critic loss isn't decreasing" or "Advantage estimates are wrong"
- **SAC vs TD3**: "Which algorithm should I use for my problem?"
- **Entropy Tuning**: "How do I set the entropy coefficient α in SAC?"
- **Policy Gradient Variance**: "My policy gradients are too noisy. How do I reduce variance?"
- **Implementation Bugs**: Critic divergence, actor-critic synchronization, target network staleness
- **Continuous Action Handling**: Tanh squashing, log determinant Jacobian, action scaling

**This skill provides practical guidance for continuous action space RL using actor-critic methods.**

Do NOT use this skill for:

- Discrete action spaces (route to value-based-methods for Q-learning/DQN)
- Pure policy gradient without value baseline (route to policy-gradient-methods)
- Model-based RL (route to model-based-rl)
- Offline RL (route to offline-rl-methods)
- Theory foundations (route to rl-foundations)

---

## Core Principle

**Actor-critic methods achieve the best of both worlds: a policy (actor) for action selection guided by a value function (critic) for stable learning. They dominate continuous control because they're designed for infinite action spaces and provide sample-efficient learning through variance reduction.**

Key insight: Continuous control has infinite actions to explore. Value-based methods (compare all action values) are infeasible. Policy gradient methods (directly optimize policy) have high variance. **Actor-critic solves this: policy directly outputs action distribution (actor), value function provides stable baseline (critic) to reduce variance.**

Use them for:

- Continuous control (robot arms, locomotion, vehicle control)
- High-dimensional action spaces (continuous angles, forces, velocities)
- Sample-efficient learning from sparse experiences
- Problems requiring exploration via stochastic policies
- Continuous state/action MDPs (deterministic or stochastic environments)

**Do not use for**:

- Discrete small action spaces (too slow compared to DQN)
- Imitation learning focused on behavior cloning (use behavior cloning directly)
- Very high-dimensional continuous spaces without careful design (curse of dimensionality)
- Planning-focused problems (route to model-based methods)

---

## Part 1: Actor-Critic Foundations

### From Policy Gradient to Actor-Critic

You understand policy gradient from policy-gradient-methods. Actor-critic extends it with **a value baseline to reduce variance**.

**Pure Policy Gradient (REINFORCE)**:

```
∇J = E_τ[∇log π(a|s) * G_t]
```

**Problem**: G_t (cumulative future reward) has high variance. All rollouts pulled toward average. Noisy gradients = slow learning.

**Actor-Critic Solution**:

```
∇J = E_τ[∇log π(a|s) * (G_t - V(s))]
       = E_τ[∇log π(a|s) * A(s,a)]

where:
- Actor: π(a|s) = policy (action distribution)
- Critic: V(s) = value function (baseline)
- Advantage: A(s,a) = G_t - V(s) = "how much better than average"
```

**Why baseline helps**:

```
Without baseline: policy gradients = [+10, -2, +5, -3, -1] (noisy, high variance)
With baseline (subtract mean=2): [+8, -4, +3, -5, -3] (same direction, but cleaner relative to baseline)

Result: Gradient points in same direction (increase high G, decrease low G) but with MUCH lower variance.
This reduces sample complexity significantly.
```

### Advantage Estimation

The core of actor-critic is **accurate advantage estimation**:

```
A(s,a) = Q(s,a) - V(s)
       = E[r + γV(s')] - V(s)
       = E[r + γV(s') - V(s)]
```

**Key insight**: Advantage = "by how much does taking action a in state s beat the average for this state?"

**Three ways to estimate advantage**:

**1. Monte Carlo (full return)**:

```python
G_t = r_t + γr_{t+1} + γ²r_{t+2} + ... (full rollout)
A(s,a) = G_t - V(s)
```

- Unbiased but high variance
- Requires complete episodes or long horizons

**2. TD(0) (one-step bootstrap)**:

```python
A(s,a) = r + γV(s') - V(s)
```

- Low variance but biased (depends on critic accuracy)
- One-step lookahead only
- If V(s') is wrong, advantage is wrong

**3. GAE - Generalized Advantage Estimation** (best practice):

```python
A_t = δ_t + (γλ)δ_{t+1} + (γλ)²δ_{t+2} + ...
δ_t = r_t + γV(s_{t+1}) - V(s_t)  [TD error]

λ ∈ [0,1] trades off bias-variance:
- λ=0: pure TD(0) (low variance, high bias)
- λ=1: pure MC (high variance, low bias)
- λ=0.95: sweet spot (good tradeoff)
```

**Why GAE**: Exponentially decaying trace over multiple steps. Reduces variance without full MC, reduces bias without pure TD.

---

### Actor-Critic Pitfall #1: Critic Not Learning Properly

**Scenario**: User trains actor-critic but critic loss doesn't decrease. Actor improves, but value function plateaus. Agent can't use accurate advantage estimates.

**Problem**:

```python
# WRONG - critic loss computed incorrectly
critic_loss = mean((V(s) - G_t)^2)  # Wrong target!
critic_loss.backward()
```

**The bug**: Critic should learn Bellman equation:

```
V(s) = E[r + γV(s')]
```

If you compute target as G_t directly, you're using Monte Carlo returns (too noisy). If you use r + γV(s'), you're bootstrapping properly.

**Correct approach**:

```python
# RIGHT - Bellman bootstrap target
V_target = r + gamma * V(s').detach()  # Detach next state value!
critic_loss = mean((V(s) - V_target)^2)
```

**Why detach() matters**: If you don't detach V(s'), gradient flows backward through value function, creating a moving target problem.

**Red Flag**: If critic loss doesn't decrease while actor loss decreases, critic isn't learning Bellman equation. Check:

1. Target computation (should be r + γV(s'), not G_t alone)
2. Detach next state value
3. Critic network is separate from actor
4. Different learning rates (critic typically higher than actor)

---

### Critic as Baseline vs Critic as Q-Function

**Important distinction**:

**A2C uses critic as baseline**:

```
V(s) = value of being in state s
A(s,a) = r + γV(s') - V(s)  [TD advantage]
Policy loss = -∇log π(a|s) * A(s,a)
```

**SAC/TD3 use critic as Q-function**:

```
Q(s,a) = expected return from taking action a in s
A(s,a) = Q(s,a) - V(s)
Policy loss = ∇log π(a|s) * Q(s,a) [deterministic policy gradient]
```

**Why the difference**: A2C updates actor and critic together (on-policy). SAC/TD3 decouple them (off-policy):

- Actor never sees the replay buffer
- Critic learns Q from replay buffer
- Actor uses critic's Q estimate (always lagging slightly)

---

## Part 2: A2C - Advantage Actor-Critic

### A2C Architecture

A2C = on-policy advantage actor-critic. Actor and critic train simultaneously with synchronized rollouts:

```
┌─────────────────────────────────────────┐
│  Environment                            │
└────────────┬────────────────────────────┘
             │ states, rewards
             ▼
┌─────────────────────────────────────────┐
│  Actor π(a|s)     Critic V(s)          │
│  Policy network   Value network         │
│  Outputs: action  Outputs: value        │
└────────┬──────────────────┬─────────────┘
         │                  │
         └──────┬───────────┘
                │
         ┌──────▼───────────┐
         │ Advantage        │
         │ A(s,a) = r+γV(s')-V(s)
         └────────┬─────────┘
                  │
         ┌────────▼────────────┐
         │ Actor Loss:         │
         │ -log π(a|s) * A(s,a)
         │                     │
         │ Critic Loss:        │
         │ (V(s) - target)²    │
         └─────────────────────┘
```

### A2C Training Loop

```python
for episode in range(num_episodes):
    states, actions, rewards, values = [], [], [], []

    state = env.reset()
    for t in range(horizon):
        # Actor samples action from policy
        action = actor(state)

        # Step environment
        next_state, reward = env.step(action)

        # Get value estimate (baseline)
        value = critic(state)

        # Store for advantage computation
        states.append(state)
        actions.append(action)
        rewards.append(reward)
        values.append(value)

        state = next_state

    # Advantage estimation (GAE)
    advantages = compute_gae(rewards, values, next_value, gamma, lambda)

    # Actor loss (policy gradient with baseline)
    actor_loss = -log_prob(actions, actor(states)) * advantages
    actor.update(actor_loss)

    # Critic loss (value function learning)
    critic_targets = rewards + gamma * values[1:] + gamma * critic(next_state)
    critic_loss = (critic(states) - critic_targets)^2
    critic.update(critic_loss)
```

### A2C vs A3C

**A2C**: Synchronous - all parallel workers update at same time (cleaner, deterministic)

```
Worker 1  ────┐
Worker 2  ────┼──► Global Model Update ──► All workers receive updated weights
Worker 3  ────┤
Worker N  ────┘
Wait for all workers before next update
```

**A3C**: Asynchronous - workers update whenever they finish (faster wall clock time, messier)

```
Worker 1  ──► Update (1) ──► Continue
Worker 2  ──────► Update (2) ──────► Continue
Worker 3  ──────────► Update (3) ──────► Continue
No synchronization barrier (race conditions possible)
```

**In practice**: A2C is preferred. A3C was important historically (enables multi-GPU training without synchronization) but A2C is cleaner.

---

## Part 3: SAC - Soft Actor-Critic

### SAC Overview

SAC = Soft Actor-Critic. The current SOTA (state-of-the-art) for continuous control. Three key innovations:

1. **Entropy regularization**: Add H(π(·|s)) to objective (maximize entropy + reward)
2. **Auto-tuning entropy coefficient**: Learn α automatically (no manual tuning!)
3. **Off-policy learning**: Learn from replay buffer (sample efficient)

### SAC's Objective Function

Standard policy gradient maximizes:

```
J(π) = E[G_t]
```

SAC maximizes:

```
J(π) = E[G_t + α H(π(·|s))]
     = E[G_t] + α E[H(π(·|s))]
```

**Where**:

- G_t = cumulative reward
- H(π(·|s)) = policy entropy (randomness)
- α = entropy coefficient (how much we value exploration)

**Why entropy**: Exploratory policies (high entropy) discover better strategies. Adding entropy to objective = agent explores automatically.

### SAC Components

```
┌─────────────────────────────────────┐
│  Replay Buffer (off-policy data)    │
└────────────┬────────────────────────┘
             │ sample batch
             ▼
    ┌────────────────────────┐
    │  Actor Network         │
    │  π(a|s) = μ(s) + σ(s) │  (Gaussian policy)
    │  Outputs: mean, std    │
    └────────────────────────┘
             │
             ▼
    ┌────────────────────────┐
    │  Two Critic Networks   │
    │  Q1(s,a), Q2(s,a)     │
    │  Learn Q-values        │
    └────────────────────────┘
             │
             ▼
    ┌────────────────────────┐
    │  Target Networks       │
    │  Q_target1, Q_target2  │
    │  (updated every N)     │
    └────────────────────────┘
             │
             ▼
    ┌────────────────────────┐
    │  Entropy Coefficient   │
    │  α (learned!)          │
    └────────────────────────┘
```

### SAC Training Algorithm

```python
# Initialize
actor = ActorNetwork()
critic1, critic2 = CriticNetwork(), CriticNetwork()
target_critic1, target_critic2 = copy(critic1), copy(critic2)
entropy_alpha = 1.0  # Learned!
target_entropy = -action_dim  # Target entropy (usually -action_dim)

for step in range(num_steps):
    # 1. Collect data (could be online or from buffer)
    state = env.reset() if step % 1000 == 0 else next_state
    action = actor.sample(state)  # π(a|s)
    next_state, reward = env.step(action)
    replay_buffer.add(state, action, reward, next_state, done)

    # 2. Sample batch from replay buffer
    batch = replay_buffer.sample(batch_size=256)
    states, actions, rewards, next_states, dones = batch

    # 3. Critic update (Q-function learning)
    # Compute target Q value using entropy-regularized objective
    next_actions = actor.sample(next_states)
    next_log_probs = actor.log_prob(next_actions, next_states)

    # Use BOTH target critics, take minimum (overestimation prevention)
    Q_target1 = target_critic1(next_states, next_actions)
    Q_target2 = target_critic2(next_states, next_actions)
    Q_target = min(Q_target1, Q_target2)

    # Entropy-regularized target
    y = reward + γ(1 - done) * (Q_target - α * next_log_probs)

    # Update both critics
    critic1_loss = MSE(critic1(states, actions), y)
    critic1.update(critic1_loss)

    critic2_loss = MSE(critic2(states, actions), y)
    critic2.update(critic2_loss)

    # 4. Actor update (policy gradient with entropy)
    # Reparameterization trick: sample actions, compute log probs
    sampled_actions = actor.sample(states)
    sampled_log_probs = actor.log_prob(sampled_actions, states)

    # Actor maximizes Q - α*log_prob (entropy regularization)
    Q1_sampled = critic1(states, sampled_actions)
    Q2_sampled = critic2(states, sampled_actions)
    Q_sampled = min(Q1_sampled, Q2_sampled)

    actor_loss = -E[Q_sampled - α * sampled_log_probs]
    actor.update(actor_loss)

    # 5. Entropy coefficient auto-tuning (SAC's KEY INNOVATION)
    # Learn α to maintain target entropy
    entropy_loss = -α * (sampled_log_probs + target_entropy)
    alpha.update(entropy_loss)

    # 6. Soft update target networks (every N steps)
    if step % update_frequency == 0:
        target_critic1 = τ * critic1 + (1-τ) * target_critic1
        target_critic2 = τ * critic2 + (1-τ) * target_critic2
```

### SAC Pitfall #1: Manual Entropy Coefficient

**Scenario**: User implements SAC but manually sets α=0.2 and training diverges. Agent explores randomly and never improves.

**Problem**: SAC's entire design is that α is **learned automatically**. Setting it manually defeats the purpose.

```python
# WRONG - treating α as fixed hyperparameter
alpha = 0.2  # Fixed!
loss = Q_target - 0.2 * log_prob  # Same penalty regardless of entropy

# Result: If entropy naturally low, penalty still high → policy forced random
#         If entropy naturally high, penalty too weak → insufficient exploration
```

**Correct approach**:

```python
# RIGHT - α is learned via entropy constraint
target_entropy = -action_dim  # For Gaussian: typically -action_dim

# Optimize α to maintain target entropy
entropy_loss = -α * (sampled_log_probs.detach() + target_entropy)
alpha_optimizer.zero_grad()
entropy_loss.backward()
alpha_optimizer.step()

# α adjusts automatically:
# - If entropy too high: α increases (more penalty) → policy becomes more deterministic
# - If entropy too low: α decreases (less penalty) → policy explores more
```

**Red Flag**: If SAC agent explores randomly without improving, check:

1. Is α being optimized? (not fixed value)
2. Is target entropy set correctly? (usually -action_dim)
3. Is log_prob computed with squashed action (after tanh)?

---

### SAC Pitfall #2: Tanh Squashing and Log Probability

**Scenario**: User implements SAC with Gaussian policy but uses policy directly. Log probabilities are computed wrong. Training is unstable.

**Problem**: SAC uses tanh squashing to bound actions:

```
Raw action from network: μ(s) + σ(s)*ε, ε~N(0,1)  → unbounded
Tanh squashed: a = tanh(raw_action)  → bounded in [-1,1]
```

But policy probability must account for this transformation:

```
π(a|s) ≠ N(μ(s), σ²(s))  [Wrong! Ignores tanh]
π(a|s) = |det(∂a/∂raw_action)|^(-1) * N(μ(s), σ²(s))
       = (1 - a²)^2 * N(μ(s), σ²(s))  [Right! Jacobian correction]

log π(a|s) = log N(μ(s), σ²(s)) - 2*log(1 - a²)
```

**The bug**: Computing log_prob without Jacobian correction:

```python
# WRONG
log_prob = normal.log_prob(raw_action) - log(1 + exp(-2*x))
# (standard normal log prob, ignores squashing)

# RIGHT
log_prob = normal.log_prob(raw_action) - log(1 + exp(-2*x))
log_prob = log_prob - 2 * (log(2) - x - softplus(-2*x))  # Add Jacobian term
```

Or simpler:

```python
# PyTorch way
dist = Normal(mu, sigma)
raw_action = dist.rsample()  # Reparameterized sample
action = torch.tanh(raw_action)
log_prob = dist.log_prob(raw_action) - torch.log(1 - action.pow(2) + 1e-6).sum(-1)
```

**Red Flag**: If SAC policy doesn't learn despite updates, check:

1. Are actions being squashed (tanh)?
2. Is log_prob computed with tanh Jacobian term?
3. Is squashing adjustment in entropy coefficient update?

---

### SAC Pitfall #3: Two Critics and Target Networks

**Scenario**: User implements SAC with one critic and gets unstable learning. "I thought SAC just needed entropy regularization?"

**Problem**: SAC uses TWO critics because of Q-function overestimation:

```
Single critic Q(s,a):
- Targets computed as: y = r + γQ_target(s', a')
- Q_target is function of Q (updated less frequently)
- In continuous space, selecting actions via max isn't feasible
- Next action sampled from π (deterministic max removed)
- But Q-values can still overestimate (stochastic environment noise)

Two critics (clipped double Q-learning):
- Use both Q1 and Q2, take minimum: Q_target = min(Q1_target, Q2_target)
- Prevents overestimation (conservative estimate)
- Both updated simultaneously
- Asymmetric: both learn, but target uses minimum
```

**Correct implementation**:

```python
# WRONG - one critic
target = reward + gamma * critic_target(next_state, next_action)

# RIGHT - two critics with min
Q1_target = critic1_target(next_state, next_action)
Q2_target = critic2_target(next_state, next_action)
target = reward + gamma * min(Q1_target, Q2_target)

# Both critics learn
critic1_loss = MSE(critic1(state, action), target)
critic2_loss = MSE(critic2(state, action), target)

# But actor only uses critic1 (or min of both)
Q_current = min(critic1(state, sampled_action), critic2(state, sampled_action))
actor_loss = -(Q_current - alpha * log_prob)
```

**Red Flag**: If SAC diverges, check:

1. Are there two Q-networks?
2. Does target use min(Q1, Q2)?
3. Are target networks updated (soft or hard)?

---

## Part 4: TD3 - Twin Delayed DDPG

### Why TD3 Exists

TD3 = Twin Delayed DDPG. It addresses SAC's cost (two networks, more computation) with deterministic policy gradient (simpler).

**DDPG** (older): Deterministic policy, single Q-network, no entropy. Fast but unstable.

**TD3** (newer): Three tricks to stabilize DDPG:

1. **Twin critics**: Two Q-networks (clipped double Q-learning)
2. **Delayed actor updates**: Update actor every d steps (not every step)
3. **Target policy smoothing**: Add noise to target action before Q evaluation

### TD3 Architecture

```
┌──────────────────────────────────┐
│  Replay Buffer                   │
└────────────┬─────────────────────┘
             │
             ▼
    ┌───────────────────────┐
    │  Actor μ(s)           │
    │  Deterministic policy │
    │  Outputs: action      │
    └───────────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │  Q1(s,a), Q2(s,a)      │
    │  Two Q-networks        │
    │  (Triple: original+2)  │
    └─────────────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │  Delayed Actor Update   │
    │  (every d steps)        │
    └─────────────────────────┘
```

### TD3 Training Algorithm

```python
for step in range(num_steps):
    # 1. Collect data
    action = actor(state) + exploration_noise
    next_state, reward = env.step(action)
    replay_buffer.add(state, action, reward, next_state, done)

    if step < min_steps_before_training:
        continue

    batch = replay_buffer.sample(batch_size)
    states, actions, rewards, next_states, dones = batch

    # 2. Critic update (BOTH Q-networks)
    # Trick #3: Target policy smoothing
    next_actions = actor_target(next_states)
    noise = torch.randn_like(next_actions) * target_noise
    noise = torch.clamp(noise, -noise_clip, noise_clip)
    next_actions = torch.clamp(next_actions + noise, -1, 1)  # Add noise, clip

    # Clipped double Q-learning: use minimum
    Q1_target = critic1_target(next_states, next_actions)
    Q2_target = critic2_target(next_states, next_actions)
    Q_target = torch.min(Q1_target, Q2_target)

    y = rewards + gamma * (1 - dones) * Q_target

    # Update both critics
    critic1_loss = MSE(critic1(states, actions), y)
    critic1_optimizer.zero_grad()
    critic1_loss.backward()
    critic1_optimizer.step()

    critic2_loss = MSE(critic2(states, actions), y)
    critic2_optimizer.zero_grad()
    critic2_loss.backward()
    critic2_optimizer.step()

    # 3. Delayed actor update (Trick #2)
    if step % policy_delay == 0:
        # Deterministic policy gradient
        Q1_current = critic1(states, actor(states))
        actor_loss = -Q1_current.mean()

        actor_optimizer.zero_grad()
        actor_loss.backward()
        actor_optimizer.step()

        # Soft update target networks
        for param, target_param in zip(critic1.parameters(), critic1_target.parameters()):
            target_param.data.copy_(tau * param.data + (1-tau) * target_param.data)
        for param, target_param in zip(critic2.parameters(), critic2_target.parameters()):
            target_param.data.copy_(tau * param.data + (1-tau) * target_param.data)
        for param, target_param in zip(actor.parameters(), actor_target.parameters()):
            target_param.data.copy_(tau * param.data + (1-tau) * target_param.data)
```

### TD3 Pitfall #1: Missing Target Policy Smoothing

**Scenario**: User implements TD3 with twin critics and delayed updates but training still unstable. "I have two critics, why isn't it stable?"

**Problem**: Target policy smoothing is critical. Without it:

```
Next action = deterministic μ_target(s')  [exact, no exploration noise]

If Q-networks overestimate for certain actions:
- Target policy always selects that exact action
- Q-target biased high for that action
- Feedback loop: overestimation → more value → policy selects it more → more overestimation
```

With smoothing:

```
Next action = μ_target(s') + ε_smoothing
- Adds small random noise to target action
- Prevents exploitation of Q-estimation errors
- Breaks feedback loop by adding randomness to target action

Important: Noise is added at TARGET action, not current action!
- Current: exploration_noise (for exploration during collection)
- Target: target_noise (for stability, noise clip small)
```

**Correct implementation**:

```python
# Trick #3: Target policy smoothing
next_actions = actor_target(next_states)
noise = torch.randn_like(next_actions) * target_policy_noise
noise = torch.clamp(noise, -noise_clip, noise_clip)
next_actions = torch.clamp(next_actions + noise, -1, 1)

# Then use these noisy actions for Q-target
Q_target = min(Q1_target(next_states, next_actions),
               Q2_target(next_states, next_actions))
```

**Red Flag**: If TD3 diverges despite two critics, check:

1. Is noise added to target action (not just actor output)?
2. Is noise clipped (noise_clip prevents too much noise)?
3. Are critic targets using smoothed actions?

---

### TD3 Pitfall #2: Delayed Actor Updates

**Scenario**: User implements TD3 with target policy smoothing and twin critics, but updates actor every step. "Do I really need delayed updates?"

**Problem**: Policy updates change actor, which changes actions chosen. If you update actor every step while critics are learning:

```
Step 1: Actor outputs a1, Q(s,a1) = 5, Actor updated
Step 2: Actor outputs a2, Q(s,a2) = 3, Actor wants to stay at a1
Step 3: Critics haven't converged, oscillate between a1 and a2
Result: Actor chases moving target, training unstable
```

With delayed updates:

```
Steps 1-4: Update critics only, let them converge
Step 5: Update actor (once per policy_delay=5)
Steps 6-9: Update critics only
Step 10: Update actor again
Result: Critic stabilizes before actor changes, smoother learning
```

**Typical settings**:

```python
policy_delay = 2  # Update actor every 2 critic updates
# or
policy_delay = 5  # More conservative, every 5 critic updates
```

**Correct implementation**:

```python
if step % policy_delay == 0:  # Only sometimes!
    actor_loss = -critic1(state, actor(state)).mean()
    actor_optimizer.zero_grad()
    actor_loss.backward()
    actor_optimizer.step()

    # Update targets on same schedule
    soft_update(critic1_target, critic1)
    soft_update(critic2_target, critic2)
    soft_update(actor_target, actor)
```

**Red Flag**: If TD3 training unstable, check:

1. Is actor updated only every policy_delay steps?
2. Are target networks updated on same schedule (policy_delay)?
3. Policy_delay typically 2-5

---

### SAC vs TD3 Decision Framework

**Both are SOTA for continuous control. How to choose?**

| Aspect | SAC | TD3 |
|--------|-----|-----|
| **Policy Type** | Stochastic (Gaussian) | Deterministic |
| **Exploration** | Entropy maximization (automatic) | Target policy smoothing |
| **Sample Efficiency** | High (two critics) | High (two critics) |
| **Stability** | Very stable (entropy helps) | Stable (three tricks) |
| **Computation** | Higher (entropy tuning) | Slightly lower |
| **Manual Tuning** | Minimal (α auto-tuned) | Moderate (policy_delay, noise) |
| **When to Use** | Default choice, off-policy | When deterministic better, simpler noise |

**Decision tree**:

1. **Do you prefer stochastic or deterministic policy?**
   - Stochastic (multiple possible actions per state) → SAC
   - Deterministic (one action per state) → TD3

2. **Sample efficiency critical?**
   - Yes, limited data → Both good, slight edge SAC
   - No, lots of data → Either works

3. **How much tuning tolerance?**
   - Want minimal tuning → SAC (α auto-tuned)
   - Don't mind tuning policy_delay, noise → TD3 (simpler conceptually)

4. **Exploration challenges?**
   - Complex exploration (entropy helps) → SAC
   - Simple exploration (policy smoothing enough) → TD3

**Practical recommendation**: Start with SAC. It's more robust (entropy auto-tuning). Switch to TD3 only if you:

- Know you want deterministic policy
- Have tuning expertise for policy_delay
- Need slightly faster computation

---

## Part 5: Continuous Action Handling

### Gaussian Policy Representation

Actor outputs **mean and standard deviation**:

```python
raw_output = actor_network(state)
mu = raw_output[:, :action_dim]
log_std = raw_output[:, action_dim:]
log_std = torch.clamp(log_std, min=log_std_min, max=log_std_max)
std = log_std.exp()

dist = Normal(mu, std)
raw_action = dist.rsample()  # Reparameterized sample
```

**Why log(std)?**: Parameterize log scale instead of scale directly.

- Numerical stability (log prevents underflow)
- Gradient flow smoother
- Prevents std from becoming negative

**Why clamp log_std?**: Prevents std from becoming too small or large.

- Too small: policy becomes deterministic, no exploration
- Too large: policy becomes random, no learning

Typical ranges:

```python
log_std_min = -20  # std >= exp(-20) ≈ 2e-9 (small exploration)
log_std_max = 2    # std <= exp(2) ≈ 7.4 (max randomness)
```

### Continuous Action Squashing (Tanh)

Raw network output unbounded. Use tanh to bound to [-1,1]:

```python
# After sampling from policy
action = torch.tanh(raw_action)
# action now in [-1, 1]

# Scale to environment action range [low, high]
action_scaled = (high - low) / 2 * action + (high + low) / 2
```

**Pitfall**: Log probability must account for squashing (already covered in SAC section).

### Exploration Noise in Continuous Control

**Off-policy methods** (SAC, TD3) need exploration during data collection:

**Method 1: Action space noise** (simpler):

```python
action = actor(state) + noise
noise = torch.randn_like(action) * exploration_std
action = torch.clamp(action, -1, 1)  # Ensure in bounds
```

**Method 2: Parameter noise** (more complex):

```
Add noise to actor network weights periodically
Action = actor_with_noisy_weights(state)
Results in correlated action noise across timesteps (more natural exploration)
```

**Typical settings**:

```python
# For SAC: exploration_std = 0.1 * max_action
# For TD3: exploration_std starts high, decays over time
```

---

## Part 6: Common Bugs and Debugging

### Bug #1: Critic Divergence

**Symptom**: Critic loss explodes, V(s) becomes huge (1e6+), agent breaks.

**Causes**:

1. **Wrong target computation**: Using wrong Bellman target
2. **No gradient clipping**: Gradients unstable
3. **Learning rate too high**: Critic overshoots
4. **Value targets too large**: Reward scale not normalized

**Diagnosis**:

```python
# Check target computation
print("Reward range:", rewards.min(), rewards.max())
print("V(s) range:", v_current.min(), v_current.max())
print("Target range:", v_target.min(), v_target.max())

# Plot value function over time
plt.plot(v_values_history)  # Should slowly increase, not explode

# Check critic loss
print("Critic loss:", critic_loss.item())  # Should decrease, not diverge
```

**Fix**:

```python
# 1. Reward normalization
rewards = (rewards - rewards.mean()) / (rewards.std() + 1e-8)

# 2. Gradient clipping
torch.nn.utils.clip_grad_norm_(critic.parameters(), max_norm=1.0)

# 3. Lower learning rate
critic_lr = 1e-4  # Instead of 1e-3

# 4. Value function target clipping (optional)
v_target = torch.clamp(v_target, -100, 100)
```

---

### Bug #2: Actor Not Learning (Constant Policy)

**Symptom**: Actor loss decreases but policy doesn't change. Same action sampled repeatedly. No improvement in return.

**Causes**:

1. **Policy output not properly parameterized**: Mean/std wrong
2. **Critic signal dead**: Q-values all same, no gradient
3. **Learning rate too low**: Actor updates too small
4. **Advantage always zero**: Critic perfect (impossible) or wrong

**Diagnosis**:

```python
# Check policy output distribution
actions = [actor.sample(state) for _ in range(1000)]
print("Action std:", np.std(actions))  # Should be >0.01
print("Action mean:", np.mean(actions))

# Check critic signal
q_values = critic(states, random_actions)
print("Q range:", q_values.min(), q_values.max())
print("Q std:", q_values.std())  # Should have variation

# Check advantage
advantages = q_values - v_baseline
print("Advantage std:", advantages.std())  # Should be >0
```

**Fix**:

```python
# 1. Ensure policy outputs have variance
assert log_std.mean() < log_std_max - 0.5  # Not clamped to max
assert log_std.mean() > log_std_min + 0.5  # Not clamped to min

# 2. Check critic learns
critic_loss should decrease

# 3. Increase actor learning rate
actor_lr = 3e-4  # Instead of 1e-4

# 4. Debug advantage calculation
if advantage.std() < 0.01:
    print("WARNING: Advantages have no variation, critic might be wrong")
```

---

### Bug #3: Entropy Coefficient Divergence (SAC)

**Symptom**: SAC entropy coefficient α explodes (1e6+), policy becomes completely random, agent stops learning.

**Cause**: Entropy constraint optimization unstable.

```python
# WRONG - entropy loss unbounded
entropy_loss = -alpha * (log_probs + target_entropy)
# If log_probs >> target_entropy, loss becomes huge positive, α explodes
```

**Fix**:

```python
# RIGHT - use log(α) to avoid explosion
log_alpha = torch.log(alpha)
log_alpha_loss = -log_alpha * (log_probs.detach() + target_entropy)
alpha_optimizer.zero_grad()
log_alpha_loss.backward()
alpha_optimizer.step()
alpha = log_alpha.exp()

# Or clip α
alpha = torch.clamp(alpha, min=1e-4, max=10.0)
```

---

### Bug #4: Target Network Never Updated

**Symptom**: Agent learns for a bit, then stops improving. Training plateaus.

**Cause**: Target networks not updated (or updated too rarely).

```python
# WRONG - never update targets
target_critic = copy(critic)  # Initialize once
for step in range(1000000):
    # ... training loop ...
    # But target_critic never updated!
```

**Fix**:

```python
# RIGHT - soft update every step (or every N steps for delayed methods)
tau = 0.005  # Soft update parameter
for step in range(1000000):
    # ... critic update ...
    # Soft update targets
    for param, target_param in zip(critic.parameters(), target_critic.parameters()):
        target_param.data.copy_(tau * param.data + (1-tau) * target_param.data)

# Or hard update (copy all weights) every N steps
if step % update_frequency == 0:
    target_critic = copy(critic)
```

---

### Bug #5: Gradient Flow Through Detached Tensors

**Symptom**: Actor loss computation succeeds, but actor parameters don't update.

**Cause**: Critic detached but actor expects gradients.

```python
# WRONG
for step in range(1000):
    q_value = critic(state, action).detach()  # Detached!
    actor_loss = -q_value.mean()
    actor.update(actor_loss)  # Gradient won't flow through q_value!

# Result: actor_loss always 0 (constant from q_value.detach())
# Actor parameters updated but toward constant target (no signal)
```

**Fix**:

```python
# RIGHT - don't detach when computing actor loss
q_value = critic(state, action)  # No detach!
actor_loss = -q_value.mean()
actor.update(actor_loss)  # Gradient flows through q_value

# Detach where appropriate:
# - Value targets: v_target = (r + gamma * v_next).detach()
# - Stop gradient in critic: q_target = (r + gamma * q_next.detach()).detach()
# But NOT when computing actor loss
```

---

## Part 7: When to Use Actor-Critic vs Alternatives

### Actor-Critic vs Policy Gradient (REINFORCE)

| Factor | Actor-Critic | Policy Gradient |
|--------|--------------|-----------------|
| **Variance** | Low (baseline reduces) | High (full return) |
| **Sample Efficiency** | High | Low |
| **Convergence Speed** | Fast | Slow |
| **Complexity** | Two networks | One network |
| **Stability** | Better | Worse (high noise) |

**Use Actor-Critic when**: Continuous actions, sample efficiency matters, training instability

**Use Policy Gradient when**: Simple problem, don't need value function, prefer simpler code

---

### Actor-Critic vs Q-Learning (DQN)

| Factor | Actor-Critic | Q-Learning |
|--------|--------------|-----------|
| **Action Space** | Continuous (natural) | Discrete (requires all Q values) |
| **Sample Efficiency** | High | Very high |
| **Stability** | Good | Can diverge (overestimation) |
| **Complexity** | Two networks | One network (but needs tricks) |

**Use Actor-Critic for**: Continuous actions, robotics, control

**Use Q-Learning for**: Discrete actions, games, navigation

---

### Actor-Critic (On-Policy A2C) vs Off-Policy (SAC, TD3)

| Factor | A2C (On-Policy) | SAC/TD3 (Off-Policy) |
|--------|-----------------|---------------------|
| **Sample Efficiency** | Moderate | High (replay buffer) |
| **Stability** | Good | Excellent |
| **Complexity** | Simpler | More complex |
| **Data Reuse** | Limited (one pass) | High (replay buffer) |
| **Parallel Training** | Excellent (A3C) | Limited (off-policy break) |

**Use A2C when**: Want simplicity, have parallel workers, on-policy is okay

**Use SAC/TD3 when**: Need sample efficiency, offline data possible, maximum stability

---

## Part 8: Implementation Checklist

### Pre-Training Checklist

- [ ] Actor outputs mean and log_std separately
- [ ] Log_std clamped: `log_std_min <= log_std <= log_std_max`
- [ ] Action squashing with tanh (bounded to [-1,1])
- [ ] Log probability computation includes tanh Jacobian (SAC/A2C)
- [ ] Critic network separate from actor
- [ ] Critic loss is value bootstrap (r + γV(s'), not G_t)
- [ ] Two critics for SAC/TD3 (or one for A2C)
- [ ] Target networks initialized as copies of main networks
- [ ] Replay buffer created (for off-policy methods)
- [ ] Advantage estimation (GAE preferred, MC acceptable)

### Training Loop Checklist

- [ ] Data collection uses current actor (not target)
- [ ] Critic updated with Bellman target: `r + γV(s').detach()`
- [ ] Actor updated with advantage signal: `-log_prob(a) * A(s,a)` or `-Q(s,a)`
- [ ] Target networks soft updated: `τ * main + (1-τ) * target`
- [ ] For SAC: entropy coefficient α being optimized
- [ ] For TD3: delayed actor updates (every policy_delay)
- [ ] For TD3: target policy smoothing (noise + clip)
- [ ] Gradient clipping applied if losses explode
- [ ] Learning rates appropriate (critic_lr typically >= actor_lr)
- [ ] Reward normalization or clipping applied

### Debugging Checklist

- [ ] Critic loss decreasing over time?
- [ ] V(s) and Q(s,a) values in reasonable range?
- [ ] Policy entropy decreasing (exploration → exploitation)?
- [ ] Actor loss decreasing?
- [ ] Return increasing over episodes?
- [ ] No NaN or Inf in losses?
- [ ] Advantage estimates have variation?
- [ ] Policy output std not stuck at min/max?

---

## Part 9: Comprehensive Pitfall Reference

### 1. Critic Loss Not Decreasing

- Wrong Bellman target (should be r + γV(s'))
- Critic weights not updating (zero gradients)
- Learning rate too low
- Target network staleness (not updated)

### 2. Actor Not Improving

- Critic broken (no signal)
- Advantage estimates all zero
- Actor learning rate too low
- Policy parameterization wrong (no variance)

### 3. Training Unstable (Divergence)

- Missing target networks
- Critic loss exploding (wrong target, high learning rate)
- Entropy coefficient exploding (SAC: should be log(α))
- Actor updates every step (should delay, especially TD3)

### 4. Policy Stuck at Random Actions (SAC)

- Manual α fixed (should be auto-tuned)
- Target entropy wrong (should be -action_dim)
- Entropy loss gradient wrong direction

### 5. Policy Output Clamped to Min/Max Std

- Log_std range too tight (check log_std_min/max)
- Network initialization pushing to extreme values
- No gradient clipping preventing adjustment

### 6. Tanh Squashing Ignored

- Log probability not adjusted for squashing
- Missing Jacobian term in SAC/policy gradient
- Action scaling inconsistent

### 7. Target Networks Never Updated

- Forgot to create target networks
- Update function called but not applied
- Update frequency too high (no learning)

### 8. Off-Policy Break (Experience Replay)

- Actor training on old data (should use current replay buffer)
- Data distribution shift (actions from old policy)
- Batch importance weights missing (PER)

### 9. Advantage Estimates Biased

- GAE parameter λ wrong (should be 0.95-0.99)
- Bootstrap incorrect (wrong value target)
- Critic too inaccurate (overcorrection)

### 10. Entropy Coefficient Issues (SAC)

- Manual tuning instead of auto-tuning
- Entropy target not set correctly
- Log(α) optimization not used (causes explosion)

---

## Part 10: Real-World Examples

### Example 1: SAC for Robotic Arm Control

**Problem**: Robotic arm needs to reach target position. Continuous joint angles.

**Setup**:

```python
state_dim = 18  # 6 joint angles + velocities
action_dim = 6  # Joint torques
action_range = [-1, 1]  # Normalized

actor = ActorNetwork(state_dim, action_dim)  # Outputs μ, log_std
critic1 = CriticNetwork(state_dim, action_dim)
critic2 = CriticNetwork(state_dim, action_dim)

target_entropy = -action_dim  # -6
alpha = 1.0
```

**Training**:

```python
for step in range(1000000):
    # Collect experience
    state = env.reset() if done else next_state
    action = actor.sample(state)
    next_state, reward, done = env.step(action)
    replay_buffer.add(state, action, reward, next_state, done)

    if len(replay_buffer) < min_buffer_size:
        continue

    batch = replay_buffer.sample(256)

    # Critic update
    next_actions = actor.sample(batch.next_states)
    next_log_probs = actor.log_prob(next_actions, batch.next_states)
    q1_target = target_critic1(batch.next_states, next_actions)
    q2_target = target_critic2(batch.next_states, next_actions)
    target = batch.rewards + gamma * (1-batch.dones) * (
        torch.min(q1_target, q2_target) - alpha * next_log_probs
    )

    critic1_loss = MSE(critic1(batch.states, batch.actions), target)
    critic2_loss = MSE(critic2(batch.states, batch.actions), target)

    # Actor update
    actions = actor.sample(batch.states)
    log_probs = actor.log_prob(actions, batch.states)
    q_values = torch.min(
        critic1(batch.states, actions),
        critic2(batch.states, actions)
    )
    actor_loss = (alpha * log_probs - q_values).mean()

    # Entropy coefficient update
    entropy_loss = -alpha * (log_probs.detach() + target_entropy).mean()

    # Optimize
    critic1_optimizer.step(critic1_loss)
    critic2_optimizer.step(critic2_loss)
    actor_optimizer.step(actor_loss)
    alpha_optimizer.step(entropy_loss)

    # Soft update targets
    soft_update(target_critic1, critic1, tau=0.005)
    soft_update(target_critic2, critic2, tau=0.005)
```

### Example 2: TD3 for Autonomous Vehicle Control

**Problem**: Vehicle continuous steering/acceleration. Needs stable, deterministic behavior.

**Setup**:

```python
state_dim = 32  # Observations (lidar, speed, etc)
action_dim = 2  # Steering angle, acceleration
action_range = [[-0.5, -1], [0.5, 1]]  # Different ranges per action

actor = ActorNetwork(state_dim, action_dim)  # Deterministic!
critic1 = CriticNetwork(state_dim, action_dim)
critic2 = CriticNetwork(state_dim, action_dim)
```

**Training**:

```python
for step in range(1000000):
    # Collect with exploration noise
    action = actor(state) + exploration_noise
    action = torch.clamp(action, *action_range)
    next_state, reward, done = env.step(action)
    replay_buffer.add(state, action, reward, next_state, done)

    batch = replay_buffer.sample(256)

    # Critic update with target policy smoothing
    next_actions = actor_target(batch.next_states)
    noise = torch.randn_like(next_actions) * target_noise
    noise = torch.clamp(noise, -noise_clip, noise_clip)
    next_actions = torch.clamp(next_actions + noise, *action_range)

    q1_target = critic1_target(batch.next_states, next_actions)
    q2_target = critic2_target(batch.next_states, next_actions)
    target = batch.rewards + gamma * (1-batch.dones) * torch.min(q1_target, q2_target)

    critic1_loss = MSE(critic1(batch.states, batch.actions), target)
    critic2_loss = MSE(critic2(batch.states, batch.actions), target)

    # Delayed actor update
    if step % policy_delay == 0:
        actor_loss = -critic1(batch.states, actor(batch.states)).mean()
        actor_optimizer.step(actor_loss)

        # Update targets
        soft_update(target_actor, actor, tau=0.005)
        soft_update(target_critic1, critic1, tau=0.005)
        soft_update(target_critic2, critic2, tau=0.005)
```

---

## Part 11: Advanced Topics

### Distributed Training

Actor-critic methods work with distributed data collection:

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│Worker 1 │  │Worker 2 │  │Worker N │
│ env     │  │ env     │  │ env     │
│ rollout │  │ rollout │  │ rollout │
└────┬────┘  └────┬────┘  └────┬────┘
     │           │           │
     └─────┬─────┴─────┬─────┘
           │           │
       ┌───▼───────────▼────┐
       │  Replay Buffer     │
       │  (or Shared Queue) │
       └───┬────────────────┘
           │
       ┌───▼──────────────┐
       │ Parameter Server │
       │ (Actor + Critics)│
       └─────────────────┘
```

**Benefits**: Fast sample collection (N workers collect in parallel)

---

### Multi-Task Learning

Use actor-critic for multiple related tasks:

```
State: [task_id, observations]
Actor: Outputs action conditional on task_id
Critic: Values state+task

Transfer learning: Pre-train on many tasks, fine-tune on new task
```

---

## Part 12: Rationalization Common Mistakes

Users often make systematic errors in actor-critic reasoning. Here's how to prevent them:

### Mistake #1: "Why use SAC when TD3 is simpler?"

**Rationalization**: "TD3 has simpler math (no entropy), just two critics and delayed updates. SAC adds entropy which seems overly complex. Can't I just use TD3?"

**Counter**: SAC's entropy IS the simplicity. By automatically tuning α, SAC handles exploration automatically. TD3 still requires manual tuning of:

- policy_delay (2? 5? 10?)
- target_policy_noise magnitude
- noise_clip value

SAC auto-tunes entropy. That's FEWER hyperparameters overall.

**Reality**: SAC is more automated. TD3 requires more expertise.

---

### Mistake #2: "My critic diverged, let me reduce learning rate"

**Rationalization**: "Critic loss is exploding. Reducing learning rate should stabilize it."

**Counter**: Blindly lowering learning rate treats symptom, not cause. If critic is diverging, check:

1. Is the Bellman target correct? (r + γV(s').detach())
2. Are you gradient clipping?
3. Are target networks being updated?

A wrong target will diverge at ANY learning rate (will just take longer).

**Reality**: Debug the Bellman equation first. Then adjust learning rate.

---

### Mistake #3: "A2C should work fine, why use off-policy?"

**Rationalization**: "A2C is on-policy and simpler. Off-policy (SAC/TD3) adds complexity with replay buffers. Can't I just use A2C for everything?"

**Counter**: A2C discards data after one pass. SAC/TD3 reuse data with replay buffer.

For continuous control with limited data:

- A2C: 1 million environment steps = 1 million gradient updates
- SAC: 1 million environment steps = 4+ million gradient updates (from replay buffer)

SAC learns 4x faster per sample.

**Reality**: Off-policy scales better. Use it when data is expensive (robotics).

---

### Mistake #4: "SAC won't explore, let me manually set α higher"

**Rationalization**: "Agent isn't exploring. SAC entropy coefficient seems too low. Let me manually increase α to force exploration."

**Counter**: Manually increasing α BREAKS SAC's design. SAC will auto-adjust α. If it's not exploring:

1. Check if α is actually being optimized (log(α) loss?)
2. Check target_entropy is correct (-action_dim?)
3. Maybe the reward is so good, SAC found it fast (not a bug!)

Manual α override means you're not using SAC, you're using plain entropy regularization. That's worse than SAC.

**Reality**: Trust SAC's auto-tuning. If exploring too little, check target_entropy.

---

### Mistake #5: "Two critics in TD3, but I'll use only one Q-value"

**Rationalization**: "TD3 has Q1 and Q2, but I'll just use Q1 for the target. It's one critic, should work fine."

**Counter**: Twin critics are critical for stability. Using only one defeats the purpose:

```python
# WRONG - only one Q, no overestimation prevention
Q_target = critic1_target(next_state, next_action)  # Just one!
target = r + gamma * Q_target

# RIGHT - minimum of two, prevents high bias
Q1_target = critic1_target(next_state, next_action)
Q2_target = critic2_target(next_state, next_action)
target = r + gamma * min(Q1_target, Q2_target)  # Conservative!
```

Single critic will overestimate and diverge.

**Reality**: Both critics must be used in target. That's the point.

---

### Mistake #6: "Tanh squashing is just for action bounds, doesn't affect gradients"

**Rationalization**: "I'll scale actions with tanh, but it's just a function. The log probability should be the same as unsquashed normal."

**Counter**: Tanh squashing changes the probability distribution:

```
π(a|s) = N(μ(s), σ(s))  [Wrong! Ignores tanh]
π(a|s) = |det(∂a/∂x)|^(-1) * N(μ(s), σ(s))  [Right! Includes Jacobian]

log π(a|s) has Jacobian term: -log(1 - a² + ε)
```

Ignoring this term makes entropy calculation wrong. SAC entropy coefficient adjusts based on WRONG entropy estimate. Policy diverges.

**Reality**: Always include Jacobian. It's not optional.

---

### Mistake #7: "Gradient clipping is for neural nets, not RL"

**Rationalization**: "Gradient clipping is for recurrent networks. Actor-critic shouldn't need it."

**Counter**: Actor-critic trains on bootstrapped targets. If critic breaks, gradients can explode:

```
Unstable critic → huge Q-values → huge actor gradients → NaN
```

Gradient clipping prevents explosion:

```python
torch.nn.utils.clip_grad_norm_(actor.parameters(), max_norm=10.0)
torch.nn.utils.clip_grad_norm_(critic.parameters(), max_norm=10.0)
```

This is protective. It doesn't hurt convergence.

**Reality**: Use gradient clipping in actor-critic. It's standard practice.

---

### Mistake #8: "Soft update is just for stability, doesn't matter if I hard update"

**Rationalization**: "Target networks update less frequently. Whether I soft update (τ=0.005) or hard update (every 1000 steps), shouldn't matter."

**Counter**: Soft vs hard update has different stability properties:

```python
# Soft update - every step
target = τ * main + (1-τ) * target  # Smooth, continuous change

# Hard update - every N steps
if step % N == 0:
    target = copy(main)  # Sudden change

# Soft update: target changes by 0.5% per step (smooth learning)
# Hard update: target changes 100% every N steps (may overshoot)
```

Hard update can cause temporary divergence when copied. Soft update is smoother.

**Reality**: Soft update is preferred. Use τ ≈ 0.005 for continuous stability.

---

## Part 13: Rationalization Decision Table

When users ask "Should I use X or Y?", use this table:

| Question | A | B | Decision |
|----------|---|---|----------|
| Stochastic or Deterministic? | Stochastic (SAC) | Deterministic (TD3) | Both valid, SAC more robust |
| Off-policy or On-policy? | Off-policy (SAC/TD3) | On-policy (A2C) | Off-policy for sample efficiency |
| Sample efficiency critical? | Yes (SAC/TD3) | No (A2C) | Use off-policy if data expensive |
| Manual tuning tolerance? | Minimal (SAC) | Moderate (TD3) | SAC: fewer hyperparameters |
| Exploration strategy? | Entropy (SAC) | Policy smoothing (TD3) | SAC: automatic entropy |
| Computation budget? | Higher (SAC) | Lower (TD3) | SAC: slightly more, worth it |
| First time AC method? | SAC (recommended) | TD3 (alternative) | Start with SAC |

---

## Part 14: Common Pitfall Deep Dives

### Pitfall #11: Advantage Estimation Bias

**What goes wrong**: Using TD(0) advantage instead of GAE. Learning slow, noisy.

```python
# Suboptimal - high bias, low variance
A(s,a) = r + γV(s') - V(s)  # One-step, if V(s') wrong, advantage wrong

# Better - balanced bias-variance
A(s,a) = δ_0 + (γλ)δ_1 + (γλ)²δ_2 + ...  # GAE combines multiple steps
```

**How to fix**:

```python
def compute_gae(rewards, values, next_value, gamma, lam):
    advantages = torch.zeros_like(rewards)
    gae = 0
    for t in reversed(range(len(rewards))):
        if t == len(rewards) - 1:
            next_val = next_value
        else:
            next_val = values[t+1]

        delta = rewards[t] + gamma * next_val - values[t]
        gae = delta + (gamma * lam) * gae
        advantages[t] = gae

    return advantages
```

---

### Pitfall #12: Network Architecture Mismatch

**What goes wrong**: Actor and critic networks very different sizes. Critic learns slow, can't keep up with actor.

```python
# WRONG - massive mismatch
actor = nn.Sequential(
    nn.Linear(state_dim, 128),
    nn.ReLU(),
    nn.Linear(128, action_dim)  # Small!
)

critic = nn.Sequential(
    nn.Linear(state_dim, 512),
    nn.ReLU(),
    nn.Linear(512, 256),
    nn.ReLU(),
    nn.Linear(256, 1)  # Huge!
)
```

**Fix**: Use similar architectures:

```python
actor = nn.Sequential(
    nn.Linear(state_dim, 256),
    nn.ReLU(),
    nn.Linear(256, 256),
    nn.ReLU(),
    nn.Linear(256, action_dim * 2)  # μ and log_std
)

critic = nn.Sequential(
    nn.Linear(state_dim + action_dim, 256),
    nn.ReLU(),
    nn.Linear(256, 256),
    nn.ReLU(),
    nn.Linear(256, 1)  # Same layer sizes
)
```

---

### Pitfall #13: Rewards Not Normalized

**What goes wrong**: Rewards in range [0, 10000]. Critic outputs huge values. Gradients unstable.

```python
# WRONG - raw rewards
reward = env.reward()  # Could be 1000+
target = reward + gamma * v_next

# RIGHT - normalize
reward_mean = running_mean(rewards)
reward_std = running_std(rewards)
reward_normalized = (reward - reward_mean) / (reward_std + 1e-8)
target = reward_normalized + gamma * v_next
```

**Running statistics**:

```python
class RunningNorm:
    def __init__(self):
        self.mean = 0
        self.var = 1
        self.count = 0

    def update(self, x):
        self.count += 1
        delta = x - self.mean
        self.mean += delta / self.count
        delta2 = x - self.mean
        self.var = (self.var * (self.count-1) + delta * delta2) / self.count
```

---

## Part 15: Red Flags Comprehensive List

| Red Flag | Diagnosis | Fix |
|----------|-----------|-----|
| Critic loss NaN | Exploding gradients, huge rewards | Reward normalization, gradient clipping |
| Critic loss stuck | Wrong target, or target network not updating | Check Bellman target, ensure soft update |
| Actor loss 0 | Critic signal dead, or advantage zero | Debug critic, check gradient flow |
| Policy std at min | Network pushing to limits | Check initialization, gradient clipping |
| Return oscillates | Actor chasing moving target | Use delayed updates (TD3) or check critic |
| Entropy coefficient explodes (SAC) | Loss unbounded | Use log(α) instead of α directly |
| Target network never used | Forgot to create/copy targets | Check target network update code |
| Action clipping needed constantly | Action range wrong, or policy diverges | Check action bounds, policy variance |
| Same action always | log_std clamped to min | Increase log_std_max or check initialization |
| Reward always same episode | Reward computed wrong, or agent stuck | Check reward function, environment |

---

## Summary: Quick Reference

### When to Choose Actor-Critic

```
Do you have continuous actions? YES → Actor-Critic
Need sample efficiency? YES → SAC or TD3
Prefer stochastic policy? YES → SAC
Prefer deterministic? YES → TD3
Want off-policy learning? YES → SAC/TD3
Want on-policy simplicity? YES → A2C
```

### Red Flags

1. Critic loss not decreasing → Check Bellman target
2. Policy not changing → Check advantage signal
3. Training diverging → Check target networks
4. SAC policy random → Check entropy coefficient (must be auto-tuned)
5. TD3 unstable → Check policy delay and target smoothing

### Quick Debugging

```python
# First: Check critic
assert critic_loss.decreasing()  # Should go down
assert -100 < v_values.mean() < 100  # Reasonable range

# Second: Check actor
assert actor_loss.decreasing() or returning improving
assert log_std not clamped to min/max
assert actions have variance

# Third: Check training
assert return increasing over episodes
assert return_std decreasing (less variance)
assert entropy coefficient changing (SAC)
```

---

## Conclusion

Actor-critic methods are the gold standard for continuous control because they:

1. **Combine benefits**: Policy (actor) + value (critic) = stable, sample-efficient learning
2. **Handle continuous actions naturally**: No need to discretize
3. **Reduce variance**: Baseline critical for fast learning
4. **Enable off-policy learning**: SAC/TD3 with replay buffer = very sample efficient
5. **Scale to complexity**: Work on high-dimensional state/action spaces

**Start with SAC** (most robust, auto-tuned entropy). Debug using the comprehensive checklist. Switch to TD3 if you need deterministic policy or have tuning expertise.

Key insight: Actor-critic's power comes from separating action selection (actor) from value estimation (critic). Maintain this separation carefully (critics must learn accurately, actors must use accurate value signals) and training will be stable and efficient.
