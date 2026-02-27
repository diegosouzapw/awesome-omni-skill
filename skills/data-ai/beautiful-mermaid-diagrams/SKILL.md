---
name: beautiful-mermaid-diagrams
description: Create beautiful diagrams using Mermaid syntax including flowcharts, sequence diagrams, class diagrams, ER diagrams, and state diagrams. Use when users ask to diagram, visualize, model, map out, or show the flow of systems, processes, architectures, or interactions.
---

# Beautiful Mermaid Diagrams

## Conventions

- MUST use English for all text in the diagram.

## Quick syntax reference

### Flowcharts

```mermaid
graph TD
  A[Start] --> B[Process] --> C[End]
```

### Sequence diagrams

```mermaid
sequenceDiagram
  Alice->>Bob: Hello
  Bob-->>Alice: Hi
```

### Class diagrams

```mermaid
classDiagram
  Animal <|-- Duck
  Animal : +String name
  Animal : +int age
```

### ER diagrams

```mermaid
erDiagram
  CUSTOMER ||--o{ ORDER : places
  CUSTOMER {int id PK}
```

## Diagram types by use case

| Use case         | Diagram type      | Keywords                        |
| ---------------- | ----------------- | ------------------------------- |
| Process flows    | `graph TD/LR`     | journeys, algorithms, workflows |
| API interactions | `sequenceDiagram` | requests, OAuth, transactions   |
| System design    | `classDiagram`    | models, MVC, patterns           |
| Database schemas | `erDiagram`       | entities, relationships, tables |
| State machines   | `stateDiagram-v2` | lifecycle, workflows, states    |

## Common shapes

| Shape     | Syntax     | Description          |
| --------- | ---------- | -------------------- |
| Rectangle | `[text]`   | Default node         |
| Rounded   | `(text)`   | Rounded corners      |
| Diamond   | `{text}`   | Decision/condition   |
| Stadium   | `([text])` | Start/end/process    |
| Circle    | `((text))` | Endpoint             |
| Database  | `[(text)]` | Data store           |
| Hexagon   | `{{text}}` | Alternative decision |

## Edge styles

| Style    | Syntax | Use case        |
| -------- | ------ | --------------- |
| Solid    | `-->`  | Default flow    |
| Dotted   | `-.->` | Optional/dashed |
| Thick    | `==>`  | Highlighted     |
| No arrow | `---`  | Related only    |

## Examples by type

| Diagram type      | Examples                                                         |
| ----------------- | ---------------------------------------------------------------- |
| Flowcharts        | [reference/flowcharts.md](reference/flowcharts.md)               |
| Sequence diagrams | [reference/sequence-diagrams.md](reference/sequence-diagrams.md) |
| Class diagrams    | [reference/class-diagrams.md](reference/class-diagrams.md)       |
| ER diagrams       | [reference/er-diagrams.md](reference/er-diagrams.md)             |
| State diagrams    | [reference/state-diagrams.md](reference/state-diagrams.md)       |

## Styling options

### Inline styles

```mermaid
graph TD
  A[Node] --> B[Custom]
  style B fill:#3b82f6,stroke:#1d4ed8
```

### Class definitions

```mermaid
graph TD
  A:::highlight --> B:::default
  classDef highlight fill:#fbbf24,stroke:#d97706
  classDef default fill:#f4f4f5,stroke:#a1a1aa
```

## Preset color themes

### Semantic colors

Use colors to convey meaning consistently across diagrams.

| Color         | Hex                   | Meaning             | Use for                        |
| ------------- | --------------------- | ------------------- | ------------------------------ |
| Green         | `#10b981` / `#059669` | Success, positive   | Completed, approved, active    |
| Red           | `#ef4444` / `#dc2626` | Error, danger       | Failed, rejected, error states |
| Yellow/Orange | `#f59e0b` / `#d97706` | Warning, caution    | Pending, review needed, alerts |
| Blue          | `#3b82f6` / `#1d4ed8` | Information, action | Links, processes, buttons      |
| Gray          | `#6b7280`             | Neutral, disabled   | Inactive, optional, background |

### Database brand colors

Use brand colors for common database systems.

| Database      | Shape      | Brand Color      | Example               |
| ------------- | ---------- | ---------------- | --------------------- |
| MongoDB       | `[(name)]` | Green `#00ED64`  | `DB[(MongoDB)]`       |
| PostgreSQL    | `[(name)]` | Blue `#4169E1`   | `DB[(PostgreSQL)]`    |
| MySQL         | `[(name)]` | Orange `#F29111` | `DB[(MySQL)]`         |
| Redis         | `[(name)]` | Red `#DC382D`    | `DB[(Redis)]`         |
| Elasticsearch | `[(name)]` | Yellow `#F4B400` | `DB[(Elasticsearch)]` |
| SQLite        | `[(name)]` | Blue `#003B57`   | `DB[(SQLite)]`        |

### Quick theme presets

Copy these classDef statements for common themes.

**Modern blue theme** (default for most diagrams):
```mermaid
graph TD
  A[Start]:::blue --> B[Process]:::blue --> C[End]:::blue
  classDef blue fill:#3b82f6,stroke:#1d4ed8,color:#fff
```

**Status theme** (success/warning/error):
```mermaid
graph TD
  A[Success]:::success --> B[Warning]:::warning --> C[Error]:::error
  classDef success fill:#10b981,stroke:#059669,color:#fff
  classDef warning fill:#f59e0b,stroke:#d97706,color:#fff
  classDef error fill:#ef4444,stroke:#dc2626,color:#fff
```

**Architecture theme** (frontend/backend/data layers):
```mermaid
graph LR
  A[Frontend]:::frontend --> B[Backend]:::backend --> C[(Database)]:::database
  classDef frontend fill:#3b82f6,stroke:#1d4ed8,color:#fff
  classDef backend fill:#8b5cf6,stroke:#7c3aed,color:#fff
  classDef database fill:#00ED64,stroke:#00b33c,color:#000
```
