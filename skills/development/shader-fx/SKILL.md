---
name: shader-fx
description: GPU-accelerated shader effects framework built on Babylon.js. Provides composable graph-based post-processing, multi-pass rendering, feedback loops, and fluid simulation. Use when writing code that uses @avtools/shader-fx for shader effects, effect chains, or GPU computations.
metadata:
  author: avtools
  version: "0.0.0"
compatibility: Requires Babylon.js. WebGPU engine for WGSL path, WebGL engine for GLSL path.
---

# @avtools/shader-fx

GPU-accelerated shader effects framework built on Babylon.js. Provides a composable, graph-based system for real-time post-processing effects and GPU computations. Supports both WebGPU (WGSL shaders via `babylon` subpath) and WebGL (GLSL shaders via `babylonGL` subpath).

## Package Location

```
packages/shader-fx/
  mod.ts                          # Re-exports babylon and babylonGL namespaces
  deno.json                       # Package config, name: @avtools/shader-fx
  babylon/
    mod.ts                        # Re-exports shaderFXBabylon.ts
    shaderFXBabylon.ts            # Core framework (WebGPU/WGSL)
  babylonGL/
    mod.ts                        # Re-exports shaderFXBabylon_GL.ts
    shaderFXBabylon_GL.ts         # Core framework (WebGL/GLSL)
  generated/
    postFX/                       # 22 generated post-processing effects (WGSL)
    fluidSimulation/              # 13 generated fluid simulation stages (WGSL)
    fluidSimulationHack/          # 13 fluid sim stages (alternate variant, WGSL)
    babylonGL/postFX/             # 9 generated post-processing effects (GLSL)
```

## Import Paths

```typescript
// Top-level namespace (rarely used directly)
import { babylon, babylonGL } from "@avtools/shader-fx";

// WebGPU API (primary)
import {
  ShaderEffect, CustomShaderEffect, PassthruEffect, CanvasPaint, FeedbackNode,
  type ShaderSource, type Dynamic, type ShaderUniforms, type UniformDescriptor,
  type RenderPrecision, type ShaderGraph, type GraphNode, type GraphEdge,
  type CustomShaderEffectOptions, type MaterialHandles, type ShaderMaterialFactory,
  type PassTextureSourceSpec, type TextureInputKey,
} from "@avtools/shader-fx/babylon";

// WebGL API (same exports, GLSL shaders)
import { ShaderEffect, CustomShaderEffect, ... } from "@avtools/shader-fx/babylonGL";

// Generated effects (WebGPU)
import { WobbleEffect } from "@avtools/shader-fx/generated/postFX/wobble.frag.generated.ts";
import { BloomEffect } from "@avtools/shader-fx/generated/postFX/bloom.frag.generated.ts";
import { VelocityAdvectionEffect } from "@avtools/shader-fx/generated/fluidSimulation/velocityAdvection.frag.generated.ts";

// Generated effects (WebGL)
import { WobbleEffect } from "@avtools/shader-fx/generated/babylonGL/postFX/wobble.frag.gl.generated.ts";
```

## Dependencies

- `babylonjs@^8.20.0` -- Babylon.js rendering engine (WebGPU and WebGL)
- Generated effects are produced by the `@avtools/power2d-codegen` package

---

## Core Concepts

### Effect Graph

Effects form a directed acyclic graph (DAG). Each effect takes one or more `ShaderSource` inputs and produces a `RenderTargetTexture` output. The framework performs topological sorting to determine render order, and `renderAll()` traverses the full graph from a terminal effect.

### ShaderSource

A `ShaderSource` is the union of all valid input types to an effect:

```typescript
type ShaderSource =
  | BABYLON.BaseTexture
  | BABYLON.RenderTargetTexture
  | HTMLCanvasElement
  | OffscreenCanvas
  | ShaderEffect          // Uses effect.output automatically
```

When a `ShaderEffect` is passed as a source, the framework automatically extracts its `.output` RenderTargetTexture. When an `HTMLCanvasElement` or `OffscreenCanvas` is passed, the framework creates and manages a dynamic texture internally.

### Dynamic Uniforms

Uniform values can be static or dynamic (functions evaluated each frame):

