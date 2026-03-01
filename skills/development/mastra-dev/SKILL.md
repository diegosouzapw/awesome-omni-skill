---
name: mastra-dev
description: "Ultimate Mastra Framework development toolkit for agent orchestration, workflow design, and MCP integration. Use when Codex should run the converted mastra-dev workflow. Inputs: command, options."
---

# Mastra Dev

Converted Claude skill workflow for Codex/OpenAI use.

## Source

Converted from `skills/mastra-dev/SKILL.md`.

## Bundled Resources

Supporting files copied from the Claude source:

- `references/README.md`
- `assets/examples`
- `scripts`
- `scripts/skill.sh`
- `assets/templates`

## Converted Instructions

The content below was adapted from the Claude source. Rewrite tool and runtime assumptions as needed when they refer to Claude-only features.

# Ultimate Mastra Development Toolkit

A comprehensive skill for developing, managing, and debugging Mastra Framework applications. This toolkit provides six major capability areas to accelerate Mastra development with automated scaffolding, server management, and expert guidance.

## Quick Start

```bash
# Check Mastra server status
$mastra-dev server status

# Analyze current Mastra setup
$mastra-dev analyze

# Create a new agent
$mastra-dev create-agent --name "contract-analyzer" --model "anthropic/claude-3-5-sonnet-20241022"

# Create a new workflow
$mastra-dev create-workflow --name "form-generation" --description "Auto-fill government forms"

# Start Mastra server
$mastra-dev server start

# Start Mastra Studio (observability UI)
$mastra-dev studio start
```

## What This Skill Does

This skill answers critical questions for Mastra developers:

### 1. How do I create a new Mastra agent?

```bash
$mastra-dev create-agent \
  --name "proposal-writer" \
  --model "openai/gpt-4-turbo" \
  --description "Expert technical writer for government proposals" \
  --instructions "You are an expert technical writer..." \
  --tools "sam-gov-lookup,company-data"
```

**What happens:**
- Generates `apps/mastra/src/agents/proposal-writer.ts` with proper structure
- Registers agent in `apps/mastra/src/config/mastra.config.ts`
- Follows Mastra best practices with TypeScript typing
- Includes Zod schemas for validation
- Auto-imports required dependencies

### 2. How do I build a workflow with multiple steps?

```bash
# Create workflow
$mastra-dev create-workflow \
  --name "contract-analysis" \
  --description "Analyze government contract for risks"

# Add steps sequentially
$mastra-dev add-step \
  --workflow "contract-analysis" \
  --step-name "extract-text" \
  --step-type "transform" \
  --input-schema '{"documentUrl": "string"}' \
  --output-schema '{"text": "string"}'

$mastra-dev add-step \
  --workflow "contract-analysis" \
  --step-name "identify-clauses" \
  --step-type "transform" \
  --input-schema '{"text": "string"}' \
  --output-schema '{"clauses": "array"}'

# Test the workflow
$mastra-dev test-workflow \
  --name "contract-analysis" \
  --input '{"documentUrl": "https://example.com/contract.pdf"}'
```

**What happens:**
- Creates workflow file at `apps/mastra/src/workflows/contract-analysis.ts`
- Generates steps with proper DAG composition (`.then()`, `.parallel()`, `.branch()`)
- Validates schema compatibility between steps
- Registers workflow in MCP server configuration
- Provides execution testing capability

### 3. How do I integrate external MCP servers?

```bash
# Add Wikipedia MCP server
$mastra-dev mcp add-client \
  --name "wikipedia" \
  --command "npx" \
  --args "-y,wikipedia-mcp"

# Add remote HTTP MCP server
$mastra-dev mcp add-client \
  --name "weather" \
  --url "https://server.smithery.ai/@smithery-ai/national-weather-service/mcp"

# Configure MCP server to expose workflows
$mastra-dev mcp configure-server \
  --agents "proposal-writer" \
  --workflows "contract-analysis"

# List all configured MCP servers
$mastra-dev mcp list-servers

# Test MCP connection
$mastra-dev mcp test --server "wikipedia"
```

**What happens:**
- Updates `apps/mastra/src/config/mcp.config.ts` with MCPClient configuration
- Supports both stdio (local) and HTTP (remote) transports
- Configures MCPServer to expose Mastra agents/workflows as tools
- Validates MCP connections and tool availability

### 4. How do I debug workflow execution?

```bash
# Debug specific workflow execution
$mastra-dev debug-workflow \
  --name "contract-analysis" \
  --execution-id "abc-123-def-456"

# Show workflow DAG visualization
$mastra-dev show-graph --workflow "contract-analysis"

# Validate all Mastra configurations
$mastra-dev validate

# View detailed logs
$mastra-dev server logs --tail 100
```

**What happens:**
- Queries `mastra.workflow_executions` table for execution state
- Retrieves step-by-step logs from `mastra.step_execution_logs`
- Identifies failed steps with error details
- Shows input/output data for each step
- Generates ASCII visualization of workflow DAG
- Validates schema compatibility and configuration correctness

### 5. What agents/workflows/tools are currently defined?

```bash
# List all agents
$mastra-dev list-agents

# Analyze specific agent
$mastra-dev analyze-agent --name "proposal-writer"

# List all workflows
$mastra-dev list-workflows

# List all tools
$mastra-dev list-tools

# Comprehensive analysis
$mastra-dev analyze
```

