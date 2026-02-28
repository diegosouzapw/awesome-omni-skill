---
name: coeditor
description: Build React applications with CopilotKit and LangGraph for AI-powered collaborative editing. Creates production-ready editor applications (text, document, or node-based) with real-time AI assistance, shared state management, and agentic workflows. Use this skill when users want to build intelligent editing interfaces with AI collaboration features.
---

# CopilotKit + LangGraph Collaborative Editor Builder

Build production-ready React applications that combine CopilotKit's AI copilot interface with LangGraph's agentic workflows for intelligent, collaborative editing experiences.

## When to Use This Skill

Use this skill when:
- User wants to build an AI-powered editor (text, document, or node-based)
- User mentions "CopilotKit", "LangGraph", "AI copilot", or "collaborative editing"
- User wants to add AI assistance to an existing React application
- User needs real-time state sharing between UI and AI agents
- User wants to build document collaboration features with AI
- User mentions "agentic workflows" or "multi-agent systems" in editing context

## Overview

This skill scaffolds applications that combine:
- **CopilotKit**: React components for AI copilot experiences (chat, suggestions, actions)
- **LangGraph**: Agent orchestration framework for complex workflows
- **React State Management**: Shared state between editor UI and AI agents
- **Editor Types**: Text editors, document editors, or node-based editors

## Prerequisites Check

Before starting, verify the following:

1. **Node.js**: Version 18.x or higher
   ```bash
   node --version
   ```

2. **Package Manager**: npm, yarn, or pnpm
   ```bash
   npm --version
   ```

3. **Python**: Version 3.9 or higher (for LangGraph backend)
   ```bash
   python3 --version
   ```

4. **OpenAI API Key** (or other LLM provider):
   ```bash
   # Will need to be configured in .env
   echo "User will need OpenAI API key or alternative LLM provider"
   ```

## Interactive Requirements Gathering

**IMPORTANT**: Before scaffolding, ask the user these questions to customize the setup:

### 1. Editor Type Selection

"What type of editor would you like to build?"

**A. Text Editor**
- Rich text editing with AI assistance
- Code editor with autocomplete and suggestions
- Markdown editor with AI enhancements
- Best for: Writing tools, code editors, note-taking apps

**B. Document Editor**
- Structured document editing (like Google Docs)
- Multi-section documents with AI collaboration
- Template-based document creation
- Best for: Documentation tools, report builders, collaborative writing

**C. Node-Based Editor**
- Visual graph/flow editor with AI assistance
- Workflow builders with intelligent suggestions
- Mind mapping with AI expansion
- Best for: Workflow designers, visual programming tools, diagramming apps

### 2. LangGraph Agent Configuration

"What type of AI agents do you need?"

Ask the user to describe their use case, then suggest appropriate agents:

**Common Agent Types:**
- **Writing Assistant**: Helps with content creation, editing, and refinement
- **Code Assistant**: Provides code suggestions, explanations, and debugging
- **Research Agent**: Searches and incorporates external information
- **Reviewer Agent**: Reviews content and provides feedback
- **Summarizer Agent**: Creates summaries and extracts key points
- **Translator Agent**: Translates between languages
- **Custom Agents**: User-defined specialized agents

**Multi-Agent Workflows:**
- Ask if they need multiple agents working together
- Determine if agents should run sequentially or in parallel
- Identify handoff points between agents

### 3. State Management Requirements

"What data needs to be shared between the editor and AI agents?"

**Common State Patterns:**
- **Document State**: Content, structure, metadata
- **Selection State**: Current cursor position, selected text/nodes
- **History State**: Undo/redo, version tracking
- **Collaboration State**: Multi-user presence, changes
- **Agent State**: Current agent task, progress, results

### 4. Backend Architecture

"How would you like to deploy the LangGraph backend?"

**A. Local Development Server**
- FastAPI server running locally
- Best for: Development, prototyping
- Setup: Python virtual environment

**B. Cloud Deployment**
- Deploy to Vercel, Railway, or cloud provider
- Best for: Production, sharing with team
- Setup: Containerized deployment

**C. Serverless Functions**
- Deploy agents as serverless functions
- Best for: Cost-effective scaling
- Setup: Vercel Functions, AWS Lambda

### 5. Additional Features

Ask about optional features:

