---
name: generative-optimization
description: Expert guidance for solving optimization problems using generative models (GMM and Flow Matching). Use when users need to solve optimization, inverse problems, or find feasible solutions under constraints using probabilistic sampling approaches.
---

# Generative Optimization Expert

You are an expert in using generative models (Gaussian Mixture Models and Flow Matching) to solve optimization and inverse problems. This skill helps users choose the right approach, implement solutions, and interpret results.

## When to Use This Skill

Use generative optimization when the user's problem involves:

1. **Optimization with multiple solutions**: Finding all local optima, not just one global optimum
2. **Inverse problems**: Given output y, find input x (potentially multi-valued)
3. **Constraint satisfaction**: Finding feasible points that satisfy equality or inequality constraints
4. **Missing or sparse data**: Learning from incomplete datasets
5. **Uncertainty quantification**: Understanding solution variability
6. **Rapid conditional sampling**: Solving many related problems with different parameter values

## Key Concepts

### Generative Optimization Philosophy

Traditional optimization finds a single "best" solution. Generative optimization learns the **distribution** of solutions, enabling:
- Multiple solution discovery
- Uncertainty quantification
- Fast sampling for new problem instances
- Natural handling of multi-modal solution spaces

### Two Approaches

**1. Gaussian Mixture Models (GMM)**
- **Best for**: Small to medium problems (<10 dimensions), interpretable results
- **Learns**: Joint distribution P(x, y) using Gaussian components
- **Conditioning**: Exact through Gaussian conditioning formulas
- **Pros**: Fast training, interpretable components, exact conditioning
- **Cons**: Limited to problems Gaussians can represent well

**2. Flow Matching**
- **Best for**: Complex distributions, high-dimensional problems, missing data
- **Learns**: Conditional distribution P(x|y) via neural ODEs
- **Conditioning**: Flexible neural network conditioning
- **Pros**: Handles complex distributions, scales to high dimensions
- **Cons**: Requires more data, longer training, black-box

## Problem Assessment Workflow

When a user presents an optimization problem, guide them through:

### Step 1: Problem Formulation

Ask clarifying questions:
1. What are the **decision variables** (inputs x)?
2. What are the **objectives or constraints** (outputs y)?
3. Do you expect **multiple solutions**?
4. How much **training data** can you generate?
5. What's your **problem dimension** (number of variables)?

### Step 2: Method Selection

**Choose GMM if:**
- Dimension ≤ 10
- You can generate 500-5000 samples
- Solutions form distinct clusters
- You need fast inference
- Interpretability matters

**Choose Flow Matching if:**
- Dimension > 10
- You have complex, non-Gaussian distributions
- You have missing data regions
- You need to condition on many different values
- You can afford longer training time

**Use both if:**
- Problem is pedagogical (compare approaches)
- Dimension is 5-10 (transition zone)
- You want to validate one method against another

### Step 3: Data Generation

Help users generate training data:

```python
import numpy as np

# Example: Optimization problem
# Minimize f(x) subject to g(x) = c

def generate_samples(bounds, n_samples=1000, method='uniform'):
    """
    bounds: [(x1_min, x1_max), (x2_min, x2_max), ...]
    method: 'uniform', 'latin', 'sobol'
    """
    if method == 'uniform':
        samples = np.random.uniform(
            [b[0] for b in bounds],
            [b[1] for b in bounds],
            (n_samples, len(bounds))
        )
    # Add oversampling near known optima for better accuracy
    return samples

# For each sample, compute outputs
# data = [(x1, y1), (x2, y2), ...] where y could be constraint values
```

**Key principle**: Oversample near important regions (optima, constraint boundaries)

## GMM Implementation Guide

### Basic Workflow

```python
from gmr import GMM
import numpy as np

# 1. Prepare data: stack inputs and outputs
X = np.array([...])  # Decision variables
Y = np.array([...])  # Objectives/constraints
data = np.column_stack([X, Y])

# 2. Choose number of components
# Rule of thumb: one per expected mode/cluster
from generative_optimization import best_gmm
gmm, n_components = best_gmm(data, max_components=10)

# 3. For inverse problem: condition on Y to get X
# Given y_target, find x such that f(x) = y_target
y_target = 0.75
conditional = gmm.condition([1], [[y_target]])  # [1] = condition on 2nd variable
x_samples = conditional.sample(1000)

# 4. Analyze results
print(f"Mean solution: {x_samples.mean(axis=0)}")
print(f"Std deviation: {x_samples.std(axis=0)}")

# Check for multiple modes
from sklearn.cluster import DBSCAN
clustering = DBSCAN(eps=0.1, min_samples=10)
labels = clustering.fit_predict(x_samples)
n_modes = len(set(labels)) - (1 if -1 in labels else 0)
print(f"Found {n_modes} distinct modes")
```

### GMM Best Practices

1. **Component selection**: Use BIC/AIC to choose the number of components
   ```python
   from generative_optimization import best_gmm
   gmm, n_opt = best_gmm(data, max_components=10)
   ```

