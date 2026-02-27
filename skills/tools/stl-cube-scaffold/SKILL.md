---
name: stl-cube-scaffold
description: Analyze an STL mesh (AABB + approximate OBB) and generate a single connected STL that includes an L×L×L cube scaffold (default 100mm) plus connectors, so the final solid's OBB is locked to the target cube size for packing/measurement checks.
---

# STL Cube Scaffold (AABB/OBB, default 100mm)

## Goals

- Compute an STL's AABB and an approximate OBB (random rotation sampling search)
- Generate a single-piece STL: original model + L×L×L cube frame/pillars + connectors (default L=100mm)

## Prerequisites

Python 3.8+ with: `numpy`, `scipy`, `manifold3d`

## Workflow

### 1) Analyze the input STL (optional)

```bash
python scripts/stl_cube_support.py <in.stl> --analyze
python scripts/stl_cube_support.py <in.stl> --analyze --target <edge_mm>
```

If the script reports that the model cannot fit into the target cube (given the current search result), stop and confirm next steps with the user (e.g. increase `<edge_mm>`, or increase `--samples/--search-points` and retry the search).

### 2) Export a cube-scaffolded STL (recommended: `frame`, default L=100mm)

```bash
python scripts/stl_cube_support.py <in.stl> \
  --export-stl <out.stl> \
  --scaffold frame \
  --target <edge_mm>
```

- If you omit `--target`, it defaults to `100` (mm).
- Check the output:
  - `AABB dim: <edge_mm> x <edge_mm> x <edge_mm> mm`
  - `Connected components: 1`

### 3) Tuning options

| Option | When to use |
|--------|-------------|
| `--connector-penetration`, `--connector-count`, `--connector-thickness` | Multiple shells (`Connected components > 1`) |
| `--samples`, `--search-points` | Search didn't find a good orientation (can't fit into cube) |
| `--objective max` | Alternative optimization objective |
| `--scaffold frame` | Lock OBB to cube size (default, recommended) |

## How It Works

- **AABB vs OBB**: An AABB (axis-aligned bounding box) depends on the model's orientation. An OBB (oriented bounding box, minimum-volume sense) is invariant under global rotation/translation.

- **Why add an L×L×L cube frame/pillars**: By unioning an explicit L×L×L frame with the model, the final solid's minimum-volume OBB is forced to L×L×L.

- **Why connectors must intersect the model**: If the scaffold is disconnected from the original mesh, slicers and mesh tools will treat it as multiple shells/parts. Connectors must have a volumetric intersection (not just touching) with the model to ensure the boolean union becomes a single connected solid.