**CopilotKit Features:**
- [ ] Chat interface (CopilotChat)
- [ ] Inline suggestions (CopilotTextarea)
- [ ] Custom actions (CopilotAction)
- [ ] Context providers (document context)
- [ ] Keyboard shortcuts

**Editor Features:**
- [ ] Real-time collaboration
- [ ] Version history
- [ ] Comments and annotations
- [ ] Export formats (PDF, Markdown, etc.)
- [ ] Templates
- [ ] Search and replace

**AI Features:**
- [ ] Streaming responses
- [ ] Multi-turn conversations
- [ ] Agent memory/context
- [ ] Custom prompts/instructions
- [ ] Fine-tuned models

## Project Structure

Based on user selections, create this structure:

```
<app-name>/
├── frontend/                   # React application
│   ├── src/
│   │   ├── components/
│   │   │   ├── Editor/        # Editor components
│   │   │   │   ├── TextEditor.tsx
│   │   │   │   ├── DocumentEditor.tsx
│   │   │   │   └── NodeEditor.tsx
│   │   │   ├── Copilot/       # CopilotKit components
│   │   │   │   ├── CopilotProvider.tsx
│   │   │   │   ├── ChatPanel.tsx
│   │   │   │   └── Actions.tsx
│   │   │   └── shared/        # Shared UI components
│   │   ├── hooks/             # Custom React hooks
│   │   │   ├── useEditor.ts
│   │   │   ├── useAgents.ts
│   │   │   └── useSharedState.ts
│   │   ├── state/             # State management
│   │   │   ├── editorStore.ts
│   │   │   ├── agentStore.ts
│   │   │   └── types.ts
│   │   ├── lib/               # Utilities
│   │   │   ├── copilot-config.ts
│   │   │   └── api-client.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
│
├── backend/                    # LangGraph backend
│   ├── agents/                # Agent definitions
│   │   ├── __init__.py
│   │   ├── writing_assistant.py
│   │   ├── code_assistant.py
│   │   └── custom_agents.py
│   ├── graphs/                # LangGraph workflows
│   │   ├── __init__.py
│   │   ├── editor_graph.py
│   │   └── multi_agent_graph.py
│   ├── api/                   # FastAPI endpoints
│   │   ├── __init__.py
│   │   ├── main.py
│   │   └── routes.py
│   ├── state/                 # Shared state management
│   │   ├── __init__.py
│   │   └── state_manager.py
│   ├── requirements.txt
│   └── pyproject.toml
│
├── shared/                     # Shared types/schemas
│   ├── types.ts
│   └── schemas.py
│
├── .env.example
├── .gitignore
├── docker-compose.yml          # Optional: for containerized dev
├── README.md
└── package.json               # Root workspace config
```

## Step-by-Step Implementation

### Step 1: Project Initialization

**1.1 Create React Frontend (Vite + TypeScript)**

```bash
npm create vite@latest <app-name> -- --template react-ts
cd <app-name>
mv <app-name> frontend
```

**1.2 Install CopilotKit Dependencies**

```bash
cd frontend
npm install @copilotkit/react-core @copilotkit/react-ui @copilotkit/react-textarea
```

**1.3 Install Editor Dependencies**

Choose based on editor type:

**For Text/Code Editor:**
```bash
npm install @monaco-editor/react      # VS Code editor
# OR
npm install @tiptap/react @tiptap/starter-kit  # Rich text
```

**For Document Editor:**
```bash
npm install slate slate-react         # Document framework
# OR
npm install @lexical/react lexical     # Facebook's editor framework
```

**For Node-Based Editor:**
```bash
npm install reactflow                  # Flow/graph editor
# OR
npm install @xyflow/react              # Advanced node editor
```

**1.4 Install State Management**

```bash
npm install zustand                    # Lightweight state management
# OR
npm install @tanstack/react-query      # Server state management
# OR
npm install jotai                      # Atomic state management
```

**1.5 Initialize Python Backend**

```bash
cd ..
mkdir backend
cd backend
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**1.6 Install LangGraph Dependencies**

```bash
pip install langgraph langchain langchain-openai fastapi uvicorn python-dotenv pydantic websockets
```

Create `requirements.txt`:
```bash
pip freeze > requirements.txt
```

### Step 2: Configure CopilotKit

**2.1 Create CopilotKit Provider (`frontend/src/components/Copilot/CopilotProvider.tsx`)**

```typescript
import { CopilotKit } from "@copilotkit/react-core";
import { CopilotSidebar } from "@copilotkit/react-ui";
import "@copilotkit/react-ui/styles.css";