```typescript
type Dynamic<T> = T | (() => T)
```

This allows uniforms to be driven by animation loops, audio analysis, or any other time-varying source without manually updating every frame.

### Multi-Pass Rendering

`CustomShaderEffect` supports multi-pass rendering. Each pass has its own `ShaderMaterial` and can route textures from earlier passes or from external inputs via `PassTextureSourceSpec`:

```typescript
type PassTextureSourceSpec<I> =
  | { binding: string; source: { kind: 'input'; key: TextureInputKey<I> } }
  | { binding: string; source: { kind: 'pass'; passIndex: number } }
```

The default behavior chains the primary texture through passes: pass N reads from pass N-1's output.

### Render Precision

```typescript
type RenderPrecision = 'unsigned_int' | 'half_float'
```

- `'half_float'` (default) -- Higher precision for accumulation effects, fluid simulation, HDR
- `'unsigned_int'` -- Standard 8-bit RGBA, lower memory usage

---

## Core API

### `ShaderEffect<I>` (abstract base class)

The root class for all effects. Provides graph traversal, rendering orchestration, and the common interface.

```typescript
abstract class ShaderEffect<I extends ShaderInputShape<I> = ShaderInputs> {
  readonly id: string                              // Unique effect ID (UUID or counter)
  debugId: string                                  // User-assigned debug label
  effectName: string                               // Effect type name
  width: number                                    // Output width (default 1280)
  height: number                                   // Output height (default 720)
  inputs: Partial<I>                               // Current input sources
  uniforms: ShaderUniforms                         // Current uniform values (may include Dynamic)
  lastRenderedFrameId?: string                      // Frame deduplication token

  abstract output: BABYLON.RenderTargetTexture     // The rendered output texture
  abstract setSrcs(fx: Partial<I>): void           // Update input sources
  abstract render(engine: BABYLON.Engine, frameId?: string): void  // Render this effect only
  abstract setUniforms(uniforms: ShaderUniforms): void             // Set uniform values
  abstract updateUniforms(): void                  // Resolve Dynamic uniforms and push to GPU
  abstract dispose(): void                         // Dispose this effect's GPU resources

  disposeAll(): void                // Recursively dispose this effect and all input effects
  getOrderedEffects(): ShaderEffect[]  // Topological sort of the effect graph (leaves first)
  getGraph(): ShaderGraph           // Returns { nodes: GraphNode[], edges: GraphEdge[] }
  renderAll(engine: BABYLON.Engine, frameId?: string): void  // Render entire graph in order
}
```

**Key behaviors:**
- `renderAll()` calls `getOrderedEffects()` to topologically sort the graph, then renders each effect once. If `frameId` is provided, effects that have already rendered for that frame ID are skipped (only `updateUniforms()` is called).
- `getOrderedEffects()` detects cycles and throws `"Cycle detected in shader graph"`.
- `disposeAll()` recursively walks the input graph and disposes every `ShaderEffect` it finds.

### `CustomShaderEffect<U, I>` (main implementation)

The concrete implementation that handles Babylon.js scenes, materials, render targets, and multi-pass rendering.

```typescript
class CustomShaderEffect<U extends object, I extends ShaderInputShape<I> = ShaderInputs>
  extends ShaderEffect<I> {

  readonly engine: BABYLON.WebGPUEngine  // (or BABYLON.Engine for GL variant)
  readonly scene: BABYLON.Scene
  readonly output: BABYLON.RenderTargetTexture
  readonly uniformMeta: UniformDescriptor[]

  // Constructor
  constructor(
    engine: BABYLON.WebGPUEngine,
    inputs: I,
    options: CustomShaderEffectOptions<U, I>
  )

  // Methods
  setSrcs(fx: Partial<I>): void
  setUniforms(uniforms: ShaderUniforms): void
  updateUniforms(): void
  render(engine: BABYLON.Engine): void
  dispose(): void

  // Accessors
  get material(): BABYLON.ShaderMaterial          // First pass material
  getPassOutput(passIndex: number): BABYLON.RenderTargetTexture | undefined
  getUniformsMeta(): UniformDescriptor[]
  getUniformRuntime(): Record<string, UniformRuntime>
  getFloatUniformNames(): string[]
  setTextureSampler(sampler: BABYLON.TextureSampler): void
}
```

