---
name: draw-io
description: Generate polished draw.io diagrams for Azure architectures with WAF-aligned design, proper layering, trust boundaries, and professional styling. Use when asked for "draw.io", "architecture diagram", "Azure diagram", "solution diagram", "WAF", "landing zone", "private endpoint", "hub-spoke", or any visual diagram in .drawio format. Outputs XML with Azure color palette, proper z-ordering, and audience-aware structure.
---

# draw.io Diagram Generation Skill

## Overview

This skill enables generation of high-quality draw.io diagrams—especially **Azure architecture diagrams**—by directly editing XML. It applies:
- **Azure best practices**: Service naming, color palette, layering patterns
- **WAF (Well-Architected Framework) thinking**: Visual cues for security, reliability, performance, cost, and operations
- **Audience-aware design**: Execs see boundaries and risk; engineers see dataflows and services

## When to Choose draw.io vs Other Formats

| Format | Best For | Trade-offs |
|--------|----------|------------|
| **draw.io (XML)** | Polished diagrams with Azure icons; customer-ready exports; high control | More effort; XML to maintain |
| **Mermaid** | Quick embeds in Markdown; version-controllable; fast iteration | No Azure icons; limited styling |
| **Canvas (.canvas)** | Interactive Obsidian presentation; drag-to-tweak | Obsidian-only; harder to share |

**Use draw.io when**: Customer expects a professional diagram for slides/docs, or you need Azure icons and precise control.

## Quick Start

When creating a draw.io diagram:

1. Use **Azure icon library** (`img/lib/azure2/...`) for official Azure icons
2. Set `defaultFontFamily="Segoe UI"` in `mxGraphModel` (Azure-style)
3. Use **container patterns**: gray fill for logical groups, white for service plans
4. Use correct **layering order** to avoid “missing arrows”:
  - Background containers FIRST (tiers/boundaries, especially anything with a fillColor)
  - Edges NEXT (connectors)
  - Foreground vertices LAST (service icons/boxes + text labels)
5. **Edges need terminals or points** to render predictably:
  - Use `source="id" target="id"` attributes (auto-routing), OR
  - Use `<mxPoint as="sourcePoint"/>` and `<mxPoint as="targetPoint"/>` when an edge is not connected to a vertex
6. Use **solid blue edges** for data flow, **dashed edges** for monitoring/auth
7. **Use separate text elements for arrow labels** - NOT edge labels (see "Arrow Label Placement Rules")
8. Set `labelBackgroundColor=none;` (or specific color if needed) to avoid default white background artifacts
9. Set `page="0"` to disable page view (useful for transparent exports)
10. **Align components** for straight lines (same center y for horizontal, same center x for vertical)
11. Set `dx`/`dy` to keep the diagram origin in the top-left viewport (these are editor offsets, not canvas size)
12. Verify with PNG export

---

## Azure Icon Library

draw.io includes official Azure icons at `img/lib/azure2/`. Always prefer these over colored rectangles for professional diagrams.

### Using Azure Icons

```xml
<!-- Azure icon with label below -->
<mxCell id="app-service" value="Web App"
  style="aspect=fixed;html=1;points=[];align=center;image;fontSize=12;image=img/lib/azure2/compute/App_Services.svg;labelPosition=center;verticalLabelPosition=bottom;verticalAlign=top;labelBackgroundColor=none;"
  vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="64" height="64" as="geometry" />
</mxCell>
```

### Key Icon Paths

| Service | Icon Path | Notes |
|---------|-----------|-------|
| App Service | `img/lib/azure2/compute/App_Services.svg` | |
| Functions | `img/lib/azure2/compute/Function_Apps.svg` | |
| Container Apps / Instances | `img/lib/azure2/compute/Container_Instances.svg` | Use for both ACA and ACI |
| AKS | `img/lib/azure2/compute/Kubernetes_Services.svg` | |
| SQL Database | `img/lib/azure2/databases/SQL_Database.svg` | |
| Cosmos DB | `img/lib/azure2/databases/Azure_Cosmos_DB.svg` | |
| Redis Cache | `img/lib/azure2/databases/Cache_Redis.svg` | |
| Front Door | `img/lib/azure2/networking/Front_Doors.svg` | |
| APIM | `img/lib/azure2/app_services/API_Management_Services.svg` | |
| AI Search | `img/lib/azure2/app_services/Search_Services.svg` | |
| **AI Foundry** | `img/lib/azure2/ai_machine_learning/AI_Studio.svg` | Correct icon (not Machine_Learning) |
| Azure OpenAI | `img/lib/azure2/ai_machine_learning/Azure_OpenAI.svg` | Or use AI_Studio for Foundry models |
| Entra ID | `img/lib/azure2/identity/Azure_Active_Directory.svg` | |
| Azure Monitor | `img/lib/azure2/management_governance/Monitor.svg` | |
| Log Analytics | `img/lib/azure2/analytics/Log_Analytics_Workspaces.svg` | |
| App Insights | `img/lib/azure2/devops/Application_Insights.svg` | |
| Service Bus | `img/lib/azure2/integration/Service_Bus.svg` | |

