---
name: figma-design
description: Access Figma designs, extract design systems, and retrieve component specifications. Use when implementing UI from Figma mockups, extracting design tokens, or analyzing design files.
metadata:
  short-description: Figma design system and component access
---

# Figma Design

Access Figma designs, design systems, and components for frontend implementation. Extract design tokens, component specifications, and visual assets to ensure pixel-perfect implementation.

## Quick Start

1) **Identify design source** - Get Figma file URL, frame names, or component IDs from designer
2) **Extract design tokens** - Use `get_design_system` or `export_tokens` for colors, typography, spacing
3) **Get component details** - Use `get_node_info`, `get_component` for structure, variants, states
4) **Download assets** - Use `download_design_assets` or `export_node_as_image` for images, icons
5) **Implement with fidelity** - Apply `frontend-philosophy` to extracted tokens, maintain visual quality

## Workflow

### 0) If any MCP call fails because Figma MCP is not connected, pause and set it up:

1. Add the Figma MCP:
   - `codex mcp add figma --url https://mcp.figma.com/mcp`
2. Enable remote MCP client:
   - Set `[features].rmcp_client = true` in `config.toml` **or** run `codex --enable rmcp_client`
3. Log in with OAuth (or configure API token):
   - `codex mcp login figma`
   - **OR** set `FIGMA_ACCESS_TOKEN` environment variable

After successful login, the user will have to restart codex. You should finish your answer and tell them so when they try again they can continue with Step 1.

### 1) Understand the Design Context

Before extracting any data, clarify:
- What Figma file or frame contains the design?
- Is there a design system or shared library?
- What component(s) need implementation?
- Are there specific variants or states (hover, active, disabled)?
- What framework is being used (React, Vue, HTML/CSS)?

**Tools to use:**
- `get_document_info` - Get file structure overview
- `get_file_nodes` - List all nodes in file
- `get_tree` - Export file structure as ASCII tree

### 2) Extract Design System Tokens

Design tokens are the foundation of consistent UI implementation.

**Colors:**
- `get_styles` - List all color styles
- `export_tokens({ format: "css" })` - Export as CSS variables
- `export_tokens({ format: "tailwind" })` - Export as Tailwind config
- `export_tokens({ format: "json" })` - Export as JSON

**Typography:**
- `get_styles` - List all text styles
- Extract font families, sizes, weights, line heights
- Map to `frontend-philosophy` distinctive fonts (avoid Inter, Roboto)

**Spacing & Layout:**
- `get_node_info` with `@layout` projection - Get padding, gaps, auto-layout
- Extract spacing scale (4px, 8px, 16px, 24px, etc.)
- Identify layout patterns (flexbox, grid)

**Effects:**
- `get_styles` - List effect styles (shadows, blurs)
- `get_css` - Extract exact CSS for shadows, borders, opacity

### 3) Get Component Specifications

For each component to implement:

**Structure:**
- `get_node_info({ node_id, select: ["@structure", "@bounds"] })` - Get hierarchy and dimensions
- `get_component` - Get component details and variants
- `list_local_components` - Audit all project components
- `list_remote_components` - Access team library components

**Styling:**
- `get_css({ node_id })` - Extract CSS properties (fills, strokes, effects, corner radius)
- `get_node_info({ node_id, select: ["@css"] })` - Get detailed styling data
- Note: Apply `frontend-philosophy` - enhance generic styles with intentional color and atmosphere

**Variants & States:**
- `get_component` returns all variants (Primary/Secondary, Small/Medium/Large)
- `get_node_info` for each variant to understand differences
- Map to component props (e.g., `variant="primary"`, `size="large"`)

**Accessibility:**
- Check for accessibility labels in Figma annotations
- Extract semantic roles (button, link, heading)
- Verify color contrast meets WCAG standards

### 4) Download Visual Assets

**Images & Icons:**
- `export_node_as_image({ node_id, format: "svg" })` - Export icons as SVG
- `export_node_as_image({ node_id, format: "png", scale: 2 })` - Export images at 2x
- `download_design_assets({ figma_url, local_path })` - Batch download all assets

**Reference Images:**
- `download_design_assets` includes `reference.png` for visual context
- Use reference image with `zai-vision_*` tool for visual comparison during implementation

### 5) Implement with Design Fidelity

Apply extracted tokens while maintaining `frontend-philosophy`:

**Typography:**
- If Figma uses generic fonts (Inter, Roboto), suggest distinctive alternatives
- Maintain hierarchy and scale from design tokens
- Apply proper font weights and line heights

**Color:**
- Extract color palette from Figma
- If palette is generic gray/blue, propose bold intentional alternatives
- Maintain contrast ratios for accessibility

**Spacing:**
- Use exact padding, margins, gaps from auto-layout properties
- Apply brave spatial composition (generous negative space OR controlled density)

**Effects:**
- Extract shadows, blurs, gradients from Figma
- Layer visual richness (gradient meshes, noise textures) per `frontend-philosophy`

**Motion:**
- Figma prototypes may specify transitions
- Add purposeful motion that enhances UX (not decorative)

### 6) Verify Implementation

**Visual Comparison:**
- Export Figma frame as PNG: `export_node_as_image`
- Screenshot implemented component
- Use `zai-vision_*` to compare for pixel-perfect accuracy

**Responsive Behavior:**
- Check Figma constraints (fixed, stretch, scale)
- Implement responsive breakpoints
- Test on multiple viewport sizes

**Accessibility:**
- Verify color contrast with Figma accessibility plugin data
- Add ARIA labels and semantic HTML
- Test keyboard navigation

## Available Tools

