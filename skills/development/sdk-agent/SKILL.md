---
name: sdk-agent
description: Create and setup Claude Agent SDK applications in TypeScript or Python. Use when the user wants to build custom agents, create new SDK projects, or set up agent development environments. Triggers on requests like "create an agent", "build an SDK app", "new agent project".
---

# Claude Agent SDK Setup

Create and configure Claude Agent SDK applications following official documentation and best practices.

## Quick Start

1. **Gather Requirements**: Language (TypeScript/Python), project name, agent type
2. **Initialize Project**: Create directory, package manager, configuration
3. **Install SDK**: Latest version from npm/PyPI
4. **Create Starter Files**: Basic agent with proper imports and error handling
5. **Setup Environment**: API keys, .env configuration
6. **Verify**: Run type checks and validation

## Language Options

### TypeScript
```bash
npm init -y
npm install @anthropic-ai/claude-agent-sdk@latest
```

Create `index.ts` with:
- Proper imports from SDK
- Basic agent configuration
- Error handling
- Type definitions

### Python
```bash
pip install claude-agent-sdk
```

Create `main.py` with:
- SDK imports
- Agent configuration
- Error handling

## Project Structure

```
project-name/
├── src/
│   └── index.ts (or main.py)
├── .env.example
├── .gitignore
├── package.json (or requirements.txt)
└── tsconfig.json (TypeScript only)
```

## Environment Setup

Create `.env.example`:
```
ANTHROPIC_API_KEY=your_api_key_here
```

Get API key from: https://console.anthropic.com/

## Verification

After setup, verify:
- TypeScript: Run `npx tsc --noEmit` - fix all type errors
- Python: Verify syntax and imports

## Resources

- TypeScript SDK: https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/sdk
- Python SDK: https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/sdk
- Overview: https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview

## Detailed References

- [references/typescript-setup.md](references/typescript-setup.md) - TypeScript configuration
- [references/python-setup.md](references/python-setup.md) - Python configuration
- [references/custom-tools.md](references/custom-tools.md) - Adding custom tools
- [references/mcp-integration.md](references/mcp-integration.md) - MCP server integration

## Common Next Steps

After initial setup:
1. Customize system prompt
2. Add custom tools via MCP
3. Configure permissions
4. Create subagents
5. Add sessions for persistence
