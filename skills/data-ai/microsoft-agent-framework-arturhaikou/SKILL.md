---
name: microsoft-agent-framework
description: Expert guidance for implementing AI agents and multi-agent workflows using Microsoft Agent Framework. Use when adding AI agent capabilities, implementing multi-agent orchestration patterns, integrating MCP tools, or building intelligent automation systems. Emphasizes gathering up-to-date information from official documentation before implementation.
compatibility: Requires .NET 8.0 or later. Supports cloud providers (Azure OpenAI, OpenAI, Anthropic, GitHub Copilot) with network access, or local models via Ollama (offline capable)
metadata:
  author: unified-ai-tracker
  version: "1.0"
  framework-version: "preview"
  nuget-package: "Microsoft.Agents.AI"
---

# Microsoft Agent Framework Development

Build AI agents and multi-agent workflows using Microsoft Agent Framework, the next-generation framework combining Semantic Kernel and AutoGen.

## When to Use This Skill

- Building AI agents with LLM capabilities
- Implementing multi-agent orchestration (Sequential, Concurrent, Group Chat, Handoff, Magentic)
- Integrating MCP (Model Context Protocol) tools
- Building workflow automation
- Managing agent sessions and state
- Connecting to Azure OpenAI, OpenAI, Anthropic, GitHub Copilot, or local models (Ollama)

## ⚠️ Critical Warning: Framework is Brand New

**Microsoft Agent Framework is in preview (2025)** - Your LLM training data predates this framework entirely.

### Why This Matters

- APIs change frequently between preview releases
- Package names, namespaces, and classes are unstable  
- Your training data **has zero information** about this framework
- Using cached knowledge will produce completely broken code
- Do NOT assume compatibility with Semantic Kernel or AutoGen

### Mandatory Pre-Implementation Checklist

Before implementing ANY Agent Framework feature:

1. ✅ Search `mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <topic>")`
2. ✅ Query `mcp_microsoft_doc_microsoft_code_sample_search(query: "<feature>", language: "csharp")`  
3. ✅ Fetch tutorials with `mcp_microsoft_doc_microsoft_docs_fetch(url: "<tutorial-url>")`
4. ✅ Verify package names, versions, namespaces from official docs
5. ✅ Check API signatures from code samples
6. ❌ **NEVER** rely on cached LLM knowledge
7. ❌ **NEVER** assume based on Semantic Kernel/AutoGen
8. ❌ **NEVER** skip documentation verification

### Tool Selection

**For Agent Framework**:
- `mcp_microsoft_doc_microsoft_docs_search` - Search Microsoft Learn
- `mcp_microsoft_doc_microsoft_code_sample_search(query, language: "csharp")` - Find code samples
- `mcp_microsoft_doc_microsoft_docs_fetch` - Get complete tutorials

**DO NOT** rely on internal knowledge - it didn't exist when you were trained.

## Framework Overview

**Microsoft Agent Framework** = Semantic Kernel + AutoGen + New Capabilities

**Core Features**:
- AI agents powered by LLMs with tool calling
- Graph-based workflows orchestrating multiple agents
- MCP integration for tools
- Session-based state management
- Pre-built orchestration patterns
- Type safety for messages and components
- Middleware and telemetry

**Agent Types**: Chat Agents (Azure OpenAI, OpenAI), Anthropic Agents, GitHub Copilot Agents, A2A Agents, Custom Agents

**Orchestration Patterns**:

| Pattern | Topology | Use Case |
|---------|----------|----------|
| Sequential | Chain | Pipelines, multi-stage processing |
| Concurrent | Broadcast | Parallel analysis, independent tasks |
| Group Chat | Star | Iterative refinement, collaboration |
| Handoff | Mesh | Dynamic delegation, expert routing |
| Magentic | Star | Complex planning, generalist tasks |

See [references/ORCHESTRATION.md](references/ORCHESTRATION.md) for detailed orchestration guidance.

## Local AI with Ollama

