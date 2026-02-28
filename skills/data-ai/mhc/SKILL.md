---
name: mhc
description: Implements Manifold-Constrained Hyper-Connections (mHC) to solve residual connection issues using Doubly Stochastic Matrices.
---
# mHC Skill

Manifold-Constrained Hyper-Connections (mHC) uses Doubly Stochastic Matrices to improve Deep Learning stability.

## Contents

- [Examples](examples.md)
    - Full JAX implementation of `sinkhorn_knopp` and `mhc_layer_forward`.
- [Deep Theory](reference.md)
    - Motivation, stability proofs, and scalability arguments.

## Usage

Use this skill when implementing Deep Transformers (1000+ layers) where standard residual connections fail (Gradient Vanishing, Representation Collapse).

```python
# Quick Ref: Sinkhorn-Knopp (See examples.md for full context)
def sinkhorn_knopp(log_matrix, n_iters=20):
    M = jnp.exp(log_matrix)
    def body(i, m):
        m /= m.sum(axis=1, keepdims=True)
        m /= m.sum(axis=0, keepdims=True)
        return m
    return jax.lax.fori_loop(0, n_iters, body, M)
```