See [[reference]] for complete icon library paths with XML examples.

**Full icon catalog**: [[azure-icons]] lists all 648 Azure icons by category (for discovery when you need an icon not listed above).

---

## Azure Architecture Principles

### Layering (Left-to-Right Flow)

Structure diagrams with clear horizontal layers:

```
Users/Clients → Edge/Gateway → Compute/App → Data → External/Dependencies
     ↓              ↓             ↓          ↓            ↓
  (left)     APIM/FrontDoor   App/AKS/ACA  Cosmos/SQL   3rd party
```

### Trust Boundaries (Always Show)

Every Azure diagram should show:
- **Internet / Untrusted**: External clients, public endpoints
- **Azure Subscription / VNet**: Your private network boundary
- **Data plane vs Control plane**: Where management vs runtime traffic flows
- **On-prem / Customer Network**: If hybrid connectivity exists

Use **dashed rectangles** or **grouped containers** for boundaries:

```xml
<!-- Trust boundary container -->
<mxCell id="vnet-boundary" value="Azure VNet (Private)" 
  style="rounded=0;whiteSpace=wrap;html=1;dashed=1;dashPattern=8 4;fillColor=none;strokeColor=#0078D4;strokeWidth=2;verticalAlign=top;fontFamily=Segoe UI;fontSize=14;fontStyle=1;"
  vertex="1" parent="1">
  <mxGeometry x="200" y="80" width="400" height="300" as="geometry" />
</mxCell>
```

### Non-Azure Entities (Generic Shapes)

**CRITICAL**: Don't use Azure icons for non-Azure things. Use generic shapes:

| Entity | Recommended Style | Why |
|--------|-------------------|-----|
| **On-Premises** | Gray rounded box (`fillColor=#E6E6E6;strokeColor=#666666;`) | No good "datacenter" icon in Azure library |
| **External Users** | Person icon (`shape=actor;`) or Gray/white box | `shape=actor` is the standard "stick figure" icon |
| **3rd Party SaaS** | White box with border | Distinguish from Azure services |
| **Internet** | Cloud shape or labeled box | Use `shape=cloud;` if needed |

```xml
<!-- On-Premises (generic gray box) -->
<mxCell id="onprem" value="On-Premises"
  style="rounded=1;whiteSpace=wrap;html=1;fillColor=#E6E6E6;strokeColor=#666666;fontFamily=Segoe UI;fontSize=12;fontStyle=1;labelBackgroundColor=none;"
  vertex="1" parent="1">
  <mxGeometry x="40" y="145" width="100" height="50" as="geometry" />
</mxCell>

<!-- External Users (Persona Icon) -->
<mxCell id="user-persona" value="Customer"
  style="shape=actor;whiteSpace=wrap;html=1;fillColor=#D4E1F5;strokeColor=#0078D4;fontFamily=Segoe UI;fontSize=12;labelPosition=center;verticalLabelPosition=bottom;verticalAlign=top;"
  vertex="1" parent="1">
  <mxGeometry x="40" y="160" width="30" height="60" as="geometry" />
</mxCell>
```

**Avoid**: `On_Premises_Data_Gateways.svg` — this is an Azure service icon (cloud-based gateway), not a datacenter!

### Azure Color Palette