**What happens:**
- Scans `apps/mastra/src/agents/*.ts` for agent definitions
- Scans `apps/mastra/src/workflows/*.ts` for workflows
- Scans `apps/mastra/src/tools/*.ts` for tools
- Parses TypeScript files to extract configurations
- Shows registration status in `mastra.config.ts`
- Displays model providers, tool usage, and dependencies

## Features

### 1. Agent Management

Create, list, and analyze Mastra agents with intelligent scaffolding.

**Commands:**

```bash
# Create new agent
$mastra-dev create-agent \
  --name <agent-name> \
  --model <provider/model> \
  --description <description> \
  [--instructions <instructions-text>] \
  [--tools <comma-separated-tool-ids>]

# List all agents
$mastra-dev list-agents

# Analyze agent configuration
$mastra-dev analyze-agent --name <agent-name>
```

**Example - Create Contract Analysis Agent:**

```bash
$mastra-dev create-agent \
  --name "contract-analyzer" \
  --model "anthropic/claude-3-5-sonnet-20241022" \
  --description "Federal contract analysis expert" \
  --instructions "You are an expert in analyzing federal contracts for compliance, risks, and key terms. Focus on FAR/DFARS requirements." \
  --tools "document-parser,sam-gov-lookup,far-compliance"
```

**Generated File:** `apps/mastra/src/agents/contract-analyzer.ts`

```typescript
import { Agent } from '@mastra/core';

export const contractAnalyzerAgent = new Agent({
  id: 'contract-analyzer',
  name: 'Federal Contract Analysis Expert',
  description: 'Federal contract analysis expert',
  instructions: `You are an expert in analyzing federal contracts for compliance, risks, and key terms. Focus on FAR/DFARS requirements.`,
  model: {
    provider: 'anthropic',
    model: 'claude-3-5-sonnet-20241022'
  },
  tools: ['document-parser', 'sam-gov-lookup', 'far-compliance']
});
```

**What Gets Updated:**
- ✅ Agent file created at `apps/mastra/src/agents/contract-analyzer.ts`
- ✅ Registered in `apps/mastra/src/config/mastra.config.ts`
- ✅ Proper TypeScript imports and exports
- ✅ Zod validation ready
- ✅ Compatible with Mastra Studio

### 2. Workflow Management

Design, build, and test DAG-based workflows with step composition.

**Commands:**

```bash
# Create new workflow
$mastra-dev create-workflow \
  --name <workflow-name> \
  --description <description> \
  [--input-schema <json-schema>] \
  [--output-schema <json-schema>]

# Add step to workflow
$mastra-dev add-step \
  --workflow <workflow-name> \
  --step-name <step-name> \
  --step-type <transform|api-call|agent-call|parallel|branch> \
  [--input-schema <json-schema>] \
  [--output-schema <json-schema>]

# Test workflow execution
$mastra-dev test-workflow \
  --name <workflow-name> \
  --input <json-input>

# List all workflows
$mastra-dev list-workflows
```

**Example - Build Form Generation Workflow:**

```bash
# Step 1: Create workflow
$mastra-dev create-workflow \
  --name "form-generation" \
  --description "Auto-fill government forms from SAM.gov data" \
  --input-schema '{"opportunityId": "string"}' \
  --output-schema '{"formData": "object", "pdfUrl": "string"}'

# Step 2: Add fetch step
$mastra-dev add-step \
  --workflow "form-generation" \
  --step-name "fetch-opportunity" \
  --step-type "api-call" \
  --input-schema '{"opportunityId": "string"}' \
  --output-schema '{"opportunity": "object"}'

# Step 3: Add transform step
$mastra-dev add-step \
  --workflow "form-generation" \
  --step-name "extract-data" \
  --step-type "transform" \
  --input-schema '{"opportunity": "object"}' \
  --output-schema '{"formData": "object"}'

# Step 4: Add PDF generation step
$mastra-dev add-step \
  --workflow "form-generation" \
  --step-name "generate-pdf" \
  --step-type "transform" \
  --input-schema '{"formData": "object"}' \
  --output-schema '{"pdfUrl": "string"}'

# Test the workflow
$mastra-dev test-workflow \
  --name "form-generation" \
  --input '{"opportunityId": "abc-123"}'
```

**Generated File:** `apps/mastra/src/workflows/form-generation.ts`

```typescript
import { createWorkflow, createStep } from '@mastra/core/workflows';
import { z } from 'zod';

const fetchOpportunityStep = createStep({
  id: 'fetch-opportunity',
  inputSchema: z.object({ opportunityId: z.string() }),
  outputSchema: z.object({ opportunity: z.any() }),
  execute: async ({ inputData }) => {
    // API call implementation
    const response = await fetch(`https://api.sam.gov/opportunities/${inputData.opportunityId}`);
    return { opportunity: await response.json() };
  }
});

const extractDataStep = createStep({
  id: 'extract-data',
  inputSchema: z.object({ opportunity: z.any() }),
  outputSchema: z.object({ formData: z.any() }),
  execute: async ({ inputData }) => {
    // Transform logic
    return { formData: { /* extracted data */ } };
  }
});

