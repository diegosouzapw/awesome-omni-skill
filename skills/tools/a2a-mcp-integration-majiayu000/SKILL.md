---
name: a2a-mcp-integration
description: Integration patterns for combining Agent-to-Agent (A2A) Protocol with Model Context Protocol (MCP) for hybrid agent communication. Use when building systems that need both agent-to-agent communication and agent-to-tool integration, implementing composite architectures, or when user mentions A2A+MCP integration, hybrid protocols, or multi-agent tool access.
allowed-tools: Read, Bash, Write, Edit, Grep, Glob
---

# A2A and MCP Integration Patterns

**Purpose:** Provide integration patterns, configuration examples, and best practices for combining Agent-to-Agent Protocol with Model Context Protocol in multi-agent systems.

**Activation Triggers:**
- A2A + MCP integration setup
- Hybrid protocol architecture
- Agent-to-agent + agent-to-tool communication
- Composite multi-agent systems
- Protocol compatibility questions
- Cross-protocol authentication
- Combined SDK usage

**Protocol Roles:**
- **A2A Protocol:** Agent-to-agent communication and task delegation
- **MCP:** Agent-to-tool communication and resource access
- **Combined:** Agents communicate via A2A while accessing tools via MCP

## Architecture Overview

### Why Combine A2A and MCP?

A2A and MCP are complementary protocols:

- **MCP** standardizes how agents connect to tools, APIs, and data sources
- **A2A** standardizes how agents communicate and coordinate with each other
- **Together** they enable agents to collaborate while accessing shared resources

### Integration Patterns

1. **Hierarchical Agent Systems**
   - Coordinator agent uses A2A to delegate tasks
   - Worker agents use MCP to access tools
   - Results flow back through A2A

2. **Federated Tool Access**
   - Multiple agents communicate via A2A
   - Each agent has MCP tool access
   - Agents share tool results through A2A messages

3. **Resource-Sharing Networks**
   - Agents discover each other via A2A
   - Agents expose MCP servers to each other
   - Dynamic tool delegation through agent network

## Quick Start

### Python Integration

```bash
# Install both SDKs
./scripts/install-python-integration.sh

# Validate installation
./scripts/validate-python-integration.sh

# Run example
python examples/python-hybrid-agent.py
```

### TypeScript Integration

```bash
# Install both SDKs
./scripts/install-typescript-integration.sh

# Validate installation
./scripts/validate-typescript-integration.sh

# Run example
ts-node examples/typescript-hybrid-agent.ts
```

## Configuration

### Environment Setup

Both protocols require environment configuration:

```bash
# A2A Configuration
A2A_API_KEY=your_a2a_key_here
A2A_BASE_URL=https://a2a.example.com

# MCP Configuration
MCP_SERVER_URL=http://localhost:3000
MCP_TRANSPORT=stdio

# Integration Settings
HYBRID_AGENT_ID=agent-001
HYBRID_AGENT_NAME=hybrid-coordinator
ENABLE_A2A=true
ENABLE_MCP=true
```

See `templates/env-integration-template.txt` for complete configuration.

### Authentication

Handle authentication for both protocols:

1. **Separate Credentials:** Each protocol uses its own auth
2. **Shared Identity:** Agent identity spans both protocols
3. **Token Forwarding:** Pass credentials through A2A messages (when appropriate)

See `templates/auth-hybrid-template.txt` for patterns.

## Integration Patterns

### Pattern 1: Coordinator-Worker with Shared Tools

**Architecture:**
- Coordinator agent receives tasks
- Delegates via A2A to worker agents
- Workers use MCP tools to complete tasks
- Results return via A2A

**Template:** `templates/coordinator-worker-pattern.py`

**Use Case:** Complex workflows requiring specialized agents with tool access

### Pattern 2: Peer-to-Peer with Tool Sharing

**Architecture:**
- Agents communicate as peers via A2A
- Each agent exposes MCP server
- Agents can request tools from each other
- Distributed tool access

**Template:** `templates/peer-tool-sharing-pattern.ts`

**Use Case:** Decentralized systems where agents have different capabilities

### Pattern 3: Agent Mesh with Centralized Tools

**Architecture:**
- Agents form A2A communication mesh
- Centralized MCP server provides tools
- All agents access same tool set
- Coordination via A2A, execution via MCP

**Template:** `templates/mesh-centralized-tools-pattern.py`