| Component Type | Fill Color | Stroke Color | Use For |
|---------------|------------|--------------|---------|
| **Compute (blue)** | `#E8F4FD` | `#0078D4` | App Service, AKS, Container Apps, Functions, VMs |
| **Data (green)** | `#E6F4EA` | `#107C10` | Cosmos DB, SQL, Storage, Redis, PostgreSQL |
| **Integration (purple)** | `#F3E8FF` | `#8661C5` | Service Bus, Event Grid, Event Hubs, Logic Apps |
| **Network/Edge (teal)** | `#E0F7FA` | `#008272` | APIM, Front Door, App Gateway, Load Balancer, VNet |
| **Identity (orange)** | `#FFF4E5` | `#FF8C00` | Entra ID, Managed Identity, Key Vault |
| **AI/ML (gradient blue)** | `#E3F2FD` | `#1976D2` | Azure OpenAI, AI Foundry, Cognitive Services, ML |
| **Monitoring (yellow)** | `#FFF8E1` | `#FFC107` | Azure Monitor, App Insights, Log Analytics |
| **Security (red accent)** | `#FFEBEE` | `#D32F2F` | WAF, Defender, Private Endpoints, NSG |

### Service Naming Convention

Name boxes by **role first, then service**:
- ✅ "API Gateway (APIM)"
- ✅ "App Backend (Container Apps)"
- ✅ "Vector Store (AI Search)"
- ❌ "Azure API Management" (too generic)
- ❌ "apim-prod-westeu-001" (too specific)

### Flow Numbering

Number flows 1→N for readability. Label arrows with **intent**, not protocol:
- ✅ "1. User authenticates"
- ✅ "2. Query knowledge base"
- ✅ "3. Generate response"
- ❌ "HTTPS 443"
- ❌ "REST API call"

---

## Edge Patterns (Arrows)

**DECISION POINT**: You are NOT required to use arrows for every relationship.
- **Use Arrows** for: Data flow, network connectivity, sequence of operations.
- **skip Arrows** when:
  - **Containment** implies relationship (e.g., App Service inside a VNet).
  - **Proximity** is clear enough (e.g., an "Identity" label next to Entra ID).
  - **Clutter** would result from too many lines (matrix of connections).
  - **Grouping** shows the association (e.g., all services in a "Management" box).

**Be decisive:** If the diagram is cleaner without an arrow, omit it.

Use consistent arrow styles to distinguish flow types:

### Data Flow (Solid Blue)
Primary request/response flow through the architecture:
```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;labelBackgroundColor=none;"
```

### Monitoring / Diagnostic Flow (Dashed)
Telemetry, logs, and metrics flowing to monitoring services:
```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;dashed=1;labelBackgroundColor=none;"
```

### Authentication / External Service Flow (Dashed)
Calls to identity providers, DNS, external APIs. If you need a label, use a separate text element (see "Arrow Label Placement Rules").
```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;dashed=1;labelBackgroundColor=none;"
```

### Cache / Return Flow (Directional)
Data retrieval from cache or return paths:
```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;endFill=0;fillColor=#dae8fc;strokeColor=#6c8ebf;labelBackgroundColor=none;"
```

Use a second edge in the opposite direction if you need an explicit return path.

### Edge Label Positioning

> **⚠️ WARNING**: Edge labels (`value="..."` on edges) have **unreliable positioning**. The offset values are inconsistent and labels often end up ON the line or in wrong positions. **Use separate text elements instead** - see "Arrow Label Placement Rules" section below for exact formulas.

**Legacy approach (NOT RECOMMENDED)**:

```xml
<!-- Label BELOW the line (positive y offset) - UNRELIABLE -->
<mxCell id="flow1" value="1. Auth" edge="1" 
  style="edgeStyle=orthogonalEdgeStyle;rounded=0;...">
  <mxGeometry relative="1" x="-0.3" y="15" as="geometry">
    <mxPoint as="offset" />
  </mxGeometry>
</mxCell>
```

**Recommended approach**: Use separate text elements with calculated positions. See "Arrow Label Placement Rules" section.

### Explicit Edge Endpoints

> **⚠️ CRITICAL**: Edges should have connection information for predictable rendering. Use EITHER:
> - `source="vertex-id"` and `target="vertex-id"` attributes (auto-connects to cell centers), OR
> - `<mxPoint x="..." y="..." as="sourcePoint" />` and `<mxPoint ... as="targetPoint" />` elements (explicit coordinates)
>
> Edges missing BOTH can render invisibly.

For precise control over where arrows start/end (instead of auto-connecting to cell centers):