const generatePdfStep = createStep({
  id: 'generate-pdf',
  inputSchema: z.object({ formData: z.any() }),
  outputSchema: z.object({ pdfUrl: z.string() }),
  execute: async ({ inputData }) => {
    // PDF generation
    return { pdfUrl: 'https://example.com/form.pdf' };
  }
});

export const formGenerationWorkflow = createWorkflow({
  id: 'form-generation',
  description: 'Auto-fill government forms from SAM.gov data',
  inputSchema: z.object({ opportunityId: z.string() }),
  outputSchema: z.object({ formData: z.any(), pdfUrl: z.string() })
})
  .then(fetchOpportunityStep)
  .then(extractDataStep)
  .then(generatePdfStep)
  .commit();
```

**Control Flow Patterns:**

- **Sequential:** `.then(stepA).then(stepB).then(stepC)`
- **Parallel:** `.parallel([stepA, stepB, stepC])`
- **Conditional:** `.branch({ when: (data) => data.score > 0.8, then: highPath, otherwise: lowPath })`

### 3. Tool Management

Create reusable tools with schema validation and type safety.

**Commands:**

```bash
# Create new tool
$mastra-dev create-tool \
  --name <tool-name> \
  --description <description> \
  [--input-schema <json-schema>] \
  [--output-schema <json-schema>]

# List all tools
$mastra-dev list-tools

# Test tool execution
$mastra-dev test-tool \
  --name <tool-name> \
  --input <json-input>
```

**Example - Create PDF Generator Tool:**

```bash
$mastra-dev create-tool \
  --name "pdf-generator" \
  --description "Generate PDF from template and data" \
  --input-schema '{"template": "string", "data": "object"}' \
  --output-schema '{"pdfUrl": "string", "size": "number"}'
```

**Generated File:** `apps/mastra/src/tools/pdf-generator.ts`

```typescript
import { createTool } from '@mastra/core/tools';
import { z } from 'zod';

export const pdfGeneratorTool = createTool({
  id: 'pdf-generator',
  description: 'Generate PDF from template and data',
  inputSchema: z.object({
    template: z.string(),
    data: z.record(z.any())
  }),
  outputSchema: z.object({
    pdfUrl: z.string(),
    size: z.number()
  }),
  execute: async ({ inputData }) => {
    // PDF generation logic
    const pdfUrl = await generatePdf(inputData.template, inputData.data);
    return {
      pdfUrl,
      size: 12345
    };
  }
});
```

### 4. Server Management

Control Mastra server lifecycle and monitor logs.

**Commands:**

```bash
# Start Mastra server
$mastra-dev server start

# Stop Mastra server
$mastra-dev server stop

# Check server status
$mastra-dev server status

# View server logs
$mastra-dev server logs [--tail <lines>]

# Start Mastra Studio (observability UI)
$mastra-dev studio start
```

**Example - Server Management Workflow:**

```bash
# Check if server is running
$mastra-dev server status
# Output: ❌ Mastra server is not running (port 6000)

# Start the server
$mastra-dev server start
# Output: ✅ Mastra server started on port 6000 (PID: 12345)

# Verify it's running
$mastra-dev server status
# Output: ✅ Mastra server is running on port 6000 (PID: 12345)
#         Uptime: 5 minutes
#         Memory: 245 MB

# View recent logs
$mastra-dev server logs --tail 50
# Output: [Shows last 50 lines from apps/mastra/logs/mastra.log]

# Start Mastra Studio for observability
$mastra-dev studio start
# Output: ✅ Mastra Studio started on http://localhost:4111
#         Connected to Mastra API at http://localhost:6000
```

**What Happens:**
- **server start:** Launches Mastra Express server via `npm run dev:mastra`
- **server stop:** Gracefully terminates process via SIGTERM
- **server status:** Checks port 6000, parses `ps aux | grep mastra`
- **server logs:** Tails from `apps/mastra/logs/mastra.log`
- **studio start:** Launches Mastra Studio CLI on port 4111

### 5. MCP Management

Configure Model Context Protocol integration for external tool access.

**Commands:**

```bash
# Add MCPClient for consuming external servers
$mastra-dev mcp add-client \
  --name <server-name> \
  --command <command> \
  [--args <comma-separated-args>] \
  [--url <http-url>]

# Configure MCPServer to expose Mastra tools
$mastra-dev mcp configure-server \
  [--agents <comma-separated-agent-ids>] \
  [--workflows <comma-separated-workflow-ids>] \
  [--tools <comma-separated-tool-ids>]

# List all MCP servers
$mastra-dev mcp list-servers

# Test MCP server connection
$mastra-dev mcp test --server <server-name>
```

**Example - Add External MCP Servers:**

```bash
# Add local stdio MCP server (Wikipedia)
$mastra-dev mcp add-client \
  --name "wikipedia" \
  --command "npx" \
  --args "-y,wikipedia-mcp"

# Add remote HTTP MCP server (Weather)
$mastra-dev mcp add-client \
  --name "weather" \
  --url "https://server.smithery.ai/@smithery-ai/national-weather-service/mcp"

# Add Puppeteer for browser automation
$mastra-dev mcp add-client \
  --name "puppeteer" \
  --command "npx" \
  --args "-y,@modelcontextprotocol/server-puppeteer"
```

**Updated Configuration:** `apps/mastra/src/config/mcp.config.ts`

```typescript
import { MCPClient } from '@mastra/mcp';