interface CopilotProviderProps {
  children: React.ReactNode;
}

export function CopilotProvider({ children }: CopilotProviderProps) {
  return (
    <CopilotKit
      runtimeUrl="/api/copilotkit"
      // Alternatively, connect directly to LangGraph backend:
      // runtimeUrl="http://localhost:8000/copilot"
    >
      <CopilotSidebar>
        {children}
      </CopilotSidebar>
    </CopilotKit>
  );
}
```

**2.2 Configure Context Providers**

CopilotKit needs access to editor state to provide relevant suggestions:

```typescript
import { useCopilotReadable, useCopilotAction } from "@copilotkit/react-core";

export function useEditorCopilot(editorState: EditorState) {
  // Make editor state readable by AI
  useCopilotReadable({
    description: "The current document content and structure",
    value: editorState,
  });

  // Define actions AI can perform
  useCopilotAction({
    name: "insertText",
    description: "Insert text at current cursor position",
    parameters: [
      {
        name: "text",
        type: "string",
        description: "The text to insert",
        required: true,
      },
    ],
    handler: async ({ text }) => {
      // Insert text into editor
      editorState.insertText(text);
    },
  });

  useCopilotAction({
    name: "replaceSelection",
    description: "Replace currently selected text",
    parameters: [
      {
        name: "newText",
        type: "string",
        description: "The replacement text",
        required: true,
      },
    ],
    handler: async ({ newText }) => {
      editorState.replaceSelection(newText);
    },
  });
}
```

### Step 3: Set Up LangGraph Backend

**3.1 Define State Schema (`backend/state/state_manager.py`)**

```python
from typing import TypedDict, Annotated, Sequence
from langchain_core.messages import BaseMessage
import operator

class EditorState(TypedDict):
    """Shared state between frontend and agents."""
    # Document content
    content: str
    # Current selection
    selection: dict
    # Chat messages
    messages: Annotated[Sequence[BaseMessage], operator.add]
    # Agent context
    current_agent: str
    # Task tracking
    task_status: str
    # Additional metadata
    metadata: dict
```

**3.2 Create Base Agent (`backend/agents/base_agent.py`)**

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

class BaseAgent:
    def __init__(self, name: str, system_prompt: str):
        self.name = name
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", system_prompt),
            ("human", "{input}"),
        ])
        self.chain = self.prompt | self.llm

    async def process(self, state: EditorState) -> EditorState:
        """Process state and return updated state."""
        raise NotImplementedError
```

**3.3 Implement Specific Agents**

Based on user requirements, create specialized agents:

**Writing Assistant (`backend/agents/writing_assistant.py`)**

```python
from .base_agent import BaseAgent
from ..state.state_manager import EditorState

class WritingAssistant(BaseAgent):
    def __init__(self):
        super().__init__(
            name="writing_assistant",
            system_prompt="""You are an expert writing assistant.
            Help users improve their writing by:
            - Suggesting better phrasing
            - Fixing grammar and style issues
            - Expanding on ideas
            - Maintaining consistent tone

            Current document context: {content}
            Current selection: {selection}
            """
        )

    async def process(self, state: EditorState) -> EditorState:
        # Get current content and selection
        content = state.get("content", "")
        selection = state.get("selection", {})

        # Process with LLM
        result = await self.chain.ainvoke({
            "input": state["messages"][-1].content,
            "content": content,
            "selection": selection,
        })

        # Update state
        state["messages"].append(result)
        state["task_status"] = "completed"

        return state
```

**Code Assistant (`backend/agents/code_assistant.py`)**

```python
from .base_agent import BaseAgent
from ..state.state_manager import EditorState

class CodeAssistant(BaseAgent):
    def __init__(self):
        super().__init__(
            name="code_assistant",
            system_prompt="""You are an expert programming assistant.
            Help users with:
            - Code completion and suggestions
            - Bug finding and fixing
            - Code explanation
            - Refactoring suggestions

            Current code: {content}
            Selected code: {selection}
            """
        )

    async def process(self, state: EditorState) -> EditorState:
        content = state.get("content", "")
        selection = state.get("selection", {})

        result = await self.chain.ainvoke({
            "input": state["messages"][-1].content,
            "content": content,
            "selection": selection,
        })

        state["messages"].append(result)
        return state
```