```xml
<!-- Option 1: Reference vertex IDs (auto-routing) -->
<mxCell id="flow1" edge="1" source="apim" target="app"
  style="edgeStyle=orthogonalEdgeStyle;rounded=0;...">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- Option 2: Explicit coordinates (precise control) -->
<mxCell id="flow4" edge="1"
  style="edgeStyle=orthogonalEdgeStyle;rounded=0;...">
  <mxGeometry relative="1" as="geometry">
    <mxPoint x="468" y="196" as="sourcePoint" />
    <mxPoint x="560" y="196" as="targetPoint" />
  </mxGeometry>
</mxCell>

<!-- Option 3: Mixed (source vertex, explicit target) -->
<mxCell id="flow3" edge="1" source="apim"
  style="edgeStyle=orthogonalEdgeStyle;rounded=0;...">
  <mxGeometry relative="1" as="geometry">
    <mxPoint x="400" y="196" as="targetPoint" />
  </mxGeometry>
</mxCell>
```

Use explicit sourcePoint/targetPoint when:
- Icons have inconsistent anchor points
- You need arrows to connect at specific spots (not center)
- Auto-routing creates awkward paths
- Edge has no source/target vertex to reference

### Minimizing Edge Bends

**Cleaner diagrams have fewer bends in edges.** Apply these principles:

#### 1. Align Components for Straight Lines
Position components so most connections are straight (0 bends) or have 1 bend max:
```
GOOD: Components aligned → straight arrow
┌──────┐         ┌──────┐
│  A   │────────▶│  B   │
└──────┘         └──────┘

BAD: Misaligned → multiple bends
┌──────┐
│  A   │──┐
└──────┘  │
          └──────▶┌──────┐
                  │  B   │
                  └──────┘
```

**Pro tip**: Match y-coordinates within 5px (e.g., source at y=118, target at y=115) → virtually straight horizontal arrow. Small differences are acceptable; large gaps create bends.

**For vertical connections** (exitX=0.5 → entryX=0.5 or entryY=0), align component **centers** exactly:
```
source_center_x = source.x + (source.width / 2)
target.x = source_center_x - (target.width / 2)
```
Example: Functions at x=400, width=68 → center=434. Monitor (width=64) should be at x=402 (402+32=434).

#### 2. Use Grid-Based Layout
- Place components on a consistent grid (e.g., x increments of 150-200px)
- Align rows by center: same `(y + height/2)` for a clean horizontal line
- Align columns by center: same `(x + width/2)` for a clean vertical line

#### 3. Edge Routing Style
Use `edgeStyle=orthogonalEdgeStyle` (default) but position components to minimize resulting bends:
- **Horizontal flow**: Place source and target at the same center y → 0 bends
- **Vertical flow**: Place source and target at the same center x → 0 bends
- **Diagonal relationship**: Accept 1-2 bends, not more

#### 4. When Bends Are Necessary
If a bend is unavoidable (e.g., routing around obstacles):
- Use **consistent bend points** (e.g., all horizontal segments at y=200)
- Avoid overlapping edges by staggering bend points (+10px offset each)
- Consider `curved=1` for softer appearance (but use sparingly)

```xml
<!-- Orthogonal edge with one clean bend -->
style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;"

<!-- Curved edge (softer, but can be harder to follow) -->
style="edgeStyle=orthogonalEdgeStyle;curved=1;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;"
```

#### 5. Layout Validation
Before finalizing:
- [ ] Most edges have 0-1 bends
- [ ] No edges with 3+ bends (redesign component positions)
- [ ] No overlapping edge segments
- [ ] Flow direction is clear (left→right or top→bottom)

---

## Arrow Label Placement Rules (MANDATORY)

**CRITICAL**: Do NOT use edge labels (`value="..."` on edge elements). Edge label positioning is unreliable and inconsistent. Always use **separate text elements** with calculated positions.

### Why Separate Text Elements?

Edge labels have unpredictable behavior:
- Position depends on edge routing (changes when components move)
- Offset values (`x`, `y` in geometry) behave inconsistently
- Labels can end up ON the line, making them unreadable

**Solution**: Create `<mxCell style="text;...">` elements with exact `x`, `y` coordinates.

### Horizontal Arrow Labels

**Rule**: ALL labels for horizontal arrows go **ABOVE** the line.