export const mcpClient = new MCPClient({
  id: 'mastra-mcp-client',
  servers: {
    wikipedia: {
      command: 'npx',
      args: ['-y', 'wikipedia-mcp']
    },
    weather: {
      url: new URL('https://server.smithery.ai/@smithery-ai/national-weather-service/mcp')
    },
    puppeteer: {
      command: 'npx',
      args: ['-y', '@modelcontextprotocol/server-puppeteer']
    }
  }
});
```

**Example - Expose Mastra Workflows via MCP:**

```bash
# Configure MCPServer to expose workflows
$mastra-dev mcp configure-server \
  --workflows "form-generation,contract-analysis"
```

**Updated Configuration:**

```typescript
import { MCPServer } from '@mastra/mcp';
import { formGenerationWorkflow } from '../workflows/form-generation.js';
import { contractAnalysisWorkflow } from '../workflows/contract-analysis.js';

export const mastraMcpServer = new MCPServer({
  id: 'mastra-workflows',
  name: 'Mastra Workflow Engine',
  version: '1.0.0',
  workflows: {
    formGeneration: formGenerationWorkflow,
    contractAnalysis: contractAnalysisWorkflow
  }
});
```

**Testing MCP Connection:**

```bash
$mastra-dev mcp test --server "wikipedia"
# Output: ✅ Connected to wikipedia MCP server
#         Available tools: search_wikipedia, get_article, get_summary
#         Transport: stdio
#         Status: healthy
```

### 6. Analysis & Debugging

Analyze Mastra setup, debug workflow execution, and validate configurations.

**Commands:**

```bash
# Analyze entire Mastra setup
$mastra-dev analyze

# Debug workflow execution
$mastra-dev debug-workflow \
  --name <workflow-name> \
  --execution-id <execution-id>

# Show workflow DAG visualization
$mastra-dev show-graph --workflow <workflow-name>

# Validate all configurations
$mastra-dev validate
```

**Example - Comprehensive Analysis:**

```bash
$mastra-dev analyze
```

**Output:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Mastra Setup Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 AGENTS (3 found)
  ✅ contract-analyzer
     Model: anthropic/claude-3-5-sonnet-20241022
     Tools: document-parser, sam-gov-lookup, far-compliance
     Status: Registered in mastra.config.ts

  ✅ proposal-writer
     Model: openai/gpt-4-turbo
     Tools: sam-gov-lookup, company-data
     Status: Registered in mastra.config.ts

  ✅ data-analyst
     Model: anthropic/claude-3-5-sonnet-20241022
     Tools: database-query, chart-generator
     Status: Registered in mastra.config.ts

📊 WORKFLOWS (2 found)
  ✅ form-generation
     Steps: 3 (fetch-opportunity, extract-data, generate-pdf)
     Type: Sequential DAG
     Status: Registered in MCP Server

  ✅ contract-analysis
     Steps: 4 (extract-text, identify-clauses, check-compliance, assess-risks)
     Type: Sequential DAG
     Status: Registered in MCP Server

📊 TOOLS (5 found)
  ✅ pdf-generator
  ✅ document-parser
  ✅ sam-gov-lookup
  ✅ far-compliance
  ✅ database-query

📊 MCP SERVERS (3 configured)
  ✅ wikipedia (stdio)
  ✅ weather (HTTP)
  ✅ puppeteer (stdio)

📊 DATABASE
  ✅ PostgreSQL connected
  ✅ Schema: mastra
  ✅ Tables: tenants, workflows, workflow_executions, step_execution_logs

📊 SERVER STATUS
  ✅ Running on port 6000
  ✅ PID: 12345
  ✅ Uptime: 2 hours 15 minutes
  ✅ Memory: 245 MB

✅ All configurations valid
```

**Example - Debug Failed Workflow:**

```bash
$mastra-dev debug-workflow \
  --name "contract-analysis" \
  --execution-id "550e8400-e29b-41d4-a716-446655440000"
```

**Output:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🐛 Workflow Execution Debug
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Workflow: contract-analysis
Execution ID: 550e8400-e29b-41d4-a716-446655440000
Status: ❌ FAILED
Started: 2026-02-11 14:23:15
Completed: 2026-02-11 14:23:47
Duration: 32 seconds

📊 STEP EXECUTION BREAKDOWN:

  ✅ Step 1: extract-text
     Status: COMPLETED
     Duration: 8s
     Input: {"documentUrl": "https://example.com/contract.pdf"}
     Output: {"text": "CONTRACT AGREEMENT..."}

  ✅ Step 2: identify-clauses
     Status: COMPLETED
     Duration: 12s
     Input: {"text": "CONTRACT AGREEMENT..."}
     Output: {"clauses": ["FAR 52.212-4", "FAR 52.212-5"]}

  ❌ Step 3: check-compliance
     Status: FAILED
     Duration: 5s
     Input: {"clauses": ["FAR 52.212-4", "FAR 52.212-5"]}
     Error: TypeError: Cannot read property 'farClause' of undefined
     Stack Trace:
       at checkComplianceStep.execute (workflows/contract-analysis.ts:45:23)

  ⏸️ Step 4: assess-risks
     Status: SKIPPED (previous step failed)

🔍 ROOT CAUSE:
  Step 'check-compliance' failed due to undefined property access.

  Likely issue: Missing FAR compliance database connection or
  incorrect data structure returned from identify-clauses step.