Microsoft Agent Framework supports **Ollama** for running local AI models - enabling cost-free development, offline work, and data privacy.

**Quick Start**:
```powershell
# Install and start Ollama
ollama serve
ollama pull phi3:mini

# Add package
dotnet add package OllamaSharp
```

**Create Agent**:
```csharp
using Microsoft.Agents.AI;
using OllamaSharp;

using OllamaApiClient chatClient = new(
    new Uri("http://localhost:11434"), "phi3:mini");

AIAgent agent = new ChatClientAgent(chatClient,
    instructions: "You are a helpful assistant.");

await agent.RunAsync("Explain async/await in C#");
```

**Recommended Models**: `phi3:mini` (fast development), `phi3.5` (production), `llama3.1` (general purpose)

See [references/OLLAMA.md](references/OLLAMA.md) for complete documentation including:
- Docker setup and GPU acceleration
- IChatClient middleware and caching
- Function calling and tool use
- Troubleshooting and best practices
- Integration with unifiedaitracker

## Critical Rules

<rules>
    <rule type="critical">
        **Mandatory Research**
        - Follow pre-implementation checklist before ANY implementation
        - NEVER skip documentation verification
        - Framework is PREVIEW - APIs change frequently
        - Your training data does not include this framework
    </rule>

    <rule type="critical">
        **Search Before Code**
        - Use `microsoft_docs_search("Microsoft Agent Framework <topic>")`
        - Use `microsoft_code_sample_search(query: "...", language: "csharp")`
        - Verify all package names, namespaces, API signatures
    </rule>

    <rule type="critical">
        **Package Verification**
        - ALL packages require **--prerelease** flag
        - Common packages: `Microsoft.Agents.AI`, `Microsoft.Agents.AI.OpenAI`, `Microsoft.Agents.AI.Anthropic`
        - Always verify names from official docs
    </rule>

    <rule type="restriction">
        **No Old Framework Assumptions**
        - Do NOT use Semantic Kernel patterns (`Kernel`, `KernelFunction`)
        - Do NOT use AutoGen patterns (`ConversableAgent`, `GroupChat`)
        - Agent Framework has different APIs - search documentation
    </rule>
</rules>

## Standard Workflow

### Phase 1: Discovery & Planning

**1. Identify Requirements**
- Single agent or multi-agent orchestration?
- Which AI provider? (Azure OpenAI, OpenAI, Anthropic, GitHub Copilot)
- Tool/MCP integration needs?
- State management requirements?

**2. Research Current Implementation**
```
Step 1: Search concepts
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <topic>")

Step 2: Find code samples
mcp_microsoft_doc_microsoft_code_sample_search(
  query: "Microsoft Agent Framework <feature>",
  language: "csharp"
)

Step 3: Get tutorial (if needed)
mcp_microsoft_doc_microsoft_docs_fetch(url: <url>)
```

**3. Verify Prerequisites**
- .NET 8.0 SDK or later
- AI provider access configured
- Environment variables ready
- Authentication credentials available

### Phase 2: Implementation

**1. Create Project**
```powershell
dotnet new console -o MyAgentProject  # or 'web' for ASP.NET Core
cd MyAgentProject
```

**2. Install Packages** (Always verify names from docs!)
```powershell
# Core (always required)
dotnet add package Microsoft.Agents.AI --prerelease

# For Azure OpenAI
dotnet add package Azure.AI.OpenAI --prerelease
dotnet add package Azure.Identity
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease

# For Anthropic
dotnet add package Microsoft.Agents.AI.Anthropic --prerelease

# For GitHub Copilot
dotnet add package GitHub.Copilot.SDK --prerelease

# For Ollama (local models) - see references/OLLAMA.md
dotnet add package OllamaSharp
```

**3. Implement Using Official Patterns**
- Follow exact patterns from official samples
- Verify class names, namespaces, method signatures
- Don't improvise based on Semantic Kernel/AutoGen