**Calculation**:
```
arrow_y = source_y + (source_height / 2)  // horizontal center of icons
label_y = arrow_y - 25                      // 25px above the arrow
label_x = ((source_x + source_width) + target_x) / 2 - (label_width / 2)  // centered
```

**Example**:
```
Source icon: x=240, width=65  → right edge = 305
Target icon: x=398            → left edge = 398
Arrow y ≈ 219 (icon centers)

label_x = (305 + 398) / 2 - (60 / 2) = 351.5 - 30 = 321
label_y = 219 - 25 = 194
```

```xml
<mxCell id="label3" value="3. Route"
  style="text;html=1;align=center;verticalAlign=middle;fontFamily=Segoe UI;fontSize=11;labelBackgroundColor=none;fontColor=#0078D4;"
  vertex="1" parent="1">
  <mxGeometry x="321" y="194" width="60" height="20" as="geometry" />
</mxCell>
```

### Vertical Arrow Labels

**Rule**: ALL labels for vertical arrows go **TO THE RIGHT** of the line.

**Calculation**:
```
arrow_x = source_x + (source_width / 2)     // vertical center of icons
label_x = arrow_x + 20                        // 20px to the right
label_y = ((source_y + source_height) + target_y) / 2 - (label_height / 2)  // centered
```

**Example**:
```
Source icon: x=540, width=64, y=185, height=68  → center_x=572, bottom=253
Target icon: y=350                               → top=350

label_x = 572 + 20 = 592 (round to 595)
label_y = (253 + 350) / 2 - 10 = 301.5 - 10 = 291 (round to 290)
```

```xml
<mxCell id="label5" value="5. Search"
  style="text;html=1;align=left;verticalAlign=middle;fontFamily=Segoe UI;fontSize=11;labelBackgroundColor=none;fontColor=#107C10;"
  vertex="1" parent="1">
  <mxGeometry x="595" y="290" width="60" height="20" as="geometry" />
</mxCell>
```

### Boundary Crossing Rules

When an arrow crosses a container boundary (e.g., VNet), place the label on the **source side** of the boundary with **5px clearance**.

**Rules**:
- **Left → Right** into a boundary: `label_right_edge < boundary_x - 5`
- **Right → Left** into a boundary: `label_left_edge > boundary_right + 5`
- **Top → Bottom** into a boundary: `label_bottom < boundary_y - 5`
- **Bottom → Top** into a boundary: `label_top > boundary_bottom + 5`

**Example**: Arrow from Users (external) to APIM (inside VNet at x=200)
```
VNet boundary x = 200
label_width = 65
label_right_edge must be < 195

label_x = 195 - 65 = 130  ✓ (label ends at 195, 5px before VNet)
```

```xml
<!-- Label for arrow crossing INTO VNet - placed outside -->
<mxCell id="label2" value="2. Request"
  style="text;html=1;align=center;verticalAlign=middle;fontFamily=Segoe UI;fontSize=11;labelBackgroundColor=none;fontColor=#0078D4;"
  vertex="1" parent="1">
  <mxGeometry x="130" y="194" width="65" height="20" as="geometry" />
</mxCell>
```

### Icon Label Clearance from Boundaries

Icon labels (below icons using `verticalLabelPosition=bottom`) need clearance from container edges.

**Rule**: `icon_y + icon_height + 18 (label) + 40 (clearance) < container_bottom`

**Example**:
```
VNet bottom = 460 (y=40, height=420)
Required clearance = 40px
Label height ≈ 18px

Maximum icon bottom = 460 - 40 - 18 = 402
If icon height = 52, max icon_y = 402 - 52 = 350
```

### Component Alignment for Straight Lines

**Horizontal flows**: Align icon centers at the same **y** (`y + height/2`).

```
apim_y = 189      (height=60)  → center = 219
app_y = 185       (height=68)  → center = 219  ✓
foundry_y = 185   (height=68)  → center = 219  ✓
openai_y = 185    (height=68)  → center = 219  ✓
```

**Vertical flows**: Icons at **same x-center**.

```
source: x=540, width=64  → center = 572
target: x=536, width=72  → center = 572  ✓ (536 + 72/2 = 572)
```

### Quick Reference Table

