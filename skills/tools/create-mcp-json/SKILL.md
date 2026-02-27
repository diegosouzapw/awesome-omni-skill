---
name: create-mcp-json
description: Create an mcp.json client configuration from a server.json. Use when the user wants to generate a ready-to-use MCP client config for connecting to an MCP server.
user-invocable: true
argument-hint: [constraints e.g. "read only tools", "no env vars"]
---

# create-mcp-json

## Sequencing Checklist

- [ ] Confirm a `server.json` is already in context (if not, ask user or suggest `/create-server-json`)
- [ ] Read the `server.json` and identify all `packages` and `remotes` entries
- [ ] Apply any user constraints from `$ARGUMENTS`
- [ ] Generate one mcp.json entry per `packages` item and one per `remotes` item
- [ ] Write descriptions that explain **when to reach for this server** (not just copy server.json)
- [ ] Pin all packages to specific versions (never `@latest`)
- [ ] Present the result and ask the user which output format they want
- [ ] Validate the generated file against `docs/mcp.schema.json` using `ajv`
- [ ] Fix any validation errors and re-validate until it passes

Generate a ready-to-use `mcp.json` client configuration from an MCP server's `server.json`.

The user provided this context about the configuration they want:

```
$ARGUMENTS
```

## Prerequisites

A `server.json` for the target server MUST already be in context before running this skill. If one is not present:

1. Ask the user to provide the `server.json`, OR
2. Suggest running `/create-server-json` first to generate one

Do NOT attempt to create an mcp.json without a server.json to derive it from.

## What is mcp.json?

An `mcp.json` is a **client-side configuration** that tells an MCP client (Claude Code, Claude Desktop, VS Code, Cursor, etc.) exactly how to connect to and run MCP servers. It is the fully-resolved, concrete form of what a `server.json` describes abstractly.

Think of it this way:
- `server.json` = "Here's how this server *can* be configured"
- `mcp.json` = "Here's exactly how this client *will* connect to its servers"

Reference the full mcp.json specification at: `docs/MCP_JSON.md` in this repository.

## Instructions

