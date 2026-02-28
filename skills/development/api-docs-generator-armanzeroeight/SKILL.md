---
name: api-docs-generator
description: Generates API documentation from code including OpenAPI specs, JSDoc, and Python docstrings. Use when documenting APIs, REST endpoints, or library functions.
---

# API Documentation Generator

## Quick Start

Generate API docs based on code type:

```bash
# OpenAPI from Express routes
npx swagger-jsdoc -d swaggerDef.js routes/*.js

# JSDoc for JavaScript
npx jsdoc src/ -d docs/

# Python docstrings
pdoc --html --output-dir docs/ mypackage/
```

## Instructions

### Step 1: Identify API Type

Determine documentation approach:

| API Type | Tool | Output |
|----------|------|--------|
| REST API | OpenAPI/Swagger | Interactive API docs |
| GraphQL | GraphQL Schema | Schema documentation |
| JavaScript Library | JSDoc | HTML reference |
| Python Library | Sphinx/pdoc | HTML reference |

### Step 2: Extract Documentation

**REST API (OpenAPI):**

Scan route files for JSDoc comments:
```javascript
/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     responses:
 *       200:
 *         description: List of users
 */
router.get('/users', getUsers);
```

Generate spec:
```bash
npx swagger-jsdoc -d swaggerDef.js routes/*.js -o openapi.json
```

**JavaScript Library (JSDoc):**

```javascript
/**
 * Adds two numbers together.
 * @param {number} a - First number
 * @param {number} b - Second number
 * @returns {number} Sum of a and b
 * @example
 * add(2, 3); // returns 5
 */
function add(a, b) {
  return a + b;
}
```

**Python (Docstrings):**

```python
def add(a: int, b: int) -> int:
    """Add two numbers together.
    
    Args:
        a: First number
        b: Second number
        
    Returns:
        Sum of a and b
        
    Example:
        >>> add(2, 3)
        5
    """
    return a + b
```

### Step 3: Generate Documentation

**OpenAPI/Swagger:**
```bash
# Generate OpenAPI spec
npx swagger-jsdoc -d swaggerDef.js routes/*.js -o openapi.json

# Serve interactive docs
npx swagger-ui-express openapi.json
```

**JSDoc:**
```bash
npx jsdoc src/ -d docs/ -r
```

**Python Sphinx:**
```bash
sphinx-apidoc -o docs/source mypackage/
cd docs && make html
```

**Python pdoc:**
```bash
pdoc --html --output-dir docs/ mypackage/
```

### Step 4: Organize Output

Structure documentation:
```
docs/
├── api/
│   ├── openapi.json       # OpenAPI specification
│   ├── index.html         # Interactive API docs
│   └── endpoints/         # Endpoint details
├── reference/
│   ├── classes/           # Class documentation
│   ├── functions/         # Function documentation
│   └── types/             # Type definitions
└── guides/
    ├── authentication.md  # Auth guide
    └── examples.md        # Usage examples
```

### Step 5: Add Examples

Include practical examples:

```markdown
## Authentication

All API requests require authentication:

\`\`\`bash
curl -H "Authorization: Bearer TOKEN" \\
  https://api.example.com/v1/users
\`\`\`

\`\`\`javascript
const response = await fetch('https://api.example.com/v1/users', {
  headers: { 'Authorization': 'Bearer TOKEN' }
});
\`\`\`

\`\`\`python
import requests

response = requests.get(
    'https://api.example.com/v1/users',
    headers={'Authorization': 'Bearer TOKEN'}
)
\`\`\`
```

## OpenAPI Specification

### Basic Structure

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API description

servers:
  - url: https://api.example.com/v1

paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

## Documentation Checklist

- [ ] All endpoints documented
- [ ] Request/response examples included
- [ ] Authentication explained
- [ ] Error codes documented
- [ ] Rate limiting described
- [ ] Code examples in multiple languages
- [ ] Interactive API explorer available