| Arrow Direction | Label Position | Gap | Alignment |
|-----------------|----------------|-----|-----------|
| Horizontal (→)  | ABOVE line     | 25px | `align=center` |
| Vertical (↓)    | RIGHT of line  | 20px | `align=left` |
| Diagonal        | ABOVE or RIGHT | 20px | Context-dependent |
| Crossing boundary | SOURCE side  | 5px from boundary | - |

### Validation Checklist

Before finalizing labels:
- [ ] All horizontal labels at consistent y (same row = same label_y)
- [ ] All vertical labels at consistent x offset (+20px from arrow)
- [ ] No labels crossing container boundaries
- [ ] No labels overlapping icons or other labels
- [ ] Icon labels have 40px clearance from container bottom

---

## Container & Grouping Patterns

Enterprise diagrams use containers for logical structure:

### Resource Group Boundary (Outer Dashed)
Wraps entire diagram to show scope:
```xml
style="rounded=0;whiteSpace=wrap;html=1;fillColor=none;dashed=1;labelBackgroundColor=none;"
```

### Logical Tier Container (Gray Fill, No Stroke)
Groups related services (App Tier, Data Tier, Monitoring):
```xml
style="rounded=0;whiteSpace=wrap;html=1;dashed=1;labelBackgroundColor=none;fillColor=#E6E6E6;strokeColor=none;"
```

### Service Plan Group (White Container)
Groups service with its hosting plan (e.g., App Service in App Service Plan):
```xml
<!-- Parent group (connectable="0") -->
style="group;labelBackgroundColor=none;strokeColor=none;"

<!-- White inner container with label -->
style="rounded=0;whiteSpace=wrap;html=1;dashed=1;labelBackgroundColor=none;fillColor=#FFFFFF;labelPosition=center;verticalLabelPosition=top;align=center;verticalAlign=bottom;spacingBottom=-35;strokeColor=none;"
value="App Service Plan"
```

See [[reference]] for complete grouping patterns with XML examples.

---

## Comparison Diagrams (Current vs Target)

When showing "before and after" or "current vs target" architectures:

### Layout Pattern
- Use **two side-by-side containers** of equal width
- Left = Current State (use orange/red border: `strokeColor=#E65100;`)
- Right = Target State (use green border: `strokeColor=#2E7D32;`)
- Add clear title labels at top of each container
- Maintain consistent vertical alignment between sides

### Color Psychology
| Color | Meaning | Stroke | Fill |
|-------|---------|--------|------|
| **Red/Orange** | Current state, problems, gaps | `#E65100` | `#FFF3E0` |
| **Green** | Target state, benefits, resolved | `#2E7D32` | `#E8F5E9` |
| **Gray** | Neutral, unchanged | `#666666` | `#F5F5F5` |
| **Blue** | Primary data flow | `#0078D4` | `#E8F4FD` |

### Annotations (Warnings, Checkmarks, Benefits)

**DON'T**: Float warning text loosely next to boxes
**DO**: Use styled annotation boxes or embed in the component

```xml
<!-- Warning annotation (red box, attached to component) -->
<mxCell id="warning1" value="⚠ No OCR&#xa;⚠ Images skipped"
  style="rounded=1;whiteSpace=wrap;html=1;fontFamily=Segoe UI;fontSize=11;fillColor=#FFEBEE;strokeColor=#D32F2F;fontColor=#C62828;align=left;spacingLeft=5;"
  vertex="1" parent="1">
  <mxGeometry x="200" y="150" width="100" height="40" as="geometry" />
</mxCell>

<!-- Benefit annotation (green box, attached to component) -->
<mxCell id="benefit1" value="✓ Full OCR + images&#xa;✓ Table extraction"
  style="rounded=1;whiteSpace=wrap;html=1;fontFamily=Segoe UI;fontSize=11;fillColor=#E8F5E9;strokeColor=#2E7D32;fontColor=#1B5E20;align=left;spacingLeft=5;"
  vertex="1" parent="1">
  <mxGeometry x="500" y="150" width="120" height="40" as="geometry" />
</mxCell>
```

---

## Text & Box Sizing Rules

Avoid text overflow and cramped layouts:

### Calculate Box Width
- **English text**: ~7-8px per character for `fontSize=12`
- **Minimum width**: 80px for any labeled box
- **Multi-line**: Use `whiteSpace=wrap;` and set a fixed width
- **Rule of thumb**: Width = (longest line chars × 8) + 20px padding

