---
name: marketplace-mcp
description: Build and deploy MCP servers to FastMCP Cloud marketplace. Use this skill when creating Python MCP servers with FastMCP 2 for deployment to fastmcp.cloud. Provides patterns, examples, and deployment guidance.
---

# FastMCP Cloud MCP Server Builder

Build production-ready MCP servers with FastMCP 2 and deploy them to fastmcp.cloud.

---

## Quick Start Workflow

### 1. Create Project Structure

```
my-mcp-server/
├── server.py          # FastMCP server (single file)
├── pyproject.toml     # Dependencies
├── .env               # Local secrets (never commit)
├── .gitignore         # Exclude .env and venv
└── README.md          # Usage documentation
```

### 2. Minimal Server Template

```python
"""MCP Server for [Service Name]."""

import os
from fastmcp import FastMCP

mcp = FastMCP(
    "my-service-mcp",
    instructions="Description of what this server does and how to use it.",
)

@mcp.tool()
async def my_tool(param: str) -> str:
    """Tool description for LLM discovery."""
    return f"Result: {param}"

if __name__ == "__main__":
    host = os.getenv("MCP_HOST", "0.0.0.0")
    port = int(os.getenv("MCP_PORT", "8000"))
    mcp.run(transport="http", host=host, port=port)
```

### 3. Deploy to fastmcp.cloud

1. Push to GitHub (public or private repo)
2. Visit [fastmcp.cloud](https://fastmcp.cloud) and authenticate with GitHub
3. Create project, set entrypoint to `server.py`
4. Server deploys to `https://your-project.fastmcp.app/mcp`

---

## Development Process

### Phase 1: Setup

**Install dependencies with uv:**
```bash
uv venv
uv pip install "fastmcp>=2.0.0,<3" httpx pydantic python-dotenv
```

**Create pyproject.toml:**
```toml
[project]
name = "my-mcp-server"
version = "0.1.0"
description = "MCP server for [service]"
requires-python = ">=3.11"
dependencies = [
    "fastmcp>=2.0.0,<3",
    "httpx>=0.27.0",
    "pydantic>=2.0.0",
]
```

**Create .gitignore:**
```
.env
.venv/
venv/
__pycache__/
*.pyc
```

### Phase 2: Implement Tools

Load the [FastMCP Patterns Guide](./reference/fastmcp_patterns.md) for detailed implementation patterns.

**Key patterns:**
- Use `@mcp.tool()` decorator for all tools
- Use Pydantic models or type hints for validation
- Support both JSON and Markdown response formats
- Implement pagination for list operations
- Use async/await for all I/O operations

### Phase 3: Test Locally

```bash
# Activate virtual environment
source .venv/bin/activate

# Load environment variables
export $(cat .env | xargs)

# Run server locally
python server.py

# Or test with MCP Inspector
fastmcp inspect server.py
```

### Phase 4: Deploy

See [Deployment Guide](./reference/deployment.md) for complete deployment instructions.

---

## Reference Documentation

### FastMCP Patterns
- [FastMCP Patterns Guide](./reference/fastmcp_patterns.md) - Tool registration, Pydantic models, response formats

### Deployment
- [Deployment Guide](./reference/deployment.md) - fastmcp.cloud deployment, secrets, CI/CD

### External Resources
- **FastMCP Documentation MCP Server**: Connect to `https://gofastmcp.com/mcp` for live documentation access
- **FastMCP Docs**: [gofastmcp.com](https://gofastmcp.com)
- **MCP Protocol**: [modelcontextprotocol.io](https://modelcontextprotocol.io)

---

## Example Server

See [example/server.py](./example/server.py) for a complete, deployable MCP server template that demonstrates all best practices.

---

## Tool Naming Convention

Use the pattern `{service}_{action}_{resource}`:
- `github_list_repos`
- `slack_send_message`
- `db_query_users`

**Always include service prefix** to avoid conflicts when multiple MCP servers are used together.

---

## Response Format Pattern

Support both JSON (for programmatic use) and Markdown (for human readability):

```python
from enum import Enum

class ResponseFormat(str, Enum):
    MARKDOWN = "markdown"
    JSON = "json"

@mcp.tool()
async def list_items(
    limit: int = 20,
    response_format: str = "json"
) -> str:
    items = await fetch_items(limit)

    if response_format == "markdown":
        lines = ["# Items", ""]
        for item in items:
            lines.append(f"- **{item['name']}**: {item['description']}")
        return "\n".join(lines)

    return json.dumps({"count": len(items), "items": items}, indent=2)
```

---

## Error Handling Pattern

```python
def handle_api_error(e: Exception) -> str:
    """Consistent error formatting."""
    if isinstance(e, httpx.HTTPStatusError):
        status = e.response.status_code
        if status == 401:
            return "Error: Authentication failed. Check API credentials."
        elif status == 404:
            return "Error: Resource not found."
        elif status == 429:
            return "Error: Rate limit exceeded. Wait before retrying."
        return f"Error: API request failed with status {status}"
    return f"Error: {type(e).__name__}: {e}"
```

---

## Secrets Management

**Local development:**
- Store secrets in `.env` file
- Load with `python-dotenv` or export manually
- Never commit `.env` to git

**fastmcp.cloud:**
- Use project settings to configure environment variables
- Secrets are encrypted and injected at runtime
- Access via `os.environ.get("SECRET_NAME")`

See [Deployment Guide](./reference/deployment.md) for detailed secrets management instructions.