**Use Case:** Teams of agents working with shared infrastructure

### Pattern 4: Layered Protocol Stack

**Architecture:**
- MCP at base layer for tool access
- A2A at orchestration layer for coordination
- Application logic at top layer
- Clean separation of concerns

**Template:** `templates/layered-stack-pattern.ts`

**Use Case:** Enterprise systems requiring protocol isolation

## Scripts

### Installation Scripts

- `install-python-integration.sh` - Install Python A2A + MCP SDKs
- `install-typescript-integration.sh` - Install TypeScript A2A + MCP SDKs
- `install-java-integration.sh` - Install Java A2A + MCP SDKs

### Validation Scripts

- `validate-python-integration.sh` - Verify Python integration setup
- `validate-typescript-integration.sh` - Verify TypeScript integration setup
- `validate-protocol-compatibility.sh` - Check protocol version compatibility

### Setup Scripts

- `setup-hybrid-agent.sh` - Initialize hybrid agent environment
- `setup-mcp-server.sh` - Configure MCP server for A2A agents
- `setup-agent-discovery.sh` - Configure A2A agent discovery with MCP tools

## Templates

### Configuration Templates

- `env-integration-template.txt` - Environment variables for both protocols
- `auth-hybrid-template.txt` - Authentication configuration
- `agent-config-hybrid.json` - Agent configuration with A2A+MCP

### Code Templates

**Python:**
- `coordinator-worker-pattern.py` - Coordinator-worker implementation
- `mesh-centralized-tools-pattern.py` - Agent mesh with central MCP
- `python-hybrid-agent.py` - Basic hybrid agent

**TypeScript:**
- `peer-tool-sharing-pattern.ts` - Peer-to-peer tool sharing
- `layered-stack-pattern.ts` - Layered protocol architecture
- `typescript-hybrid-agent.ts` - Basic hybrid agent

**Java:**
- `java-hybrid-agent.java` - Basic Java integration
- `java-coordinator-pattern.java` - Coordinator pattern in Java

**Configuration:**
- `mcp-server-config.json` - MCP server configuration for A2A agents
- `a2a-agent-card.json` - Agent card with MCP tool references

## Common Integration Scenarios

### Scenario 1: Multi-Agent Data Pipeline

**Problem:** Multiple agents process data through different tools

**Solution:**
1. Coordinator receives request via A2A
2. Delegates to specialized agents (data-fetcher, data-processor, data-storage)
3. Each agent uses MCP tools for its domain
4. Results aggregate via A2A back to coordinator

**Example:** `examples/data-pipeline-integration.py`

### Scenario 2: Distributed Research Assistant

**Problem:** Research task requires web search, document analysis, and synthesis

**Solution:**
1. Agents communicate via A2A to coordinate
2. Search agent uses MCP web search tools
3. Analysis agent uses MCP document processing tools
4. Synthesis agent combines results using MCP output tools

**Example:** `examples/research-assistant-integration.ts`

### Scenario 3: Microservice-Style Agent Architecture

**Problem:** Need modular, scalable agent system

**Solution:**
1. Each agent is a microservice with A2A interface
2. Agents use MCP to access shared databases, APIs
3. Service discovery via A2A agent cards
4. Load balancing across agent instances

**Example:** `examples/microservice-agents.py`

## Error Handling

### Protocol-Specific Errors

Handle errors from both protocols:

```python
from a2a import A2AError
from mcp import MCPError

try:
    # A2A communication
    response = await a2a_client.send_task(task)

    # MCP tool execution
    result = await mcp_client.call_tool("search", params)

except A2AError as e:
    # Handle A2A communication errors
    logger.error(f"A2A error: {e}")

except MCPError as e:
    # Handle MCP tool errors
    logger.error(f"MCP error: {e}")
```

See `examples/error-handling-integration.py` for complete patterns.

### Connection Failures

Both protocols may fail independently:

1. **A2A failure, MCP working:** Agent can execute local tools
2. **MCP failure, A2A working:** Agent can delegate to others
3. **Both failing:** Implement fallback logic

See `templates/failover-pattern.py` for resilience patterns.

## Security Considerations

### Authentication Boundaries

**Separate Auth Per Protocol:**
- A2A credentials for agent communication
- MCP credentials for tool access
- Never share credentials across protocols

### Message Security