**3.4 Create LangGraph Workflow (`backend/graphs/editor_graph.py`)**

```python
from langgraph.graph import StateGraph, END
from ..state.state_manager import EditorState
from ..agents.writing_assistant import WritingAssistant
from ..agents.code_assistant import CodeAssistant

def create_editor_graph():
    """Create LangGraph workflow for editor agents."""

    # Initialize agents
    writing_agent = WritingAssistant()
    code_agent = CodeAssistant()

    # Create graph
    workflow = StateGraph(EditorState)

    # Add nodes
    workflow.add_node("writing_assistant", writing_agent.process)
    workflow.add_node("code_assistant", code_agent.process)

    # Add conditional routing
    def route_agent(state: EditorState) -> str:
        """Route to appropriate agent based on state."""
        current_agent = state.get("current_agent", "writing_assistant")
        return current_agent

    # Set entry point
    workflow.set_conditional_entry_point(
        route_agent,
        {
            "writing_assistant": "writing_assistant",
            "code_assistant": "code_assistant",
        }
    )

    # Add edges
    workflow.add_edge("writing_assistant", END)
    workflow.add_edge("code_assistant", END)

    return workflow.compile()
```

**3.5 Create FastAPI Server (`backend/api/main.py`)**

```python
from fastapi import FastAPI, WebSocket
from fastapi.middleware.cors import CORSMiddleware
from ..graphs.editor_graph import create_editor_graph
from ..state.state_manager import EditorState
import json

app = FastAPI()

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vite dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize graph
graph = create_editor_graph()

@app.post("/copilot")
async def copilot_endpoint(request: dict):
    """CopilotKit-compatible endpoint."""
    # Extract state from request
    state: EditorState = {
        "content": request.get("context", {}).get("content", ""),
        "selection": request.get("context", {}).get("selection", {}),
        "messages": request.get("messages", []),
        "current_agent": request.get("agent", "writing_assistant"),
        "task_status": "pending",
        "metadata": {},
    }

    # Process through graph
    result = await graph.ainvoke(state)

    # Return response
    return {
        "message": result["messages"][-1].content,
        "state": result,
    }

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket for real-time updates."""
    await websocket.accept()

    try:
        while True:
            # Receive state update from frontend
            data = await websocket.receive_text()
            state = json.loads(data)

            # Process through graph
            result = await graph.ainvoke(state)

            # Send back to frontend
            await websocket.send_json(result)
    except Exception as e:
        print(f"WebSocket error: {e}")
    finally:
        await websocket.close()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Step 4: Implement Editor Component

Based on editor type, create the appropriate component:

**4.1 Text Editor (Monaco/VS Code)**

```typescript
// frontend/src/components/Editor/TextEditor.tsx
import Editor from '@monaco-editor/react';
import { useEditorStore } from '../../state/editorStore';
import { useEditorCopilot } from '../../hooks/useEditorCopilot';

export function TextEditor() {
  const { content, updateContent, selection } = useEditorStore();

  // Connect to CopilotKit
  useEditorCopilot({ content, selection });

  return (
    <Editor
      height="90vh"
      defaultLanguage="typescript"
      value={content}
      onChange={(value) => updateContent(value || '')}
      onCursorSelectionChange={(selection) => {
        useEditorStore.setState({ selection });
      }}
      theme="vs-dark"
      options={{
        minimap: { enabled: true },
        fontSize: 14,
        wordWrap: 'on',
      }}
    />
  );
}
```

**4.2 Document Editor (Slate)**

```typescript
// frontend/src/components/Editor/DocumentEditor.tsx
import { createEditor } from 'slate';
import { Slate, Editable, withReact } from 'slate-react';
import { useState, useMemo } from 'react';
import { useEditorCopilot } from '../../hooks/useEditorCopilot';

