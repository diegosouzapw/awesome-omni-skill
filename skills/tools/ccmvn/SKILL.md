---
name: ccmvn
description: Run Maven builds using the ccmvn tool, which provides Maven proxy support for sandboxed environments. Use this skill when you need to run Maven commands like clean install, package, test, or compile.
argument-hint: "[maven-command] [options]"
user-invocable: true
---

# Maven Build with ccmvn

The `ccmvn` tool enables Maven to run in restricted sandbox environments where outbound HTTPS is blocked or DNS resolution is unavailable. It works by creating a local HTTP proxy that Maven uses to access Maven Central.

## When to Use This Skill

- Running Maven builds (`clean install`, `package`, `test`, `compile`, etc.)
- Building Java projects that depend on Maven Central
- Working in sandboxed environments with network restrictions
- Needing to execute Maven with custom parameters and options

## Prerequisites

Before using this skill, ensure:

1. **Maven is installed**: The system must have Maven 3.x installed
2. **Node.js is available**: Required for the proxy server (typically pre-installed in Claude Code)
3. **Proxy configuration** (for sandboxed Claude Code): The `HTTPS_PROXY` environment variable should be set if in a restricted sandbox

## Basic Usage

### Simple Build

\`\`\`bash
npx ccmvn clean install
\`\`\`

### Other Common Commands

\`\`\`bash
# Package the project
npx ccmvn package

# Run tests
npx ccmvn test

# Compile only
npx ccmvn compile

# Custom parameters
npx ccmvn clean install -DskipTests
npx ccmvn clean install -X  # Debug output
\`\`\`

## How ccmvn Works

ccmvn creates a three-layer proxy chain:

\`\`\`
Maven → Local Proxy → Upstream Sandbox Proxy → Maven Central
       (HTTP)        (HTTP CONNECT)         (HTTP/2 over TLS)
\`\`\`

### Architecture Layers

1. **Maven → Local Proxy** (localhost:8080)
   - Plain HTTP communication
   - Maven configured via \`~/.m2/settings.xml\`
   - No authentication required locally

2. **Local Proxy → Upstream Proxy**
   - HTTP CONNECT tunneling with JWT authentication
   - Upstream proxy performs DNS resolution for sandboxed environments

3. **Local Proxy → Maven Central**
   - HTTP/2 over TLS with ALPN negotiation
   - Secure connection to Maven Central repositories

### Automatic Configuration

ccmvn automatically:
- Creates \`~/.m2/settings.xml\` if it doesn't exist
- Configures Maven to use the local proxy as a mirror for Maven Central
- Starts the proxy server before running Maven
- Cleans up the proxy server after Maven completes

## Common Maven Commands

| Command | Purpose |
|---------|---------|
| \`ccmvn clean\` | Remove build artifacts |
| \`ccmvn compile\` | Compile source code |
| \`ccmvn test\` | Run tests |
| \`ccmvn package\` | Create JAR/WAR file |
| \`ccmvn install\` | Install to local repository |
| \`ccmvn deploy\` | Deploy to remote repository |
| \`ccmvn clean install\` | Full clean build and install |

## Advanced Options

### Skip Tests During Build

\`\`\`bash
npx ccmvn clean install -DskipTests
\`\`\`

### Run with Debug Output

\`\`\`bash
npx ccmvn clean install -X
\`\`\`

### Specify Specific Goals

\`\`\`bash
npx ccmvn clean compile test-compile
\`\`\`

### Use Custom Settings File

Maven will automatically use \`~/.m2/settings.xml\`. To verify or modify proxy settings, check this file after running ccmvn.

## Troubleshooting

### Proxy Connection Issues

If you see proxy-related errors:
1. Ensure \`HTTPS_PROXY\` environment variable is correctly set (if in a sandboxed environment)
2. Check that the proxy server can reach Maven Central
3. Verify your proxy credentials are correct

### Maven Not Found

If Maven is not installed:
\`\`\`bash
# Install Maven (varies by system)
brew install maven  # macOS
apt-get install maven  # Ubuntu/Debian
\`\`\`

### Maven Central Timeout

If builds timeout accessing Maven Central:
1. Check your network connection
2. Try again - temporary network issues can cause timeouts
3. Use \`-X\` flag for debug output to see where it's hanging

### Settings.xml Issues

ccmvn automatically creates \`~/.m2/settings.xml\` if missing. If you have a custom settings file:
1. Backup your current \`~/.m2/settings.xml\`
2. Remove it and let ccmvn create a fresh one
3. Or manually add the proxy mirror configuration

## Examples

### Building a Spring Boot Application

\`\`\`bash
npx ccmvn clean package -DskipTests
\`\`\`

### Running Unit Tests

\`\`\`bash
npx ccmvn clean test
\`\`\`

### Full Build with All Checks

\`\`\`bash
npx ccmvn clean install
\`\`\`

### Building Specific Module (in multi-module project)

\`\`\`bash
npx ccmvn clean install -pl module-name
\`\`\`

## Environment Variables

### HTTPS_PROXY (Sandboxed Environments Only)

Format: \`http://username:jwt_token@host:port\`

This is automatically used by ccmvn for sandboxed Claude Code environments. In local development, this is typically not needed.

## Security

- **Local proxy only**: The proxy server is only accessible on localhost
- **TLS validation**: TLS certificates are validated for Maven Central
- **End-to-end encryption**: Connection through the proxy chain maintains encryption
- **Token protection**: JWT tokens are only used for authentication to the upstream proxy

## Performance Tips

1. **First build**: Initial builds will download dependencies - this takes longer
2. **Incremental builds**: Subsequent builds with cached dependencies are much faster
3. **Skip tests if not needed**: Use \`-DskipTests\` to speed up builds during development
4. **Use clean install sparingly**: Only use \`clean\` when you have problems or need a fresh build

## Resources

- [ccmvn on npm](https://www.npmjs.com/package/ccmvn)
- [ccmvn on GitHub](https://github.com/ctalau/ccmvn)
- [Maven Documentation](https://maven.apache.org/guides/)
- [Maven Central Repository](https://repo.maven.apache.org/maven2/)
