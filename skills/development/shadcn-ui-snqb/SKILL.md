---
name: shadcn-ui
description: Access shadcn/ui v4 component library via MCP server. Query components, blocks, demos, and metadata across React, Svelte, Vue, and React Native frameworks. Use for production-ready, accessible component foundations.
---

# shadcn-ui MCP Skill

**MCP Server:** `@jpisnice/shadcn-ui-mcp-server`
**Status:** Installed and configured
**Frameworks:** React, Svelte, Vue, React Native

## What This Skill Does

Provides direct access to the shadcn/ui v4 component library through an MCP server. Query components, get usage examples, explore complete blocks (dashboards, forms), and retrieve metadata—all via natural language.

## When to Use This Skill

- Building UI with React, Svelte, Vue, or React Native
- Need production-ready, accessible component foundations
- Want to see component demos and usage patterns
- Need dependency information for components
- Building dashboards, forms, or complex UI blocks
- Comparing component implementations across frameworks

## What shadcn-ui Provides

### Components
- 50+ production-ready UI components
- Full TypeScript definitions
- Accessibility built-in (ARIA, keyboard navigation)
- Radix UI primitives under the hood
- Tailwind CSS styling

### Blocks
- Complete dashboard implementations
- Form templates with validation
- Calendar and date picker blocks
- Authentication flows
- Data table examples

### Metadata
- Component dependencies
- Configuration requirements
- Installation instructions
- Peer dependencies
- Usage examples

## How to Query Components

### Basic Component Query
```
"Show me the shadcn button component"
"Get the shadcn card component"
"What's the shadcn dialog component?"
```

### Framework-Specific Query
```
"Show me the shadcn button for React"
"Get the shadcn card for Svelte"
"What's the shadcn select for Vue?"
```

### Block Query
```
"Show me the shadcn dashboard block"
"Get the shadcn form block"
"What blocks are available in shadcn?"
```

### Metadata Query
```
"What dependencies does shadcn select need?"
"How do I install the shadcn button?"
"What peer dependencies does shadcn dialog require?"
```

## Multi-Framework Support

One of shadcn-ui's unique advantages: **same component logic, different framework implementations**.

| Framework | Example Query |
|-----------|---------------|
| **React** | "Show me shadcn button for React" |
| **Svelte** | "Show me shadcn button for Svelte" |
| **Vue** | "Show me shadcn button for Vue" |
| **React Native** | "Show me shadcn button for React Native" |

## Component Categories

### Form Components
- Button, Input, Textarea
- Select, Checkbox, Radio Group
- Switch, Slider
- Form, Label
- Date Picker, Calendar

### Layout Components
- Card, Separator
- Accordion, Tabs
- Sheet, Dialog, Drawer
- Collapsible, Resizable

### Navigation
- Navigation Menu, Breadcrumb
- Dropdown Menu, Context Menu
- Command, Menubar

### Feedback
- Alert, Toast, Progress
- Skeleton, Badge
- Alert Dialog

### Data Display
- Table, Data Table
- Avatar, Tooltip
- Hover Card, Popover

## Usage Workflow

### 1. Query for Component
```
User: "Show me the shadcn button component"
```

### 2. Review Component Details
You'll get:
- TypeScript source code
- Component props and variants
- Usage examples
- Dependencies required
- Configuration needed

### 3. Implement and Customize
- Copy base component structure
- Apply your own styling/theming
- Customize variants as needed
- Add project-specific logic

## Example: Building a Dashboard

### Step 1: Query Blocks
```
"Show me the shadcn dashboard block"
```

**What you get:**
- Complete dashboard layout
- Pre-configured components (cards, tables, charts)
- Navigation structure
- Responsive grid system

### Step 2: Query Individual Components
```
"Show me shadcn card component"
"Show me shadcn table component"
```

### Step 3: Customize
- Apply your design system colors
- Modify typography
- Adjust spacing and layout
- Add custom functionality

## Dependencies and Installation

### Check Dependencies
```
"What dependencies does the shadcn select component need?"
```

**Response includes:**
- Required packages (@radix-ui/react-select, etc.)
- Peer dependencies (React, Tailwind CSS)
- Installation commands
- Configuration requirements

