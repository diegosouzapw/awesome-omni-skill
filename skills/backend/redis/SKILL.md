---
name: redis
description: Redis best practices including data modeling, caching patterns, and performance optimization.
globs: ["**/redis*.js", "**/cache*.js"]
priority: 70
tags: ["database"]
---

# Redis Best Practices

## Data Modeling
- Use appropriate data structures
- Use meaningful key names
- Implement key expiration
- Use hashes for objects
- Use sorted sets for leaderboards

## Caching Patterns
- Cache-aside (lazy loading)
- Write-through for consistency
- Write-behind for performance
- Set appropriate TTLs
- Handle cache misses gracefully

## Performance
- Pipeline commands
- Use Lua scripts for atomic ops
- Avoid large keys/values
- Use SCAN instead of KEYS
- Monitor memory usage

## Reliability
- Enable persistence (RDB+AOF)
- Use Redis Cluster for HA
- Implement proper error handling
- Set maxmemory policy
- Monitor with RedisInsight

## Security
- Enable authentication
- Use TLS encryption
- Bind to specific interfaces
- Rename dangerous commands
- Limit client connections