**4. Configure Environment**
```powershell
# Azure OpenAI
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
$env:AZURE_OPENAI_DEPLOYMENT_NAME="gpt-4o-mini"

# Anthropic
$env:ANTHROPIC_API_KEY="your-api-key"
$env:ANTHROPIC_DEPLOYMENT_NAME="claude-haiku-4-5"

# For Ollama setup, see references/OLLAMA.md
```

### Phase 3: Testing & Validation

```powershell
dotnet build
dotnet run
```

Verify agent behavior, tool calling, state management, and error handling.

## Quick Reference

### Creating Basic Agent

**Always search first**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework create agent Azure OpenAI")
mcp_microsoft_doc_microsoft_code_sample_search(query: "AIAgent Azure OpenAI", language: "csharp")
```

**Azure OpenAI Pattern** (verify before using):
```csharp
using Azure.AI.OpenAI;
using Azure.Identity;
using Microsoft.Extensions.AI;
using Microsoft.Agents.AI;

var client = new AzureOpenAIClient(
    new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")),
    new DefaultAzureCredential());

var chatClient = client
    .GetChatClient(Environment.GetEnvironmentVariable("AZURE_OPENAI_DEPLOYMENT_NAME"))
    .AsIChatClient();

var agent = chatClient.AsAIAgent(new()
{
    Instructions = "You are a helpful assistant."
});

var response = await agent.RunAsync("What is Microsoft Agent Framework?");
```

**Ollama (Local) Pattern** (verify before using):
```csharp
using Microsoft.Agents.AI;
using OllamaSharp;

using OllamaApiClient chatClient = new(
    new Uri("http://localhost:11434"), 
    "phi3:mini");

AIAgent agent = new ChatClientAgent(
    chatClient,
    instructions: "You are a helpful assistant.",
    name: "LocalAssistant");

var response = await agent.RunAsync("What is Microsoft Agent Framework?");
```

### MCP Tools

Search: `"Microsoft Agent Framework MCP tools"`

### Multi-Agent Workflows

Search: `"Microsoft Agent Framework <pattern> orchestration"` where pattern is:
- sequential - Pipeline processing
- concurrent - Parallel analysis  
- group chat - Collaborative refinement
- handoff - Dynamic expert routing
- magentic - Complex task planning

See [references/ORCHESTRATION.md](references/ORCHESTRATION.md) for details.

### State Management

Search: `"Microsoft Agent Framework session state management"`

## Implementation Patterns

See [references/PATTERNS.md](references/PATTERNS.md) for detailed patterns:

1. **Single Agent with Tools** - Function calling and tool integration
2. **Sequential Workflow** - Pipeline processing with multiple agents
3. **Group Chat** - Collaborative refinement with manager
4. **Human-in-the-Loop** - Checkpoint-based approval workflows
5. **A2A Communication** - Agent-to-Agent protocol for remote agents

## Common Use Cases

**Customer Support Agent**:
```
Search: "Microsoft Agent Framework chat agent Azure OpenAI"
        "Microsoft Agent Framework tools function calling"
        "Microsoft Agent Framework session state management"
