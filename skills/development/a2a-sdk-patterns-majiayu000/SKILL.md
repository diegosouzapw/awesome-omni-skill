---
name: a2a-sdk-patterns
description: SDK installation and setup patterns for Agent-to-Agent Protocol across Python, TypeScript, Java, C#, and Go. Use when implementing A2A protocol, setting up SDKs, configuring authentication, or when user mentions SDK installation, language-specific setup, or A2A integration.
allowed-tools: Read, Bash, Write, Edit, Grep, Glob
---

# Agent-to-Agent Protocol SDK Patterns

**Purpose:** Provide installation, configuration, and usage patterns for A2A Protocol SDKs across multiple programming languages.

**Activation Triggers:**
- SDK installation requests
- Language-specific A2A setup
- Authentication configuration
- Package dependency issues
- SDK version compatibility
- Import/setup errors

**Supported Languages:**
- Python (3.8+)
- TypeScript/JavaScript (Node 18+)
- Java (11+)
- C# (.NET 6+)
- Go (1.20+)

## Quick Start by Language

### Python

```bash
# Install SDK
./scripts/install-python.sh

# Verify installation
./scripts/validate-python.sh
```

### TypeScript

```bash
# Install SDK
./scripts/install-typescript.sh

# Verify installation
./scripts/validate-typescript.sh
```

### Java

```bash
# Install SDK
./scripts/install-java.sh

# Verify installation
./scripts/validate-java.sh
```

### C#

```bash
# Install SDK
./scripts/install-csharp.sh

# Verify installation
./scripts/validate-csharp.sh
```

### Go

```bash
# Install SDK
./scripts/install-go.sh

# Verify installation
./scripts/validate-go.sh
```

## Installation Scripts

All installation scripts are in `scripts/` directory:

- `install-python.sh` - Install Python SDK via pip
- `install-typescript.sh` - Install TypeScript SDK via npm/yarn
- `install-java.sh` - Install Java SDK via Maven/Gradle
- `install-csharp.sh` - Install C# SDK via NuGet
- `install-go.sh` - Install Go SDK via go get

Validation scripts verify installation and dependencies:

- `validate-python.sh` - Check Python SDK installation
- `validate-typescript.sh` - Check TypeScript SDK installation
- `validate-java.sh` - Check Java SDK installation
- `validate-csharp.sh` - Check C# SDK installation
- `validate-go.sh` - Check Go SDK installation

## Configuration Templates

Templates are in `templates/` directory:

**Environment Setup:**
- `env-template.txt` - Environment variable template (all languages)
- `python-config.py` - Python configuration example
- `typescript-config.ts` - TypeScript configuration example
- `java-config.xml` - Java Maven configuration
- `csharp-config.csproj` - C# project configuration
- `go-mod.txt` - Go module configuration

**Authentication:**
- `auth-api-key-template.txt` - API key authentication
- `auth-oauth-template.txt` - OAuth authentication
- `auth-jwt-template.txt` - JWT authentication

## Common Setup Patterns

### Environment Variables

All SDKs use environment variables for configuration:

```bash
# Required
A2A_API_KEY=your_api_key_here
A2A_BASE_URL=https://api.a2a.example.com

# Optional
A2A_TIMEOUT=30
A2A_RETRY_ATTEMPTS=3
A2A_LOG_LEVEL=info
```

**CRITICAL:** Always use placeholders in committed files. Create `.env.example` with placeholder values only.

### Authentication Setup

All SDKs support three authentication methods:

1. **API Key** - Simplest, for server-to-server
2. **OAuth 2.0** - For user-delegated access
3. **JWT** - For service-to-service with custom claims

See `templates/auth-*-template.txt` for implementation patterns.

### Error Handling

All SDKs provide consistent error handling:

- `A2AConnectionError` - Network/connectivity issues
- `A2AAuthenticationError` - Invalid credentials
- `A2ARateLimitError` - Rate limit exceeded
- `A2AValidationError` - Invalid request data

See `examples/error-handling-*.md` for language-specific patterns.

## Language-Specific Considerations

### Python

- Requires Python 3.8+
- Install via pip: `pip install a2a-protocol`
- Async support via asyncio
- Type hints available
- See `examples/python-basic.py`

### TypeScript

- Requires Node 18+
- Install via npm: `npm install @a2a/protocol`
- Full TypeScript definitions included
- Promise-based async/await
- See `examples/typescript-basic.ts`

### Java

- Requires Java 11+
- Maven: Add to pom.xml
- Gradle: Add to build.gradle
- Thread-safe client
- See `examples/java-basic.java`

### C#

- Requires .NET 6+
- NuGet: `dotnet add package A2A.Protocol`
- Async/await support
- Dependency injection ready
- See `examples/csharp-basic.cs`

### Go

- Requires Go 1.20+
- Install: `go get github.com/a2a/protocol-go`
- Context-aware operations
- Goroutine-safe
- See `examples/go-basic.go`

## Troubleshooting

### Installation Issues

**Package not found:**
```bash
# Python
pip install --upgrade pip
pip install a2a-protocol

# TypeScript
npm cache clean --force
npm install @a2a/protocol

# Java
mvn clean install -U

# C#
dotnet restore --force

# Go
go clean -modcache
go get -u github.com/a2a/protocol-go
```

**Version conflicts:**
Run the appropriate validation script to check dependencies:
```bash
./scripts/validate-<language>.sh
```

### Authentication Errors

1. Check environment variables are set
2. Verify API key format (no extra spaces/newlines)
3. Ensure base URL is correct
4. Check API key permissions

### Connection Issues

1. Verify network connectivity
2. Check firewall/proxy settings
3. Validate base URL is accessible
4. Review timeout settings

## Security Best Practices

**Environment Variables:**
- NEVER commit actual API keys
- Use `.env` files (add to `.gitignore`)
- Create `.env.example` with placeholders
- Use secret management in production (Vault, AWS Secrets Manager, etc.)

**API Keys:**
- Rotate keys regularly
- Use different keys for dev/staging/prod
- Implement key expiration
- Monitor key usage

**Network Security:**
- Always use HTTPS
- Validate SSL certificates
- Implement request signing for sensitive operations
- Use VPN/private networks for production

## Examples

Complete examples for each language:

- `examples/python-basic.py` - Basic Python usage
- `examples/python-async.py` - Async Python usage
- `examples/typescript-basic.ts` - Basic TypeScript usage
- `examples/java-basic.java` - Basic Java usage
- `examples/csharp-basic.cs` - Basic C# usage
- `examples/go-basic.go` - Basic Go usage
- `examples/error-handling-python.md` - Python error handling
- `examples/error-handling-typescript.md` - TypeScript error handling
- `examples/error-handling-java.md` - Java error handling

## Resources

**Official Documentation:**
- Python SDK: https://docs.a2a-protocol.org/python
- TypeScript SDK: https://docs.a2a-protocol.org/typescript
- Java SDK: https://docs.a2a-protocol.org/java
- C# SDK: https://docs.a2a-protocol.org/csharp
- Go SDK: https://docs.a2a-protocol.org/go

**GitHub Repositories:**
- Python: https://github.com/a2a/protocol-python
- TypeScript: https://github.com/a2a/protocol-ts
- Java: https://github.com/a2a/protocol-java
- C#: https://github.com/a2a/protocol-dotnet
- Go: https://github.com/a2a/protocol-go

---

**Version:** 1.0.0
**Protocol Compatibility:** A2A Protocol 1.0+
