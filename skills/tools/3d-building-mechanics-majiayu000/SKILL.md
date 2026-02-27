---
name: 3d-building-mechanics
description: "Complete Three.js building system with spatial indexing, structural physics, and multiplayer networking. Use for survival/crafting games, sandbox games, multiplayer construction, or any 3D building mechanics."
---

# 3D Building Mechanics - Advanced Skill

Complete Three.js building system with performance optimization, structural physics, and multiplayer networking.

## When to Use This Skill

Use when building:
- Survival/crafting games with base building
- Creative sandbox games
- Multiplayer construction games
- Any 3D building mechanics in Three.js

## Quick Start

```javascript
import { SpatialHashGrid } from './scripts/spatial-hash-grid.js';
import { HeuristicValidator } from './scripts/heuristic-validator.js';
import { ClientPrediction } from './scripts/client-prediction.js';

// Set up spatial indexing
const spatialIndex = new SpatialHashGrid(10);

// Set up structural validation (Rust/Valheim style)
const validator = new HeuristicValidator({ mode: 'heuristic' });

// For multiplayer - client prediction
const prediction = new ClientPrediction(buildingSystem);
```

## File Structure

```
references/
  performance-at-scale.md    - Spatial partitioning, chunks, instancing
  structural-physics-advanced.md - Arcade vs heuristic vs realistic
  multiplayer-networking.md  - Authority models, delta sync, conflicts

scripts/
  spatial-hash-grid.js   - O(1) spatial queries for uniform distribution
  octree.js              - Adaptive spatial queries for clustered bases
  chunk-manager.js       - World streaming for large maps
  performance-profiler.js - Benchmarking utilities
  
  heuristic-validator.js - Fast stability checking (Fortnite/Rust/Valheim)
  stability-optimizer.js - Caching and batch updates
  damage-propagation.js  - Damage states, cascading collapse
  physics-engine-lite.js - Optional realistic physics
  
  delta-compression.js   - Only send what changed
  client-prediction.js   - Optimistic placement with rollback
  conflict-resolver.js   - Handle simultaneous builds
  building-network-manager.js - Complete server/client networking
```

## Key Patterns

### Spatial Indexing Decision
- <1,000 pieces: Simple array
- 1,000-5,000 uniform: SpatialHashGrid
- 1,000-5,000 clustered: Octree  
- 5,000+: ChunkManager + Octree per chunk

### Structural Physics Modes
- **Arcade** (Fortnite): Connectivity only, instant collapse
- **Heuristic** (Rust/Valheim): Stability %, predictable rules
- **Realistic**: Full stress/strain, computationally expensive

### Multiplayer Authority
- Server-authoritative with client prediction
- Delta compression (only send changes)
- Conflict resolution: first-write, timestamp, or lock-based
