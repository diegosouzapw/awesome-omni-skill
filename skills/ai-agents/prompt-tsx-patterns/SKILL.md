---
name: prompt-tsx-patterns
description: Deep dive into prompt-tsx patterns used in vscode-copilot-chat, including component lifecycle, async rendering, priority system, and token budget management
keywords: prompt-tsx, vscode, prompts, react, tsx, token budget, priority
---

This skill provides comprehensive guidance on using prompt-tsx in the vscode-copilot-chat extension. It covers the specific patterns and conventions used in this codebase.

## What is Prompt-TSX?

Prompt-TSX is a React-like framework for building AI prompts using TypeScript and JSX. It provides:
- Component-based prompt composition
- Token budget management
- Priority-based pruning
- Type-safe prompt generation

## Core Concepts

### PromptElement Base Class

All prompt components extend `PromptElement`:

```typescript
import { PromptElement, BasePromptElementProps } from '@vscode/prompt-tsx';

interface MyPromptProps extends BasePromptElementProps {
  readonly userQuery: string;
  readonly files?: string[];
}

class MyPrompt extends PromptElement<MyPromptProps> {
  render() {
    return (
      <>
        <SystemMessage priority={1000}>
          System instructions here<br />
        </SystemMessage>
        <UserMessage priority={900}>
          {this.props.userQuery}
        </UserMessage>
      </>
    );
  }
}
```

**Key Points**:
- Props must extend `BasePromptElementProps`
- Render method returns JSX (PromptPiece)
- Can be sync or async
- Components are classes, not functions

### The Line Break Rule

**CRITICAL**: JSX collapses whitespace and newlines!

```typescript
// ❌ WRONG - These will be on the same line
<SystemMessage priority={1000}>
  You are a helpful assistant.
  Please follow these guidelines:
  1. Be concise
  2. Be accurate
</SystemMessage>

// ✅ CORRECT - Use <br /> for line breaks
<SystemMessage priority={1000}>
  You are a helpful assistant.<br />
  Please follow these guidelines:<br />
  1. Be concise<br />
  2. Be accurate<br />
</SystemMessage>
```

**Why**: Prompt-TSX renders to plain text. Without explicit `<br />` tags, all text is concatenated into a single line.

## Priority System

Priority controls:
1. **Rendering order** (high priority first)
2. **Pruning decisions** (low priority pruned first when over budget)

### Priority Ranges in This Codebase

Based on patterns in `src/extension/prompts/`:

```typescript
// Core instructions - always included
const PRIORITY_SYSTEM_INSTRUCTIONS = 1000;

// User's current message - highest user priority
const PRIORITY_USER_MESSAGE = 900;

// Recent conversation - important context
const PRIORITY_RECENT_HISTORY = 800;

// Conversation history - context but can be pruned
const PRIORITY_HISTORY = 700;

// User attachments - files, code snippets
const PRIORITY_ATTACHMENTS = 600;

// Contextual info - workspace, file listings
const PRIORITY_CONTEXT = 500;

// Documentation, examples - helpful but optional
const PRIORITY_BACKGROUND = 100;
```

### Priority Best Practices

1. **Space by 10s**: Use 700, 710, 720 not 700, 701, 702
2. **Group related content**: Similar priority for related pieces
3. **Consider pruning**: What should be removed first?
4. **Document choices**: Comment why you chose a priority

```typescript
// Good: Clear priority hierarchy
<>
  {/* Critical instructions - never prune */}
  <SystemMessage priority={1000}>...</SystemMessage>

  {/* User query - high priority */}
  <UserMessage priority={900}>...</UserMessage>

  {/* Recent context - medium priority */}
  <History priority={700} flexGrow={1} />

  {/* Background info - prune first */}
  <Documentation priority={100} />
</>
```

## Token Budget Management

### FlexGrow

`flexGrow` allows components to expand to fill available token space:

```typescript
// Component with flexGrow will use remaining tokens
<History
  priority={700}
  flexGrow={1}  // Take all remaining space
/>

// Multiple flex components share proportionally
<>
  <History priority={700} flexGrow={2} />    // Gets 2/3 of space
  <Examples priority={500} flexGrow={1} />   // Gets 1/3 of space
</>
```

### FlexReserve

`flexReserve` reserves tokens before rendering:

```typescript
<History
  priority={700}
  flexGrow={1}
  flexReserve="/5"  // Reserve 1/5 (20%) of budget before rendering
/>
```

**Use cases**:
- When you know minimum tokens needed
- Prevent other components from using all space
- Guarantee space for important but flex content

### TextChunk for Large Content

`TextChunk` enables intelligent truncation:

```typescript
<TextChunk
  breakOn="\n\n"      // Break on paragraph boundaries
  breakOnWhitespace   // Or break on any whitespace
  priority={500}
>
  {longDocumentation}
</TextChunk>
```

**How it works**:
- If content fits in budget: rendered fully
- If too large: truncated at break point
- Preserves readability by breaking cleanly

## Async Rendering

Components can perform async operations:

```typescript
class FileContentPrompt extends PromptElement<FileContentProps> {
  async render() {
    // Async work happens IN render
    const content = await this.readFile(this.props.filePath);
    const metadata = await this.getMetadata(this.props.filePath);

    return (
      <>
        <SystemMessage priority={1000}>
          File: {this.props.filePath}<br />
          Size: {metadata.size} bytes<br />
        </SystemMessage>
        <TextChunk priority={500} breakOnWhitespace>
          {content}
        </TextChunk>
      </>
    );
  }

  private async readFile(path: string): Promise<string> {
    // Implementation
  }
}
```

