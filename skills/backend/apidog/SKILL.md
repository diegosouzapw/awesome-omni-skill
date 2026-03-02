---
name: apidog
description: Apidog API development platform documentation - API design, testing, mocking, and documentation
---

# Apidog Skill

Comprehensive assistance with Apidog - an all-in-one API development platform for designing, testing, mocking, and documenting APIs.

## When to Use This Skill

This skill should be triggered when:
- Designing RESTful APIs or defining endpoints (OpenAPI/Swagger specs)
- Creating API test scenarios and automated testing workflows
- Setting up API mocking for frontend development or testing
- Migrating from Postman, Insomnia, or other API tools
- Working with API documentation generation and sharing
- Debugging API requests and responses
- Managing environments and variables in API workflows
- Writing pre/post processors or test scripts
- Implementing API security schemes (OAuth, API keys, JWT)
- Generating code from API specifications
- Working with schemas and data models (JSON Schema)
- Database operations in API testing (MySQL, MongoDB, Redis)

## Quick Reference

### Basic API Endpoint Design

```json
{
  "method": "POST",
  "path": "/api/users",
  "parameters": {
    "body": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "email": { "type": "string", "format": "email" }
      },
      "required": ["name", "email"]
    }
  },
  "responses": {
    "201": {
      "description": "User created",
      "body": {
        "type": "object",
        "properties": {
          "id": { "type": "integer" },
          "name": { "type": "string" },
          "email": { "type": "string" }
        }
      }
    }
  }
}
```

### Using Variables in Requests

```javascript
// Environment variables
{{base_url}}/api/users/{{user_id}}

// Dynamic values
{
  "timestamp": "{{$timestamp}}",
  "uuid": "{{$guid}}",
  "random_email": "{{$randomEmail}}",
  "random_int": "{{$randomInt}}"
}
```

### Post-Processor Script for Extracting Data

```javascript
// Extract token from response
const response = pm.response.json();
pm.environment.set("auth_token", response.data.token);

// Extract user ID
pm.environment.set("user_id", response.data.user.id);

// Assert response status
pm.test("Status is 200", function() {
  pm.response.to.have.status(200);
});
```

### Basic Assertion Examples

```javascript
// Status code assertion
pm.test("Status code is 200", function() {
  pm.response.to.have.status(200);
});

// Response body validation
pm.test("Response has user data", function() {
  const jsonData = pm.response.json();
  pm.expect(jsonData).to.have.property('user');
  pm.expect(jsonData.user).to.have.property('email');
});

// Response time check
pm.test("Response time is less than 500ms", function() {
  pm.expect(pm.response.responseTime).to.be.below(500);
});
```

### Setting Up Mock Data

```javascript
// Smart Mock - Automatically generates data based on field names
{
  "id": "@integer(1, 1000)",
  "name": "@name",
  "email": "@email",
  "avatar": "@image('200x200')",
  "created_at": "@datetime"
}

// Custom Mock with conditions
{
  "status": "@pick(['active', 'pending', 'disabled'])",
  "age": "@integer(18, 65)",
  "balance": "@float(0, 10000, 2, 2)"
}
```

### Pre-Processor Script for Dynamic Headers

```javascript
// Generate timestamp-based signature
const timestamp = Date.now();
const signature = CryptoJS.MD5(timestamp + "secret_key").toString();

pm.request.headers.add({
  key: "X-Timestamp",
  value: timestamp.toString()
});

pm.request.headers.add({
  key: "X-Signature",
  value: signature
});
```

### Test Scenario with Data Passing

```javascript
// Step 1: Login and save token
// POST /api/login
pm.test("Login successful", function() {
  const response = pm.response.json();
  pm.environment.set("token", response.access_token);
});

// Step 2: Use token in next request
// GET /api/profile
// Headers: Authorization: Bearer {{token}}

// Step 3: Extract and use profile data
pm.test("Profile fetched", function() {
  const profile = pm.response.json();
  pm.environment.set("user_name", profile.name);
});
```

### Database Operation in Tests

```javascript
// MySQL query in pre-processor
const db = require('db');
const result = db.query('SELECT * FROM users WHERE email = ?', [pm.environment.get('email')]);
pm.environment.set('user_id', result[0].id);
```

### Schema Composition (oneOf, allOf, anyOf)

```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "type": { "const": "credit_card" },
        "card_number": { "type": "string" }
      }
    },
    {
      "type": "object",
      "properties": {
        "type": { "const": "bank_account" },
        "account_number": { "type": "string" }
      }
    }
  ]
}
```

### Security Scheme Configuration

```json
{
  "securitySchemes": {
    "BearerAuth": {
      "type": "http",
      "scheme": "bearer",
      "bearerFormat": "JWT"
    },
    "ApiKeyAuth": {
      "type": "apiKey",
      "in": "header",
      "name": "X-API-Key"
    }
  }
}
```

## Key Concepts

### Design-First vs Request-First Mode
- **Design-First**: Define API specifications first, then implement and test
- **Request-First**: Send requests directly, then optionally save as endpoints
- Apidog supports both workflows seamlessly

### Schemas and Components
- **Schemas**: Reusable data models (JSON Schema format)
- **Components**: Shared parameters, headers, responses across endpoints
- Enables DRY principles in API design

### Smart Mock
Automatically generates realistic mock data based on:
- Field names (email, phone, address, etc.)
- Data types (string, number, boolean)
- Formats (date-time, uuid, url)
- Custom mock rules and priorities

