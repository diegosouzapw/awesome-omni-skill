---
name: aws-ai-assistant
description: Building AI Solutions on AWS - Bedrock AgentCore agents, MCP server development, Knowledge Bases RAG, generative AI architecture, CDK infrastructure, and AWS AI service integration. Use when designing, building, or deploying any AI system on Amazon Web Services.
---

# Building AI Solutions on AWS

Comprehensive AWS AI skill covering the full lifecycle of AI solution development on Amazon Web Services — from architecture design through agent building, MCP server creation, knowledge base setup, and production deployment. Built from AWS Well-Architected best practices, official documentation, and real-world patterns.

## When to Use This Skill

- Designing AI architectures on AWS (chatbots, agents, copilots, content generators)
- Building AI agents with Amazon Bedrock AgentCore
- Creating and deploying Model Context Protocol (MCP) servers on AWS
- Setting up RAG systems with Amazon Bedrock Knowledge Bases
- Deploying AI infrastructure with CDK, CloudFormation, or Terraform
- Integrating AWS AI services (Bedrock, SageMaker, Comprehend, Rekognition, etc.)
- Reviewing AWS AI architectures for Well-Architected compliance
- Implementing responsible AI with Bedrock Guardrails
- Optimizing AI workload costs on AWS

## Amazon Bedrock AgentCore

### AgentCore Architecture Overview

Amazon Bedrock AgentCore is the agentic platform for building, deploying, and operating AI agents at scale. It provides nine integrated services:

| Service | Purpose |
|---------|---------|
| **Runtime** | Execute agents with 8-hour windows, session isolation, A2A protocol |
| **Gateway** | Transform APIs/Lambda into agent tools, connect MCP servers |
| **Identity** | OAuth authorization, identity-aware access, secure token vault |
| **Memory** | Episodic memory for agent learning across interactions |
| **Code Interpreter** | Secure, isolated code execution environments |
| **Browser** | Web interaction capabilities at scale |
| **Observability** | CloudWatch dashboards, OpenTelemetry, metrics tracking |
| **Policy** | Cedar-based deterministic access controls |
| **Evaluations** | 13 pre-built evaluators for quality monitoring |

### Building an Agent with AgentCore

```python
import boto3
import json

bedrock_agent = boto3.client('bedrock-agent')

# Create an agent
response = bedrock_agent.create_agent(
    agentName='customer-support-agent',
    description='AI agent for customer support with tool access',
    foundationModel='anthropic.claude-sonnet-4-5-20250929-v1:0',
    instruction="""You are a customer support agent. Help users with their inquiries
    by searching the knowledge base and taking actions through available tools.
    Always verify user identity before making account changes.
    Be concise, helpful, and professional.""",
    idleSessionTTLInSeconds=1800,
)

agent_id = response['agent']['agentId']

# Create an action group (tools)
bedrock_agent.create_agent_action_group(
    agentId=agent_id,
    agentVersion='DRAFT',
    actionGroupName='customer-tools',
    actionGroupExecutor={'lambda': 'arn:aws:lambda:us-east-1:123456789:function:customer-tools'},
    apiSchema={
        'payload': json.dumps({
            'openapi': '3.0.0',
            'paths': {
                '/lookup-order': {
                    'post': {
                        'summary': 'Look up customer order by order ID',
                        'operationId': 'lookupOrder',
                        'parameters': [
                            {'name': 'orderId', 'in': 'query', 'required': True, 'schema': {'type': 'string'}}
                        ]
                    }
                }
            }
        })
    }
)
```

### AgentCore Policy with Cedar

Define deterministic access controls using the Cedar policy language:

```cedar
// Allow the agent to read customer data
permit(
    principal == Agent::"customer-support-agent",
    action == Action::"invoke-tool",
    resource == Tool::"lookup-order"
) when {
    context.userAuthenticated == true
};

// Block the agent from deleting data
forbid(
    principal == Agent::"customer-support-agent",
    action == Action::"invoke-tool",
    resource == Tool::"delete-account"
);

// Allow refunds only up to $100
permit(
    principal == Agent::"customer-support-agent",
    action == Action::"invoke-tool",
    resource == Tool::"process-refund"
) when {
    context.refundAmount <= 100
};
```

### AgentCore Evaluations

Monitor agent quality with 13 pre-built evaluators:

