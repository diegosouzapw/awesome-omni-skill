---
name: nodejs
description: Node.js server development patterns including async patterns, error handling, and security best practices.
globs: ["**/package.json", "**/tsconfig.json", "**/server.js", "**/index.js", "**/app.js"]
priority: 90
tags: ["tool"]
---

# Node.js Best Practices

## Project Structure
- src/ for source code
- src/lib/ for utilities
- src/types/ for TypeScript types
- tests/ for test files
- Keep entry point minimal

## Async Patterns
- Use async/await everywhere
- Handle all promise rejections
- Use Promise.all for parallel tasks
- Use Promise.allSettled when all must complete
- Set timeouts for external calls
- Use AbortController for cancellation

## Error Handling
- Use centralized error handler
- Create custom error classes
- Log errors with context (request ID, user)
- Return appropriate status codes
- Never expose stack traces in production

## Environment
- Use dotenv for local config
- Validate env variables at startup (envalid)
- Use different configs per environment
- Never commit secrets
- Use secret managers in production

## Security
- Validate all input (Zod, Joi)
- Use helmet for HTTP headers
- Rate limit APIs (express-rate-limit)
- Keep dependencies updated (npm audit)
- Use npm/yarn lockfiles

## Performance
- Use clustering for multi-core
- Enable gzip compression
- Implement caching (Redis)
- Use connection pooling
- Profile with clinic.js
