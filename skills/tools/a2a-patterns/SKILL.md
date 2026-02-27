---
name: a2a-patterns
description: Agent-to-Agent (A2A) protocol implementation patterns for Google ADK - exposing agents via A2A, consuming external agents, multi-agent communication, and protocol configuration. Use when building multi-agent systems, implementing A2A protocol, exposing agents as services, consuming remote agents, configuring agent cards, or when user mentions A2A, agent-to-agent, multi-agent collaboration, remote agents, or agent orchestration.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# A2A Protocol Implementation Patterns

## Instructions

This skill provides comprehensive patterns for implementing the Agent2Agent (A2A) protocol in Google's Agent Development Kit (ADK). The A2A protocol standardizes communication between AI agents, enabling multi-agent collaboration across different platforms and frameworks.

## What is A2A?

The Agent2Agent (A2A) protocol enables AI agents to:
- Discover each other's capabilities through Agent Cards
- Communicate securely using standardized JSON-RPC messages
- Collaborate across different frameworks (CrewAI, LangGraph, ADK)
- Work across deployment platforms (Cloud Run, Agent Engine, GKE)

**Key Concept:** A2A focuses on agent-to-agent collaboration in natural modalities, complementing MCP (Model Context Protocol) which handles tool/data connections.

## Core Patterns

### 1. Exposing Agents via A2A (Server-Side)

**When to use:** Make your ADK agent available for other agents to consume

**Template:** `templates/a2a-server.py`

**Key Components:**
- `AgentCard` at `/.well-known/agent.json` - Advertises capabilities
- `AgentExecutor` - Handles incoming requests
- `DefaultRequestHandler` - Processes JSON-RPC messages
- `A2AStarletteApplication` - HTTP server implementation

**Script:** `scripts/expose-agent.sh`

### 2. Consuming External Agents (Client-Side)

**When to use:** Integrate remote A2A agents as sub-agents

**Template:** `templates/a2a-client.py`

**Key Components:**
- `A2ACardResolver` - Discovers remote agent capabilities
- `send_task` tool - Sends messages to remote agents
- Session tracking - Maintains context across interactions

**Script:** `scripts/consume-agent.sh`

### 3. Multi-Agent Communication

**When to use:** Orchestrate multiple specialized agents collaborating on complex tasks

**Template:** `templates/multi-agent-orchestration.py`

**Pattern:**
- Coordinator agent routes tasks
- Specialist agents handle specific domains
- Agent-to-agent messaging via A2A protocol
- Result aggregation and synthesis

**Example:** `examples/purchasing-concierge/`

### 4. Agent Card Configuration

**When to use:** Define agent capabilities for discovery

**Template:** `templates/agent-card.json`

**Contents:**
- Agent metadata (name, description, version)
- Capabilities and skills
- Supported modalities (text, audio, video)
- Endpoint URLs and protocol version
- Streaming support indicators

**Script:** `scripts/generate-agent-card.sh`

## Implementation Patterns

### Server-Side: Exposing an Agent

```python
# templates/a2a-server.py structure
from adk import Agent
from a2a import AgentExecutor, DefaultRequestHandler, AgentCard

class MyAgentExecutor(AgentExecutor):
    """Handle incoming A2A requests"""
    async def execute(self, request):
        # Process request using your agent
        result = await self.agent.run(request.message)
        return result

# Configure Agent Card
agent_card = AgentCard(
    name="my-agent",
    description="Agent description",
    capabilities=["skill1", "skill2"],
    endpoint="https://my-agent.example.com"
)

# Expose via HTTP
from a2a import A2AStarletteApplication

app = A2AStarletteApplication(
    executor=MyAgentExecutor(),
    card=agent_card
)
```

**Deployment:**
```bash
# Deploy to Cloud Run
bash scripts/expose-agent.sh --platform cloud-run

# Deploy to Agent Engine
bash scripts/expose-agent.sh --platform agent-engine

# Deploy to GKE
bash scripts/expose-agent.sh --platform gke
```

### Client-Side: Consuming an Agent