```python
# Configure evaluations for an agent
evaluation_config = {
    'evaluators': [
        {'type': 'CORRECTNESS', 'weight': 0.3},
        {'type': 'HELPFULNESS', 'weight': 0.2},
        {'type': 'SAFETY', 'weight': 0.2},
        {'type': 'TOOL_SELECTION_ACCURACY', 'weight': 0.15},
        {'type': 'TOOL_PARAMETER_ACCURACY', 'weight': 0.1},
        {'type': 'GOAL_SUCCESS_RATE', 'weight': 0.05},
    ],
    'schedule': 'CONTINUOUS',
    'alertThreshold': 0.7,
}
```

## MCP Server Development on AWS

### Official AWS MCP Servers

AWS provides official MCP servers via the `awslabs` organization:

| Server | Package | Purpose |
|--------|---------|---------|
| CDK | `awslabs.cdk-mcp-server` | CDK and CloudFormation assistance |
| Documentation | `awslabs.aws-documentation-mcp-server` | AWS documentation search |
| API | `awslabs.aws-api-mcp-server` | Execute AWS API calls via natural language |
| Knowledge | `awslabs.aws-knowledge-mcp-server` | AWS knowledge and regional availability |
| Pricing | `awslabs.aws-pricing-mcp-server` | AWS pricing information |
| Bedrock KB | `awslabs.bedrock-kb-retrieval-mcp-server` | Knowledge Base retrieval |
| Cost Analysis | `awslabs.cost-analysis-mcp-server` | Cost and usage analysis |
| Nova Canvas | `awslabs.nova-canvas-mcp-server` | Image generation with Nova |
| Lambda Tool | `awslabs.lambda-tool-mcp-server` | Expose Lambda as MCP tools |
| Terraform | `awslabs.terraform-mcp-server` | Terraform assistance |
| Core | `awslabs.core-mcp-server` | Core AWS MCP utilities |

### Installing AWS MCP Servers

```json
{
  "mcpServers": {
    "aws-cdk": {
      "command": "uvx",
      "args": ["awslabs.cdk-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    },
    "aws-docs": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    },
    "bedrock-kb": {
      "command": "uvx",
      "args": ["awslabs.bedrock-kb-retrieval-mcp-server@latest"],
      "env": {
        "KNOWLEDGE_BASE_ID": "your-kb-id",
        "AWS_REGION": "us-east-1",
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    }
  }
}
```

### Building a Custom MCP Server

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { BedrockRuntimeClient, InvokeModelCommand } from "@aws-sdk/client-bedrock-runtime";
import { S3Client, GetObjectCommand } from "@aws-sdk/client-s3";

const server = new McpServer({
  name: "my-aws-ai-server",
  version: "1.0.0",
});

const bedrockClient = new BedrockRuntimeClient({ region: "us-east-1" });
const s3Client = new S3Client({ region: "us-east-1" });

// Tool: Invoke a Bedrock model
server.tool(
  "invoke-model",
  "Invoke an Amazon Bedrock foundation model with a prompt",
  {
    prompt: { type: "string", description: "The prompt to send to the model" },
    modelId: { type: "string", description: "Bedrock model ID (e.g., anthropic.claude-sonnet-4-5-20250929-v1:0)" },
  },
  async ({ prompt, modelId }) => {
    const command = new InvokeModelCommand({
      modelId,
      contentType: "application/json",
      body: JSON.stringify({
        anthropic_version: "bedrock-2023-05-31",
        messages: [{ role: "user", content: prompt }],
        max_tokens: 1024,
      }),
    });
    const response = await bedrockClient.send(command);
    const result = JSON.parse(new TextDecoder().decode(response.body));
    return {
      content: [{ type: "text", text: result.content[0].text }],
    };
  }
);

