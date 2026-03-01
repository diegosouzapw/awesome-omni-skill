---
name: json-visualization-dev
description: Develop and maintain the JSON Visualization web application - a Next.js tool for visualizing JSON/YAML/CSV/XML data as interactive graphs. Use when working with this codebase, adding features, fixing bugs, or understanding the graph visualization, data conversion, or type generation systems.
license: See LICENSE.md in project root
compatibility: Requires Node.js >=24.x, pnpm package manager
metadata:
  author: json-visualization
  version: "1.0"
  tech-stack: "Next.js, React 19, TypeScript, Zustand, Styled-components, Mantine v8"
---

# JSON Visualization Development Skill

This skill helps you work with the JSON Visualization codebase - an open-source web application for visualizing and manipulating JSON data.

## When to use this skill

- Adding new features to the editor, graph visualization, or converters
- Fixing bugs in data parsing, rendering, or state management
- Understanding the codebase architecture and data flow
- Creating new data format converters or type generators
- Modifying UI components or styling

## Project overview

**What it does**: Converts JSON, YAML, CSV, XML, TOML into interactive graphs/trees with features like format conversion, validation, code generation, and image export.

**Tech stack**:
- Next.js (React 19) + TypeScript
- Zustand (state management)
- Styled-components + Mantine v8 (UI)
- Monaco Editor (code editor)
- Reaflow (graph visualization)

## Quick start

```bash
# Install dependencies
pnpm install

# Start dev server
pnpm dev  # http://localhost:3000

# Lint and format
pnpm lint
pnpm lint:fix
```

## Architecture

### Core directories

```
src/
├── pages/              # Next.js routes (index, editor, converters, type generators)
├── features/
│   ├── editor/        # Main editor (TextEditor, GraphView, TreeView, Toolbar)
│   └── modals/        # Import, Download, Type, Schema, JQ, JPath modals
├── store/             # Zustand stores (useFile, useConfig, useJson, useModal)
├── components/        # Reusable UI (buttons, animations, effects)
├── layout/            # Page layouts (Navbar, Footer, Landing sections)
├── lib/utils/         # Utilities (jsonAdapter, json2go, generateType, search)
├── hooks/             # Custom hooks (useFocusNode, useJsonQuery)
├── constants/         # Theme, global styles, graph config
├── enums/             # FileFormat, TypeLanguage, ViewMode
└── types/             # TypeScript type definitions
```

### Data flow

1. **Input** → `useFile` store → Parse via `jsonParser.ts` → Graph nodes
2. **Render** → `GraphView` (Reaflow + custom nodes/edges)
3. **Actions** → Toolbar → Modals → Store updates → Re-render

### Key files

- `src/store/useFile.ts` - File operations, content management (160 LOC)
- `src/features/editor/views/GraphView/lib/jsonParser.ts` - JSON → graph parser (207 LOC)
- `src/lib/utils/jsonAdapter.ts` - Format conversions (104 LOC)
- `src/features/editor/views/GraphView/index.tsx` - Main graph view (198 LOC)

## Code style guidelines

### TypeScript

```typescript
// ✅ Use type imports
import type { MenuItemProps } from "@mantine/core";

// ❌ Don't use regular imports for types
import { MenuItemProps } from "@mantine/core";
```

### Import order (enforced by Prettier)

1. React (`react`, `react/*`)
2. Next.js (`next`, `next/*`)
3. `@mantine/core`
4. Other `@mantine` packages
5. `styled-components`
6. Third-party modules
7. Internal `src/` imports
8. Relative imports (`./`, `../`)

### Naming conventions

- **Components**: PascalCase (`Navbar.tsx`, `GraphView.tsx`)
- **Hooks**: camelCase with `use` prefix (`useFocusNode.ts`)
- **Stores**: camelCase with `use` prefix, default export (`useFile.ts`)
- **Styled components**: Prefix with `Styled` (`StyledButton`)
- **Functions**: camelCase (`fetchUrl`, `setContents`)

### Formatting

- Double quotes only
- Semicolons required
- Max 100 chars per line
- No multiple empty lines
- Avoid parens for single arrow function params: `x => x * 2`

## Common tasks

### Adding a new converter

1. Create route in `src/pages/converter/[format1]-to-[format2].tsx`
2. Use `ConverterLayout/ToolPage.tsx` wrapper
3. Conversion logic is in `lib/utils/jsonAdapter.ts`

### Adding a new type generator

1. Create route in `src/pages/type/[format]-to-[language].tsx`
2. Use `TypeLayout/TypegenWrapper.tsx` wrapper
3. Generation logic in `lib/utils/generateType.ts` or `json2go.js`

### Creating a Zustand store

```typescript
import { create } from "zustand";

interface MyState {
  value: string;
}

interface MyActions {
  setValue: (value: string) => void;
}

const useMyStore = create<MyState & MyActions>()((set, get) => ({
  value: "",
  setValue: value => set({ value }),
}));

export default useMyStore;
```

### Creating a styled component

```typescript
import styled from "styled-components";

const StyledButton = styled.button`
  background-color: #f7c948;
  color: #1a1a1a;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  transition: all 0.3s ease;

  &:hover {
    background-color: #37ff8b;
  }
`;
```

## Design system

**Colors**:
- Background: `#f7f3e6` (warm beige)
- Primary text: `#1a1a1a`
- Accent: `#37ff8b` (neon green)
- Yellow: `#f7c948`

**Fonts**:
- Global: Playfair Display (serif)
- Code: JetBrains Mono (monospace)

## Important notes

- **No tests**: Project has no test suite - don't create tests unless requested
- **Package manager**: Use `pnpm` only (not npm/yarn)
- **Privacy-first**: All processing is client-side
- **Node version**: >=24.x required
- **ESLint**: `src/enums` directory is ignored

## Getting help

- See [references/ARCHITECTURE.md](references/ARCHITECTURE.md) for detailed architecture
- See [references/COMPONENTS.md](references/COMPONENTS.md) for component catalog
- See [references/STATE.md](references/STATE.md) for state management patterns
- Check `AGENTS.md` in project root for full guidelines