```

**Document Processing Pipeline (Sequential)**:
```
Search: "Microsoft Agent Framework sequential workflow orchestration"
Implement: Extraction → Analysis → Summarization agents
```

**Collaborative Analysis (Group Chat)**:
```
Search: "Microsoft Agent Framework group chat RoundRobinGroupChatManager"
Implement: Multiple analyst agents with iterative refinement
```

**Expert Routing (Handoff)**:
```
Search: "Microsoft Agent Framework handoff orchestration mesh"
Implement: Domain experts with dynamic delegation
```

**Complex Automation (Magentic)**:
```
Search: "Microsoft Agent Framework magentic planner"
Implement: Specialized agents with planner-based coordination
```

**Local/Offline Development**:
```
Use: Ollama with phi3 models for cost-free development
See: references/OLLAMA.md for complete setup
```

## Integration with unifiedaitracker

**Approach**:
1. Add packages to `unifiedaitracker.Application`
2. Create agent services (e.g., `TicketAnalysisService`)
3. Register in `unifiedaitracker.Infrastructure/Extensions.cs`
4. Use via dependency injection in controllers
5. Configure OpenAI connection in AppHost

**Search before implementing**:
```
"Microsoft Agent Framework dependency injection ASP.NET Core"
"Microsoft Agent Framework configuration options"
```

## Best Practices

1. **Always Verify Documentation First** - Search, find samples, fetch tutorials
2. **Use Official Code Samples as Templates** - Don't improvise
3. **Handle Prerelease Carefully** - Always use `--prerelease`, expect breaking changes
4. **Implement Error Handling** - Search for current patterns
5. **Use Telemetry and Monitoring** - Built-in observability
6. **Secure Configurations** - Use environment variables or Key Vault
7. **Test Thoroughly** - Various inputs, tool calling, error scenarios
8. **Design for Observability** - Use middleware and filters

## Troubleshooting

See [references/TROUBLESHOOTING.md](references/TROUBLESHOOTING.md) for detailed solutions.

**Agent Framework Issues**:
- **Package Not Found** → Missing `--prerelease` flag or wrong package name
- **API Signature Mismatch** → Using outdated patterns, search for current API
- **Authentication Failures** → Verify environment variables and Azure RBAC
- **Workflow Not Executing** → Verify workflow builder syntax from docs
- **Agent Not Using Tools** → Check tool definition and registration

**Ollama Issues**: See [references/OLLAMA.md](references/OLLAMA.md#troubleshooting) for Ollama-specific troubleshooting

## Migration

**From Semantic Kernel or AutoGen**: Do NOT assume compatibility.

Fetch migration guides:
```
mcp_microsoft_doc_microsoft_docs_fetch(
  url: "https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/"
)
```

## Success Indicators

✅ **Using skill effectively**:
- Search docs before every implementation
- Use official code samples as templates
- Verify all package names and API signatures
- Use `--prerelease` flag consistently
- Code matches current patterns

❌ **Warning signs**:
- Implementing from memory
- Using Semantic Kernel/AutoGen patterns
- Skipping documentation search
- Guessing at package names/APIs

## Essential Commands

```powershell
# Search docs
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <topic>")

# Find code samples
mcp_microsoft_doc_microsoft_code_sample_search(query: "...", language: "csharp")

# Install packages (ALWAYS use --prerelease for Agent Framework)
dotnet add package Microsoft.Agents.AI --prerelease
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease

# For local Ollama (see references/OLLAMA.md for complete setup)
dotnet add package OllamaSharp
ollama serve && ollama pull phi3:mini
```

## Additional Resources

See [references/QUICK_REFERENCE.md](references/QUICK_REFERENCE.md) for command cheat sheet.

**Official docs**:
- Overview: https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview
- Quickstart: https://learn.microsoft.com/en-us/agent-framework/tutorials/quick-start
- User Guide: https://learn.microsoft.com/en-us/agent-framework/user-guide/overview
- ChatClientAgent: https://learn.microsoft.com/en-us/agent-framework/user-guide/agents/agent-types/chat-client-agent
- GitHub: https://github.com/microsoft/agent-framework

**Local AI with Ollama**: See [references/OLLAMA.md](references/OLLAMA.md) for complete documentation

---

## Summary

Microsoft Agent Framework is **brand new (2025)** and in **preview**:

1. **NEVER trust cached knowledge** - framework didn't exist when you were trained
2. **ALWAYS search documentation first** - APIs change frequently
3. **ALWAYS verify package names** - use official docs
4. **ALWAYS use official code samples** - don't improvise
5. **ALWAYS use --prerelease flag** - all packages are in preview

**Use Microsoft documentation MCP tools to gather current information before every implementation.**

**Never skip the mandatory pre-implementation checklist.**

**For local development**: Use Ollama with Microsoft's Phi-3 models for cost-free, offline-capable development with the same agent code.