💡 SUGGESTED FIX:
  1. Verify far-compliance tool is properly configured
  2. Check inputData schema in check-compliance step
  3. Add null checking: if (!inputData.clauses) throw new Error(...)
  4. Review step schema compatibility

📝 LOGS:
  [2026-02-11 14:23:40] ERROR: FAR compliance check failed
  [2026-02-11 14:23:40] ERROR: Database connection timeout
```

**Example - Show Workflow DAG:**

```bash
$mastra-dev show-graph --workflow "form-generation"
```

**Output (ASCII DAG):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Workflow DAG: form-generation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Input: { opportunityId: string }
  │
  ├─> [fetch-opportunity]
  │     Input: { opportunityId: string }
  │     Output: { opportunity: object }
  │
  ├─> [extract-data]
  │     Input: { opportunity: object }
  │     Output: { formData: object }
  │
  ├─> [generate-pdf]
  │     Input: { formData: object }
  │     Output: { pdfUrl: string }
  │
  └─> Output: { formData: object, pdfUrl: string }

Composition: Sequential (.then chain)
Total Steps: 3
Schema Validation: ✅ All schemas compatible
```

**Example - Validate All Configurations:**

```bash
$mastra-dev validate
```

**Output:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Configuration Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ mastra.config.ts
   - All agents registered correctly
   - All workflows registered correctly
   - Storage configuration valid (PostgresStore)
   - MCP servers configured properly

✅ mcp.config.ts
   - MCPClient configuration valid
   - 3 servers configured (wikipedia, weather, puppeteer)
   - MCPServer exports 2 workflows

✅ Agent Files
   - contract-analyzer.ts: Valid TypeScript, proper imports
   - proposal-writer.ts: Valid TypeScript, proper imports
   - data-analyst.ts: Valid TypeScript, proper imports

✅ Workflow Files
   - form-generation.ts: Valid DAG, schemas compatible
   - contract-analysis.ts: Valid DAG, schemas compatible

✅ Tool Files
   - pdf-generator.ts: Valid TypeScript, Zod schemas correct
   - document-parser.ts: Valid TypeScript, Zod schemas correct

✅ Database Schema
   - mastra schema exists
   - All required tables present
   - Migrations up to date

⚠️ WARNINGS (1):
  - Tool 'far-compliance' referenced in agent but not found in tools directory
    Recommendation: Create tools/far-compliance.ts or remove from agent

✅ Overall Status: VALID (1 warning)
```

## Configuration

The skill supports optional configuration via `.mastra-dev-config.json` in your project root:

```json
{
  "mastraPath": "apps/mastra",
  "defaultModel": "anthropic/claude-3-5-sonnet-20241022",
  "defaultProvider": "anthropic",
  "serverPort": 6000,
  "studioPort": 4111,
  "autoRegister": true,
  "templateDefaults": {
    "agent": {
      "instructions": "You are a helpful AI assistant.",
      "tools": []
    },
    "workflow": {
      "composition": "sequential"
    }
  },
  "logging": {
    "level": "info",
    "format": "pretty"
  }
}
```

**Configuration Options:**

- `mastraPath` - Relative path to Mastra app (default: "apps/mastra")
- `defaultModel` - Default LLM model for agents
- `defaultProvider` - Default LLM provider
- `serverPort` - Mastra server port (default: 6000)
- `studioPort` - Mastra Studio port (default: 4111)
- `autoRegister` - Automatically register agents/workflows in config files
- `templateDefaults` - Default values for generated files
- `logging` - Log level and format configuration

## Integration with Mastra App

This skill integrates seamlessly with the existing Mastra app at `/home/artsmc/applications/low-code/apps/mastra`:

### Directory Structure

```
apps/mastra/
├── src/
│   ├── agents/                  # ← Generated agent files
│   │   ├── contract-analyzer.ts
│   │   ├── proposal-writer.ts
│   │   └── data-analyst.ts
│   ├── workflows/               # ← Generated workflow files
│   │   ├── hello-world.ts       # Existing example
│   │   ├── form-generation.ts
│   │   └── contract-analysis.ts
│   ├── tools/                   # ← Generated tool files
│   │   ├── pdf-generator.ts
│   │   ├── document-parser.ts
│   │   └── sam-gov-lookup.ts
│   ├── config/
│   │   ├── mastra.config.ts     # ← Auto-updated registration
│   │   └── mcp.config.ts        # ← Auto-updated MCP config
│   └── app.ts                   # Express server (unchanged)
└── package.json
```

### Auto-Registration

When you create agents, workflows, or tools, the skill automatically:

1. **Generates TypeScript files** in the correct directories
2. **Updates `mastra.config.ts`** to register new constructs
3. **Updates `mcp.config.ts`** to expose workflows as MCP tools
4. **Validates TypeScript syntax** before writing
5. **Preserves existing code** (no destructive edits)

**Example Registration Update:**

Before:
```typescript
// apps/mastra/src/config/mastra.config.ts
export const mastra = new Mastra({
  storage,
  agents: {},
  workflows: {},
  tools: {}
});
```

After running `$mastra-dev create-agent --name "proposal-writer"`:
```typescript
import { proposalWriterAgent } from '../agents/proposal-writer.js';