### Example: Fitting "Python Library (No OCR, No Images)"
- 35 characters × 8px = 280px + 20px padding = **~300px minimum width**
- Or split to 2 lines: "Python Library" + "(No OCR, No Images)" → width ~150px

### Nested Container Spacing
- **Minimum padding**: 15px between parent container edge and child boxes
- **Minimum gap**: 10px between sibling boxes
- **Label clearance**: 25px from top of container to first child (for title)

```xml
<!-- Container with proper padding -->
<mxCell id="container" value="Document Processing"
  style="rounded=0;whiteSpace=wrap;html=1;fillColor=#E8F5E9;strokeColor=#2E7D32;verticalAlign=top;fontFamily=Segoe UI;fontSize=14;fontStyle=1;spacingTop=5;"
  vertex="1" parent="1">
  <mxGeometry x="400" y="60" width="280" height="200" as="geometry" />
</mxCell>

<!-- First child 15px from edges, 25px from top (after label) -->
<mxCell id="child1" ...>
  <mxGeometry x="415" y="95" width="100" height="60" as="geometry" />
</mxCell>
```

### Title & Subtitle Pattern
Don't cram metadata into the title. Use a separate subtitle text element:

```xml
<!-- Main title -->
<mxCell id="title" value="CZ AI Platform: Current vs Target Architecture"
  style="text;html=1;align=center;fontFamily=Segoe UI;fontSize=20;fontStyle=1;labelBackgroundColor=none;"
  vertex="1" parent="1">
  <mxGeometry x="200" y="10" width="500" height="30" as="geometry" />
</mxCell>

<!-- Subtitle (audience, date) -->
<mxCell id="subtitle" value="Audience: D&amp;A Leadership | Jan 28, 2026"
  style="text;html=1;align=center;fontFamily=Segoe UI;fontSize=12;fontColor=#666666;labelBackgroundColor=none;"
  vertex="1" parent="1">
  <mxGeometry x="200" y="38" width="500" height="20" as="geometry" />
</mxCell>
```

---

## WAF-Aligned Design

When relevant, highlight 1-3 WAF pillars that are in scope for the audience and design:

### Security (Show These)
- Trust boundaries (VNet, private endpoints)
- Identity flow (Entra ID → Managed Identity → services)
- Data encryption indicators (lock icons or labels)
- Private endpoints for PaaS services (if required by the security design)

### Reliability (Show These)
- Multi-region or zone redundancy (if in scope)
- Failover paths (if in scope)
- Health probes / monitoring connections
- Circuit breaker patterns

### Performance (Show These)
- Caching layers (Redis, CDN) when performance is a goal
- Async patterns (queues, events) when decoupling is in scope
- Regional distribution (if in scope)
- Autoscaling indicators (if known)

### Cost Optimization (Show These)
- Reserved vs on-demand annotations (if decided)
- Spot VM indicators (if decided)
- Tier choices (Premium vs Standard) when known
- Shared services when relevant

### Operational Excellence (Show These)
- Monitoring/logging connections (dotted lines to Azure Monitor)
- Deployment pipelines (if in scope)
- Alert paths (if in scope)

---

## Audience-Focused Diagrams

### For Executives
- **Emphasize**: Trust boundaries, data residency, compliance zones, business flows
- **Minimize**: Technical details, specific service names, internal connections
- **Style**: Larger boxes, fewer elements (5-8 max), clear "in/out" boundaries

### For Architects
- **Emphasize**: Relevant WAF pillars, service interactions, scaling hints
- **Minimize**: Implementation details, exact configurations
- **Style**: Medium detail (8-14 elements), numbered flows, boundary labels

### For Engineers
- **Emphasize**: Exact services, APIs, data flows, connection strings, ports
- **Add**: Schema hints, retry policies, timeout values
- **Style**: High detail, service-specific icons, subnet/NSG visibility

---

## Core Rules

### Font Settings

```xml
<!-- In mxGraphModel -->
<mxGraphModel defaultFontFamily="Segoe UI" page="0" ...>

<!-- In EVERY text element's style -->
<mxCell style="text;fontFamily=Segoe UI;fontSize=14;..." />
```

### Element Ordering (Z-Order)

> **Note**: The Quick Start section above has the authoritative z-order rule. This section provides additional context.