### `CustomShaderEffectOptions<U, I>`

```typescript
interface CustomShaderEffectOptions<U, I> {
  factory: ShaderMaterialFactory<U, string>         // Creates MaterialHandles for each pass
  textureInputKeys: Array<TextureInputKey<I>>       // Input texture names (must have >= 1)
  textureBindingKeys?: string[]                     // Shader binding names (defaults to textureInputKeys)
  passTextureSources?: readonly (readonly PassTextureSourceSpec<I>[])[]  // Per-pass texture routing
  passCount?: number                                // Number of render passes (default 1)
  primaryTextureKey?: keyof I & string              // Key chained across passes (default: first input)
  width?: number                                    // Output width (default 1280)
  height?: number                                   // Output height (default 720)
  sampler?: BABYLON.TextureSampler                  // Custom sampler
  materialName?: string                             // Base name for materials
  sampleMode?: 'nearest' | 'linear'                // Texture filtering (default 'linear')
  precision?: RenderPrecision                       // Render target format (default 'half_float')
  uniformMeta?: UniformDescriptor[]                 // Metadata for uniform introspection/UI
}
```

### `PassthruEffect`

Simple passthrough that copies a source texture to a render target. Useful as an initial node in a chain or for capturing a texture.

```typescript
class PassthruEffect extends CustomShaderEffect<PassthruUniforms, PassthruInputs> {
  constructor(
    engine: BABYLON.WebGPUEngine,
    inputs: { src: ShaderSource },
    width?: number,       // default 1280
    height?: number,      // default 720
    sampleMode?: 'nearest' | 'linear',  // default 'linear'
    precision?: RenderPrecision,         // default 'half_float'
  )
}
```

### `CanvasPaint`

Renders a `ShaderSource` directly to the screen (default framebuffer) rather than to a render target. This is the terminal node that displays the effect chain on a canvas.

```typescript
class CanvasPaint extends CustomShaderEffect<Record<string, never>, CanvasPaintInputs> {
  constructor(
    engine: BABYLON.WebGPUEngine,  // or BABYLON.Engine for GL
    inputs: { src: ShaderSource },
    width?: number,
    height?: number,
    sampleMode?: 'nearest' | 'linear',
    precision?: RenderPrecision,
    targetCanvas?: HTMLCanvasElement,  // For multi-view rendering
  )
}
```

**Key behavior:** `render()` calls `engine.restoreDefaultFramebuffer()` and renders to the screen via `scene.render()` instead of to a render target. Supports multi-view rendering via `targetCanvas` parameter.

### `FeedbackNode`

Creates temporal feedback loops by cycling a previous frame's output back as input. Essential for iterative effects like fluid simulation, reaction-diffusion, trails, and motion blur.

```typescript
class FeedbackNode extends ShaderEffect<FeedbackInputs> {
  constructor(
    engine: BABYLON.WebGPUEngine,
    startState: ShaderEffect,           // Initial state (rendered on first frame)
    width?: number,
    height?: number,
    sampleMode?: 'nearest' | 'linear',
    precision?: RenderPrecision,
  )

  setFeedbackSrc(effect: ShaderEffect): void  // Set the effect whose output feeds back
}
```

**Key behavior:**
- On the first frame, copies `startState.output` to its own output.
- After the first frame, swaps to reading from the feedback source's output.
- The feedback source is NOT added to `inputs` to avoid creating a cycle in the DAG traversal. The feedback connection is managed separately.

---

## Type Definitions

### `UniformDescriptor`

Metadata for a shader uniform, enabling introspection and UI generation:

```typescript
interface UniformDescriptor {
  name: string                        // Logical uniform name (e.g., 'xStrength')
  kind: 'f32' | 'i32' | 'u32' | 'bool' | 'vec2f' | 'vec3f' | 'vec4f' | 'mat4x4f'
  bindingName: string                 // Actual shader binding name (e.g., 'uniforms_xStrength')
  default?: unknown                   // Default value
  isArray?: boolean
  arraySize?: number
  ui?: {
    min?: number
    max?: number
    step?: number
  }
}
```

