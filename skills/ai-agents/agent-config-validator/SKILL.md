---
name: agent-config-validator
description: Validate AgentConfig definitions for the Agent Framework. Use when creating or modifying agent configurations to ensure correct structure, valid tool references, and proper sub-agent composition. Validates TypeScript interfaces and Python Pydantic models.
allowed-tools: Read, Grep, Glob
---

# Agent Configuration Validation

Validates AgentConfig definitions against the Agent Framework schema.

## AgentConfig Schema

```typescript
interface ModelConfig {
  provider: 'gemini' | 'openai' | 'anthropic' | 'ollama' | 'custom';
  model: string;           // e.g., "gpt-4o", "claude-3-haiku", "llama3.2"
  baseUrl?: string;        // For custom/Ollama endpoints
  apiKeyEnvVar?: string;   // Environment variable name for API key
}

interface AgentConfig {
  id: string;                              // Required: unique identifier
  name: string;                            // Required: display name
  type: 'llm' | 'sequential' | 'parallel' | 'loop' | 'custom';  // Required
  modelConfig: ModelConfig;                // Required for type='llm'
  description: string;                     // Required: for orchestrator routing
  instruction: string;                     // Required: system prompt
  tools: string[];                         // MCP tool IDs
  subAgents: string[];                     // Sub-agent IDs
  outputKey?: string;                      // For state passing
  maxIterations?: number;                  // For LoopAgent (default: 3)
  stateSchema?: Record<string, any>;       // Optional state definition
  createdAt: Date;
  isActive: boolean;
}
```

## Validation Rules

### Required Fields

| Field | Type | Condition |
|-------|------|-----------|
| `id` | string | Always required, must be unique |
| `name` | string | Always required |
| `type` | enum | Must be one of: 'llm', 'sequential', 'parallel', 'loop', 'custom' |
| `description` | string | Always required (used for routing) |
| `instruction` | string | Required for 'llm' type |
| `modelConfig` | object | Required for 'llm' type |

### Type-Specific Rules

#### LLM Agent
- Must have `modelConfig` with valid `provider` and `model`
- Must have `instruction` (system prompt)
- `outputKey` recommended for state passing

#### Sequential Agent
- Must have at least 2 agents in `subAgents`
- Order matters (first to last execution)

#### Parallel Agent
- Must have at least 2 agents in `subAgents`
- Each sub-agent should have unique `outputKey`

#### Loop Agent
- Must have `subAgents`
- Should specify `maxIterations` (default: 3)

### Reference Validation

- **Tool references**: Each tool ID in `tools[]` must exist in MCP registry
- **Sub-agent references**: Each ID in `subAgents[]` must be a valid agent ID
- **Model provider**: Must be a supported provider with valid credentials

## Example Validations

### Valid LLM Agent

```json
{
  "id": "research-agent",
  "name": "Research Agent",
  "type": "llm",
  "modelConfig": {
    "provider": "gemini",
    "model": "gemini-2.5-flash"
  },
  "description": "Gathers and summarizes information from various sources",
  "instruction": "You are a research assistant...",
  "tools": ["web_search", "document_reader"],
  "subAgents": [],
  "outputKey": "research_results",
  "isActive": true
}
```

### Invalid Configuration (Missing Fields)

```json
{
  "id": "broken-agent",
  "name": "Broken",
  "type": "llm"
  // MISSING: modelConfig, description, instruction
}
```

## Validation Commands

```bash
# Validate Python Pydantic model
uv run python -c "from agent.state.models import AgentConfig; AgentConfig.model_validate(config)"

# Check TypeScript interface
bun run typecheck
```

## Common Issues

1. **Missing description**: Orchestrator can't route to agent
2. **Invalid tool references**: Tools won't load at runtime
3. **Circular sub-agent references**: Causes infinite loops
4. **Missing outputKey in parallel**: Results overwrite each other