export function DocumentEditor() {
  const editor = useMemo(() => withReact(createEditor()), []);
  const [value, setValue] = useState(initialValue);

  // Connect to CopilotKit
  useEditorCopilot({ content: JSON.stringify(value) });

  return (
    <Slate editor={editor} value={value} onChange={setValue}>
      <Editable
        placeholder="Start writing..."
        renderLeaf={renderLeaf}
        renderElement={renderElement}
      />
    </Slate>
  );
}
```

**4.3 Node-Based Editor (ReactFlow)**

```typescript
// frontend/src/components/Editor/NodeEditor.tsx
import ReactFlow, {
  MiniMap,
  Controls,
  Background,
  useNodesState,
  useEdgesState,
} from 'reactflow';
import 'reactflow/dist/style.css';
import { useEditorCopilot } from '../../hooks/useEditorCopilot';

export function NodeEditor() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);

  // Connect to CopilotKit
  useEditorCopilot({
    content: JSON.stringify({ nodes, edges }),
    selection: selectedNodes,
  });

  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        fitView
      >
        <Controls />
        <MiniMap />
        <Background variant="dots" gap={12} size={1} />
      </ReactFlow>
    </div>
  );
}
```

### Step 5: State Management

**5.1 Create Editor Store (Zustand)**

```typescript
// frontend/src/state/editorStore.ts
import { create } from 'zustand';

interface EditorState {
  content: string;
  selection: {
    start: number;
    end: number;
    text: string;
  };
  history: string[];
  currentAgent: string;

  // Actions
  updateContent: (content: string) => void;
  updateSelection: (selection: EditorState['selection']) => void;
  setAgent: (agent: string) => void;
  undo: () => void;
  redo: () => void;
}

export const useEditorStore = create<EditorState>((set, get) => ({
  content: '',
  selection: { start: 0, end: 0, text: '' },
  history: [],
  currentAgent: 'writing_assistant',

  updateContent: (content) =>
    set((state) => ({
      content,
      history: [...state.history, state.content],
    })),

  updateSelection: (selection) =>
    set({ selection }),

  setAgent: (agent) =>
    set({ currentAgent: agent }),

  undo: () => {
    const { history } = get();
    if (history.length > 0) {
      set({
        content: history[history.length - 1],
        history: history.slice(0, -1),
      });
    }
  },

  redo: () => {
    // Implement redo logic
  },
}));
```

**5.2 Create Agent Store**

```typescript
// frontend/src/state/agentStore.ts
import { create } from 'zustand';

interface Agent {
  id: string;
  name: string;
  description: string;
  status: 'idle' | 'working' | 'completed' | 'error';
}

interface AgentState {
  agents: Agent[];
  activeAgent: string | null;
  taskQueue: Task[];

  // Actions
  setActiveAgent: (agentId: string) => void;
  updateAgentStatus: (agentId: string, status: Agent['status']) => void;
  addTask: (task: Task) => void;
}

export const useAgentStore = create<AgentState>((set) => ({
  agents: [
    {
      id: 'writing_assistant',
      name: 'Writing Assistant',
      description: 'Helps with writing and editing',
      status: 'idle',
    },
    {
      id: 'code_assistant',
      name: 'Code Assistant',
      description: 'Helps with coding and debugging',
      status: 'idle',
    },
  ],
  activeAgent: null,
  taskQueue: [],

  setActiveAgent: (agentId) =>
    set({ activeAgent: agentId }),

  updateAgentStatus: (agentId, status) =>
    set((state) => ({
      agents: state.agents.map((agent) =>
        agent.id === agentId ? { ...agent, status } : agent
      ),
    })),

  addTask: (task) =>
    set((state) => ({
      taskQueue: [...state.taskQueue, task],
    })),
}));
```

### Step 6: Connect Frontend to Backend

**6.1 Create API Client (`frontend/src/lib/api-client.ts`)**

```typescript
interface CopilotRequest {
  context: {
    content: string;
    selection: any;
  };
  messages: any[];
  agent: string;
}

export class AgentAPIClient {
  private baseUrl: string;

  constructor(baseUrl: string = 'http://localhost:8000') {
    this.baseUrl = baseUrl;
  }

  async sendMessage(request: CopilotRequest) {
    const response = await fetch(`${this.baseUrl}/copilot`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(request),
    });

    if (!response.ok) {
      throw new Error(`API error: ${response.statusText}`);
    }

    return response.json();
  }

  // WebSocket connection for real-time updates
  connectWebSocket(onMessage: (data: any) => void) {
    const ws = new WebSocket(`ws://localhost:8000/ws`);

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      onMessage(data);
    };

    return ws;
  }
}
```

**6.2 Create Custom Hook (`frontend/src/hooks/useAgents.ts`)**

```typescript
import { useEffect, useState } from 'react';
import { AgentAPIClient } from '../lib/api-client';
import { useEditorStore } from '../state/editorStore';
import { useAgentStore } from '../state/agentStore';