### `UniformRuntime`

Runtime tracking for uniform values (used for UI sliders, range detection):

```typescript
interface UniformRuntime {
  isDynamic: boolean       // true if the value is a function
  current?: number         // Most recently resolved value
  min?: number             // Observed minimum
  max?: number             // Observed maximum
}
```

### `MaterialHandles<U, TName>`

The bridge between the framework and Babylon.js shader materials:

```typescript
interface MaterialHandles<U, TName extends string = string> {
  material: BABYLON.ShaderMaterial
  setTexture(name: TName, texture: BABYLON.BaseTexture): void
  setTextureSampler(name: TName, sampler: BABYLON.TextureSampler): void
  setUniforms(uniforms: Partial<U>): void
}
```

### `ShaderGraph`

```typescript
interface ShaderGraph {
  nodes: GraphNode[]
  edges: GraphEdge[]
}
interface GraphNode {
  id: string
  name: string
  ref: ShaderEffect
}
interface GraphEdge {
  from: string    // Source effect ID
  to: string      // Destination effect ID
}
```

---

## Generated Effects

All generated effects follow a consistent pattern. Each `.generated.ts` file exports:
- `XxxVertexSource` -- WGSL (or GLSL) vertex shader string
- `XxxFragmentSources` -- Array of fragment shader strings (one per pass)
- `XxxPassCount` -- Number of render passes
- `XxxPrimaryTextureName` -- Primary texture binding name
- `XxxPassTextureSources` -- Per-pass texture routing spec
- `XxxUniformMeta` -- Uniform metadata array
- `XxxUniforms` interface -- TypeScript interface for uniform values
- `setXxxUniforms()` -- Function to push uniforms to a ShaderMaterial
- `XxxInputs` interface -- TypeScript interface for texture inputs
- `createXxxMaterial()` -- Factory function returning MaterialHandles
- `XxxEffect` class -- Concrete `CustomShaderEffect` subclass

### Post-Processing Effects (postFX)

| Effect | Inputs | Key Uniforms | Passes | Description |
|--------|--------|-------------|--------|-------------|
| `WobbleEffect` | src | xStrength, yStrength, time | 1 | Sinusoidal UV distortion |
| `BloomEffect` | src | preBlackLevel, preGamma, preBrightness, minBloomRadius, maxBloomRadius, bloomThreshold, bloomSCurve, bloomFill, bloomIntensity, outputMode, inputImage | 2 | Multi-pass bloom with preprocessing |
| `PixelateEffect` | src | pixelSize | 1 | Pixelation/mosaic |
| `AlphaThresholdEffect` | src | threshold | 1 | Alpha cutoff |
| `PolygonMaskEffect` | src | (polygon data) | 1 | Polygon-shaped masking |
| `TransformEffect` | src | rotate, anchor(vec2f), translate(vec2f), scale(vec2f) | 1 | 2D affine transform |
| `HorizontalBlurEffect` | src | radius | 1 | 1D horizontal gaussian blur |
| `VerticalBlurEffect` | src | radius | 1 | 1D vertical gaussian blur |
| `EdgeEffect` | src | (edge detection params) | 1 | Edge detection |
| `InvertEffect` | src | -- | 1 | Color inversion |
| `LayerBlendEffect` | src1, src2 | -- | 1 | Alpha-based layer compositing |
| `LevelEffect` | src | (level params) | 1 | Levels adjustment |
| `LimitEffect` | src | (limit params) | 1 | Value limiting |
| `MathOpEffect` | src | (math params) | 1 | Mathematical operations |
| `InputCropEffect` | src | (crop params) | 1 | Input cropping |
| `DualViewEffect` | src | (view params) | 1 | Dual-view display |
| `AlphaTimeTagEffect` | src | (time params) | 1 | Alpha-based time tagging |
| `FloodFillStepEffect` | src | (fill params) | 1 | Flood fill step (JFA) |
| `FluidSimEffect` | src | (fluid params) | 1 | Single-pass fluid sim |
| `FluidVisualizeEffect` | src | (vis params) | 1 | Fluid visualization |
| `ReactionDiffusionEffect` | src | (RD params) | 1 | Reaction-diffusion step |
| `ReactionVisualizeEffect` | src | (vis params) | 1 | Reaction-diffusion visualization |