**A2A Messages:**
- End-to-end encryption between agents
- Signature verification for agent identity
- Do not include MCP credentials in A2A messages

**MCP Communication:**
- Secure tool access with proper authentication
- Validate tool responses before sharing via A2A
- Sandbox tool execution

### Network Security

**Hybrid Deployment:**
- A2A may be internet-facing for agent discovery
- MCP should be internal for tool security
- Use VPN/private networks for MCP traffic
- Implement network segmentation

See `examples/security-best-practices.md` for detailed guidance.

## Performance Optimization

### Protocol Selection

Choose the right protocol for each interaction:

**Use A2A when:**
- Delegating tasks to other agents
- Coordinating multi-agent workflows
- Sharing results between agents

**Use MCP when:**
- Accessing tools and APIs
- Reading/writing data sources
- Executing specialized functions

### Connection Pooling

Both protocols benefit from connection pooling:

```python
# A2A connection pool
a2a_pool = A2AConnectionPool(
    max_connections=10,
    timeout=30
)

# MCP connection pool
mcp_pool = MCPConnectionPool(
    max_connections=5,
    timeout=15
)
```

See `templates/connection-pooling.py` for implementation.

### Caching Strategies

**Agent Discovery Caching:**
- Cache A2A agent cards (refresh periodically)
- Cache MCP tool schemas
- Invalidate on protocol updates

**Result Caching:**
- Cache expensive MCP tool results
- Share cache across A2A agent network
- Implement cache coherence protocol

## Testing

### Integration Tests

Test both protocols together:

```bash
# Run integration test suite
./scripts/test-integration.sh

# Test specific pattern
./scripts/test-pattern.sh coordinator-worker

# Test protocol compatibility
./scripts/test-protocol-versions.sh
```

### Mock Servers

Use mock servers for development:

```bash
# Start mock A2A server
./scripts/start-mock-a2a.sh

# Start mock MCP server
./scripts/start-mock-mcp.sh

# Run tests against mocks
./scripts/test-with-mocks.sh
```

## Examples

Complete working examples:

**Python:**
- `python-hybrid-agent.py` - Basic hybrid agent
- `data-pipeline-integration.py` - Multi-agent data pipeline
- `microservice-agents.py` - Microservice architecture
- `error-handling-integration.py` - Error handling patterns

**TypeScript:**
- `typescript-hybrid-agent.ts` - Basic hybrid agent
- `research-assistant-integration.ts` - Distributed research
- `peer-coordination.ts` - Peer-to-peer coordination

**Configuration:**
- `docker-compose-integration.yml` - Docker setup for hybrid system
- `kubernetes-hybrid-agents.yaml` - Kubernetes deployment

**Documentation:**
- `security-best-practices.md` - Security guidelines
- `troubleshooting-integration.md` - Common issues and solutions
- `architecture-patterns.md` - Detailed architecture patterns

## Troubleshooting

### Common Issues

**Protocol Version Mismatch:**
```bash
# Check versions
./scripts/validate-protocol-compatibility.sh

# Upgrade if needed
pip install --upgrade a2a-protocol mcp-sdk
```

**Authentication Errors:**
```bash
# Verify both protocol credentials
echo $A2A_API_KEY
echo $MCP_SERVER_URL

# Test separately
python -c "from a2a import Client; Client().ping()"
python -c "from mcp import Client; Client().ping()"
```

**Connection Issues:**
- Check A2A agent is reachable
- Verify MCP server is running
- Test network connectivity separately
- Review firewall rules

See `examples/troubleshooting-integration.md` for detailed solutions.

## Resources

**Official Documentation:**
- A2A Protocol: https://a2a-protocol.org
- MCP: https://modelcontextprotocol.io
- Integration Guide: https://docs.a2a-protocol.org/mcp-integration

**GitHub Repositories:**
- A2A+MCP Examples: https://github.com/a2a/mcp-integration-examples
- Python Integration: https://github.com/a2a/python-mcp-integration
- TypeScript Integration: https://github.com/a2a/typescript-mcp-integration

**Community:**
- A2A Discord: https://discord.gg/a2a-protocol
- MCP Discussion: https://github.com/modelcontextprotocol/specification/discussions
- Integration Patterns: https://community.a2a-protocol.org/integrations

---

**Version:** 1.0.0
**A2A Protocol Compatibility:** 1.0+
**MCP Compatibility:** 1.0+
**Last Updated:** 2025-12-20