**Key points**:
- Use `async render()` for async operations
- All async work happens in render method
- Don't store promises in state
- Always await before returning JSX

## Special Components

### Tag

Create XML-like structured content:

```typescript
<Tag name="context" attrs={{ type: "file", id: "main.ts" }}>
  {fileContent}
</Tag>
```

Renders as:
```xml
<context type="file" id="main.ts">
  [fileContent]
</context>
```

**Use for**:
- Structured data in prompts
- Semantic markup
- Tool result formatting

### References

Track variable usage in prompts:

```typescript
<references value={[new PromptReference({ variableName: 'fileName' })]} />
```

**Purpose**: Tell the system which variables are used in the prompt

### Meta

Attach metadata that survives pruning:

```typescript
<meta value={new ToolResultMetadata(toolCallId, result)} />
```

**Purpose**: Preserve important metadata even if content is pruned

### KeepWith

Keep related content together during pruning:

```typescript
const KeepWith = useKeepWith();

<>
  <KeepWith priority={2}>
    <ToolCallRequest>...</ToolCallRequest>
  </KeepWith>
  <KeepWith priority={1}>
    <ToolCallResponse>...</ToolCallResponse>
  </KeepWith>
</>
```

**Effect**: Both elements pruned together, not separately

## Patterns from This Codebase

### Pattern: System + User Message

```typescript
render() {
  return (
    <>
      <SystemMessage priority={1000}>
        {this.props.systemInstructions}
      </SystemMessage>
      <UserMessage priority={900}>
        {this.props.userQuery}
      </UserMessage>
    </>
  );
}
```

### Pattern: History with Flex

```typescript
<History
  priority={700}
  flexGrow={1}
  flexReserve="/5"
  messages={this.props.conversationHistory}
/>
```

### Pattern: File Context

```typescript
<FileContext
  priority={600}
  flexGrow={2}
  files={this.props.attachedFiles}
/>
```

### Pattern: Tool Results

```typescript
{this.props.toolResults.map((result, i) => (
  <ToolResultComponent
    key={i}
    priority={850}  // Higher than history, lower than user message
    result={result}
  />
))}
```

## Common Mistakes

### 1. Forgetting Line Breaks

```typescript
// ❌ Will render on one line
<SystemMessage priority={1000}>
  Line 1
  Line 2
</SystemMessage>

// ✅ Explicit line breaks
<SystemMessage priority={1000}>
  Line 1<br />
  Line 2<br />
</SystemMessage>
```

### 2. Priority Conflicts

```typescript
// ❌ Same priority - unpredictable order
<SystemMessage priority={1000}>...</SystemMessage>
<AnotherMessage priority={1000}>...</AnotherMessage>

// ✅ Different priorities
<SystemMessage priority={1000}>...</SystemMessage>
<AnotherMessage priority={990}>...</AnotherMessage>
```

### 3. Async Without Await

```typescript
// ❌ Promise not awaited
async render() {
  const data = this.fetchData();  // Returns Promise!
  return <>{data}</>;  // Renders "[object Promise]"
}

// ✅ Await the promise
async render() {
  const data = await this.fetchData();
  return <>{data}</>;
}
```

### 4. Large Content Without TextChunk

```typescript
// ❌ Could exceed token budget
<UserMessage priority={900}>
  {hugeDocument}
</UserMessage>

// ✅ Use TextChunk for intelligent truncation
<TextChunk breakOnWhitespace priority={900}>
  {hugeDocument}
</TextChunk>
```

## Testing Prompt Components

### Manual Testing

1. **Create test props**:
   ```typescript
   const testProps: MyPromptProps = {
     userQuery: 'test query',
     files: ['file1.ts', 'file2.ts']
   };
   ```

2. **Instantiate and render**:
   ```typescript
   const prompt = new MyPrompt(testProps);
   const result = await prompt.render();
   ```

3. **Inspect output**:
   - Check priorities are correct
   - Verify line breaks appear
   - Confirm token usage reasonable

### Testing Strategies

- **Unit tests**: Test component logic
- **Integration tests**: Test full prompt composition
- **Token budget tests**: Test with tight budgets
- **Priority tests**: Verify pruning order

## Real Examples from This Codebase

See the `references/` directory for:
- `component-patterns.md` - Actual component implementations
- `priority-examples.md` - Priority usage patterns
- `async-rendering.md` - Async rendering examples

These reference files contain real code from this codebase that you can learn from and adapt.

## Quick Reference

**Must-know rules**:
1. ✅ Use `<br />` for line breaks
2. ✅ Props extend `BasePromptElementProps`
3. ✅ Higher priority = rendered first, pruned last
4. ✅ Use `async render()` for async operations
5. ✅ Use `TextChunk` for large content
6. ✅ Space priorities by 10s (700, 710, 720)

**Common components**:
- `<SystemMessage priority={N}>` - System instructions
- `<UserMessage priority={N}>` - User input
- `<TextChunk breakOn="...">` - Truncatable content
- `<Tag name="..." attrs={{}}>` - Structured markup
- `<references value={...} />` - Variable tracking
- `<meta value={...} />` - Metadata

Remember: Prompt-TSX is your interface to the AI. Master it, and you master how the AI sees and understands requests!