### Fluid Simulation Stages (fluidSimulation)

These are individual pipeline stages composable into a full Navier-Stokes fluid simulation:

| Effect | Inputs | Key Uniforms | Description |
|--------|--------|-------------|-------------|
| `VelocityAdvectionEffect` | velocity | timeStep, dissipation | Semi-Lagrangian velocity advection |
| `DyeAdvectionEffect` | velocity, dye | timeStep, dissipation | Advects dye/color field by velocity |
| `CurlEffect` | velocity | -- | Computes curl of velocity field |
| `VorticityConfinementEffect` | velocity, curl | strength, timeStep | Restores small-scale vortices |
| `DivergenceEffect` | velocity | -- | Computes divergence of velocity |
| `PressureJacobiEffect` | pressure, divergence | -- | Jacobi iteration for pressure solve |
| `PressureDampEffect` | pressure | damping | Damps pressure field |
| `GradientSubtractionEffect` | velocity, pressure | -- | Projects velocity to be divergence-free |
| `ForceApplicationEffect` | velocity | (force params) | Applies external forces to velocity |
| `DyeForceApplicationEffect` | dye | (force params) | Applies dye/color forces |
| `SplatEffect` | src | (splat params) | Gaussian splat injection |
| `SplatUnifiedEffect` | src | (splat params) | Unified splat variant |
| `AddEffect` | src1, src2 | -- | Additive blending of two fields |

### WebGL Variants (babylonGL/postFX)

A subset of effects available as GLSL for WebGL fallback:
- wobble, bloom, pixelate, alphaThreshold, polygonMask, transform, horizontalBlur, verticalBlur, layerBlend

---

## Usage Patterns

### Pattern 1: Simple Effect Chain

```typescript
import { PassthruEffect, CanvasPaint } from "@avtools/shader-fx/babylon";
import { WobbleEffect } from "@avtools/shader-fx/generated/postFX/wobble.frag.generated.ts";

// engine is a BABYLON.WebGPUEngine
const passthru = new PassthruEffect(engine, { src: someTexture }, 1280, 720);
const wobble = new WobbleEffect(engine, { src: passthru }, 1280, 720);
const canvasPaint = new CanvasPaint(engine, { src: wobble }, 1280, 720);

// Set uniforms (static values)
wobble.setUniforms({ xStrength: 0.02, yStrength: 0.01, time: 0 });

// Render loop
engine.runRenderLoop(() => {
  wobble.setUniforms({ time: performance.now() / 1000 });
  canvasPaint.renderAll(engine);
});
```

### Pattern 2: Dynamic Uniforms

```typescript
import { WobbleEffect } from "@avtools/shader-fx/generated/postFX/wobble.frag.generated.ts";

const wobble = new WobbleEffect(engine, { src: passthru });

// Pass functions instead of values -- evaluated each frame during updateUniforms()
wobble.setUniforms({
  xStrength: () => Math.sin(performance.now() / 1000) * 0.05,
  yStrength: 0.02,
  time: () => performance.now() / 1000,
});
```

### Pattern 3: Canvas as Input Source

```typescript
import { PassthruEffect } from "@avtools/shader-fx/babylon";

const drawCanvas = document.createElement('canvas');
drawCanvas.width = 1280;
drawCanvas.height = 720;
const ctx = drawCanvas.getContext('2d')!;

// The framework creates a DynamicTexture internally and updates it each render
const passthru = new PassthruEffect(engine, { src: drawCanvas }, 1280, 720);

engine.runRenderLoop(() => {
  ctx.clearRect(0, 0, 1280, 720);
  ctx.fillStyle = 'red';
  ctx.fillRect(100, 100, 200, 200);
  passthru.renderAll(engine);
});
```

### Pattern 4: Multi-Input Effects