```python
# templates/a2a-client.py structure
from adk import Agent
from a2a import A2ACardResolver, send_task

# Discover remote agent
resolver = A2ACardResolver()
agent_card = await resolver.resolve("https://remote-agent.example.com")

# Create tool to communicate with remote agent
send_task_tool = send_task(
    agent_url=agent_card.endpoint,
    session_id="unique-session-id"
)

# Use in your agent
my_agent = Agent(
    tools=[send_task_tool],
    # ... other config
)

# Agent can now invoke remote agent
result = await my_agent.run("Ask the remote agent to do something")
```

### Multi-Agent Orchestration

```python
# templates/multi-agent-orchestration.py structure
from adk import Agent
from a2a import A2ACardResolver, send_task

# Discover specialist agents
resolver = A2ACardResolver()
research_agent = await resolver.resolve("https://research-agent.example.com")
analysis_agent = await resolver.resolve("https://analysis-agent.example.com")
writing_agent = await resolver.resolve("https://writing-agent.example.com")

# Coordinator agent
coordinator = Agent(
    name="coordinator",
    tools=[
        send_task(agent_url=research_agent.endpoint),
        send_task(agent_url=analysis_agent.endpoint),
        send_task(agent_url=writing_agent.endpoint)
    ],
    instructions="""
    You coordinate multiple specialist agents:
    1. Use research agent to gather information
    2. Use analysis agent to process findings
    3. Use writing agent to synthesize results
    """
)

# Execute multi-agent workflow
result = await coordinator.run("Research and write a report on AI agents")
```

## Agent Card Structure

```json
{
  "id": "my-agent",
  "name": "My Agent",
  "description": "Description of agent capabilities",
  "version": "1.0.0",
  "url": "https://my-agent.example.com",
  "capabilities": {
    "skills": [
      {
        "name": "skill1",
        "description": "First skill description"
      },
      {
        "name": "skill2",
        "description": "Second skill description"
      }
    ],
    "modalities": ["text", "image"],
    "streaming": true
  },
  "protocol": {
    "version": "0.3",
    "transport": "grpc"
  }
}
```

**Generation:**
```bash
bash scripts/generate-agent-card.sh \
  --name "my-agent" \
  --description "Agent description" \
  --skills "skill1,skill2" \
  --modalities "text,image" \
  --url "https://my-agent.example.com"
```

## Protocol Configuration

### gRPC Transport (A2A v0.3+)

```python
# templates/grpc-config.py
from a2a import A2AStarletteApplication, GrpcTransport

app = A2AStarletteApplication(
    executor=MyAgentExecutor(),
    transport=GrpcTransport(
        host="0.0.0.0",
        port=50051,
        secure=True,
        cert_file="/path/to/cert.pem",
        key_file="/path/to/key.pem"
    )
)
```

### Security Cards (A2A v0.3+)

```python
# templates/security-card.py
from a2a import SecurityCard, sign_card

# Create security card
security_card = SecurityCard(
    issuer="my-organization",
    audience=["trusted-agent-1", "trusted-agent-2"],
    permissions=["read", "write"]
)

# Sign the card
signed_card = sign_card(
    card=security_card,
    private_key="/path/to/private-key.pem"
)
```

### JSON-RPC Message Format

**Request:**
```json
{
  "id": "request-uuid",
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": "Task description",
    "session_id": "session-uuid",
    "context": {}
  }
}
```

**Response:**
```json
{
  "id": "request-uuid",
  "jsonrpc": "2.0",
  "result": {
    "message": "Agent response",
    "artifacts": [],
    "status": "completed"
  }
}
```

## Scripts

### 1. Expose Agent via A2A

```bash
bash scripts/expose-agent.sh --platform cloud-run --region us-central1
```

**What it does:**
- Generates Agent Card at `/.well-known/agent.json`
- Creates Dockerfile with A2A server
- Deploys to specified platform
- Configures networking and security
- Returns agent endpoint URL

### 2. Consume Remote Agent

```bash
bash scripts/consume-agent.sh --url https://remote-agent.example.com
```

**What it does:**
- Resolves Agent Card from remote URL
- Validates capabilities
- Generates client code
- Creates `send_task` tool wrapper
- Provides integration example

### 3. Generate Agent Card

```bash
bash scripts/generate-agent-card.sh \
  --name "my-agent" \
  --description "Agent description" \
  --skills "research,analysis,writing"
```

**What it does:**
- Creates JSON Agent Card
- Validates against A2A schema
- Generates `/.well-known/agent.json`
- Provides endpoint configuration

