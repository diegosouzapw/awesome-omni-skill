---
name: d2-diagram
description: Comprehensive tool for creating D2 diagrams based on descriptions and requirements. This skill should be used when creating visual diagrams, system architectures, flowcharts, network topologies, data flows, or any visual representation that can be expressed as a diagram. Triggers include requests to "create a diagram," "visualize," "draw an architecture," "show relationships," or when converting textual descriptions into visual diagrams.
---

# D2 Diagram Creation Skill

## Overview

Generate clear, well-structured, and visually appealing D2 diagrams from textual descriptions. D2 is a modern diagram scripting language that transforms text into professional diagrams with support for icons, styling, containers, and multiple layout options.

## When to Use This Skill

Use this skill when:
- Creating system architecture diagrams
- Visualizing data flows or processes
- Drawing network topologies
- Illustrating software component relationships
- Creating flowcharts or decision trees
- Designing database schemas
- Showing microservices architectures
- Any request involving "create a diagram," "visualize," "draw," or "show relationships"

## Prerequisites

**CRITICAL**: Before creating any diagram, read the reference files to understand D2 syntax and available icons:
1. Read `references/d2_syntax_reference.md` for syntax patterns and examples
2. Read `references/d2_validated_icons_list.md` for available icon URLs

## Diagram Creation Workflow

### Step 1: Analyze the Requirements

Carefully review the provided description to identify:

1. **Core Components**: What are the main elements/entities?
2. **Relationships**: How do components connect or interact?
3. **Hierarchy**: Are there groupings or containers?
4. **Flow Direction**: What's the logical flow (left-to-right, top-to-bottom)?
5. **Special Requirements**: Styling, colors, icons, or specific visual needs?

**Example Analysis:**

Request: "Create a diagram showing a web application with users connecting to a load balancer, which distributes traffic to multiple app servers. The app servers connect to a database."

Analysis:
- Core: Users, Load Balancer, App Servers, Database
- Relationships: Users → Load Balancer → App Servers → Database
- Hierarchy: Could group app servers together
- Direction: Left-to-right flow
- Shapes: person for users, hexagon for load balancer, rectangles for servers, cylinder for database

### Step 2: Design the Structure

Create a mental model of the diagram structure:

1. **Identify Central Element**: Place the most important component centrally
2. **Plan Containers**: Group related elements together
3. **Arrange Flow**: Position elements to show natural progression
4. **Consider Symmetry**: Balance the layout for visual appeal

### Step 3: Select Shapes and Icons

Choose appropriate shapes and icons for each element:

**Shape Selection Guidelines:**
- `person` - Users, people, actors
- `cylinder` - Databases, data storage
- `hexagon` - Gateways, processors, services
- `cloud` - Cloud services, external systems
- `rectangle` - Default for most components, servers, applications
- `diamond` - Decision points, conditionals
- `circle` - Start/end points, events
- `queue` - Message queues, buffers
- `stored_data` - Data stores, caches

**Icon Selection:**
- Consult `references/d2_validated_icons_list.md` for available icons
- Match icons to component type (e.g., AWS Lambda for serverless functions)
- Use `shape: image` to display icon without border
- If no suitable icon exists, use appropriate shape with clear labeling

### Step 4: Apply Styling

Create consistent visual appearance:

1. **Define Classes**: Create reusable styles for similar components
2. **Use Color Coding**: Apply colors to distinguish different types or states
3. **Add Visual Hierarchy**: Use size, color, and positioning to show importance
4. **Maintain Consistency**: Apply uniform styling to similar elements

**Styling Best Practices:**
```d2
classes: {
  primary_service: {
    style.fill: "#4A90E2"
    style.stroke: "#2E5C8A"
    style.stroke-width: 2
  }
  
  database_style: {
    shape: cylinder
    style.fill: "#48C774"
    style.multiple: true
  }
}
```

### Step 5: Construct the Diagram

Build the diagram following D2 syntax:

1. **Start with Direction**: Set overall flow direction
2. **Define Core Elements**: Create main shapes with labels
3. **Add Containers**: Group related components
4. **Create Connections**: Link elements with appropriate arrows
5. **Apply Styling**: Add colors, icons, and visual properties
6. **Add Notes**: Include explanatory text where helpful