```typescript
import { LayerBlendEffect } from "@avtools/shader-fx/generated/postFX/layerBlend.frag.generated.ts";

const layer1 = new PassthruEffect(engine, { src: texture1 });
const layer2 = new PassthruEffect(engine, { src: texture2 });

// LayerBlend takes two inputs: src1 and src2
const blend = new LayerBlendEffect(engine, { src1: layer1, src2: layer2 });
blend.renderAll(engine);
```

### Pattern 5: Feedback Loops (Temporal Effects)

```typescript
import { FeedbackNode, PassthruEffect } from "@avtools/shader-fx/babylon";
import { WobbleEffect } from "@avtools/shader-fx/generated/postFX/wobble.frag.generated.ts";

// Create initial state
const initialState = new PassthruEffect(engine, { src: blackTexture });

// FeedbackNode: first frame uses initialState, then switches to feedback source
const feedback = new FeedbackNode(engine, initialState, 1280, 720);

// Processing effect reads from the feedback node
const wobble = new WobbleEffect(engine, { src: feedback }, 1280, 720);
wobble.setUniforms({ xStrength: 0.01, yStrength: 0.01, time: () => performance.now() / 1000 });

// Close the loop: feedback reads from wobble's output on subsequent frames
feedback.setFeedbackSrc(wobble);

// Display
const paint = new CanvasPaint(engine, { src: wobble });

engine.runRenderLoop(() => {
  // renderAll handles the entire graph
  // FeedbackNode is NOT in wobble's input graph to avoid cycles;
  // it must be rendered explicitly before the rest
  feedback.render(engine);
  paint.renderAll(engine);
});
```

### Pattern 6: Multi-Pass Effects (Blur Example)

```typescript
import { HorizontalBlurEffect } from "@avtools/shader-fx/generated/postFX/horizontalBlur.frag.generated.ts";
import { VerticalBlurEffect } from "@avtools/shader-fx/generated/postFX/verticalBlur.frag.generated.ts";

// Two-pass separable Gaussian blur
const hBlur = new HorizontalBlurEffect(engine, { src: inputEffect }, 1280, 720);
const vBlur = new VerticalBlurEffect(engine, { src: hBlur }, 1280, 720);

// BloomEffect is a single effect with 2 internal passes:
// pass0 preprocesses, pass1 samples from pass0's output
import { BloomEffect } from "@avtools/shader-fx/generated/postFX/bloom.frag.generated.ts";
const bloom = new BloomEffect(engine, { src: inputEffect }, 1280, 720);
```

### Pattern 7: Fluid Simulation Pipeline

```typescript
import { FeedbackNode, PassthruEffect } from "@avtools/shader-fx/babylon";
import { VelocityAdvectionEffect } from "@avtools/shader-fx/generated/fluidSimulation/velocityAdvection.frag.generated.ts";
import { CurlEffect } from "@avtools/shader-fx/generated/fluidSimulation/curl.frag.generated.ts";
import { VorticityConfinementEffect } from "@avtools/shader-fx/generated/fluidSimulation/vorticityConfinement.frag.generated.ts";
import { DivergenceEffect } from "@avtools/shader-fx/generated/fluidSimulation/divergence.frag.generated.ts";
import { PressureJacobiEffect } from "@avtools/shader-fx/generated/fluidSimulation/pressureJacobi.frag.generated.ts";
import { GradientSubtractionEffect } from "@avtools/shader-fx/generated/fluidSimulation/gradientSubtraction.frag.generated.ts";
import { ForceApplicationEffect } from "@avtools/shader-fx/generated/fluidSimulation/forceApplication.frag.generated.ts";
import { DyeAdvectionEffect } from "@avtools/shader-fx/generated/fluidSimulation/dyeAdvection.frag.generated.ts";

const w = 512, h = 512;
const precision = 'half_float';

// Velocity pipeline with feedback
const velInit = new PassthruEffect(engine, { src: zeroTexture }, w, h, 'linear', precision);
const velFeedback = new FeedbackNode(engine, velInit, w, h, 'linear', precision);

const forceApp = new ForceApplicationEffect(engine, { velocity: velFeedback }, w, h, 'nearest', precision);
const velAdvect = new VelocityAdvectionEffect(engine, { velocity: forceApp }, w, h, 'linear', precision);
velAdvect.setUniforms({ timeStep: 1.0, dissipation: 0.99 });

const curl = new CurlEffect(engine, { velocity: velAdvect }, w, h, 'nearest', precision);
const vorticity = new VorticityConfinementEffect(engine, { velocity: velAdvect, curl: curl }, w, h, 'nearest', precision);

const divergence = new DivergenceEffect(engine, { velocity: vorticity }, w, h, 'nearest', precision);

// Pressure solve (multiple Jacobi iterations)
const pressureInit = new PassthruEffect(engine, { src: zeroTexture }, w, h, 'nearest', precision);
const pressureFeedback = new FeedbackNode(engine, pressureInit, w, h, 'nearest', precision);
const jacobi = new PressureJacobiEffect(engine, { pressure: pressureFeedback, divergence: divergence }, w, h, 'nearest', precision);
pressureFeedback.setFeedbackSrc(jacobi);

const gradSub = new GradientSubtractionEffect(engine, { velocity: vorticity, pressure: jacobi }, w, h, 'nearest', precision);
velFeedback.setFeedbackSrc(gradSub);

// Dye pipeline
const dyeInit = new PassthruEffect(engine, { src: zeroTexture }, w, h, 'linear', precision);
const dyeFeedback = new FeedbackNode(engine, dyeInit, w, h, 'linear', precision);
const dyeAdvect = new DyeAdvectionEffect(engine, { velocity: gradSub, dye: dyeFeedback }, w, h, 'linear', precision);
dyeFeedback.setFeedbackSrc(dyeAdvect);
```