export function useAgents() {
  const [client] = useState(() => new AgentAPIClient());
  const { content, selection, currentAgent } = useEditorStore();
  const { updateAgentStatus } = useAgentStore();

  const sendToAgent = async (message: string) => {
    updateAgentStatus(currentAgent, 'working');

    try {
      const response = await client.sendMessage({
        context: { content, selection },
        messages: [{ role: 'user', content: message }],
        agent: currentAgent,
      });

      updateAgentStatus(currentAgent, 'completed');
      return response;
    } catch (error) {
      updateAgentStatus(currentAgent, 'error');
      throw error;
    }
  };

  return { sendToAgent };
}
```

### Step 7: Configuration Files

**7.1 Environment Variables**

Create `.env.example`:
```bash
# Frontend
VITE_API_URL=http://localhost:8000

# Backend
OPENAI_API_KEY=your-api-key-here
ANTHROPIC_API_KEY=optional-claude-key
LANGCHAIN_API_KEY=optional-langsmith-key
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=coeditor-project
```

**7.2 Docker Compose (Optional)**

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    environment:
      - VITE_API_URL=http://backend:8000
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./backend:/app
```

### Step 8: Development Scripts

**8.1 Root Package.json**

```json
{
  "name": "coeditor-app",
  "private": true,
  "workspaces": ["frontend", "backend"],
  "scripts": {
    "dev": "concurrently \"npm run dev:frontend\" \"npm run dev:backend\"",
    "dev:frontend": "cd frontend && npm run dev",
    "dev:backend": "cd backend && python -m uvicorn api.main:app --reload",
    "build": "npm run build:frontend",
    "build:frontend": "cd frontend && npm run build",
    "type-check": "cd frontend && npm run type-check"
  },
  "devDependencies": {
    "concurrently": "^8.2.0"
  }
}
```

## Advanced Patterns

### Multi-Agent Workflows

For complex tasks requiring multiple agents:

```python
# backend/graphs/multi_agent_graph.py
from langgraph.graph import StateGraph, END

def create_multi_agent_workflow():
    workflow = StateGraph(EditorState)

    # Add multiple agents
    workflow.add_node("researcher", researcher_agent.process)
    workflow.add_node("writer", writer_agent.process)
    workflow.add_node("reviewer", reviewer_agent.process)

    # Create workflow: research → write → review
    workflow.set_entry_point("researcher")
    workflow.add_edge("researcher", "writer")
    workflow.add_edge("writer", "reviewer")

    # Conditional loop back for revisions
    workflow.add_conditional_edges(
        "reviewer",
        should_revise,
        {
            "revise": "writer",
            "approve": END,
        }
    )

    return workflow.compile()
```

### Streaming Responses

For better UX with long-running tasks:

```typescript
// Frontend streaming
async function* streamAgentResponse(message: string) {
  const response = await fetch('/api/copilot/stream', {
    method: 'POST',
    body: JSON.stringify({ message }),
  });

  const reader = response.body?.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    yield chunk;
  }
}
```

### Context-Aware Actions

Make agents aware of editor context:

```typescript
useCopilotAction({
  name: "improveSelection",
  description: "Improve the currently selected text",
  handler: async () => {
    const { selection } = useEditorStore.getState();
    const improved = await sendToAgent(
      `Improve this text: ${selection.text}`
    );
    replaceSelection(improved);
  },
});
```

## Testing

### Frontend Tests

```typescript
// frontend/src/components/Editor/__tests__/TextEditor.test.tsx
import { render, screen } from '@testing-library/react';
import { TextEditor } from '../TextEditor';

describe('TextEditor', () => {
  it('renders editor', () => {
    render(<TextEditor />);
    expect(screen.getByRole('textbox')).toBeInTheDocument();
  });

  it('updates content on change', async () => {
    // Test implementation
  });
});
```

### Backend Tests

```python
# backend/tests/test_agents.py
import pytest
from agents.writing_assistant import WritingAssistant

@pytest.mark.asyncio
async def test_writing_assistant():
    agent = WritingAssistant()
    state = {
        "content": "This is a test",
        "messages": [{"role": "user", "content": "Improve this"}],
    }
    result = await agent.process(state)
    assert "messages" in result
```

