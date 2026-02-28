---
name: agent-card-templates
description: A2A agent card JSON templates with schema validation and examples for different agent types. Use when creating agent cards, implementing A2A protocol discovery, setting up agent metadata, configuring authentication schemes, defining agent capabilities, or when user mentions agent card, agent discovery, A2A metadata, service endpoint configuration, or agent authentication setup.
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Agent Card Templates

**Purpose:** Provide reusable JSON templates for creating A2A (Agent-to-Agent) protocol agent cards following the official specification.

**Activation Triggers:**
- Creating new A2A agent cards
- Implementing agent discovery endpoints
- Configuring authentication schemes
- Defining agent capabilities and skills
- Setting up service endpoints
- Validating agent card JSON structure

**Key Resources:**
- `templates/schema.json` - Complete JSON schema for validation
- `templates/basic-agent-card.json` - Simple agent card template
- `templates/multi-capability-agent-card.json` - Agent with multiple skills
- `templates/authenticated-agent-card.json` - Agent with auth requirements
- `templates/streaming-agent-card.json` - Agent with streaming support
- `examples/` - Real-world agent card examples
- `scripts/validate-agent-card.sh` - Schema validation script
- `scripts/generate-agent-card.sh` - Interactive card generator
- `scripts/test-agent-card.sh` - Format and structure testing

## Security: API Key Handling

**CRITICAL:** When generating any configuration files or code:

❌ NEVER hardcode actual API keys or secrets
❌ NEVER include real credentials in examples
❌ NEVER commit sensitive values to git

✅ ALWAYS use placeholders: `your_service_key_here`
✅ ALWAYS create `.env.example` with placeholders only
✅ ALWAYS add `.env*` to `.gitignore` (except `.env.example`)
✅ ALWAYS read from environment variables in code
✅ ALWAYS document where to obtain keys

**Placeholder format:** `{service}_{env}_your_key_here`

## Agent Card Structure

### Required Fields

**Basic Identity:**
- `id` - Unique identifier (URI or UUID)
- `name` - Human-readable display name
- `protocolVersion` - A2A protocol version (e.g., "0.3")
- `serviceEndpoint` - Base URL for agent's A2A service
- `provider` - Organization information (name, contactEmail, url)

**Security:**
- `securitySchemes` - Authentication scheme definitions
- `security` - Required auth combinations

**Capabilities:**
- `capabilities` - Feature flags (streaming, pushNotifications)

### Optional Fields

- `description` - Detailed agent purpose explanation
- `logo` - URL to logo image
- `version` - Agent implementation version
- `skills` - Array of agent capabilities with schemas
- `extensions` - Extension support declarations
- `metadata` - Custom key-value pairs

## Template Categories

### 1. Basic Agent Card
**Use for:** Simple agents with minimal configuration
**Template:** `templates/basic-agent-card.json`
**Features:**
- Required fields only
- Simple API key authentication
- No streaming or advanced capabilities
- Single skill definition

### 2. Multi-Capability Agent Card
**Use for:** Agents with multiple skills and capabilities
**Template:** `templates/multi-capability-agent-card.json`
**Features:**
- Multiple skill definitions
- Input/output schemas for each skill
- Capability flags enabled
- Comprehensive metadata

### 3. Authenticated Agent Card
**Use for:** Agents with complex authentication requirements
**Template:** `templates/authenticated-agent-card.json`
**Features:**
- Multiple authentication schemes (API Key, Bearer, OAuth2)
- Alternative auth combinations
- Security scopes and permissions
- OpenID Connect support

### 4. Streaming Agent Card
**Use for:** Agents supporting real-time updates
**Template:** `templates/streaming-agent-card.json`
**Features:**
- Streaming capability enabled
- Push notification support
- WebHook configuration
- SSE (Server-Sent Events) ready

## Authentication Schemes

### Supported Types

**API Key:**
```json
{
  "type": "apiKey",
  "name": "X-API-Key",
  "in": "header"
}
```

**Bearer Token:**
```json
{
  "type": "http",
  "scheme": "bearer"
}
```

**OAuth 2.0:**
```json
{
  "type": "oauth2",
  "flows": {
    "authorizationCode": {
      "authorizationUrl": "https://provider.example/oauth/authorize",
      "tokenUrl": "https://provider.example/oauth/token",
      "scopes": {
        "read": "Read access",
        "write": "Write access"
      }
    }
  }
}
```

