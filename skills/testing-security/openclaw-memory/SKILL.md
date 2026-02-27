---
name: openclaw-memory
description: Production-grade memory management for OpenClaw. Search, compress, encrypt, authenticate, rate limit, and audit OpenClaw memory (MEMORY.md, AGENTS.md, memory/) with enterprise-grade security.
version: 1.2.0
---

# OpenClaw Memory Management Skill

> 🧠 Search, compress, encrypt, authenticate, rate limit, and audit OpenClaw memory (MEMORY.md, AGENTS.md, memory/) with enterprise-grade security

**Version**: 1.2.0 - ENHANCED SECURITY
**Author**: Pi-Agent 🐺📿
**License**: MIT

> ⚠️ **Security**: This skill includes 11 defense layers against command injection, path traversal, prompt injection, DoS attacks, and unauthorized access.

---

## Quick Start

### Basic Usage

```bash
cd ~/pi-mono-workspace/skills/openclaw-memory

# Search memory
./openclaw-memory.sh search "trading strategies"

# View statistics
./openaw-memory.sh stats

# List recent entries
./openclaw-memory.sh recent
```

### Encryption (V1.2.0)

```bash
# Generate encryption key
./openclaw-memory.sh key generate

# Encrypt a file
./openclaw-memory.sh encrypt MEMORY.md

# Decrypt a file
./openclaw-memory.sh decrypt MEMORY.md.enc
```

### Authentication (V1.2.0)

```bash
# Initialize authentication
./openclaw-memory.sh auth init

# Add a user
./openclaw-memory.sh auth add-user alice secret123

# Enable authentication
export OPENCLAW_AUTH=true
export OPENCLAW_API_KEY="ocm_abc123..."
```

### Rate Limiting (V1.2.0)

```bash
# Initialize rate limiting
./openclaw-memory.sh rate-limit init

# Enable rate limiting
export OPENCLAW_RATE_LIMIT=true

# Check rate limit status
./openclaw-memory.sh rate-limit status
```

---

## Commands

### Memory Commands

| Command | Description |
|---------|-------------|
| `search <query>` | Search MEMORY.md and memory/*.md for content |
| `compress [level]` | Compress conversation history (default: level 1) |
| `stats` | Show memory statistics |
| `agents` | List all agents and their roles |
| `recent [n]` | Show recent memory entries (default: 5) |
| `clean` | Remove stale memory files (>90 days old) |

### Encryption Commands (V1.2.0)

| Command | Description |
|---------|-------------|
| `key generate` | Generate encryption key |
| `encrypt <file>` | Encrypt a memory file |
| `decrypt <file>` | Decrypt a memory file |
| `key list` | List encrypted files |

### Authentication Commands (V1.2.0)

| Command | Description |
|---------|-------------|
| `auth init` | Initialize authentication system |
| `auth add-user <user> <pass>` | Add a new user |
| `auth remove-user <user>` | Remove a user |
| `auth list` | List all users |
| `auth status` | Show authentication status |
| `auth clean-sessions` | Clean expired sessions |

### Rate Limiting Commands (V1.2.0)

| Command | Description |
|---------|-------------|
| `rate-limit init` | Initialize rate limiting |
| `rate-limit check` | Check rate limit (consume token) |
| `rate-limit status` | Get rate limit status |
| `rate-limit stats` | Show rate limit statistics |
| `rate-limit reset [client]` | Reset rate limit for client |
| `rate-limit cleanup` | Clean old client data |

---

## Features

### Memory Management
- 🔍 **Memory Search**: Search across MEMORY.md and memory/*.md files
- 🗜 **Memory Compression**: Compress old conversation history (3 levels)
- 📊 **Memory Statistics**: View memory usage and file counts
- 🤖 **Agent Listing**: Display AGENTS.md contents
- 📅 **Recent Entries**: Show recently added memory files
- 🧹 **Clean Old**: Remove stale memory files (>90 days)

### Security (V1.2.0)
- 🔒 **AES-256-GCM Encryption**: Military-grade encryption for sensitive files
- 🔐 **User Authentication**: API key and session-based access control
- 🚦 **Token Bucket Rate Limiting**: Production-grade rate limiting
- 🔍 **Permission Auditing**: Security audit with auto-fix capabilities
- 🛡️ **Security Hardened**: Protection against injection attacks, path traversal, DoS

---

## Examples

### Search for trading strategies

```bash
./openclaw-memory.sh search "trading strategies"
```

### Compress memory (Level 1)

```bash
./openclaw-memory.sh compress 1
```

### Encrypt sensitive files

```bash
./openclaw-memory.sh key generate
./openclaw-memory.sh encrypt MEMORY.md
```

### View memory statistics

```bash
./openclaw-memory.sh stats
```

### Run security audit

```bash
./openclaw-memory.sh audit
```

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE` | Current directory | Path to OpenClaw workspace |
| `OPENCLAW_AUTH` | false | Enable authentication |
| `OPENCLAW_API_KEY` | - | API key for authentication |
| `OPENCLAW_SESSION` | - | Session token for authentication |
| `OPENCLAW_RATE_LIMIT` | false | Enable rate limiting |
| `OPENCLAW_CLIENT_ID` | hostname:pid | Client identifier for rate limiting |

---

## Best Practices

### Memory Organization

1. **Durable Facts** → Add to `MEMORY.md`
   - User preferences
   - Important decisions
   - System configurations
   - Long-term goals

2. **Daily Logs** → Create `memory/YYYY-MM-DD.md`
   - Daily activities
   - Session summaries
   - Short-term observations

3. **Agent Definitions** → Edit `AGENTS.md`
   - Agent roles
   - Sub-agent capabilities
   - Tool configurations

### Compression Schedule

```bash
# Weekly: Level 1 compression
0 0 * * 0 openclaw-memory.sh compress 1

# Monthly: Level 2 compression
0 0 1 * * openclaw-memory.sh compress 2

# Quarterly: Level 3 compression (archive)
0 0 1 1,4,7,10 * openclaw-memory.sh compress 3
```

---

## Security

The skill includes 11 defense layers:

| Threat | Mitigation |
|--------|------------|
| Command Injection | Input validation, dangerous character filtering |
| Path Traversal | Path resolution, symlink checks, whitelist |
| Option Injection | Fixed strings, `--` delimiter |
| Denial of Service | Operation limits (max results, max files) |
| Prompt Injection | Input sanitization, no AI processing |
| Unauthorized Access | AES-256-GCM encryption, authentication |

### Security Logging

All security events are logged to `/tmp/openclaw-memory.log`:
- Command executions
- Input validation failures
- Path traversal attempts
- Dangerous character detection
- Authentication attempts
- Rate limit violations

---

## Support

- **OpenClaw Discord**: https://discord.gg/clawd
- **OpenClaw Docs**: https://docs.openclaw.ai

---

**Built by**: Pi-Agent 🐺📿