## Deployment

### Frontend Deployment (Vercel)

```bash
cd frontend
vercel deploy --prod
```

### Backend Deployment (Railway/Render)

```bash
# Dockerfile for backend
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Best Practices Checklist

When building the application, ensure:

### Architecture
- [ ] Clear separation between frontend and backend
- [ ] Type-safe communication (TypeScript + Pydantic)
- [ ] Shared state schema between frontend and backend
- [ ] Proper error handling and fallbacks

### CopilotKit Integration
- [ ] Context providers configured for editor state
- [ ] Custom actions defined for common tasks
- [ ] Proper streaming for long responses
- [ ] Keyboard shortcuts configured

### LangGraph Implementation
- [ ] Agents properly isolated with clear responsibilities
- [ ] State transitions well-defined
- [ ] Error recovery mechanisms
- [ ] Logging and observability (LangSmith)

### Editor Experience
- [ ] Responsive UI with loading states
- [ ] Undo/redo functionality
- [ ] Auto-save and persistence
- [ ] Keyboard shortcuts
- [ ] Accessibility (ARIA labels)

### Performance
- [ ] Debounced state updates
- [ ] Lazy loading for heavy components
- [ ] Optimized bundle size
- [ ] Caching for API responses

### Security
- [ ] API key protection (server-side only)
- [ ] Input validation
- [ ] Rate limiting
- [ ] CORS properly configured
- [ ] Sanitized user inputs

## Common Pitfalls to Avoid

- ❌ Exposing API keys in frontend code
- ❌ Not handling agent failures gracefully
- ❌ Blocking UI while waiting for agent responses
- ❌ Not validating state updates from agents
- ❌ Forgetting to cleanup WebSocket connections
- ❌ Not implementing proper error boundaries
- ❌ Ignoring TypeScript errors
- ❌ Not testing on different screen sizes
- ❌ Hardcoding backend URLs
- ❌ Not implementing proper loading states

## Example Use Cases

### 1. AI-Powered Writing App
- **Editor**: Rich text (TipTap)
- **Agents**: Writing assistant, grammar checker, style improver
- **Features**: Real-time suggestions, tone adjustment, expansion

### 2. Code Collaboration Tool
- **Editor**: Monaco (VS Code)
- **Agents**: Code assistant, reviewer, documentation generator
- **Features**: Code completion, bug detection, auto-documentation

### 3. Workflow Designer
- **Editor**: ReactFlow nodes
- **Agents**: Workflow optimizer, validator, template suggester
- **Features**: Smart node suggestions, validation, auto-layout

## Reference Documentation

For detailed implementation patterns and API references:
- `references/copilotkit-patterns.md` - CopilotKit integration patterns
- `references/langgraph-agents.md` - LangGraph agent implementations
- `references/state-management.md` - State synchronization patterns
- `references/editor-integrations.md` - Editor-specific implementations

## Post-Setup Checklist

After scaffolding, guide the user to:

1. ✅ Configure environment variables (.env)
2. ✅ Set up OpenAI or LLM provider API key
3. ✅ Test frontend development server
4. ✅ Test backend API server
5. ✅ Verify CopilotKit connection
6. ✅ Test agent responses
7. ✅ Customize agent prompts
8. ✅ Add custom actions
9. ✅ Implement additional features
10. ✅ Deploy to production

## Troubleshooting

### CopilotKit not connecting
- Verify `runtimeUrl` is correct
- Check CORS configuration on backend
- Ensure backend is running

### Agents not responding
- Check API keys are configured
- Verify LangGraph state schema matches
- Check backend logs for errors

### Editor state not syncing
- Verify `useCopilotReadable` is called
- Check WebSocket connection
- Ensure state updates are properly dispatched

## Version Compatibility

This skill targets:
- **React**: 18+
- **CopilotKit**: Latest (@copilotkit/react-core)
- **LangGraph**: Latest (langgraph)
- **Python**: 3.9+
- **Node.js**: 18+
- **TypeScript**: 5+

## Next Steps

After scaffolding:
1. Customize agent prompts for your use case
2. Add domain-specific actions
3. Implement additional editor features
4. Set up analytics and monitoring
5. Plan deployment strategy
6. Create user documentation