### 4. Validate A2A Configuration

```bash
bash scripts/validate-a2a.sh --config agent-card.json
```

**What it does:**
- Checks Agent Card schema
- Validates endpoint accessibility
- Tests JSON-RPC message format
- Verifies security configuration

## Templates

### Python Templates

- `templates/a2a-server.py` - Server-side agent implementation
- `templates/a2a-client.py` - Client-side agent consumption
- `templates/multi-agent-orchestration.py` - Multi-agent coordination
- `templates/grpc-config.py` - gRPC transport configuration
- `templates/security-card.py` - Security card implementation

### Go Templates

- `templates/go/a2a-server.go` - Go server implementation
- `templates/go/a2a-client.go` - Go client implementation

### Configuration Templates

- `templates/agent-card.json` - Agent Card JSON structure
- `templates/deployment-config.yaml` - Cloud Run/GKE deployment
- `templates/security-policy.json` - Security configuration

## Examples

### Example 1: Research Agent Cluster

**Location:** `examples/research-cluster/`

**Architecture:**
- Coordinator agent orchestrates research workflow
- Search agent gathers information
- Analysis agent processes findings
- Writing agent synthesizes results

**Communication:** All agents communicate via A2A protocol

### Example 2: E-Commerce Assistant

**Location:** `examples/ecommerce-assistant/`

**Architecture:**
- Customer-facing agent handles inquiries
- Inventory agent checks product availability
- Pricing agent calculates costs
- Payment agent processes transactions

**Pattern:** Hierarchical agent structure with A2A messaging

### Example 3: Code Review System

**Location:** `examples/code-review/`

**Architecture:**
- Manager agent coordinates review process
- Style agent checks code formatting
- Security agent scans for vulnerabilities
- Performance agent analyzes efficiency

**Integration:** Each specialist agent is independent A2A service

### Example 4: Data Pipeline

**Location:** `examples/data-pipeline/`

**Architecture:**
- Ingestion agent collects data
- Transformation agent processes data
- Validation agent checks quality
- Storage agent persists results

**Workflow:** Sequential A2A agent execution

## Production Best Practices

### 1. Agent Discovery

- Host Agent Cards at `/.well-known/agent.json`
- Use semantic versioning for capabilities
- Document all available skills clearly
- Update cards when capabilities change

### 2. Error Handling

```python
try:
    result = await send_task(remote_agent, task)
except A2AConnectionError:
    # Handle network failures
    result = fallback_handler(task)
except A2AAuthenticationError:
    # Handle auth failures
    log_security_event()
except A2ATimeoutError:
    # Handle timeouts
    retry_with_backoff()
```

### 3. Security

- Sign Agent Cards with private keys
- Validate incoming requests
- Use HTTPS/gRPC with TLS
- Implement authentication and authorization
- Rate limit agent-to-agent calls

### 4. Monitoring

```python
# Track A2A metrics
metrics.record('a2a.request.count', {'agent': 'remote-agent'})
metrics.record('a2a.latency', latency_ms)
metrics.record('a2a.error.rate', error_count)
```

### 5. Testing

```bash
# Test agent exposure
bash scripts/validate-a2a.sh --config agent-card.json

# Test agent consumption
bash scripts/test-a2a-client.sh --url https://remote-agent.example.com

# Test multi-agent workflow
bash scripts/test-orchestration.sh --config multi-agent-config.yaml
```

## Deployment Platforms

### Cloud Run

```bash
# Deploy A2A agent to Cloud Run
gcloud run deploy my-agent \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

**Agent Card URL:** `https://my-agent-[hash]-uc.a.run.app/.well-known/agent.json`

### Agent Engine

```bash
# Deploy to Agent Engine
adk deploy --platform agent-engine --agent my-agent
```

**Features:** Managed scaling, built-in monitoring, A2A native

### GKE (Google Kubernetes Engine)

```bash
# Deploy to GKE
kubectl apply -f templates/deployment-config.yaml
```

**Benefits:** Full control, custom scaling, multi-region

## Framework Interoperability

A2A agents can use different frameworks:

**ADK Agent:**
```python
from adk import Agent
agent = Agent(name="adk-agent", ...)
```

**CrewAI Agent:**
```python
from crewai import Agent
agent = Agent(role="crew-agent", ...)
```