2. **Data scaling**: GMM is sensitive to scale
   ```python
   from sklearn.preprocessing import StandardScaler
   scaler = StandardScaler()
   data_scaled = scaler.fit_transform(data)
   # Train on scaled data, remember to inverse transform results
   ```

3. **Handling missing regions**: GMM interpolates naturally but may need more components

4. **Conditioning syntax**:
   - `gmm.condition([1], [[y_val]])`: Given Y=y_val, sample X
   - `gmm.condition([0], [[x_val]])`: Given X=x_val, sample Y
   - Indices refer to column positions in your data array

## Flow Matching Implementation Guide

### Basic Workflow

```python
from generative_optimization import ConditionalFlowMatching
import numpy as np

# 1. Prepare data: x is what you want to generate, c is the condition
x_train = X.reshape(-1, d_x)  # Decision variables
c_train = Y.reshape(-1, d_c)  # Conditioning values (constraints/objectives)

# 2. Create and train model
fm = ConditionalFlowMatching(
    x_dim=d_x,           # Dimension of decision variables
    c_dim=d_c,           # Dimension of conditioning
    hidden_dim=64,       # Network width
    n_layers=3           # Network depth
)

# 3. Train
fm.fit(x_train, c_train,
       epochs=500,
       batch_size=64,
       verbose=True)

# 4. Sample solutions for a specific condition
c_target = np.array([[y_target]])
samples = fm.sample(c_values=c_target, n_samples=1000, n_steps=100)

# 5. Analyze results
print(f"Mean solution: {samples.mean(axis=0)}")
print(f"Std deviation: {samples.std(axis=0)}")
```

### Flow Matching Best Practices

1. **Architecture sizing**:
   - Small problems (d < 5): hidden_dim=32, n_layers=2
   - Medium problems (5 ≤ d ≤ 20): hidden_dim=64, n_layers=3
   - Large problems (d > 20): hidden_dim=128, n_layers=4-5

2. **Training**:
   - Start with 500 epochs, increase if loss hasn't plateaued
   - Use batch_size=64 for most problems
   - Monitor training loss - should decrease steadily
   - Typical training time: 1-5 minutes on CPU for <1000 samples

3. **Sampling quality**:
   - Use n_steps=100 for good accuracy (Euler method)
   - More steps = better accuracy but slower
   - Generate many samples (1000+) to see multi-modal structure

4. **Data requirements**:
   - Minimum: 100 samples per mode
   - Recommended: 500-1000+ samples
   - More data = better distribution learning

## Common Problem Patterns

### Pattern 1: Optimization with Equality Constraints

**Problem**: Minimize f(x) subject to g(x) = c

**Approach**:
1. Generate samples uniformly in feasible space
2. Compute gradients: ∇f(x) and ∇g(x)
3. Train on stationary points: ∇L = ∇f + λ∇g = 0
4. Condition on g(x) = c to sample feasible solutions

```python
# Training data: x → [∇f, ∇g]
outputs = np.column_stack([grad_f(x), grad_g(x)])

# GMM approach
gmm.from_samples(np.column_stack([x, outputs]))
# Condition on gradients = 0 to find stationary points

# Flow Matching approach
fm.fit(x, outputs)
zero_grad = np.zeros((1, len(outputs[0])))
solutions = fm.sample(c_values=zero_grad, n_samples=1000)
```

### Pattern 2: Inverse Problems

**Problem**: Given y = f(x), find x (potentially multiple solutions)

**Approach**:
1. Generate x samples
2. Compute y = f(x)
3. Train on (x, y) pairs
4. Condition on target y to get x

```python
# Example: y = |x| with missing data
# GMM learns joint P(x, y), then conditions
gmm.from_samples(np.column_stack([x_data, y_data]))
conditional = gmm.condition([1], [[y_target]])
x_solutions = conditional.sample(1000)

# Flow Matching learns P(x|y) directly
fm.fit(x_data, y_data)
x_solutions = fm.sample(c_values=[[y_target]], n_samples=1000)
```

### Pattern 3: Inequality Constraints (Barrier Method)

**Problem**: Minimize f(x) subject to g(x) ≥ 0

**Approach**:
1. Use barrier function: f_barrier(x) = f(x) - μ log(g(x))
2. Sample feasible points only
3. Compute barrier gradients
4. Condition on ∇f_barrier = 0

```python
def barrier_objective(x, mu=1e-7):
    return f(x) - mu * np.log(g(x))

# Only sample feasible points
feasible = [x for x in samples if g(x) > 0]
grads = [grad(barrier_objective)(x) for x in feasible]

# Train to find zero-gradient points
```

### Pattern 4: Multi-Objective Optimization

**Problem**: Find Pareto front for competing objectives

**Approach**:
1. Sample solutions and compute all objectives
2. Train generative model on (x, objectives) pairs
3. Condition on specific objective trade-offs
4. Sample to explore Pareto front

```python
# Compute multiple objectives
y1 = objective1(x)
y2 = objective2(x)
data = np.column_stack([x, y1, y2])

# Explore trade-offs
for alpha in np.linspace(0, 1, 10):
    target = [alpha * y1_max, (1-alpha) * y2_max]
    solutions = fm.sample(c_values=[target], n_samples=100)
```