1. Read the `server.json` already in context
2. If `$ARGUMENTS` includes constraints (e.g., "read only tools", "no write access", "minimal env vars"), factor those into the configuration — for example by selecting specific `packageArguments` or omitting optional environment variables
3. Generate one mcp.json entry per `packages` item AND one per `remotes` item in the server.json
4. Present the result and ask the user which output format they want (see [Output Formats](#output-formats))

## Transformation Rules

### From `packages[]` (stdio servers)

| server.json field | mcp.json field | Notes |
|---|---|---|
| `title` | `title` | Direct copy |
| `description` | `description` | See [Writing descriptions](#writing-descriptions) |
| `packages[N].transport.type` | `type` | Usually `"stdio"` |
| `packages[N].registryType` / `runtimeHint` | `command` | See command mapping below |
| `packages[N].identifier` + `packageArguments` | `args` | See args construction below |
| `packages[N].environmentVariables` | `env` | Include ALL env vars as `${ENV_VAR_NAME}` templated variables — see [Environment variable handling](#environment-variable-handling) |

#### Command mapping

| `registryType` | `runtimeHint` | `command` |
|---|---|---|
| `npm` | `npx` (default) | `"npx"` |
| `pypi` | `uvx` (default) | `"uvx"` |
| `oci` | — | `"docker"` |

#### Version pinning

**ALWAYS** pin to a specific version — never use `@latest`, `@^1.0.0`, or leave the version off. This avoids supply chain attacks where a compromised latest version gets pulled automatically on every launch.

- **npm**: `@modelcontextprotocol/server-github@0.6.2` not `@modelcontextprotocol/server-github` or `@modelcontextprotocol/server-github@latest`
- **pypi**: `weather-mcp-server==0.5.0` not `weather-mcp-server` or `weather-mcp-server@latest`
- **oci**: `docker.io/mcp/server:1.0.2` not `docker.io/mcp/server:latest`

If the server.json has no `version` field, look it up on the package registry (PyPI, npm, etc.) to find the latest version and pin to that. Note the version you chose in the output.

#### Args construction

- **npm**: `["-y", "<identifier>@<version>", ...packageArguments]`
- **pypi**: `["<identifier>==<version>", ...packageArguments]`
- **oci**: `["run", "-i", "--rm", ...runtimeArguments, "<identifier>:<version>", ...packageArguments]`

For `packageArguments`, resolve them as follows:
- `"positional"` args: append `value` directly
- `"named"` args: append `name` then `value` (e.g., `"--host", "localhost"`)
- If the argument has `isRequired: true` and no `value`, use `valueHint` as a placeholder comment or ask the user

### From `remotes[]` (HTTP/SSE servers)

| server.json field | mcp.json field | Notes |
|---|---|---|
| `title` | `title` | From top-level server.json |
| `description` | `description` | See [Writing descriptions](#writing-descriptions) |
| `remotes[N].type` | `type` | `"streamable-http"` or `"sse"` |
| `remotes[N].url` | `url` | Resolve any `{variable}` templates — ask user for values |
| `remotes[N].headers` | `headers` | Secret header values use `${ENV_VAR_NAME}` interpolation |

### Writing descriptions

Do NOT just copy the server.json `description` verbatim — it's often too generic (and capped at 100 chars). Instead, write a description that helps the user instantly understand **when to reach for this server** and **what it can actually do**, especially vs. similar servers.

A good description should answer: "If I'm scanning a list of 20 servers, what tells me *this* is the one I need right now?"

Guidelines:
- **Start with "Use when..."** to frame the description as a trigger — what situation should make the user think of this server
- **Name the concrete actions/tools** the server provides (e.g., "search logs, find error patterns, correlate across services" — not "provides access to logs")
- **Include the scope/platform** when it differentiates (e.g., "CloudWatch Logs" not just "logs")
- **Mention what makes it unique** vs. similar servers (e.g., "cross-service log correlation" is a differentiator vs. a generic log reader)
- Keep it to 1-2 sentences

Examples:
- Bad: `"GitHub API integration"` — too vague, could be anything
- Good: `"Use when you need to work with GitHub repos. Create/manage issues, PRs, branches, and files. Search code, review diffs, and merge PRs."`
- Bad: `"Provides access to AWS CloudWatch Logs"` — doesn't say what you can do with them
- Good: `"Use when debugging or investigating AWS services. Search and filter CloudWatch Logs, find error patterns, and correlate log events across multiple services."`

### Environment variable handling

**Bias towards being explicit.** Do not assume the user has CLIs, credentials, or cloud profiles pre-configured. Include ALL environment variables from the server.json — both required and optional — as `${ENV_VAR_NAME}` templated variables in the `env` block. This makes the configuration self-documenting and ensures the user knows exactly what needs to be set up.

- **All env vars** get included in `env`, using `${ENV_VAR_NAME}` interpolation — the variable name inside `${}` matches the env var key exactly (UPPER_SNAKE_CASE)
- **Secrets** (`isSecret: true`) always use `${ENV_VAR_NAME}` (e.g., `${AWS_SECRET_ACCESS_KEY}`)
- **Non-secrets** also use `${ENV_VAR_NAME}` by default (e.g., `${AWS_REGION}`, `${AWS_PROFILE}`) — don't hardcode values or silently omit them
- **Vars with a `default`** may use the default value directly only if the default is universally sensible; otherwise use `${ENV_VAR_NAME}`
- Examples: `${GITHUB_PERSONAL_ACCESS_TOKEN}`, `${AWS_ACCESS_KEY_ID}`, `${AWS_REGION}`, `${SENTRY_AUTH_TOKEN}`

### Applying user constraints from `$ARGUMENTS`

If the user specifies constraints, apply them:

- **"read only tools"** / **"read only"**: If the server supports tool filtering via `packageArguments` or env vars (e.g., `ENABLED_TOOLS`, `--read-only`), configure accordingly. Note this in the output.
- **"no env vars"**: Omit optional env vars, only include `isRequired: true` vars
- **"minimal"**: Only include required fields, skip `title` and `description`
- **Custom constraints**: Interpret and apply as best as possible, noting what was done

## Output Formats

### Flat mcp.json format (default)

The root object is a map of server names to configurations. This is the format defined in the mcp.json spec:

```json
{
  "server-name": {
    "title": "Human-Readable Name",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@scope/package-name"],
    "env": {
      "API_KEY": "${API_KEY}"
    }
  }
}
```

### Claude Code `.mcp.json` format

Wraps servers under an `mcpServers` key. Use this when the user is configuring Claude Code specifically:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@scope/package-name"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

Note: Claude Code defaults `type` to `"stdio"`, so the `type` field can be omitted for stdio servers. For `sse` or `streamable-http` servers it must be included.

## Examples

### npm package → mcp.json

Given this server.json:
```json
{
  "name": "io.github.modelcontextprotocol/server-github",
  "title": "GitHub",
  "description": "GitHub API integration",
  "packages": [{
    "registryType": "npm",
    "identifier": "@modelcontextprotocol/server-github",
    "version": "0.6.2",
    "transport": { "type": "stdio" },
    "environmentVariables": [{
      "name": "GITHUB_PERSONAL_ACCESS_TOKEN",
      "description": "GitHub PAT",
      "isRequired": true,
      "isSecret": true
    }]
  }]
}
```

Produces:
```json
{
  "github": {
    "title": "GitHub",
    "description": "Create/manage issues, PRs, branches, and files in GitHub repos. Search code, review diffs, and merge PRs.",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github@0.6.2"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
    }
  }
}
```

### pypi package → mcp.json

Given a PyPI server.json:
```json
{
  "name": "io.github.example/weather",
  "title": "Weather",
  "packages": [{
    "registryType": "pypi",
    "identifier": "weather-mcp-server",
    "version": "0.5.0",
    "runtimeHint": "uvx",
    "transport": { "type": "stdio" },
    "environmentVariables": [
      { "name": "WEATHER_API_KEY", "isRequired": true, "isSecret": true },
      { "name": "WEATHER_UNITS", "default": "celsius" }
    ]
  }]
}
```

Produces:
```json
{
  "weather": {
    "title": "Weather",
    "type": "stdio",
    "command": "uvx",
    "args": ["weather-mcp-server==0.5.0"],
    "env": {
      "WEATHER_API_KEY": "${WEATHER_API_KEY}",
      "WEATHER_UNITS": "celsius"
    }
  }
}
```

### Remote server → mcp.json

Given a remote server.json:
```json
{
  "name": "io.github.example/analytics",
  "title": "Analytics",
  "remotes": [{
    "type": "streamable-http",
    "url": "https://mcp.analytics.example.com/mcp",
    "headers": [{
      "name": "Authorization",
      "isRequired": true,
      "isSecret": true
    }]
  }]
}
```

Produces:
```json
{
  "analytics": {
    "title": "Analytics",
    "type": "streamable-http",
    "url": "https://mcp.analytics.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${ANALYTICS_AUTH_TOKEN}"
    }
  }
}
```

### OAuth remote → mcp.json (no auth config needed)

```json
{
  "linear": {
    "title": "Linear",
    "description": "Create, update, and search issues, projects, and cycles in Linear. Manage labels, assignees, and workflow states.",
    "type": "streamable-http",
    "url": "https://mcp.linear.app/mcp"
  }
}
```

OAuth is handled out-of-band by the MCP client via well-known metadata endpoints. No explicit auth config is needed in the mcp.json.

## Validation

After writing the mcp.json file, validate it against the schema at `docs/mcp.schema.json` in this repository:

```bash
# Install validator if needed (check first)
which ajv || npm install -g ajv-cli

# Validate the generated file against the schema
ajv validate -s docs/mcp.schema.json -d <path-to-generated-mcp.json> --spec=draft7 --strict=false
```

If validation fails, read the errors, fix the generated file, and re-validate until it passes. Do not consider the task complete until validation succeeds.

### Additional checks (not covered by schema)

Before writing the file, also verify:

- [ ] Secret values use `${ENV_VAR_NAME}` interpolation, not literal values
- [ ] Non-secret env vars with defaults use the default value directly
- [ ] User constraints from `$ARGUMENTS` have been applied and noted