// Resource: Expose S3 documents
server.resource(
  "s3-document",
  "s3://{bucket}/{key}",
  async (uri) => {
    const [bucket, ...keyParts] = uri.replace("s3://", "").split("/");
    const key = keyParts.join("/");
    const command = new GetObjectCommand({ Bucket: bucket, Key: key });
    const response = await s3Client.send(command);
    const content = await response.Body.transformToString();
    return {
      contents: [{ uri, mimeType: "text/plain", text: content }],
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Deploying MCP Servers on AWS

Architecture for secure remote MCP server deployment:

```
User/Agent -> CloudFront (CDN + DDoS) -> WAF (Request filtering)
    -> ALB (Load balancing) -> ECS Fargate (MCP Server containers)
    -> OAuth 2.0 Provider (Cognito) for authentication
    -> VPC Private Subnets for network isolation
```

CDK deployment pattern:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';
import * as wafv2 from 'aws-cdk-lib/aws-wafv2';

export class McpServerStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    const vpc = new ec2.Vpc(this, 'McpVpc', { maxAzs: 2 });
    const cluster = new ecs.Cluster(this, 'McpCluster', { vpc });

    const taskDef = new ecs.FargateTaskDefinition(this, 'McpTask', {
      memoryLimitMiB: 512,
      cpu: 256,
    });

    taskDef.addContainer('mcp-server', {
      image: ecs.ContainerImage.fromAsset('./mcp-server'),
      portMappings: [{ containerPort: 3000 }],
      logging: ecs.LogDrivers.awsLogs({ streamPrefix: 'mcp-server' }),
      healthCheck: {
        command: ['CMD-SHELL', 'curl -f http://localhost:3000/health || exit 1'],
        interval: cdk.Duration.seconds(30),
      },
    });

    const service = new ecs.FargateService(this, 'McpService', {
      cluster,
      taskDefinition: taskDef,
      desiredCount: 2,
      assignPublicIp: false,
    });

    const alb = new elbv2.ApplicationLoadBalancer(this, 'McpAlb', {
      vpc,
      internetFacing: true,
    });

    const listener = alb.addListener('McpListener', { port: 443 });
    listener.addTargets('McpTarget', {
      port: 3000,
      targets: [service],
      healthCheck: { path: '/health' },
    });
  }
}
```

## Amazon Bedrock Knowledge Bases

### RAG Architecture with S3 Vectors

```
Documents (S3) -> Data Ingestion Pipeline -> Chunking Strategy
    -> Embedding Model (Titan Embeddings) -> S3 Vectors (storage)
    -> Retrieval API -> Agent/Application -> Response with citations
```

### Setting Up a Knowledge Base with CDK

```typescript
import * as bedrock from '@cdklabs/generative-ai-cdk-constructs/lib/cdk-lib/bedrock';

const kb = new bedrock.KnowledgeBase(this, 'KnowledgeBase', {
  embeddingsModel: bedrock.BedrockFoundationModel.TITAN_EMBED_TEXT_V2_1024,
  instruction: 'Use this knowledge base to answer questions about our products and documentation.',
});

const dataSource = new bedrock.S3DataSource(this, 'DataSource', {
  bucket: documentsBucket,
  knowledgeBase: kb,
  chunkingStrategy: bedrock.ChunkingStrategy.SEMANTIC,
});
```

### Chunking Strategy Selection Guide

| Strategy | Best For | Chunk Size | Overlap |
|----------|----------|------------|---------|
| **Semantic** | Mixed content types, varied document lengths | Auto-determined | Semantic boundaries |
| **Hierarchical** | Structured documents (manuals, wikis) | Parent: 1500, Child: 300 | Section boundaries |
| **Fixed-Size** | Uniform documents, simple setup | 1000 tokens | 200 tokens |
| **Custom Lambda** | Domain-specific splitting requirements | Custom | Custom |

### Vector Storage Selection Guide

| Storage | Cost | Scale | Best For |
|---------|------|-------|----------|
| **S3 Vectors** | Lowest (90% savings) | Trillions of vectors | Cost-optimized production |
| **OpenSearch Serverless** | Medium | Millions | Hybrid text + vector search |
| **Aurora pgvector** | Medium | Millions | Existing PostgreSQL users |
| **Pinecone** | Higher | Billions | Managed, metadata filtering |
| **Redis Enterprise** | Higher | Millions | Ultra-low latency |

## Generative AI Architecture Patterns

### Pattern 1: Conversational AI Agent

```
User -> API Gateway -> Lambda (auth) -> Bedrock Agent
    -> AgentCore Runtime (session, memory)
    -> AgentCore Gateway (tools: Lambda, APIs, MCP servers)
    -> Bedrock Knowledge Base (RAG for context)
    -> Bedrock Guardrails (safety, PII)
    -> Response with citations
```

### Pattern 2: Multi-Agent Orchestration

```
User Request -> Supervisor Agent (routing, planning)
    -> Specialist Agent 1 (e.g., research) via A2A protocol
    -> Specialist Agent 2 (e.g., analysis) via A2A protocol
    -> Specialist Agent 3 (e.g., writing) via A2A protocol
    -> Supervisor Agent (aggregation, quality check)
    -> Response
```

### Pattern 3: Enterprise Knowledge Copilot

```
Enterprise Data Sources -> S3 (documents, wikis, tickets)
    -> Bedrock Knowledge Base (multimodal, S3 Vectors)
    -> Bedrock Agent with Guardrails
    -> Cognito Authentication
    -> React Frontend (FAST template)
    -> CloudWatch Observability
```

### Pattern 4: Content Generation Pipeline

```
Request -> API Gateway -> Step Functions (orchestration)
    -> Bedrock (text generation, Claude/Nova)
    -> Bedrock (image generation, Nova Canvas)
    -> S3 (output storage)
    -> SNS (notification)
    -> CloudWatch (monitoring)
```

## AWS AI Service Selection Guide

| Use Case | Primary Service | Supporting Services |
|----------|----------------|-------------------|
| Conversational agents | Bedrock AgentCore | Knowledge Bases, Guardrails |
| Document Q&A (RAG) | Bedrock Knowledge Bases | S3 Vectors, Bedrock Models |
| Text generation | Bedrock (Claude/Nova) | Guardrails, CloudWatch |
| Image generation | Bedrock (Nova Canvas) | S3, CloudFront |
| Speech-to-text | Amazon Transcribe | S3, Comprehend |
| Text-to-speech | Amazon Polly | S3, CloudFront |
| Document processing | Amazon Textract | Comprehend, S3 |
| Sentiment analysis | Amazon Comprehend | S3, QuickSight |
| Image/video analysis | Amazon Rekognition | S3, Lambda |
| Enterprise search | Amazon Kendra | S3, RDS, SharePoint |
| Code assistance | Amazon Q Developer | CodeWhisperer |
| Custom ML models | Amazon SageMaker | S3, ECR, CloudWatch |
| ML model serving | SageMaker Endpoints | Auto Scaling, CloudWatch |

## AI Infrastructure as Code

### CDK Generative AI Constructs

```typescript
import { bedrock, opensearchserverless } from '@cdklabs/generative-ai-cdk-constructs';

// RAG with OpenSearch Serverless
const vectorStore = new opensearchserverless.VectorCollection(this, 'VectorStore');

const kb = new bedrock.KnowledgeBase(this, 'KB', {
  embeddingsModel: bedrock.BedrockFoundationModel.TITAN_EMBED_TEXT_V2_1024,
  vectorStore: vectorStore,
});

// Agent with knowledge base
const agent = new bedrock.Agent(this, 'Agent', {
  foundationModel: bedrock.BedrockFoundationModel.ANTHROPIC_CLAUDE_SONNET_V1,
  instruction: 'You are a helpful assistant that answers questions using the knowledge base.',
  knowledgeBases: [kb],
});

// Guardrails
const guardrail = new bedrock.Guardrail(this, 'Guardrail', {
  name: 'content-safety',
  blockedInputMessaging: 'I cannot process this request.',
  blockedOutputsMessaging: 'I cannot provide this response.',
  contentFilters: [
    {
      type: bedrock.ContentFilterType.SEXUAL,
      inputStrength: bedrock.FilterStrength.HIGH,
      outputStrength: bedrock.FilterStrength.HIGH,
    },
    {
      type: bedrock.ContentFilterType.HATE,
      inputStrength: bedrock.FilterStrength.HIGH,
      outputStrength: bedrock.FilterStrength.HIGH,
    },
  ],
});
```

### CloudFormation for AgentCore

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Bedrock AgentCore AI Stack

Resources:
  AgentRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: bedrock.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: BedrockInvoke
          PolicyDocument:
            Statement:
              - Effect: Allow
                Action:
                  - bedrock:InvokeModel
                Resource: !Sub arn:aws:bedrock:${AWS::Region}::foundation-model/*

  Agent:
    Type: AWS::Bedrock::Agent
    Properties:
      AgentName: my-ai-agent
      FoundationModel: anthropic.claude-sonnet-4-5-20250929-v1:0
      Instruction: |
        You are a helpful AI assistant. Use the available tools
        and knowledge base to answer user questions accurately.
      AgentResourceRoleArn: !GetAtt AgentRole.Arn
      IdleSessionTTLInSeconds: 1800

  KnowledgeBase:
    Type: AWS::Bedrock::KnowledgeBase
    Properties:
      Name: my-knowledge-base
      RoleArn: !GetAtt KBRole.Arn
      KnowledgeBaseConfiguration:
        Type: VECTOR
        VectorKnowledgeBaseConfiguration:
          EmbeddingModelArn: !Sub arn:aws:bedrock:${AWS::Region}::foundation-model/amazon.titan-embed-text-v2:0
      StorageConfiguration:
        Type: S3_VECTORS
        S3VectorsConfiguration:
          BucketArn: !GetAtt VectorBucket.Arn

Outputs:
  AgentId:
    Value: !GetAtt Agent.AgentId
  KnowledgeBaseId:
    Value: !GetAtt KnowledgeBase.KnowledgeBaseId
```

## Model Selection Guide

| Model | Strengths | Latency | Cost | Best For |
|-------|-----------|---------|------|----------|
| **Claude Sonnet** | Complex reasoning, coding, analysis | Medium | Medium | General-purpose, coding |
| **Claude Haiku** | Fast, efficient, accurate | Low | Low | High-volume, simple tasks |
| **Nova Pro** | Balanced, multimodal | Medium | Medium | General-purpose |
| **Nova Lite** | Fast, cost-effective | Low | Low | Simple tasks, high volume |
| **Nova Micro** | Text-only, fastest | Lowest | Lowest | Classification, routing |
| **Llama 3** | Open source, customizable | Medium | Medium | Fine-tuning, self-hosting |
| **Mistral** | Multilingual, efficient | Low | Low | Code, multilingual tasks |

## Common Pitfalls

1. **No Guardrails** — Deploying agents without Bedrock Guardrails for content safety and PII protection
2. **Overly permissive IAM** — Using broad `bedrock:*` permissions instead of specific actions and resources
3. **No observability** — Running agents without AgentCore Observability or CloudWatch monitoring
4. **Expensive vector storage** — Using managed vector databases when S3 Vectors offers 90% savings
5. **Missing Cedar policies** — Deploying agents without deterministic access controls via AgentCore Policy
6. **ClickOps infrastructure** — Creating resources manually instead of using CDK/CloudFormation/Terraform
7. **Single-region deployment** — Not planning for multi-region availability in production
8. **No evaluation** — Deploying agents without continuous quality evaluation via AgentCore Evaluations
9. **Wrong model for the task** — Using expensive models (Claude Opus) for simple tasks that Nova Micro can handle
10. **Unprotected MCP servers** — Deploying remote MCP servers without OAuth 2.0, WAF, or VPC isolation

## Reference Assets

| Asset | Source | Description |
|-------|--------|-------------|
| Bedrock AgentCore Docs | `docs.aws.amazon.com/bedrock-agentcore` | Complete AgentCore service documentation |
| AWS MCP Servers | `github.com/awslabs/mcp` | Official MCP servers for AWS (11+ servers) |
| Gen AI CDK Constructs | `awslabs.github.io/generative-ai-cdk-constructs` | CDK constructs for AI workloads |
| Well-Architected Gen AI Lens | `docs.aws.amazon.com/wellarchitected` | Six-pillar architecture guidance |
| MCP Server Deployment Guide | `aws.amazon.com/solutions/guidance/deploying-mcp-servers` | Secure MCP deployment on AWS |
| FAST Template | AWS blog | Fullstack AgentCore Solution Template |
| IaC MCP Server | AWS DevOps Blog | AI-powered CDK/CloudFormation assistance |
| AgentCore Policy | AWS News Blog | Cedar-based agent access controls |
| AgentCore Evaluations | AWS ML Blog | 13 pre-built quality evaluators |
| Bedrock Knowledge Bases | AWS Bedrock Docs | RAG with multimodal retrieval and S3 Vectors |

## Visual Diagramming with Excalidraw

Use the Excalidraw MCP server to generate interactive diagrams directly in the conversation. Describe what you need in natural language and Excalidraw renders it as an interactive canvas with hand-drawn style.

### When to Use

- AWS AI architecture diagrams showing service interactions
- AgentCore agent flow diagrams (Runtime -> Gateway -> Tools -> Response)
- MCP server deployment architecture (CloudFront -> WAF -> ALB -> ECS)
- RAG pipeline topology (Documents -> Chunking -> Embeddings -> Vector Store -> Retrieval)
- Multi-agent orchestration diagrams with A2A protocol flows
- VPC network architecture for AI workloads

### Workflow

1. Describe the diagram you need — be specific about AWS services, data flows, and security boundaries
2. Review the rendered interactive diagram in the chat
3. Request refinements by describing what to change (add/remove/rearrange elements)
4. Use fullscreen mode for detailed editing when needed

### Tips for Effective Diagrams

- Name specific AWS services and their connections (e.g., "API Gateway connects to Lambda authorizer and Bedrock Agent")
- Specify layout direction when it matters (e.g., "left-to-right flow" or "top-down hierarchy")
- Include security boundaries (VPC, subnets, IAM) in the diagram description
- Request AWS-style architecture diagrams for documentation