### Pattern 8: Custom Effect from Scratch

```typescript
import * as BABYLON from "babylonjs";
import { CustomShaderEffect, type ShaderSource, type MaterialHandles } from "@avtools/shader-fx/babylon";

interface MyInputs { src: ShaderSource }
interface MyUniforms { intensity: number }

function createMyMaterial(scene: BABYLON.Scene, options?: { name?: string }): MaterialHandles<MyUniforms, 'src'> {
  const name = options?.name ?? 'MyMaterial';
  const vertexName = `${name}VertexShader`;
  const fragmentName = `${name}FragmentShader`;

  BABYLON.ShaderStore.ShadersStoreWGSL[vertexName] = `/* your WGSL vertex shader */`;
  BABYLON.ShaderStore.ShadersStoreWGSL[fragmentName] = `/* your WGSL fragment shader */`;

  const material = new BABYLON.ShaderMaterial(name, scene, {
    vertex: name, fragment: name,
  }, {
    attributes: ['position', 'uv'],
    uniforms: ['intensity'],
    samplers: ['src'],
    samplerObjects: ['srcSampler'],
    shaderLanguage: BABYLON.ShaderLanguage.WGSL,
  });

  return {
    material,
    setTexture: (n, tex) => material.setTexture(n, tex),
    setTextureSampler: (n, s) => material.setTextureSampler('srcSampler', s),
    setUniforms: (u) => { if (u.intensity !== undefined) material.setFloat('intensity', u.intensity); },
  };
}

class MyEffect extends CustomShaderEffect<MyUniforms, MyInputs> {
  effectName = 'MyEffect';
  constructor(engine: BABYLON.WebGPUEngine, inputs: MyInputs, width = 1280, height = 720) {
    super(engine, inputs, {
      factory: (scene, opts) => createMyMaterial(scene, opts),
      textureInputKeys: ['src'],
      width, height,
      materialName: 'MyMaterial',
    });
  }
}
```

### Pattern 9: Graph Introspection

```typescript
const graph = canvasPaint.getGraph();
console.log('Nodes:', graph.nodes.map(n => n.name));
console.log('Edges:', graph.edges.map(e => `${e.from} -> ${e.to}`));

const ordered = canvasPaint.getOrderedEffects();
console.log('Render order:', ordered.map(e => e.effectName));

// Uniform introspection
const meta = bloom.getUniformsMeta();
// [{ name: 'preBlackLevel', kind: 'f32', bindingName: 'uniforms_preBlackLevel', default: 0.05 }, ...]

const runtime = bloom.getUniformRuntime();
// { preBlackLevel: { isDynamic: false, current: 0.05, min: 0.05, max: 0.05 }, ... }
```