### Pre/Post Processors
- **Pre-processors**: Run before request (setup, auth, dynamic data)
- **Post-processors**: Run after response (assertions, data extraction, validation)
- Support scripts, database operations, variable extraction, assertions

### Test Scenarios
- Chain multiple API requests together
- Pass data between requests using variables
- Add conditional logic and loops
- Generate comprehensive test reports

### Environments & Variables
- **Environment variables**: Configuration per environment (dev, staging, prod)
- **Global variables**: Shared across all environments
- **Temporary variables**: Exist only during test scenario execution
- **Vault secrets**: Secure storage (HashiCorp, Azure, AWS integration)

## Reference Files

This skill includes comprehensive documentation in `references/`:

- **api.md** - Complete Apidog documentation including:
  - **Getting Started**: Introduction, navigation, basic concepts, quick start guides
  - **Migration**: Import from Postman, Insomnia, Swagger/OpenAPI, cURL, HAR files
  - **API Design**: Endpoint specifications, schemas, components, security schemes, modules
  - **Development & Debugging**: Requests, responses, environments, variables, code generation
  - **Pre/Post Processors**: Assertions, scripts, database operations, variable extraction
  - **Mock Data**: Smart mock, custom mock, mock scripts, cloud vs self-hosted
  - **Testing**: Test scenarios, automation, reports, data passing between requests
  - **Dynamic Values Modules**: Faker.js integration for realistic test data (50+ modules)
  - **Collaboration**: Team management, permissions, version control
  - **Documentation**: Auto-generation, sharing, customization
  - **Integration**: CI/CD, webhooks, CLI, API

Use the reference file when you need:
- Detailed feature explanations
- Advanced configuration options
- Migration guides and import procedures
- Comprehensive scripting examples
- Database integration patterns
- Complete dynamic values reference

## Working with This Skill

### For Beginners
Start with these core concepts:
1. **Creating your first endpoint**: Define method, path, parameters, and responses
2. **Making requests**: Use the request panel to test endpoints
3. **Basic assertions**: Validate status codes and response bodies
4. **Environments**: Set up dev/staging/prod configurations
5. **Mock data**: Enable mock server for frontend development

Refer to the Quick Start section in `references/api.md` for step-by-step tutorials.

### For Intermediate Users
Focus on:
1. **Schema design**: Create reusable data models with JSON Schema
2. **Test scenarios**: Chain requests and pass data between them
3. **Pre/Post processors**: Add custom logic with scripts
4. **Dynamic values**: Use Faker.js modules for realistic test data
5. **Security schemes**: Implement OAuth, JWT, API key authentication

### For Advanced Users
Explore:
1. **Database operations**: Query MySQL/MongoDB/Redis in tests
2. **Custom scripts**: Use JavaScript libraries and external languages
3. **Schema composition**: Leverage oneOf, allOf, anyOf for complex models
4. **CI/CD integration**: Automate testing in pipelines
5. **Team collaboration**: Version control, branches, merge conflicts
6. **API documentation**: Customize and publish interactive docs

### For Migration
If coming from other tools:
- **Postman users**: Import collections, environments, and globals
- **Insomnia users**: Import workspaces and requests
- **Swagger users**: Import OpenAPI 2.0/3.0 specifications
- **cURL users**: Paste cURL commands directly
- Check `references/api.md` Migration section for detailed guides

### Navigation Tips
- Use Ctrl/Cmd+K for quick search across documentation
- Check the Table of Contents in api.md for specific topics
- Dynamic Values Modules section lists all available Faker.js functions
- Script examples section shows common patterns for assertions and data manipulation

## Common Workflows

### Workflow 1: Design → Mock → Test → Document
1. Design API endpoints with schemas
2. Enable smart mock for realistic data
3. Write test scenarios with assertions
4. Auto-generate and share documentation

### Workflow 2: Import → Enhance → Automate
1. Import from Postman/Swagger
2. Add schemas and components for reusability
3. Create test scenarios with data passing
4. Integrate with CI/CD pipeline

### Workflow 3: Request-First Development
1. Send ad-hoc requests to explore APIs
2. Save successful requests as endpoints
3. Add schemas from response bodies
4. Build test scenarios from saved requests

## Resources

### Official Documentation
All content is sourced from https://docs.apidog.com/

### Dynamic Values Reference
Apidog includes 50+ Faker.js modules for test data:
- Person (names, emails, phones)
- Internet (URLs, IPs, domains)
- Commerce (products, prices)
- Date/Time (timestamps, future/past dates)
- Location (addresses, coordinates)
- Finance (credit cards, transactions)
- And many more - see Dynamic Values Modules in api.md

### Database Support
- MySQL / PostgreSQL
- MongoDB
- Redis
- Oracle

### Scripting Languages
- JavaScript (primary)
- Python, PHP, Go, Java (via external execution)

### Import/Export Formats
- OpenAPI 2.0 / 3.0
- Postman Collection v1/v2
- Insomnia v4
- HAR (HTTP Archive)
- cURL
- WSDL
- Markdown

## Notes

- This skill was automatically generated from official Apidog documentation
- Code examples preserve syntax from source docs with proper language detection
- Quick reference examples are extracted from real-world usage patterns
- Reference file maintains hierarchical structure of official documentation

## Updating

To refresh this skill with updated documentation:
1. Re-run the scraper with the same configuration
2. The skill will be rebuilt with the latest information from docs.apidog.com