Elements render in XML declaration order (first = bottom layer). For diagrams with **filled containers** (like gray tier backgrounds), the correct order is:

1. **Background containers** (boundaries, tiers with `fillColor`)
2. **Edges** (arrows/connectors)
3. **Foreground vertices** (service icons, text labels)

This prevents arrows from being hidden behind filled containers.

```xml
<root>
  <mxCell id="0" />
  <mxCell id="1" parent="0" />

  <!-- 1. CONTAINERS FIRST (filled backgrounds) -->
  <mxCell id="tier1" style="fillColor=#E6E6E6;..." vertex="1" ... />

  <!-- 2. EDGES NEXT (render above containers) -->
  <mxCell id="arrow1" edge="1" ... />

  <!-- 3. VERTICES LAST (render on top) -->
  <mxCell id="icon1" vertex="1" ... />
</root>
```

### Label-Arrow Spacing

Labels must be at least 20px away from arrow lines:

```xml
<!-- Arrow at Y=220 -->
<mxCell id="arrow">
  <mxGeometry>
    <mxPoint y="220" as="sourcePoint"/>
  </mxGeometry>
</mxCell>

<!-- Label at Y=180 (40px above arrow) - CORRECT -->
<mxCell id="label" value="Process">
  <mxGeometry y="180" width="60" height="20" />
</mxCell>
```

### Multi-byte Text Width

Allocate sufficient width to prevent unwanted line breaks:

```xml
<!-- 8 Japanese characters × 35px = 280px minimum -->
<mxCell id="title" value="システム構成図">
  <mxGeometry width="300" height="40" />
</mxCell>
```

---

## Instruction Template

When asked to create a draw.io diagram:

1. **Clarify audience**: Exec overview vs architect detail vs engineer spec?
2. **Identify scope**: Context (high-level) vs Container (services) vs Component (internal)?
3. **Gather components**: Which Azure services? What trust boundaries?
4. **Plan WAF elements**: Which pillars to emphasize (if applicable)?
5. **Generate XML** with all rules applied
6. **Verify** with PNG export

## PNG Verification

Always recommend PNG export for visual verification:

```bash
# macOS
drawio -x -f png -s 2 -t -o output.png input.drawio
open output.png

# Linux
drawio -x -f png -s 2 -t -o output.png input.drawio
xdg-open output.png

# Windows (PowerShell)
& "$env:ProgramFiles\draw.io\draw.io.exe" -x -f png -s 2 -t -o output.png input.drawio
Start-Process output.png
```

---

## Supporting Files

- [[reference]] - Complete XML structure reference with Azure shapes
- [[examples]] - Production-ready Azure architecture examples
- [[checklist]] - Pre-commit validation checklist (including WAF)
- [[azure-icons]] - Full Azure icon catalog (by category)

---

## MCP Tool Integration

When draw.io MCP tools are available, use them with these exact parameter formats:

### mcp_drawio_create_new_diagram

Create a new diagram from scratch. Provide complete `mxGraphModel` XML:

```json
{
  "xml": "<mxGraphModel dx=\"800\" dy=\"600\" ...><root>...</root></mxGraphModel>"
}
```

### mcp_drawio_edit_diagram

Edit existing diagram by operations. **Always call `mcp_drawio_get_diagram` first** to see current cell IDs.

```json
{
  "operations": [
    {
      "operation": "add",
      "cell_id": "new-cell-1",
      "new_xml": "<mxCell id=\"new-cell-1\" value=\"Label\" ... />"
    },
    {
      "operation": "update",
      "cell_id": "existing-id",
      "new_xml": "<mxCell id=\"existing-id\" value=\"New Label\" ... />"
    },
    {
      "operation": "delete",
      "cell_id": "cell-to-remove"
    }
  ]
}
```

**Common mistake**: Using `changes` instead of `operations`, or `op` instead of `operation`. The correct format is:
- Top-level array: `operations`
- Each item: `operation` (enum: "add", "update", "delete"), `cell_id`, `new_xml` (except delete)

### Workflow

1. `mcp_drawio_start_session` - Opens browser preview
2. `mcp_drawio_create_new_diagram` - Create initial diagram
3. `mcp_drawio_get_diagram` - Fetch current state (before edits)
4. `mcp_drawio_edit_diagram` - Apply changes
5. `mcp_drawio_export_diagram` - Save to .drawio file