### Pattern 10: Using Frame IDs for Deduplication

```typescript
let frameCounter = 0;
engine.runRenderLoop(() => {
  const frameId = `frame-${frameCounter++}`;
  // If multiple terminal effects share subgraphs, frameId prevents double-rendering
  canvasPaint1.renderAll(engine, frameId);
  canvasPaint2.renderAll(engine, frameId);  // Shared effects only updateUniforms, not re-render
});
```

---

## Important Caveats

1. **WebGPU vs WebGL:** The `babylon` subpath uses `BABYLON.WebGPUEngine` and WGSL shaders. The `babylonGL` subpath uses `BABYLON.Engine` and GLSL shaders. The APIs are structurally identical but the engine types and shader languages differ. Not all generated effects are available in the GL variant (only 9 of the 22 postFX effects have GL versions).

2. **Scene management:** Each `CustomShaderEffect` creates its own `BABYLON.Scene` internally. This is by design for isolation but means creating many effects creates many scenes. Always call `dispose()` or `disposeAll()` when effects are no longer needed.

3. **FeedbackNode and cycles:** `FeedbackNode.setFeedbackSrc()` intentionally does NOT add the feedback source to the node's `inputs` map. This prevents cycles during `renderAll()` traversal. Because of this, `FeedbackNode` must be rendered separately or placed correctly in the render order. The feedback source's output is only swapped in after the first frame.

4. **Generated code -- do not edit:** All files under `generated/` are auto-generated by `@avtools/power2d-codegen`. Manual edits will be overwritten. To modify effects, edit the source definitions in `@avtools/power2d` and regenerate.

5. **Canvas inputs:** When passing an `HTMLCanvasElement` or `OffscreenCanvas` as a `ShaderSource`, the framework creates a `DynamicTexture` (WebGL) or uses `createDynamicTexture` + `updateDynamicTexture` (WebGPU). The canvas is re-uploaded to the GPU every frame during `render()`. This is convenient but not free; for high-performance scenarios, consider pre-rendering to a Babylon texture.

6. **Default resolution:** All effects default to 1280x720. Always pass explicit width/height when your target resolution differs.

7. **Sampler defaults:** All effects use `CLAMP_ADDRESSMODE` for wrap modes by default. The default sampling mode is `BILINEAR_SAMPLINGMODE` unless `sampleMode: 'nearest'` is specified. Fluid simulation stages typically use `'nearest'` for field data and `'linear'` for advection/dye.

8. **Pass texture routing:** For multi-pass effects (like Bloom with 2 passes), the `passTextureSources` specification controls which textures feed into which passes. By default, the primary texture is chained through passes (pass 1 reads pass 0's output). Custom routing allows any pass to read from any earlier pass or from external inputs.

9. **Disposing:** Call `dispose()` on individual effects or `disposeAll()` on the terminal effect to clean up. Failing to dispose will leak GPU resources (render targets, materials, scenes, meshes).

10. **CanvasPaint is terminal:** `CanvasPaint` renders to the screen framebuffer, not to a render target. Its `output` property exists for structural consistency but it renders to screen in `render()`. Do not chain another effect after `CanvasPaint`.

---

## Monorepo Relationships

```
@avtools/shader-fx
  depends on: babylonjs (npm)
  consumed by: @avtools/power2d (re-exports and uses shader effects)
              apps/browser-projections (uses effects for live visuals)
              apps/deno-notebooks (experimental usage)

@avtools/power2d-codegen
  generates: packages/shader-fx/generated/**
  reads: @avtools/power2d effect definitions
  writes: WGSL and GLSL .generated.ts files

@avtools/power2d
  defines: effect specifications (shader source, uniforms, passes)
  uses: @avtools/shader-fx/babylon as the runtime
```

The code generation flow is: effect definitions in `@avtools/power2d` are processed by `@avtools/power2d-codegen` to produce the `.generated.ts` files in `@avtools/shader-fx/generated/`. These generated files import the core framework from `@avtools/shader-fx/babylon` (or `babylonGL`) and export ready-to-use effect classes.
