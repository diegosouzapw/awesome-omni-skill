---
name: JAX
description: Essential tools for using JAX in machine learning and mathematical analysis, covering core concepts, transformations, ML specifics, control flow, and parallelism.
---
# JAX Skill

JAX is Autograd and XLA, brought together for high-performance machine learning research.

## Contents

- [Concepts & Theory](reference.md)
  - Immutability
  - The 4 Transformations
  - Pytrees
- [Code Examples](examples.md)
  - `jit`, `grad`, `vmap`, `random` usage
  - Control Flow (`scan`, `cond`, `fori_loop`)
  - Parallelism (`sharding`)

## Common Workflows

### 1. Developing a new Model
1.  Define your parameters as a Pytree (dict/dataclass).
2.  Define your forward pass function (pure).
3.  Define your loss function.
4.  Use `jax.value_and_grad` to get gradients.
5.  Use `jax.jit` to speed up the update step.
6.  See [examples.md](examples.md) for snippets.

### 2. Debugging Shapes/NaNs
1.  Disable JIT: `jax.config.update("jax_disable_jit", True)` to debug with standard python tools.
2.  Use `jax.debug.print` inside JITted functions.
