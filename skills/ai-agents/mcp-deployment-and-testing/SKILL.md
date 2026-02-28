---
name: MCP Deployment and Testing
description: This skill should be used when the user asks to "deploy MCP server", "test MCP", "use ngrok", "MCP Inspector", "connect to ChatGPT", "create connector", "troubleshoot MCP", "debug server", or needs guidance on deploying and testing MCP servers for the OpenAI Apps SDK.
version: 0.1.0
---

# MCP Deployment and Testing for OpenAI Apps SDK

## Overview

Deploying and testing MCP servers requires proper HTTPS setup, local development tools, and ChatGPT connector configuration. This skill covers the complete deployment workflow from local testing to production.

## Local Development Setup

### Running the Server

**Python:**
```bash
# Install dependencies
pip install mcp

# Run server
python server.py
# Server running on http://localhost:8000/mcp
```

**TypeScript:**
```bash
# Install dependencies
npm install @modelcontextprotocol/sdk express

# Run server
npx tsx server.ts
# Server running on http://localhost:8000/mcp
```

### Exposing with ngrok

ChatGPT requires HTTPS. Use ngrok for local development:

```bash
# Install ngrok
brew install ngrok  # macOS
# or download from ngrok.com

# Expose local server
ngrok http 8000

# Output:
# Forwarding https://abc123.ngrok.io -> http://localhost:8000
```

Save the HTTPS URL for ChatGPT connector setup.

## MCP Inspector

Test MCP servers without ChatGPT integration:

### Installation

```bash
npx @modelcontextprotocol/inspector@latest
```

### Usage

```bash
# Test local server
npx @modelcontextprotocol/inspector http://localhost:8000/mcp

# Test ngrok URL
npx @modelcontextprotocol/inspector https://abc123.ngrok.io/mcp
```

### Inspector Features

- **List Tools** - View all available tools and schemas
- **Call Tools** - Execute tools with test parameters
- **View Resources** - List and fetch MCP resources
- **Test Widgets** - Preview widget HTML rendering
- **Debug Responses** - Inspect structuredContent and _meta

## ChatGPT Connector Setup

### Enable Developer Mode

1. Open ChatGPT settings
2. Navigate to "Developer settings"
3. Enable "Developer mode"

### Create Connector

1. Go to ChatGPT → Settings → Connectors
2. Click "Add connector"
3. Enter your MCP server URL (HTTPS required)
4. Name your connector
5. Save

### Add to Conversation

1. Start a new conversation
2. Click the connector icon
3. Select your connector
4. Test with natural language prompts

## Testing Workflow

### 1. Local Server Test

```bash
# Start server
python server.py

# Test with curl
curl http://localhost:8000/mcp \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"method": "tools/list"}'
```

### 2. MCP Inspector Test

```bash
# Run inspector
npx @modelcontextprotocol/inspector http://localhost:8000/mcp

# In inspector:
# - Verify tools list correctly
# - Test each tool with sample inputs
# - Check widget rendering
```

### 3. ngrok Integration Test

```bash
# Expose server
ngrok http 8000

# Test HTTPS endpoint
curl https://abc123.ngrok.io/mcp \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"method": "tools/list"}'
```

### 4. ChatGPT Integration Test

1. Add connector with ngrok URL
2. Test natural language tool invocation
3. Verify widget rendering
4. Check response accuracy

## Debugging

### Enable Debug Logging

**Python:**
```python
import logging
logging.basicConfig(level=logging.DEBUG)

mcp = FastMCP("server", debug=True)
```

**TypeScript:**
```typescript
const server = new Server(
  { name: "server", version: "1.0.0" },
  { capabilities: { tools: {} }, debug: true }
);
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Connection refused | Check server is running on correct port |
| SSL error | Use ngrok for HTTPS |
| Tool not found | Verify tool registration and naming |
| Widget not rendering | Check `openai/outputTemplate` URI |
| Auth failure | Verify tokens and security config |

### Request Logging

**Python:**
```python
@mcp.middleware
async def log_requests(request, call_next):
    print(f"Request: {request.method} - {request.params}")
    response = await call_next(request)
    print(f"Response: {response}")
    return response
```

## Production Deployment

### Hosting Options

| Platform | Notes |
|----------|-------|
| Railway | Easy Python/Node deployment |
| Render | Free tier available |
| Fly.io | Edge deployment |
| AWS Lambda | Serverless option |
| Google Cloud Run | Container-based |

### Railway Deployment

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login and init
railway login
railway init

# Deploy
railway up
```

### Docker Deployment

**Dockerfile (Python):**
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000
CMD ["python", "server.py"]
```

**Build and run:**
```bash
docker build -t mcp-server .
docker run -p 8000:8000 mcp-server
```

### Environment Variables

Production configuration:

```bash
# .env.production
PORT=8000
HOST=0.0.0.0
DATABASE_URL=postgresql://...
API_KEY=your_production_key
LOG_LEVEL=INFO
```

### Health Checks

Add a health check endpoint:

**Python:**
```python
@mcp.tool()
def health_check() -> dict:
    """Server health check."""
    return {
        "structuredContent": {
            "status": "healthy",
            "version": "1.0.0",
            "timestamp": datetime.now().isoformat()
        }
    }
```

## Production Checklist

- [ ] HTTPS enabled (required)
- [ ] Environment variables configured
- [ ] Debug mode disabled
- [ ] Logging configured
- [ ] Error handling implemented
- [ ] Rate limiting enabled
- [ ] Health check endpoint
- [ ] Monitoring/alerting setup
- [ ] Database backups (if applicable)
- [ ] CORS configured correctly

## Troubleshooting

### Server Not Responding

```bash
# Check if server is running
curl http://localhost:8000/mcp

# Check port binding
lsof -i :8000

# Check logs
tail -f server.log
```

### Widget Not Loading

1. Check resource registration
2. Verify `mimeType: "text/html+skybridge"`
3. Inspect `_meta.openai/outputTemplate`
4. Test widget HTML in MCP Inspector

### Tool Call Failures

1. Verify tool name matches exactly
2. Check inputSchema validation
3. Review error response format
4. Test with MCP Inspector first

## Additional Resources

### Reference Files

For detailed deployment guides:
- **`references/hosting-guide.md`** - Platform-specific deployment
- **`references/monitoring.md`** - Logging and monitoring setup

### Example Files

Working examples in `examples/`:
- **`examples/Dockerfile`** - Docker deployment example
- **`examples/railway.json`** - Railway configuration

### Official Documentation

- Apps SDK Deployment: https://developers.openai.com/apps-sdk/build/deploy-your-app/
- ChatGPT Connectors: https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-connectors-in-chatgpt-beta