### Common Dependencies
Most shadcn components require:
- React 18+ (for React version)
- Tailwind CSS 3+
- Radix UI primitives
- class-variance-authority
- clsx, tailwind-merge

## Best Practices

### ✅ DO

1. **Use as Foundation**
   - shadcn provides structure and accessibility
   - Build your unique design on top

2. **Query Before Building**
   - Check if shadcn has a component before building from scratch
   - Save time on accessibility and edge cases

3. **Check Multiple Frameworks**
   - If switching frameworks, query for that implementation
   - Maintain consistent UX across platforms

4. **Use Blocks for Rapid Prototyping**
   - Dashboard, form, calendar blocks save hours
   - Customize blocks to match your brand

5. **Review Metadata**
   - Check dependencies before installing
   - Understand peer dependencies and conflicts

### ❌ DON'T

1. **Don't Use Components As-Is in Production**
   - Always customize to match your design system
   - Add brand colors, typography, spacing

2. **Don't Skip Accessibility Review**
   - shadcn provides accessible foundations
   - Don't break them with aggressive customizations

3. **Don't Mix Framework Implementations**
   - Stick to one framework per project
   - Cross-framework mixing causes conflicts

4. **Don't Ignore Dependencies**
   - Review required packages before installation
   - Check for version conflicts with existing dependencies

## Integration with Other Skills

### With `frontend-design` Skill
```
frontend-design provides: Bold aesthetics, creative direction, unique design
shadcn-ui provides: Component structure, accessibility, best practices
```

**Workflow:**
1. Query shadcn-ui for base component
2. Apply frontend-design aesthetic transformation
3. Result: Production-ready + unforgettable design

### With `interface-composition` Skill
```
interface-composition provides: Visual hierarchy, balance, composition rules
shadcn-ui provides: Component building blocks
```

**Workflow:**
1. Query shadcn-ui for components
2. Apply interface-composition principles (visual weight, balance, hierarchy)
3. Result: Well-composed, professional UI

## Troubleshooting

### Component Not Found
- Try full name: "shadcn-ui button component"
- Specify framework: "button for React"
- Check spelling and naming

### Dependency Conflicts
- Review peer dependencies before installation
- Check if Radix UI versions are compatible
- Ensure Tailwind CSS is configured

### Framework-Specific Issues
- Verify you're querying for the correct framework
- Check framework-specific documentation
- Review implementation examples

## Quick Reference

**Query Patterns:**
```
"Show me shadcn [component]"
"Get shadcn [component] for [framework]"
"What's in the shadcn [block] block?"
"What dependencies does shadcn [component] need?"
"How do I install shadcn [component]?"
```

**Common Components:**
Button, Card, Input, Select, Dialog, Table, Form, Calendar, Dropdown Menu, Toast, Sheet, Tabs, Accordion, Avatar, Badge, Checkbox, Radio, Switch, Slider

**Available Blocks:**
Dashboard, Form, Calendar, Authentication, Data Table

**Supported Frameworks:**
React, Svelte 5, Vue, React Native

---

## Key Takeaways

`★ Insight ─────────────────────────────────────`
**1. shadcn-ui = Production-Ready Foundation**
- Accessibility built-in (ARIA, keyboard nav)
- Best practices enforced
- TypeScript definitions included
- Battle-tested component patterns

**2. Multi-Framework Consistency**
- Same component logic across React, Svelte, Vue, React Native
- Maintain consistent UX when switching platforms
- Faster development across different projects

**3. Blocks Save Time**
- Complete dashboard, form, calendar implementations
- 60-80% faster scaffolding
- Focus on customization, not structure

**4. Always Customize**
- shadcn components are STARTING POINTS
- Add your design system (colors, typography, spacing)
- Make it match your brand
- Never ship generic shadcn aesthetics
─────────────────────────────────────────────────`

## Resources

- [shadcn/ui Official Docs](https://ui.shadcn.com/)
- [GitHub - shadcn-ui-mcp-server](https://github.com/Jpisnice/shadcn-ui-mcp-server)
- [Radix UI Documentation](https://www.radix-ui.com/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)