### Document Navigation
- `get_document_info` - Get file overview and structure
- `get_file_nodes` - List all nodes in file
- `get_tree` - Export file structure as ASCII tree with node IDs
- `get_selection` - Get currently selected elements in Figma
- `set_current_page` - Switch to specific page

### Design System Extraction
- `get_styles` - List all styles (color, text, effect, grid)
- `export_tokens` - Export design tokens to CSS/JSON/Tailwind
- `get_css` - Extract CSS properties from nodes
- `get_local_components` - Audit project components
- `get_remote_components` - Access team library components

### Component Analysis
- `get_node_info` - Inspect specific node properties (use projections: `@structure`, `@css`, `@layout`, `@typography`, `@tokens`)
- `get_nodes_info` - Batch inspect multiple nodes
- `get_component` - Get component details and variants
- `create_component_instance` - Use components in designs

### Asset Export
- `export_node_as_image` - Export as PNG/SVG/JPG
- `download_design_assets` - Batch download assets with reference image
- `scan_text_nodes` - Find all text layers for content audit

### Query API (Advanced)
- `query` - Use Figma Query DSL for token-efficient searches
  - Projections: `@structure`, `@bounds`, `@css`, `@layout`, `@typography`, `@tokens`, `@images`, `@all`
  - Filters: `$match`, `$eq`, `$in`, `$gt`, `$lt`
  - Example: `{ "from": ["COMPONENT"], "where": { "name": { "$match": "Button*" } }, "select": ["@structure", "@css"] }`

## Practical Workflows

### Design Handoff
**Goal:** Extract complete design specifications for implementation

1. `get_document_info` - Understand file structure
2. `get_tree` - Get node IDs for target frames
3. `export_tokens({ format: "tailwind" })` - Export design system
4. `get_node_info({ node_id, select: ["@all"] })` - Get component details
5. `download_design_assets` - Download assets + reference.png
6. Implement component using extracted data

### Design System Audit
**Goal:** Inventory design tokens and components for consistency

1. `get_styles` - List all color, text, effect styles
2. `list_local_components` - Get all components with usage count
3. `export_tokens({ format: "json" })` - Export tokens for analysis
4. Identify inconsistencies (duplicate colors, similar components)
5. Recommend consolidation or cleanup

### Component Library Migration
**Goal:** Convert Figma components to code components

1. `list_local_components` - Get component inventory
2. For each component:
   - `get_component` - Get variants and properties
   - `get_css` - Extract styling
   - `get_node_info({ select: ["@layout"] })` - Get layout properties
3. Generate component code (React, Vue, etc.)
4. Create Storybook stories or documentation

### Visual QA & Pixel Perfection
**Goal:** Ensure implementation matches design exactly

1. `export_node_as_image({ node_id, format: "png", scale: 2 })` - Export design
2. Screenshot implemented component
3. Use `zai-vision_*` for visual comparison
4. Identify discrepancies (spacing, colors, shadows)
5. Fix implementation and re-verify

### Design Token Synchronization
**Goal:** Keep code tokens in sync with Figma

1. `export_tokens({ format: "css" })` - Export current tokens
2. Compare with existing CSS variables or Tailwind config
3. Identify changes (new colors, updated spacing)
4. Update code design system
5. Document changes and notify team

## Tips for Maximum Productivity

- **Use projections for efficient queries** - `select: ["@structure", "@css"]` fetches only needed data
- **Batch node inspections** - Use `get_nodes_info` for multiple nodes instead of individual calls
- **Export tokens early** - Get design system tokens before diving into components
- **Download reference images** - Visual context helps with implementation decisions
- **Map Figma variants to props** - Component variants become component props (e.g., `variant`, `size`, `state`)
- **Use Query DSL for large files** - More token-efficient than fetching entire files
- **Cache component IDs** - Reuse node IDs across multiple queries
- **Verify with visual comparison** - Always compare exported design vs. implementation

## Troubleshooting

- **Authentication Errors**: Re-run OAuth (`codex mcp login figma`); verify workspace access; check API token permissions
- **File Not Found**: Verify Figma URL is correct and accessible; check file hasn't been moved or deleted
- **Node Not Found**: Use `get_tree` to find correct node IDs; verify node hasn't been deleted
- **Export Failures**: Check node type supports export (frames, components); verify export settings; try different format (PNG vs SVG)
- **Missing Fonts**: Pre-load custom fonts with `load_font_async` before setting font names
- **Component Import Timeout**: Use `getNodeByIdAsync` for local components; increase timeout for remote library components
- **Token Export Issues**: Verify design uses variables/styles (not raw values); check file has defined color/text styles
- **Rate Limits**: Batch operations; use Query DSL for efficient data fetching; implement exponential backoff

## Integration with Frontend Workflow

When working with the `frontend` agent:

1. **Load this skill first**: `skill("figma-design")` before implementation
2. **Extract design tokens** as foundation for styling
3. **Get component specs** for structure and variants
4. **Apply frontend-philosophy** to elevate generic Figma designs:
   - Replace generic fonts with distinctive alternatives
   - Enhance color palettes with bold, intentional choices
   - Add atmosphere and depth (gradients, shadows, textures)
   - Implement purposeful motion
5. **Verify visual fidelity** with exported reference images

See `reference/design-handoff-workflow.md` for detailed step-by-step process.

## References and Examples

- `reference/design-token-extraction.md` - How to export and apply design tokens
- `reference/component-implementation-guide.md` - Step-by-step component conversion
- `reference/figma-query-dsl.md` - Query DSL syntax and examples
- `examples/button-component-extraction.md` - Complete button component workflow
- `examples/design-system-migration.md` - Migrating Figma design system to code