### Step 6: Review and Refine

Before finalizing:

**Quality Checklist:**
- [ ] All components clearly labeled
- [ ] Connections properly directed and labeled
- [ ] Appropriate shapes/icons used
- [ ] Consistent styling applied
- [ ] Logical flow is evident
- [ ] No overlapping or cluttered elements
- [ ] Container groupings make sense
- [ ] Icons are from validated list
- [ ] Overall layout is balanced

## Creating Specific Diagram Types

### System Architecture

```d2
direction: right

classes: {
  service: {
    style.fill: "#4A90E2"
    style.stroke: "#2E5C8A"
  }
  
  db: {
    shape: cylinder
    style.fill: "#48C774"
  }
}

users: Users {
  shape: person
  icon: https://icons.terrastruct.com/essentials/365-user.svg
}

frontend: Frontend {
  class: service
  icon: https://icons.terrastruct.com/tech/react.svg
}

backend: Backend Services {
  api: API Gateway {
    class: service
    icon: https://icons.terrastruct.com/aws/Networking%20&%20Content%20Delivery/Amazon-API-Gateway.svg
  }
  
  auth: Auth Service {
    class: service
  }
  
  data: Data Service {
    class: service
  }
}

database: PostgreSQL {
  class: db
  icon: https://icons.terrastruct.com/tech/postgresql.svg
}

users -> frontend: Access
frontend -> backend.api: REST API
backend.api -> backend.auth: Authenticate
backend.api -> backend.data: Fetch Data
backend.data -> database: Query
```

### Data Flow Diagram

```d2
direction: right

source: Data Source {
  shape: cylinder
  style.fill: "#E8F5E9"
  style.multiple: true
}

ingestion: Data Ingestion {
  shape: hexagon
  style.fill: "#FFF9C4"
}

processing: Data Processing {
  transform: Transform
  validate: Validate
  enrich: Enrich
  
  style.fill: "#E3F2FD"
}

storage: Data Warehouse {
  shape: stored_data
  style.fill: "#F3E5F5"
}

analytics: Analytics Layer {
  shape: rectangle
  style.fill: "#FCE4EC"
}

viz: Visualization {
  shape: page
  icon: https://icons.terrastruct.com/essentials/dashboard.svg
}

source -> ingestion: Extract {
  style.stroke: "#4CAF50"
  style.stroke-width: 2
}

ingestion -> processing.transform: Raw Data {
  style.animated: true
}

processing.transform -> processing.validate
processing.validate -> processing.enrich

processing.enrich -> storage: Store {
  style.stroke: "#2196F3"
  style.stroke-width: 2
}

storage -> analytics: Query
analytics -> viz: Display
```

### Network Topology

```d2
direction: down

internet: Internet {
  shape: cloud
  icon: https://icons.terrastruct.com/essentials/214-worldwide.svg
}

firewall: Firewall {
  shape: hexagon
  style.fill: "#FF5252"
  icon: https://icons.terrastruct.com/infra/firewall.svg
}

dmz: DMZ {
  direction: right
  
  lb: Load Balancer {
    shape: hexagon
    icon: https://icons.terrastruct.com/infra/load-balancer.svg
  }
  
  web1: Web Server 1 {
    shape: rectangle
  }
  
  web2: Web Server 2 {
    shape: rectangle
  }
  
  style.fill: "#FFF3E0"
}

internal: Internal Network {
  direction: right
  
  app1: App Server 1
  app2: App Server 2
  
  db: Database Cluster {
    shape: cylinder
    style.multiple: true
    icon: https://icons.terrastruct.com/tech/postgresql.svg
  }
  
  style.fill: "#E8F5E9"
}

internet -> firewall
firewall -> dmz.lb
dmz.lb -> dmz.web1
dmz.lb -> dmz.web2
dmz.web1 -> internal.app1
dmz.web2 -> internal.app2
internal.app1 -> internal.db
internal.app2 -> internal.db
```

### Microservices Architecture

