---
name: react-fluent-ui-patterns
description: Skill for React TypeScript frontend development with Fluent UI Copilot components. Use when creating UI components, handling SSE streams, working with chat interfaces, or implementing theme support.
---

# React Fluent UI Development Skill

This skill provides patterns and guidance for developing the React TypeScript frontend in the Foundry Agent Accelerator.

## Project Structure

```
src/frontend/
├── src/
│   ├── main.tsx              # App entry point
│   ├── components/
│   │   ├── App.tsx           # Root component
│   │   ├── agents/           # Chat-related components
│   │   │   ├── AgentPreview.tsx
│   │   │   ├── chatbot/      # Chat input components
│   │   │   └── hooks/        # Custom React hooks
│   │   └── core/             # Shared/reusable components
│   │       └── theme/        # Theme provider
│   └── types/
│       └── chat.ts           # TypeScript interfaces
└── package.json
```

## Component Documentation Format

Every component file MUST start with:

```tsx
/**
 * =============================================================================
 * COMPONENT NAME - Brief Description
 * =============================================================================
 * 
 * WHAT THIS FILE DOES:
 * --------------------
 * 1. First responsibility
 * 2. Second responsibility
 * 
 * HOW TO CUSTOMIZE:
 * -----------------
 * - Customization instructions
 * 
 * =============================================================================
 */
```

## Interface Naming Convention

Prefix all interface names with `I`:

```tsx
interface IAgent {
  id: string;
  name: string;
  description?: string | null;
  model: string;
  metadata?: Record<string, any>;
}

interface IMessage {
  id: string;
  content: string;
  role: 'user' | 'assistant';
}
```

## Component Pattern

```tsx
import { ReactNode, useState, useCallback } from "react";
import { Button, Body1 } from "@fluentui/react-components";
import styles from "./ComponentName.module.css";

interface IComponentNameProps {
  title: string;
  onAction?: () => void;
}

export function ComponentName({ title, onAction }: IComponentNameProps): ReactNode {
  // -------------------------------------------------------------------------
  // STATE
  // -------------------------------------------------------------------------
  const [isActive, setIsActive] = useState(false);

  // -------------------------------------------------------------------------
  // HANDLERS
  // -------------------------------------------------------------------------
  const handleClick = useCallback(() => {
    setIsActive(!isActive);
    onAction?.();
  }, [isActive, onAction]);

  // -------------------------------------------------------------------------
  // RENDER
  // -------------------------------------------------------------------------
  return (
    <div className={styles.container}>
      <Body1>{title}</Body1>
      <Button onClick={handleClick} appearance="primary">
        Click Me
      </Button>
    </div>
  );
}
```

## CSS Modules

### File naming
- Component: `ComponentName.tsx`
- Styles: `ComponentName.module.css`

### CSS Module Pattern

```css
/* ComponentName.module.css */
.container {
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 8px;
}

.header {
  font-size: 1.5rem;
  font-weight: 600;
}

/* Use kebab-case for multi-word classes */
.action-button {
  margin-top: 8px;
}
```

### Usage

```tsx
import styles from "./ComponentName.module.css";

<div className={styles.container}>
  <h1 className={styles.header}>Title</h1>
  <button className={styles.actionButton}>Action</button>
</div>
```

## Fluent UI Components

```tsx
import {
  Body1,
  Button,
  Caption1,
  Title2,
  Spinner,
  Input,
  Textarea,
} from "@fluentui/react-components";

import {
  ChatRegular,
  SendRegular,
  SettingsRegular,
  DismissRegular,
} from "@fluentui/react-icons";
```

## Fluent UI Copilot Components

For chat interfaces:

```tsx
import { CopilotProvider } from "@fluentui-copilot/react-provider";
import { CopilotChat } from "@fluentui-copilot/react-copilot-chat";
```

## SSE (Server-Sent Events) Pattern

Handle streaming responses from the backend:

```tsx
const processStream = async (response: Response) => {
  const reader = response.body?.getReader();
  const decoder = new TextDecoder();
  
  if (!reader) return;
  
  let buffer = "";
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n\n");
    buffer = lines.pop() || "";
    
    for (const line of lines) {
      if (line.startsWith("data: ")) {
        const data = JSON.parse(line.slice(6));
        
        switch (data.type) {
          case "message":
            appendToMessage(data.content);
            break;
          case "completed_message":
            setFinalMessage(data.content);
            break;
          case "stream_end":
            setIsLoading(false);
            break;
        }
      }
    }
  }
};
```

## Custom Hooks

Place in `hooks/` directory:

```tsx
// hooks/useFormatTimestamp.ts
import { useMemo } from "react";

export function useFormatTimestamp(timestamp: number): string {
  return useMemo(() => {
    const date = new Date(timestamp);
    return date.toLocaleTimeString("en-US", {
      hour: "2-digit",
      minute: "2-digit",
    });
  }, [timestamp]);
}
```

## Theme Support

Use the ThemeProvider:

```tsx
import { ThemeProvider } from "./core/theme/ThemeProvider";

return (
  <ThemeProvider>
    <div className="app-container">
      <YourComponent />
    </div>
  </ThemeProvider>
);
```

## File Attachment Handling

```tsx
interface IFileAttachment {
  name: string;
  type: string;  // MIME type
  data: string;  // Base64-encoded
}

const handleFileSelect = async (file: File): Promise<IFileAttachment> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => {
      const base64 = (reader.result as string).split(",")[1];
      resolve({ name: file.name, type: file.type, data: base64 });
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
};
```

## Markdown Rendering

```tsx
import ReactMarkdown from "react-markdown";
import remarkGfm from "remark-gfm";
import remarkMath from "remark-math";
import rehypeKatex from "rehype-katex";

export function MessageContent({ content }: { content: string }): ReactNode {
  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm, remarkMath]}
      rehypePlugins={[rehypeKatex]}
    >
      {content}
    </ReactMarkdown>
  );
}
```

## Component Location Guide

| Component Type | Location |
|----------------|----------|
| Agent/Chat related | `components/agents/` |
| Shared/Reusable | `components/core/` |
| Theme related | `components/core/theme/` |
| Chat input | `components/agents/chatbot/` |
| Custom hooks | `components/agents/hooks/` |