**Basic Auth:**
```json
{
  "type": "http",
  "scheme": "basic"
}
```

## Usage Workflow

### 1. Select Template

Choose template based on agent requirements:
```bash
# List available templates
ls templates/

# View template content
cat templates/basic-agent-card.json
```

### 2. Generate Agent Card

Use interactive generator:
```bash
./scripts/generate-agent-card.sh

# Or specify template directly
./scripts/generate-agent-card.sh --template basic
```

**Generator prompts for:**
- Agent name and description
- Service endpoint URL
- Provider information
- Authentication scheme
- Capabilities and skills

### 3. Validate Agent Card

Validate against schema:
```bash
# Validate structure and required fields
./scripts/validate-agent-card.sh agent-card.json

# Test format and completeness
./scripts/test-agent-card.sh agent-card.json
```

**Validation checks:**
- Required fields present
- Valid JSON syntax
- Schema compliance
- URL format validation
- Authentication scheme correctness

### 4. Deploy Agent Card

Host at standard location:
```
https://<base_url>/.well-known/agent.json
```

Or alternative:
```
https://<base_url>/.well-known/agent-card.json
```

## Skill Definition

Each skill in the agent card includes:

```json
{
  "name": "skill-identifier",
  "description": "What the skill does",
  "inputSchema": {
    "type": "object",
    "properties": {
      "param1": {"type": "string"}
    },
    "required": ["param1"]
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "result": {"type": "string"}
    }
  },
  "inputModes": ["text/plain", "application/json"],
  "outputModes": ["application/json"],
  "tags": ["category", "feature"],
  "examples": [
    {
      "input": {"param1": "example"},
      "output": {"result": "example output"}
    }
  ]
}
```

## Scripts Reference

### validate-agent-card.sh
**Purpose:** Validate agent card against JSON schema
**Usage:**
```bash
./scripts/validate-agent-card.sh <agent-card.json>
```
**Checks:**
- JSON syntax validity
- Required fields presence
- Schema compliance
- URL format validation

### generate-agent-card.sh
**Purpose:** Interactive agent card generator
**Usage:**
```bash
./scripts/generate-agent-card.sh [--template TEMPLATE]
```
**Options:**
- `--template basic|multi|authenticated|streaming`
- `--output FILE` - Output file path
- `--interactive` - Prompt for all values

### test-agent-card.sh
**Purpose:** Test agent card format and structure
**Usage:**
```bash
./scripts/test-agent-card.sh <agent-card.json>
```
**Tests:**
- Well-formed JSON
- Required fields present
- Valid authentication schemes
- Capability flags consistency
- Service endpoint accessibility

## Examples Reference

**`examples/calculator-agent.md`** - Simple math calculation agent
**`examples/translation-agent.md`** - Multi-language translation service
**`examples/data-analysis-agent.md`** - Complex data processing agent

Each example includes:
- Complete agent card JSON
- Authentication configuration
- Skill definitions
- Usage scenarios
- Deployment instructions

## Best Practices

✓ **Use standard well-known URI** - `/.well-known/agent.json`
✓ **Include comprehensive descriptions** - Help with discovery
✓ **Define clear input/output schemas** - Enable validation
✓ **Specify authentication requirements** - Security first
✓ **Version your agent cards** - Track changes
✓ **Test card accessibility** - Ensure HTTP GET works
✓ **Document all skills** - Include examples
✓ **Use placeholder credentials** - Never hardcode secrets

## Common Issues

**Issue:** Agent card not discovered
**Fix:** Verify `/.well-known/agent.json` is accessible via HTTP GET

**Issue:** Authentication failures
**Fix:** Check `securitySchemes` and `security` match implementation

**Issue:** Schema validation errors
**Fix:** Run `validate-agent-card.sh` to identify missing/invalid fields

**Issue:** Skills not recognized
**Fix:** Ensure input/output schemas are valid JSON Schema format

## Resources

**A2A Protocol Specification:** https://a2a-protocol.org/latest/specification/
**Agent Card Concepts:** https://agent2agent.info/docs/concepts/agentcard/
**JSON Schema Reference:** https://json-schema.org/

---

**Supported A2A Protocol Version:** 0.3+
**Template Version:** 1.0.0
**Last Updated:** 2025-12-20