```d2
direction: right

classes: {
  service: {
    shape: rectangle
    style.fill: "#E3F2FD"
    style.stroke: "#1976D2"
  }
  
  database: {
    shape: cylinder
    style.fill: "#C8E6C9"
  }
  
  queue: {
    shape: queue
    style.fill: "#FFF9C4"
  }
}

users: Users {
  shape: person
}

gateway: API Gateway {
  shape: hexagon
  style.fill: "#B39DDB"
}

services: Microservices {
  user_service: User Service {
    class: service
  }
  
  order_service: Order Service {
    class: service
  }
  
  payment_service: Payment Service {
    class: service
  }
  
  notification_service: Notification Service {
    class: service
  }
}

queues: Message Queues {
  order_queue: Order Queue {
    class: queue
  }
  
  notification_queue: Notification Queue {
    class: queue
  }
}

databases: Databases {
  user_db: User DB {
    class: database
  }
  
  order_db: Order DB {
    class: database
  }
  
  payment_db: Payment DB {
    class: database
  }
}

users -> gateway
gateway -> services.user_service
gateway -> services.order_service
gateway -> services.payment_service

services.user_service -> databases.user_db
services.order_service -> databases.order_db
services.payment_service -> databases.payment_db

services.order_service -> queues.order_queue: Publish
services.payment_service -> queues.order_queue: Subscribe

services.notification_service -> queues.notification_queue: Subscribe
```

## Best Practices

### Clarity and Simplicity

1. **Start Simple**: Begin with core elements, add complexity gradually
2. **Use Descriptive Labels**: Choose clear, informative names for all components
3. **Limit Complexity**: If diagram becomes cluttered, split into multiple diagrams or use layers
4. **Show Key Relationships**: Focus on important connections, omit trivial ones

### Visual Design

1. **Consistent Styling**: Apply uniform styles to similar component types using classes
2. **Color Coding**: Use colors purposefully to distinguish types or states
3. **Appropriate Shapes**: Match shapes to what they represent
4. **Balance Layout**: Distribute elements evenly, avoid crowding
5. **Direction Matters**: Choose flow direction that matches natural reading or process flow

### Icons and Shapes

1. **Verify Icon URLs**: Only use icons from `references/d2_validated_icons_list.md`
2. **Use Icons Sparingly**: Too many icons can clutter; use for key components
3. **Consistent Icon Style**: Stick to one icon set (e.g., all AWS, all tech icons)
4. **Fallback to Shapes**: If no suitable icon, use appropriate shape with label

### Containers and Grouping

1. **Logical Grouping**: Group related components together
2. **Nested Containers**: Use for hierarchical structures
3. **Clear Boundaries**: Make container purposes obvious through naming
4. **Avoid Over-Nesting**: Too many levels makes diagrams hard to read

### Connections

1. **Label Important Connections**: Add labels to clarify relationships
2. **Use Appropriate Arrows**: Choose arrow types that match relationship
3. **Connection Styling**: Style connections to show different types or states
4. **Avoid Crossing Lines**: Minimize connection crossings for clarity

## Common Pitfalls to Avoid

### ❌ Don't Use Fake Icon URLs

**Wrong:**
```d2
server: {
  icon: https://example.com/fake-icon.svg  # This will fail
}
```

**Right:**
```d2
server: {
  icon: https://icons.terrastruct.com/tech/docker.svg  # Verified URL
  shape: image
}
```

Or use a shape without an icon:
```d2
server: {
  shape: rectangle
}
```

### ❌ Don't Over-Complicate

**Wrong:**
```d2
# Too many nested levels
system.layer1.layer2.layer3.layer4.component
```

**Right:**
```d2
# Clear, manageable hierarchy
system: {
  frontend
  backend
  database
}
```

### ❌ Don't Forget Direction

**Wrong:**
```d2
# No direction specified, layout may be suboptimal
A -> B -> C -> D
```

**Right:**
```d2
direction: right  # Clear flow direction
A -> B -> C -> D
```

### ❌ Don't Mix Styling Approaches

**Wrong:**
```d2
# Inconsistent styling
element1: {
  style.fill: "#FF0000"
}

element2: {
  style.fill: "#00FF00"
}

element3: {
  style.fill: "#0000FF"
}
```