**LangGraph Agent:**
```python
from langgraph import StateGraph
agent = StateGraph(...)
```

**All communicate via A2A protocol** - Framework is transparent to clients

## Integration with Gemini API

```python
# templates/gemini-integration.py
from adk import Agent
from vertexai.preview.generative_models import GenerativeModel

# ADK agent using Gemini
agent = Agent(
    name="gemini-agent",
    model=GenerativeModel("gemini-2.0-flash-exp"),
    tools=[send_task_tool]
)

# Expose via A2A
from a2a import A2AStarletteApplication
app = A2AStarletteApplication(
    executor=GeminiAgentExecutor(agent)
)
```

## A2A + MCP Integration

```python
# templates/a2a-mcp-integration.py
from adk import Agent
from a2a import send_task
from mcp import use_mcp_server

# Agent with both A2A (agents) and MCP (tools)
agent = Agent(
    name="hybrid-agent",
    # A2A: Communicate with other agents
    tools=[
        send_task(agent_url="https://research-agent.example.com")
    ],
    # MCP: Connect to data sources
    mcps=[
        use_mcp_server("filesystem"),
        use_mcp_server("database")
    ]
)
```

**Use Case:** Agent uses MCP for data access, A2A for agent collaboration

## Requirements

**Environment Variables:**
- `GOOGLE_CLOUD_PROJECT` - GCP project ID (for Gemini/Vertex AI)
- `GOOGLE_APPLICATION_CREDENTIALS` - Service account key path
- `A2A_AGENT_URL` - Your agent's public URL (for card generation)

**Dependencies:**
```bash
pip install google-adk[a2a]
pip install google-cloud-aiplatform
pip install grpcio  # For gRPC transport
```

**Infrastructure:**
- GCP project with Vertex AI API enabled
- Cloud Run or Agent Engine (for deployment)
- Domain with HTTPS (for production Agent Cards)

## Security: API Key Handling

**CRITICAL:** When generating any configuration files or code:

- NEVER hardcode actual API keys or secrets
- NEVER include real credentials in examples
- NEVER commit sensitive values to git

- ALWAYS use placeholders: `your_service_key_here`
- ALWAYS create `.env.example` with placeholders only
- ALWAYS add `.env*` to `.gitignore` (except `.env.example`)
- ALWAYS read from environment variables in code
- ALWAYS document where to obtain keys

**Placeholder format:** `{service}_{env}_your_key_here`

Example:
```bash
# .env.example (safe to commit)
GOOGLE_CLOUD_PROJECT=your_project_id_here
GOOGLE_APPLICATION_CREDENTIALS=/path/to/your_service_account_key.json
A2A_AGENT_URL=https://your_agent_url_here

# .env (NEVER commit)
GOOGLE_CLOUD_PROJECT=actual-project-id
GOOGLE_APPLICATION_CREDENTIALS=/actual/path/to/key.json
A2A_AGENT_URL=https://my-agent-xyz.run.app
```

## Troubleshooting

**Agent Card Not Found:**
- Verify `/.well-known/agent.json` is accessible
- Check CORS configuration
- Validate JSON schema

**Connection Refused:**
- Confirm agent is running
- Check firewall rules
- Verify endpoint URL

**Authentication Failed:**
- Validate security card signature
- Check permissions
- Review audience list

**Message Format Error:**
- Verify JSON-RPC 2.0 format
- Check message structure
- Validate parameter types

## Resources

**Official Documentation:**
- A2A Protocol: https://a2a-protocol.org/
- ADK A2A Guide: https://google.github.io/adk-docs/a2a/
- Google Cloud Blog: https://cloud.google.com/blog/products/ai-machine-learning/agent2agent-protocol-is-getting-an-upgrade

**Code Examples:**
- Python A2A Samples: https://github.com/a2aproject/a2a-samples/tree/main/samples/python/agents
- Purchasing Concierge Codelab: https://codelabs.developers.google.com/intro-a2a-purchasing-concierge

**Community:**
- A2A GitHub: https://github.com/a2aproject
- ADK Python: https://github.com/google/adk-python

---

**Plugin:** google-adk
**Version:** 1.0.0
**Protocol Version:** A2A v0.3+
**Language Support:** Python (stable), Go (stable)
**Deployment Platforms:** Cloud Run, Agent Engine, GKE
