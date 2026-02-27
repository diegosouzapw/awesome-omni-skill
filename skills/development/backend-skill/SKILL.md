---
name: backend-skill
description: Design and implement backend functionality including route generation, request/response handling, and database connectivity.
---

# Backend Skill – API & Database Development

## Instructions

1. **Project setup**
   - Initialize backend project (Node.js / Express / Fastify / Django / Flask)
   - Configure environment variables
   - Follow clean folder structure (routes, controllers, services, models)

2. **Route generation**
   - Define RESTful routes (GET, POST, PUT, DELETE)
   - Use proper URL naming conventions
   - Apply versioning (e.g. `/api/v1/`)
   - Separate routes by resource

3. **Request & response handling**
   - Validate incoming requests
   - Handle query params, route params, and body data
   - Return consistent JSON responses
   - Use proper HTTP status codes
   - Implement centralized error handling

4. **Database connection**
   - Connect to database (MongoDB / PostgreSQL / MySQL)
   - Define schemas or models
   - Perform CRUD operations
   - Handle connection errors gracefully
   - Use migrations or schema versioning

5. **Security & middleware**
   - Use authentication middleware (JWT / sessions)
   - Protect routes with authorization
   - Sanitize inputs to prevent injection attacks
   - Enable CORS and rate limiting

6. **Testing & debugging**
   - Test endpoints using Postman or automated tests
   - Log requests and errors
   - Handle edge cases and failures

## Best Practices
- Keep routes thin, move logic to services
- Use async/await with proper error handling
- Return meaningful error messages
- Follow REST standards
- Write reusable and modular code
- Never expose secrets in code

## Example Structure
```txt
src/
 ├─ routes/
 │   └─ user.routes.js
 ├─ controllers/
 │   └─ user.controller.js
 ├─ services/
 │   └─ user.service.js
 ├─ models/
 │   └─ user.model.js
 ├─ middleware/
 │   └─ auth.middleware.js
 └─ app.js