**Right:**
```d2
# Consistent styling with classes
classes: {
  primary: {
    style.fill: "#4A90E2"
  }
}

element1: { class: primary }
element2: { class: primary }
element3: { class: primary }
```

## Advanced Techniques

### Layout Optimization

```d2
vars: {
  d2-config: {
    layout-engine: elk  # Try 'elk' for better hierarchical layouts
    pad: 50             # Adjust padding
  }
}

# Use direction to control flow
direction: right

# Use grid for structured layouts
grid_container: {
  grid-rows: 2
  grid-columns: 3
  
  item1; item2; item3
  item4; item5; item6
}
```

### Reusable Styles with Classes

```d2
classes: {
  critical: {
    style.fill: "#FF5252"
    style.stroke: "#D32F2F"
    style.stroke-width: 3
  }
  
  normal: {
    style.fill: "#4CAF50"
    style.stroke: "#388E3C"
  }
  
  inactive: {
    style.fill: "#BDBDBD"
    style.opacity: 0.5
  }
}

# Apply classes
prod_server: { class: critical }
dev_server: { class: normal }
old_server: { class: inactive }
```

### Interactive Elements

```d2
component: {
  link: https://docs.example.com/component
  tooltip: Click to view documentation
}
```

### Notes and Annotations

```d2
important_element: Critical Component

important_element.note: |md
  This component handles:
  - User authentication
  - Session management
  - Rate limiting
| {
  shape: text
  near: top-center
}
```

## Output Format

Always provide:

1. **Complete D2 Code**: Ready to use, properly formatted
2. **Clear Comments**: Explain key sections
3. **Proper Syntax**: Follow D2 language specifications
4. **Working Icons**: Only use validated icon URLs

The output should be a complete `.d2` file that can be:
- Rendered using D2 CLI: `d2 diagram.d2 output.svg`
- Used in D2 playground: https://play.d2lang.com
- Integrated into documentation systems

## Resources

### Reference Files

- **`references/d2_syntax_reference.md`**: Comprehensive D2 syntax guide with examples
- **`references/d2_validated_icons_list.md`**: Verified icon URLs organized by category

### Quick Reference

**Basic Shape:**
```d2
element: Label
```

**Shape with Type:**
```d2
element: Label {
  shape: cylinder
}
```

**Connection:**
```d2
A -> B: Connection Label
```

**Container:**
```d2
container: Container Name {
  child1
  child2
  child1 -> child2
}
```

**Styling:**
```d2
element: {
  style.fill: "#4A90E2"
  style.stroke: "#2E5C8A"
}
```

**Icon:**
```d2
element: {
  icon: https://icons.terrastruct.com/tech/docker.svg
  shape: image
}
```

## Tips for Success

1. **Read References First**: Always consult syntax and icon references before starting
2. **Plan Before Coding**: Sketch out structure mentally or on paper
3. **Iterate**: Start simple, refine iteratively
4. **Test Layouts**: Try different directions and layout engines
5. **Use Examples**: Adapt provided examples to your needs
6. **Stay Organized**: Use containers to maintain structure
7. **Be Consistent**: Apply uniform styling throughout
8. **Document Choices**: Add comments explaining non-obvious decisions

## Troubleshooting

**Diagram looks cluttered?**
- Simplify: Remove non-essential elements
- Use containers to group related items
- Try different layout engine (elk vs dagre)
- Split into multiple diagrams or layers

**Icons not appearing?**
- Verify icon URL is from validated list
- Check URL is accessible (HTTPS required)
- Use `shape: image` to remove border
- Fallback to shapes if icon unavailable

**Layout not as expected?**
- Try different `direction` setting
- Experiment with layout engine (`elk` vs `dagre`)
- Adjust container groupings
- Use grid layouts for structured arrangements

**Connections crossing awkwardly?**
- Reorder element definitions
- Adjust container organization
- Try different flow direction
- Simplify connection paths

---

**Remember**: The goal is to create clear, informative diagrams that effectively communicate system structure and relationships. When in doubt, prioritize clarity over complexity.