export const mastra = new Mastra({
  storage,
  agents: {
    proposalWriter: proposalWriterAgent  // ← Auto-added
  },
  workflows: {},
  tools: {}
});
```

### Development Workflow

1. **Start Mastra server** (if not running):
   ```bash
   $mastra-dev server start
   ```

2. **Create constructs** as needed:
   ```bash
   $mastra-dev create-agent --name "my-agent"
   $mastra-dev create-workflow --name "my-workflow"
   $mastra-dev create-tool --name "my-tool"
   ```

3. **Edit generated files** to implement logic:
   - Agents: Add custom instructions and tool configurations
   - Workflows: Implement step `execute` functions
   - Tools: Add business logic to `execute` functions

4. **Test locally**:
   ```bash
   $mastra-dev test-workflow --name "my-workflow" --input '{...}'
   $mastra-dev test-tool --name "my-tool" --input '{...}'
   ```

5. **Use Mastra Studio** for visual debugging:
   ```bash
   $mastra-dev studio start
   # Open http://localhost:4111
   ```

6. **Validate before commit**:
   ```bash
   $mastra-dev validate
   ```

## Templates

The skill uses four TypeScript templates for code generation:

### 1. Agent Template

**File:** `templates/agent.template.ts`

**Variables:**
- `{{agentName}}` - Camel-cased agent name (e.g., "contractAnalyzer")
- `{{agentId}}` - Kebab-cased agent ID (e.g., "contract-analyzer")
- `{{agentDisplayName}}` - Human-readable name (e.g., "Contract Analyzer")
- `{{description}}` - Agent description
- `{{instructions}}` - Agent behavior instructions
- `{{provider}}` - LLM provider (e.g., "anthropic", "openai")
- `{{model}}` - Model name (e.g., "claude-3-5-sonnet-20241022")
- `{{tools}}` - Comma-separated tool IDs

**Example Usage:**
```bash
$mastra-dev create-agent \
  --name "contract-analyzer" \
  --model "anthropic/claude-3-5-sonnet-20241022" \
  --description "Federal contract analysis expert"
```

### 2. Workflow Template

**File:** `templates/workflow.template.ts`

**Variables:**
- `{{workflowName}}` - Camel-cased workflow name
- `{{workflowId}}` - Kebab-cased workflow ID
- `{{workflowDisplayName}}` - Human-readable name
- `{{description}}` - Workflow description
- `{{inputSchema}}` - Zod object schema for input
- `{{outputSchema}}` - Zod object schema for output
- `{{steps}}` - Step definitions (generated)
- `{{composition}}` - DAG composition (`.then()`, `.parallel()`, etc.)

**Example Usage:**
```bash
$mastra-dev create-workflow \
  --name "form-generation" \
  --description "Auto-fill government forms"
```

### 3. Tool Template

**File:** `templates/tool.template.ts`

**Variables:**
- `{{toolName}}` - Camel-cased tool name
- `{{toolId}}` - Kebab-cased tool ID
- `{{toolDisplayName}}` - Human-readable name
- `{{description}}` - Tool description
- `{{inputSchema}}` - Zod object schema for input
- `{{outputSchema}}` - Zod object schema for output
- `{{executeBody}}` - Function body implementation

**Example Usage:**
```bash
$mastra-dev create-tool \
  --name "pdf-generator" \
  --description "Generate PDF from template"
```

### 4. Step Template

**File:** `templates/step.template.ts`

**Variables:**
- `{{stepName}}` - Camel-cased step name
- `{{stepId}}` - Kebab-cased step ID
- `{{stepDisplayName}}` - Human-readable name
- `{{description}}` - Step description
- `{{inputSchema}}` - Zod object schema for input
- `{{outputSchema}}` - Zod object schema for output
- `{{executeBody}}` - Function body implementation

**Example Usage:**
```bash
$mastra-dev add-step \
  --workflow "my-workflow" \
  --step-name "fetch-data" \
  --step-type "api-call"
```

## Best Practices

### 1. Agent Development

✅ **DO:**
- Use clear, specific instructions
- Choose appropriate models for tasks (Opus for complex reasoning, Sonnet for speed)
- Register only necessary tools (avoid tool overload)
- Test agents with `agent.generate()` before production use
- Use memory systems for conversation context

❌ **DON'T:**
- Use generic instructions like "You are a helpful assistant"
- Register all available tools (increases latency and confusion)
- Skip testing agent behavior
- Hardcode API keys or secrets in agent definitions

### 2. Workflow Design

✅ **DO:**
- Define clear input/output schemas for every step
- Use `.parallel()` for independent operations
- Implement retry policies for unstable external APIs
- Add error handling with lifecycle callbacks
- Validate schema compatibility between steps
- Use `.commit()` to finalize workflow definitions

❌ **DON'T:**
- Create circular dependencies (DAG must be acyclic)
- Skip schema definitions (causes runtime errors)
- Use `.branch()` without proper conditionals
- Forget to call `.commit()` at the end
- Ignore step execution order requirements

### 3. Tool Development

✅ **DO:**
- Write descriptive tool descriptions for agents
- Use Zod for comprehensive input validation
- Handle edge cases and errors gracefully
- Return structured data matching output schema
- Test tools independently before agent integration

❌ **DON'T:**
- Use vague descriptions (agents won't know when to call)
- Skip input validation (leads to runtime errors)
- Return inconsistent output structures
- Assume external APIs are always available
- Expose sensitive operations without safeguards

### 4. MCP Integration

✅ **DO:**
- Use stdio transport for local MCP servers
- Use HTTP transport for remote MCP servers
- Test MCP connections with `$mastra-dev mcp test`
- Document which agents use which MCP tools
- Version MCP server configurations

❌ **DON'T:**
- Mix stdio and HTTP incorrectly
- Skip connection testing
- Hardcode URLs in MCP config
- Forget to restart server after MCP changes

### 5. Testing & Debugging

✅ **DO:**
- Test workflows with representative data
- Use Mastra Studio for visual debugging
- Check logs regularly during development
- Validate configurations before committing
- Debug failed executions with execution IDs

❌ **DON'T:**
- Skip testing with real data
- Ignore workflow execution failures
- Deploy without validating configurations
- Debug in production (use staging/dev environments)

## Troubleshooting

### Issue 1: "Mastra server is not running"

**Symptoms:**
```
❌ Mastra server is not running (port 6000)
```

**Solutions:**
```bash
# Start the server
$mastra-dev server start

