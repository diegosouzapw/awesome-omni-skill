---
name: managing-traefik
description: Manages Traefik reverse proxy for local development. Use when routing domains to local services, configuring CORS, checking service health, or debugging connectivity issues.
---

# Traefik Hub MCP

Local development reverse proxy management.

## Quick Start

**Generate Docker labels for a new service?**
```
generate_labels name="myapp" domain="myapp.localhost" port=3000
```
Returns docker-compose label configuration to copy into your project.

**Something not working?**
```
doctor
```
Comprehensive health check with actionable tips.

## Tool Categories

### Project Setup
| Tool | Use When |
|------|----------|
| `generate_labels` | Creating docker-compose labels for Docker services |

### CORS Management
| Tool | Use When |
|------|----------|
| `get_cors` | Checking current allowed origins |
| `update_cors` | Adding/removing CORS origins |

### Health & Debugging
| Tool | Use When |
|------|----------|
| `doctor` | First step for any problem (comprehensive check) |
| `check_health` | Testing if specific domain responds |
| `check_setup` | Verifying MCP configuration |

### Inspection
| Tool | Use When |
|------|----------|
| `list_routers` | Seeing all configured routes |
| `list_services` | Seeing all backend services |
| `list_middlewares` | Seeing available middlewares |
| `get_router` | Getting details of specific route |

### Stack Control
| Tool | Use When |
|------|----------|
| `init_stack` | First time setup (creates network + starts Traefik) |
| `start_traefik` | Starting stopped Traefik |
| `stop_traefik` | Stopping Traefik |
| `create_network` | Creating traefik-public network only |

## Common Workflows

### New Project Setup
```
1. generate_labels name="myapp" domain="myapp.localhost" port=3000
2. update_cors add=["http://myapp.localhost"]
```
Add the generated labels to your project's docker-compose.yml, then update CORS if needed.

### Debug Service Not Reachable
```
1. doctor                           # Check all components
2. check_health domain="X.localhost" # Test specific domain
3. list_routers                     # Verify route exists
4. container_logs name="traefik"    # Check Traefik logs
```

### Add CORS Origin
```
1. get_cors                         # See current origins
2. update_cors add=["http://newapp.localhost"]
```

### Generate Docker Labels
```
generate_labels name="myapp" domain="myapp.localhost" port=3000
```
Returns docker-compose label configuration to copy into your project.

## Decision Guide

| Situation | Tool |
|-----------|------|
| First time ever | `init_stack` |
| New Docker service | `generate_labels` |
| CORS errors | `get_cors` → `update_cors` |
| Service unreachable | `doctor` |
| Check specific domain | `check_health` |

## Available Middlewares

Reference with `@file` suffix (e.g., `cors-dev@file`):
- `cors-dev` - CORS headers for local development
- `secure-headers` - Security headers
- `rate-limit` - Rate limiting (100 req/s)
- `strip-api-prefix` - Strip `/api` prefix from path