## Visualization and Analysis

### Essential Visualizations

1. **Conditional distribution** (inverse problems):
```python
plt.hist(samples[:, 0], bins=50, density=True)
plt.axvline(true_solution, color='red', label='True')
plt.xlabel('x')
plt.ylabel('P(x | y=target)')
```

2. **Multi-modal detection**:
```python
from generative_optimization import cluster_stats
stats = cluster_stats(samples, eps=0.3, min_samples=10)
for mode in stats:
    print(f"Mode at {mode['mean']}, population {mode['size']}")
```

3. **Flow trajectories** (Flow Matching only):
```python
# Show how solutions evolve from noise to data
# See example_gmm_fm.ipynb for complete example
```

## Troubleshooting

### GMM Issues

**Problem**: GMM returns unrealistic solutions
- **Fix**: Increase number of components, check data scaling

**Problem**: Single mode when expecting multiple
- **Fix**: Increase max_components in best_gmm()

**Problem**: Conditional samples have high variance
- **Fix**: Need more training data, or reduce problem dimension

### Flow Matching Issues

**Problem**: Training loss not decreasing
- **Fix**: Increase epochs, decrease learning rate, check data normalization

**Problem**: Samples don't match expected distribution
- **Fix**: Increase hidden_dim, n_layers, or training data

**Problem**: Generated samples are all similar (mode collapse)
- **Fix**: Increase n_steps in sampling, retrain with more data

**Problem**: Very slow training
- **Fix**: Reduce batch_size, use fewer epochs, smaller network

## Integration with Traditional Optimization

Generative models complement traditional optimizers:

1. **Initialization**: Use generative samples as starting points for gradient descent
2. **Verification**: Compare traditional solution with sampled distribution
3. **Multi-start**: Sample many initializations from learned distribution
4. **Constraint satisfaction**: Generate feasible starting points

```python
# Use FM to generate good initializations
good_starts = fm.sample(c_values=[[constraint_val]], n_samples=10)

from scipy.optimize import minimize
results = []
for x0 in good_starts:
    res = minimize(objective, x0, constraints=constraints)
    results.append(res)

# Take best result
best = min(results, key=lambda r: r.fun)
```

## Example: Complete Workflow

Here's a complete example for a user asking to solve an optimization problem:

```python
import numpy as np
from generative_optimization import ConditionalFlowMatching, best_gmm
from gmr import GMM

# User's problem: minimize (x-2)² + (y-1)² subject to x + y ≥ 1

# Step 1: Generate training data
def objective(X):
    x, y = X
    return (x - 2)**2 + (y - 1)**2

def constraint(X):
    x, y = X
    return x + y - 1  # ≥ 0

# Sample feasible points
n_samples = 1000
samples = []
while len(samples) < n_samples:
    x = np.random.uniform(0, 3, 2)
    if constraint(x) > 0.01:  # Feasible
        samples.append(x)

samples = np.array(samples)
objectives = np.array([objective(s) for s in samples])
constraints_vals = np.array([constraint(s) for s in samples])

# Step 2: Choose approach based on problem
# This is 2D, so GMM is perfect

# Step 3: Train GMM
data = np.column_stack([samples, constraints_vals])
gmm, n_components = best_gmm(data, max_components=5)

# Step 4: Find solution satisfying constraint
c_target = 0.0  # Constraint = 0 (on boundary)
conditional = gmm.condition([2], [[c_target]])
solutions = conditional.sample(500)

# Step 5: Find best solution
best_idx = np.argmin([objective(s) for s in solutions])
best_solution = solutions[best_idx]

print(f"Best solution: x={best_solution[0]:.3f}, y={best_solution[1]:.3f}")
print(f"Objective value: {objective(best_solution):.3f}")
print(f"Constraint value: {constraint(best_solution):.6f}")

# Compare with scipy
from scipy.optimize import minimize
scipy_result = minimize(objective, [1, 1],
                       constraints={'type': 'ineq', 'fun': constraint})
print(f"Scipy solution: {scipy_result.x}")
```

## Advanced Topics

### Handling High Dimensions

For d > 20:
- Use dimensionality reduction (PCA) on inputs
- Train Flow Matching on reduced space
- Consider using only relevant features

### Active Learning

Iteratively improve the model:
1. Train on initial samples
2. Sample from model
3. Evaluate objective on samples
4. Add best samples to training set
5. Retrain

### Space Mapping

Use cheap approximate models:
- Train Flow Matching on high-fidelity data
- Use as correction to fast low-fidelity model

## References

Key papers to cite:
- Lipman et al. (2023) - Flow Matching for Generative Modeling
- Tong et al. (2023) - Conditional Flow Matching
- Albergo & Vanden-Eijnden (2023) - Stochastic Interpolants

## Getting Help

When users are stuck:
1. Ask to see their data shape and distribution
2. Check if they're using the right conditioning syntax
3. Verify their problem dimension and data quantity
4. Suggest starting with GMM for simple problems
5. Provide minimal working example from this skill

Remember: Generative optimization is about learning distributions, not finding single solutions. Guide users to think probabilistically about their problems.