# If that fails, check if port is in use
lsof -i :6000

# Kill existing process if needed
kill -9 <PID>

# Start again
$mastra-dev server start
```

### Issue 2: "Agent not found in mastra.config.ts"

**Symptoms:**
```
⚠️ Agent 'my-agent' exists but not registered in mastra.config.ts
```

**Solutions:**
```bash
# Re-run create-agent with --force flag (if implemented)
$mastra-dev create-agent --name "my-agent" --force

# Or manually add to mastra.config.ts:
# 1. Import: import { myAgent } from '../agents/my-agent.js';
# 2. Register: agents: { myAgent }
```

### Issue 3: "Workflow execution failed with schema error"

**Symptoms:**
```
❌ Step 'fetch-data' failed: Input validation error
Expected { userId: string }, got { user_id: string }
```

**Solutions:**
- Verify step schemas match data flow
- Check previous step's output schema
- Update input schema to match actual data structure
- Use schema transformation step if needed

```bash
# Debug the workflow
$mastra-dev debug-workflow --name "my-workflow" --execution-id "abc-123"

# Check DAG visualization
$mastra-dev show-graph --workflow "my-workflow"
```

### Issue 4: "MCP server connection timeout"

**Symptoms:**
```
❌ Failed to connect to MCP server 'wikipedia'
Connection timeout after 5000ms
```

**Solutions:**
```bash
# Test MCP server independently
npx -y wikipedia-mcp

# Check if command is correct
$mastra-dev mcp list-servers

# Remove and re-add server
$mastra-dev mcp remove-client --name "wikipedia"
$mastra-dev mcp add-client --name "wikipedia" --command "npx" --args "-y,wikipedia-mcp"

# Restart Mastra server
$mastra-dev server stop
$mastra-dev server start
```

### Issue 5: "Mastra Studio shows blank page"

**Symptoms:**
- Studio loads but shows empty interface
- "Cannot connect to server" error

**Solutions:**
```bash
# Ensure Mastra server is running first
$mastra-dev server status

# If not running, start it
$mastra-dev server start

# Then start Studio
$mastra-dev studio start

# Check CORS configuration in apps/mastra/src/app.ts
# Should include: http://localhost:4111 in allowed origins
```

### Issue 6: "TypeScript compilation errors after generation"

**Symptoms:**
```
Error: Cannot find module '@mastra/core'
```

**Solutions:**
```bash
# Install dependencies
cd apps/mastra
npm install

# Or from monorepo root
npm install

# Rebuild
npm run build:mastra
```

### Issue 7: "Database connection error"

**Symptoms:**
```
❌ PostgreSQL connection failed
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Solutions:**
```bash
# Check if PostgreSQL is running
pg_isready -h localhost -p 5432

# Start PostgreSQL (Docker)
cd apps/mastra
docker-compose up -d postgres

# Or use system PostgreSQL
sudo service postgresql start

# Verify DATABASE_URL environment variable
echo $DATABASE_URL
```

### Issue 8: "Permission denied when running skill"

**Symptoms:**
```
bash: /home/artsmc/.claude/skills$mastra-dev/skill.sh: Permission denied
```

**Solutions:**
```bash
# Make skill executable
chmod +x /home/artsmc/.claude/skills$mastra-dev/skill.sh

# Or run with bash explicitly
bash /home/artsmc/.claude/skills$mastra-dev/skill.sh server status
```

### Issue 9: "Python not found"

**Symptoms:**
```
❌ Error: Python 3 is required
```

**Solutions:**
```bash
# Check Python installation
python3 --version

# Install Python 3 (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install python3

# Install Python 3 (macOS)
brew install python3
```

### Issue 10: "Workflow not exposed in MCP server"

**Symptoms:**
- Workflow exists but not available as MCP tool
- External MCP clients can't see workflow

**Solutions:**
```bash
# Configure workflow for MCP exposure
$mastra-dev mcp configure-server --workflows "my-workflow"

# Verify MCP configuration
$mastra-dev mcp list-servers

# Restart Mastra server to apply changes
$mastra-dev server stop
$mastra-dev server start

# Test MCP connection
$mastra-dev mcp test --server "mastra-workflows"
```

