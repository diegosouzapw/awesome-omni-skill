---
name: architecture-diagrams
description: This skill should be used when the user asks to "create a diagram", "draw architecture", "make a Mermaid diagram", "update the system diagram", "visualize data flow", or when generating flowcharts, module diagrams, or dependency graphs. Provides standards for clear, unambiguous Mermaid diagrams.
---

# Architecture Diagram Standards

Apply these standards when creating or updating Mermaid diagrams.

## Core Rules

### Every Arrow Needs a Label

Unlabeled arrows force readers to guess the relationship.

```mermaid
%% BAD
A --> B

%% GOOD
A -->|float[] samples| B
A -->|HTTP 200| B
A -->|calls| B
```

### No Dead Ends

Every process node needs input AND output arrows. Data doesn't disappear.

```mermaid
%% BAD: normalize just ends
RawData --> Normalize

%% GOOD: show what normalized data becomes
RawData -->|int16[]| Normalize -->|float[] peak=1.0| Smoother
```

### Single Abstraction Level Per Diagram

Don't mix high-level modules with implementation functions.

| Level | Shows | Example Nodes |
|-------|-------|---------------|
| System | External boundaries | AudioJones, WASAPI, Display |
| Module | Internal components | audio.c, waveform.c, visualizer.c |
| Function | Implementation detail | ProcessWaveformBase, CubicInterp |

Create separate diagrams for each level.

### Connect All Subgraphs

Isolated subgraphs indicate missing relationships. If a subgraph modifies data elsewhere, show the arrow.

```mermaid
%% BAD: UI floats alone
subgraph UI
    Panel --> Slider
end
subgraph Core
    Config --> Render
end

%% GOOD: show what UI affects
subgraph UI
    Panel --> Slider
end
subgraph Core
    Config --> Render
end
Slider -->|modifies| Config
```

## Arrow Conventions

Pick ONE meaning per diagram and state it in a legend or title:

| Arrow Type | Meaning | Use When |
|------------|---------|----------|
| `-->` | Data flows from A to B | Showing data transformation pipelines |
| `-->` | A calls/invokes B | Showing control flow or dependencies |
| `-.->` | Async/event-based | Callbacks, message queues |
| `==>` | High-volume/critical path | Emphasizing main data path |

**Bidirectional**: Use two arrows with separate labels, not `<-->`.

```mermaid
Client -->|request| Server
Server -->|response| Client
```

## Required Elements

### Legend

Every diagram needs a legend explaining:

- Arrow meaning (data flow vs dependency)
- Shape meanings if non-obvious
- Any color coding

```mermaid
%% Legend:
%% → data flow (payload type on label)
%% [box] processing function
%% [(cylinder)] persistent buffer
%% {{diamond}} decision point
```

### Title

Descriptive title that clarifies arrow semantics:

- "Data Flow: Audio Samples to Screen" (arrows = data movement)
- "Module Dependencies" (arrows = import/call relationships)

## Mermaid Syntax Reference

### Labeled Edges

```mermaid
A -->|label| B      %% arrow with label
A -- label --> B    %% alternative syntax
A -.->|label| B     %% dotted with label
A ==>|label| B      %% thick with label
```

### Subgraph Direction

```mermaid
subgraph Module[Module Name]
    direction LR
    A --> B
end
```

Note: Subgraph direction ignored if nodes link outside the subgraph.

### Node Shapes

```mermaid
A[Rectangle]        %% process/function
B[(Database)]       %% persistent storage
C((Circle))         %% start/end point
D{Diamond}          %% decision
E{{Hexagon}}        %% preparation/setup
F[/Parallelogram/]  %% input/output
```

## Verification Checklist

Before finalizing any diagram:

- [ ] Every arrow has a label describing what flows/relationship
- [ ] No orphaned nodes or subgraphs
- [ ] Every process has both input and output arrows
- [ ] Single abstraction level throughout
- [ ] Legend explains arrow and shape meanings
- [ ] Title clarifies diagram's semantic intent

## Anti-patterns

| Problem | Fix |
|---------|-----|
| `A --> B --> C --> D` (unlabeled chain) | Add data type labels to each arrow |
| Subgraph with no external connections | Add arrows showing how it interacts |
| Function node with only input arrow | Show output or mark as "side effect: X" |
| Mixed modules and functions | Split into separate diagrams |
| Colors without legend | Add legend or remove colors |

## Example: Complete Data Flow

```mermaid
flowchart LR
    subgraph Capture[Audio Capture]
        CB[Callback] -->|int16[] stereo| RB[(Ring Buffer)]
    end

    subgraph Process[Waveform Processing]
        RB -->|int16[4096]| Norm[Normalize]
        Norm -->|float[1024] peak=1.0| Smooth[Smooth]
        Smooth -->|float[2048] palindrome| Interp[CubicInterp]
    end

    subgraph Render[Visualization]
        Interp -->|Vector2[]| Draw[DrawCircular]
        Draw -->|pixels| Accum[(accumTexture)]
        Accum -->|texture| Blur[Gaussian Blur]
        Blur -->|decayed texture| Accum
        Accum -->|final frame| Screen[Display]
    end

%% Legend:
%% → data flow with payload type
%% [(name)] persistent buffer
%% [name] processing function
```
