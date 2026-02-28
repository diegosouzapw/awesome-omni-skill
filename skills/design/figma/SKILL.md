---
name: "Figma"
description: "Use MCP Figma for design system integration, asset export, and design-to-code workflows."
version: "1.0.0"
category: "modules"
author: "Rulebook"
tags: ["modules", "mcp"]
dependencies: []
conflicts: []
---
<!-- FIGMA:START -->
# Figma MCP Instructions

**CRITICAL**: Use MCP Figma for design system integration, asset export, and design-to-code workflows.

## Core Operations

### File Operations
```typescript
// Get file
figma.getFile({ file_key: 'file-key' })

// Get file nodes
figma.getFileNodes({
  file_key: 'file-key',
  ids: ['node-id-1', 'node-id-2']
})

// Get file versions
figma.getFileVersions({ file_key: 'file-key' })
```

### Image Export
```typescript
// Export images
figma.getImage({
  file_key: 'file-key',
  ids: 'node-id',
  format: 'png',  // png, jpg, svg, pdf
  scale: 2  // @2x resolution
})

// Export multiple
figma.getImage({
  file_key: 'file-key',
  ids: 'node-1,node-2,node-3',
  format: 'svg',
  svg_outline_text: true
})
```

### Components
```typescript
// Get components
figma.getFileComponents({ file_key: 'file-key' })

// Get component sets
figma.getFileComponentSets({ file_key: 'file-key' })

// Get team components
figma.getTeamComponents({ team_id: 'team-id' })
```

### Styles
```typescript
// Get file styles
figma.getFileStyles({ file_key: 'file-key' })

// Get team styles
figma.getTeamStyles({ team_id: 'team-id' })
```

### Comments
```typescript
// Get comments
figma.getComments({ file_key: 'file-key' })

// Post comment
figma.postComment({
  file_key: 'file-key',
  message: 'Approved for development',
  comment_id: 'parent-comment-id'  // For replies
})
```

## Common Patterns

### Design Token Export
```typescript
// Export design tokens from Figma styles
const { data: { styles } } = await figma.getFileStyles({ file_key: fileKey })

const tokens = {
  colors: {},
  typography: {},
  spacing: {}
}

for (const style of Object.values(styles)) {
  if (style.style_type === 'FILL') {
    tokens.colors[style.name] = extractColor(style)
  } else if (style.style_type === 'TEXT') {
    tokens.typography[style.name] = extractTextStyle(style)
  }
}

// Write to tokens.json
fs.writeFileSync('tokens.json', JSON.stringify(tokens, null, 2))
```

### Component Sync
```typescript
// Sync Figma components to code
const { data: components } = await figma.getFileComponents({ file_key: fileKey })

for (const component of Object.values(components)) {
  // Export component as SVG
  const { data: images } = await figma.getImage({
    file_key: fileKey,
    ids: component.node_id,
    format: 'svg'
  })
  
  // Save to assets folder
  const svg = await fetch(images[component.node_id]).then(r => r.text())
  fs.writeFileSync(`assets/icons/${component.name}.svg`, svg)
}
```

### Screenshot Generation
```typescript
// Generate screenshots for documentation
const screens = ['home-screen', 'login-screen', 'dashboard']

for (const screenId of screens) {
  const { data: images } = await figma.getImage({
    file_key: fileKey,
    ids: screenId,
    format: 'png',
    scale: 2
  })
  
  const imageUrl = images[screenId]
  const response = await fetch(imageUrl)
  const buffer = await response.buffer()
  
  fs.writeFileSync(`docs/screenshots/${screenId}.png`, buffer)
}
```

### Design Review Automation
```typescript
// Check for new comments
const { data: comments } = await figma.getComments({ file_key: fileKey })

const unresolved = comments.filter(c => !c.resolved_at)

if (unresolved.length > 0) {
  console.log(`${unresolved.length} unresolved design comments`)
  
  // Create Jira issues for unresolved comments
  for (const comment of unresolved) {
    await jira.issues.createIssue({
      fields: {
        project: { key: 'DESIGN' },
        summary: `Design feedback: ${comment.message.substring(0, 50)}`,
        description: comment.message,
        issuetype: { name: 'Task' }
      }
    })
  }
}
```

### Design System Documentation
```typescript
// Generate component documentation
const { data: components } = await figma.getFileComponents({ file_key: fileKey })

let markdown = '# Design System Components\n\n'

for (const [id, component] of Object.entries(components)) {
  // Export thumbnail
  const { data: images } = await figma.getImage({
    file_key: fileKey,
    ids: id,
    format: 'png',
    scale: 1
  })
  
  markdown += `## ${component.name}\n\n`
  markdown += `![${component.name}](${images[id]})\n\n`
  markdown += `**Description:** ${component.description || 'No description'}\n\n`
}

fs.writeFileSync('docs/design-system.md', markdown)
```

## Best Practices

✅ **DO:**
- Cache file data to reduce API calls
- Use version history for tracking changes
- Export assets at appropriate resolutions
- Document component usage
- Use meaningful component names
- Keep design tokens in sync
- Handle rate limits (requests/minute)

❌ **DON'T:**
- Export entire files repeatedly
- Ignore version control
- Hardcode file keys
- Skip error handling
- Export at wrong resolutions
- Commit API tokens

## Configuration

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-figma"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "your-personal-access-token"
      }
    }
  }
}
```

**Setup:**
1. Generate personal access token: Account Settings → Personal Access Tokens
2. Grant appropriate scopes (file content, comments)
3. Store token securely

## Integration Patterns

### CI/CD Asset Pipeline
```bash
# Export icons on every design update
figma-export --file-key=$FIGMA_FILE --format=svg --output=src/assets/icons

# Optimize SVGs
svgo --folder src/assets/icons

# Commit if changes detected
git diff --quiet src/assets/icons || git commit -m "chore: Update design assets"
```

### Design-to-Code Workflow
```typescript
// 1. Detect design changes
const currentVersion = await figma.getFile({ file_key: fileKey })
const lastVersion = loadLastProcessedVersion()

if (currentVersion.version !== lastVersion) {
  // 2. Export updated components
  await exportComponents(fileKey)
  
  // 3. Generate code
  await generateComponentCode()
  
  // 4. Run tests
  await runVisualRegressionTests()
  
  // 5. Create PR if tests pass
  if (testsPass) {
    await createPullRequest('Update components from Figma')
  }
}
```

<!-- FIGMA:END -->