## Use Cases

### Use Case 1: Building a Government Contract Analysis System

**Scenario:** Create an AI system that analyzes federal contracts for compliance and risk.

```bash
# 1. Create contract analysis agent
$mastra-dev create-agent \
  --name "contract-analyzer" \
  --model "anthropic/claude-3-5-sonnet-20241022" \
  --description "Expert in federal contract analysis" \
  --instructions "Analyze contracts for FAR/DFARS compliance, identify risks, and extract key terms"

# 2. Create tools for contract processing
$mastra-dev create-tool \
  --name "pdf-parser" \
  --description "Extract text from PDF contracts"

$mastra-dev create-tool \
  --name "far-lookup" \
  --description "Look up FAR/DFARS clauses"

$mastra-dev create-tool \
  --name "risk-calculator" \
  --description "Calculate contract risk score"

# 3. Create contract analysis workflow
$mastra-dev create-workflow \
  --name "contract-analysis" \
  --description "End-to-end contract analysis pipeline"

# 4. Add workflow steps
$mastra-dev add-step --workflow "contract-analysis" --step-name "extract-text"
$mastra-dev add-step --workflow "contract-analysis" --step-name "identify-clauses"
$mastra-dev add-step --workflow "contract-analysis" --step-name "check-compliance"
$mastra-dev add-step --workflow "contract-analysis" --step-name "assess-risks"
$mastra-dev add-step --workflow "contract-analysis" --step-name "generate-report"

# 5. Test the workflow
$mastra-dev test-workflow \
  --name "contract-analysis" \
  --input '{"contractUrl": "https://example.com/contract.pdf"}'

# 6. Start Mastra Studio to monitor execution
$mastra-dev studio start
```

### Use Case 2: Automating Government Form Filling

**Scenario:** Auto-fill complex government forms using SAM.gov API data.

```bash
# 1. Add SAM.gov MCP server for data access
$mastra-dev mcp add-client \
  --name "sam-gov" \
  --url "https://api.sam.gov/mcp"

# 2. Create form generation workflow
$mastra-dev create-workflow \
  --name "form-generation" \
  --description "Auto-fill government forms from opportunity data"

# 3. Add steps
$mastra-dev add-step --workflow "form-generation" --step-name "fetch-opportunity"
$mastra-dev add-step --workflow "form-generation" --step-name "extract-requirements"
$mastra-dev add-step --workflow "form-generation" --step-name "fill-form-fields"
$mastra-dev add-step --workflow "form-generation" --step-name "generate-pdf"

# 4. Create PDF generator tool
$mastra-dev create-tool \
  --name "pdf-generator" \
  --description "Generate PDF from form data"

# 5. Expose workflow via MCP for external access
$mastra-dev mcp configure-server --workflows "form-generation"

# 6. Test end-to-end
$mastra-dev test-workflow \
  --name "form-generation" \
  --input '{"opportunityId": "abc-123"}'
```

### Use Case 3: Multi-Agent Research System

**Scenario:** Build a research system with specialized agents working together.

```bash
# 1. Create specialized agents
$mastra-dev create-agent \
  --name "researcher" \
  --model "anthropic/claude-3-opus-20240229" \
  --description "Deep research expert"

$mastra-dev create-agent \
  --name "summarizer" \
  --model "anthropic/claude-3-5-sonnet-20241022" \
  --description "Expert summarizer"

$mastra-dev create-agent \
  --name "fact-checker" \
  --model "openai/gpt-4-turbo" \
  --description "Fact verification specialist"

# 2. Add external research tools via MCP
$mastra-dev mcp add-client --name "wikipedia" --command "npx" --args "-y,wikipedia-mcp"
$mastra-dev mcp add-client --name "arxiv" --command "npx" --args "-y,arxiv-mcp"

# 3. Create research workflow
$mastra-dev create-workflow \
  --name "research-pipeline" \
  --description "Multi-agent research system"

# 4. Add parallel research steps
$mastra-dev add-step --workflow "research-pipeline" --step-name "gather-sources"
$mastra-dev add-step --workflow "research-pipeline" --step-name "parallel-research" --step-type "parallel"
$mastra-dev add-step --workflow "research-pipeline" --step-name "synthesize-findings"
$mastra-dev add-step --workflow "research-pipeline" --step-name "fact-check"
$mastra-dev add-step --workflow "research-pipeline" --step-name "generate-report"

# 5. Validate setup
$mastra-dev validate

# 6. Start server and Studio
$mastra-dev server start
$mastra-dev studio start
```

## Metadata

- **Version:** 1.0.0
- **Status:** Active
- **Author:** AIForge Development Team
- **Last Updated:** 2026-02-11
- **Dependencies:** Python 3.x (stdlib only), Mastra Framework, TypeScript, Node.js 20+
- **Compatibility:** Mastra v1.3.0+, AIForge monorepo structure
- **License:** MIT

## Related Skills

- `$feature-new` - Complete feature development workflow
- `$security-quality-assess` - Security vulnerability scanning
- `$pm-db` - Project management database

## Support

For issues, questions, or contributions:
- **GitHub Issues:** https://github.com/your-org/aiforge/issues
- **Documentation:** /home/artsmc/applications/low-code/job-queue/product-forge/
- **Mastra Docs:** https://mastra.ai/docs
