---
name: express
description: Express.js server best practices including middleware, error handling, and security.
globs: ["**/server.js", "**/app.js", "**/routes/**/*.js", "**/middleware/**/*.js"]
priority: 85
tags: ["framework"]
---

# Express.js Best Practices

## Project Structure
- src/routes/ for route handlers
- src/middleware/ for middleware
- src/controllers/ for business logic
- src/models/ for data models
- src/utils/ for utilities

## Middleware
- Use helmet for security headers
- Use cors for CORS handling
- Use morgan for logging
- Use compression for gzip
- Use express.json() for body parsing
- Order middleware correctly

## Error Handling
- Use async error wrapper or express-async-errors
- Centralized error handler middleware
- Send appropriate status codes
- Log errors with context
- Never expose stack traces in production

## Security
- Validate all inputs with Joi/Zod
- Sanitize user data
- Use rate limiting
- Implement CSRF protection
- Use secure session config

## Performance
- Use caching (Redis)
- Implement pagination
- Use streaming for large responses
- Enable gzip compression
- Use connection pooling for databases
