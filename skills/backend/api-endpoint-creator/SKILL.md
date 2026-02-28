---
name: api-endpoint-creator
description: Guides standardized REST API endpoint creation following team conventions. Use when creating new API endpoints.
version: 1.0.0
author: Backend Team
category: custom
token_estimate: ~3200
---

<objective>
Codify REST API design conventions and best practices for creating consistent, well-documented, and tested API endpoints. Ensure all endpoints follow the same patterns for authentication, error handling, validation, and documentation.
</objective>

<when_to_use>
Use this skill when:

- Creating a new REST API endpoint
- Adding routes to an existing API
- Refactoring endpoints to follow team standards
- Building CRUD operations for new resources
- Extending API functionality

Do NOT use this skill when:

- Building GraphQL APIs (use graphql-design skill)
- Creating internal-only functions (not exposed via API)
- Working on non-REST protocols (WebSocket, gRPC)
</when_to_use>

<prerequisites>
Before using this skill, ensure:

- API framework is set up (Flask, FastAPI, Express, etc.)
- Authentication system is in place
- Database models are defined
- OpenAPI/Swagger documentation structure exists
- Testing framework is configured
</prerequisites>

<workflow>
<step>
<name>Define Endpoint Specification</name>

Plan the endpoint before implementation:

**Endpoint Details:**
```yaml
# Endpoint specification template
method: POST
path: /api/v1/resources
description: Create a new resource
auth_required: true
rate_limit: 10 requests/minute
request_body:
  content_type: application/json
  schema:
    name: string (required, max 100 chars)
    description: string (optional, max 1000 chars)
    tags: array of strings (optional)
response:
  success: 201 Created
  errors: 400 Bad Request, 401 Unauthorized, 409 Conflict
```

**URL Structure Conventions:**

Follow REST principles:
- `/api/v1/resources` - Collection endpoint (GET all, POST new)
- `/api/v1/resources/{id}` - Item endpoint (GET, PUT, PATCH, DELETE)
- `/api/v1/resources/{id}/subresources` - Nested resources
- `/api/v1/resources/actions` - Special actions (e.g., /search, /bulk)

**HTTP Methods:**
- `GET` - Retrieve resource(s), no side effects
- `POST` - Create new resource
- `PUT` - Replace entire resource
- `PATCH` - Update partial resource
- `DELETE` - Remove resource
</step>

<step>
<name>Implement Request Handling</name>

Create the endpoint with proper structure:

**Python/Flask Example:**
```python
from flask import Blueprint, request, jsonify
from functools import wraps
from marshmallow import Schema, fields, ValidationError

# Define request schema
class CreateResourceSchema(Schema):
    name = fields.String(required=True, validate=lambda x: len(x) <= 100)
    description = fields.String(validate=lambda x: len(x) <= 1000)
    tags = fields.List(fields.String())

create_resource_schema = CreateResourceSchema()

@api_bp.route('/api/v1/resources', methods=['POST'])
@require_auth  # Authentication decorator
@rate_limit(max_requests=10, window=60)  # Rate limiting
def create_resource():
    """Create a new resource.

    Request body:
        {
            "name": "Resource name",
            "description": "Optional description",
            "tags": ["tag1", "tag2"]
        }

    Returns:
        201: Resource created successfully
        400: Invalid request data
        401: Authentication required
        409: Resource already exists
    """
    # 1. Parse and validate request
    try:
        data = create_resource_schema.load(request.get_json())
    except ValidationError as e:
        return jsonify({'error': 'Validation failed', 'details': e.messages}), 400

    # 2. Authorization check (can user create resources?)
    if not current_user.has_permission('create_resource'):
        return jsonify({'error': 'Permission denied'}), 403

    # 3. Business logic validation
    existing = Resource.query.filter_by(
        name=data['name'],
        user_id=current_user.id
    ).first()
    if existing:
        return jsonify({'error': 'Resource with this name already exists'}), 409

    # 4. Create resource
    try:
        resource = Resource(
            name=data['name'],
            description=data.get('description', ''),
            tags=data.get('tags', []),
            user_id=current_user.id,
            created_at=datetime.utcnow()
        )
        db.session.add(resource)
        db.session.commit()

        # 5. Return response
        return jsonify(resource.to_dict()), 201

    except Exception as e:
        db.session.rollback()
        logger.error(f"Failed to create resource: {e}")
        return jsonify({'error': 'Failed to create resource'}), 500
```

**Node.js/Express Example:**
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');

router.post('/api/v1/resources',
    // Authentication middleware
    requireAuth,

    // Rate limiting middleware
    rateLimit({ max: 10, windowMs: 60000 }),

    // Validation middleware
    body('name').isString().isLength({ max: 100 }).notEmpty(),
    body('description').optional().isString().isLength({ max: 1000 }),
    body('tags').optional().isArray(),

    async (req, res) => {
        // 1. Check validation
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                error: 'Validation failed',
                details: errors.array()
            });
        }

        // 2. Authorization
        if (!req.user.hasPermission('create_resource')) {
            return res.status(403).json({ error: 'Permission denied' });
        }

        // 3. Business logic
        const existing = await Resource.findOne({
            name: req.body.name,
            userId: req.user.id
        });
        if (existing) {
            return res.status(409).json({
                error: 'Resource with this name already exists'
            });
        }

        // 4. Create resource
        try {
            const resource = await Resource.create({
                name: req.body.name,
                description: req.body.description || '',
                tags: req.body.tags || [],
                userId: req.user.id
            });

            // 5. Return response
            res.status(201).json(resource.toJSON());
        } catch (error) {
            console.error('Failed to create resource:', error);
            res.status(500).json({ error: 'Failed to create resource' });
        }
    }
);
```

**Key Components:**
1. **Input validation** - Validate request format and data types
2. **Authentication** - Verify user is authenticated
3. **Authorization** - Check user has permission for this action
4. **Business logic** - Check business rules (uniqueness, relationships)
5. **Error handling** - Catch and handle errors appropriately
6. **Response** - Return appropriate status code and data
</step>

<step>
<name>Implement Error Responses</name>

Use consistent error response format:

**Standard Error Format:**
```json
{
    "error": "Brief error message",
    "details": "More detailed explanation or validation errors",
    "code": "ERROR_CODE",
    "timestamp": "2025-01-20T10:30:00Z"
}
```

**Common HTTP Status Codes:**
- `200 OK` - Successful GET, PUT, PATCH, DELETE
- `201 Created` - Successful POST
- `204 No Content` - Successful DELETE with no response body
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource already exists or conflict with current state
- `422 Unprocessable Entity` - Validation errors
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error

**Error Handler Example:**
```python
from flask import jsonify
from datetime import datetime

def handle_api_error(error_message, status_code=400, details=None, code=None):
    """Create standardized error response."""
    response = {
        'error': error_message,
        'timestamp': datetime.utcnow().isoformat() + 'Z'
    }
    if details:
        response['details'] = details
    if code:
        response['code'] = code
    return jsonify(response), status_code

# Usage:
return handle_api_error(
    'Resource not found',
    status_code=404,
    code='RESOURCE_NOT_FOUND'
)
```
</step>

<step>
<name>Add Pagination (for Collection Endpoints)</name>

Implement pagination for list endpoints:

**Pagination Parameters:**
```python
@api_bp.route('/api/v1/resources', methods=['GET'])
@require_auth
def list_resources():
    """List resources with pagination.

    Query parameters:
        page: Page number (default: 1)
        per_page: Items per page (default: 20, max: 100)
        sort: Sort field (default: created_at)
        order: Sort order (asc/desc, default: desc)
    """
    # Parse pagination params
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 20, type=int), 100)
    sort = request.args.get('sort', 'created_at')
    order = request.args.get('order', 'desc')

    # Validate sort field (prevent SQL injection)
    allowed_sort_fields = ['created_at', 'updated_at', 'name']
    if sort not in allowed_sort_fields:
        return handle_api_error(f'Invalid sort field. Use: {allowed_sort_fields}')

    # Query with pagination
    query = Resource.query.filter_by(user_id=current_user.id)

    # Apply sorting
    sort_column = getattr(Resource, sort)
    if order == 'desc':
        query = query.order_by(sort_column.desc())
    else:
        query = query.order_by(sort_column.asc())

    # Paginate
    pagination = query.paginate(page=page, per_page=per_page, error_out=False)

    # Build response
    return jsonify({
        'items': [r.to_dict() for r in pagination.items],
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total_pages': pagination.pages,
            'total_items': pagination.total,
            'has_next': pagination.has_next,
            'has_prev': pagination.has_prev
        }
    }), 200
```

**Pagination Response Format:**
```json
{
    "items": [
        {"id": 1, "name": "Resource 1"},
        {"id": 2, "name": "Resource 2"}
    ],
    "pagination": {
        "page": 1,
        "per_page": 20,
        "total_pages": 5,
        "total_items": 95,
        "has_next": true,
        "has_prev": false
    }
}
```
</step>

<step>
<name>Create Tests</name>

Write comprehensive tests for the endpoint:

**Test Structure:**
```python
import pytest
from app import create_app, db
from app.models import Resource, User

@pytest.fixture
def client():
    """Create test client."""
    app = create_app('testing')
    with app.test_client() as client:
        with app.app_context():
            db.create_all()
            yield client
            db.drop_all()

@pytest.fixture
def auth_headers():
    """Create auth headers for testing."""
    user = User.create(email='test@example.com', password='password')
    token = user.generate_auth_token()
    return {'Authorization': f'Bearer {token}'}

# Test happy path
def test_create_resource_with_valid_data_returns_201(client, auth_headers):
    """Test creating resource with valid data."""
    data = {
        'name': 'Test Resource',
        'description': 'Test description',
        'tags': ['tag1', 'tag2']
    }
    response = client.post('/api/v1/resources',
                          json=data,
                          headers=auth_headers)

    assert response.status_code == 201
    json_data = response.get_json()
    assert json_data['name'] == 'Test Resource'
    assert json_data['description'] == 'Test description'
    assert json_data['tags'] == ['tag1', 'tag2']
    assert 'id' in json_data
    assert 'created_at' in json_data

# Test authentication
def test_create_resource_without_auth_returns_401(client):
    """Test endpoint requires authentication."""
    data = {'name': 'Test Resource'}
    response = client.post('/api/v1/resources', json=data)

    assert response.status_code == 401
    assert 'error' in response.get_json()

# Test validation
def test_create_resource_with_missing_name_returns_400(client, auth_headers):
    """Test name field is required."""
    data = {'description': 'Description without name'}
    response = client.post('/api/v1/resources',
                          json=data,
                          headers=auth_headers)

    assert response.status_code == 400
    json_data = response.get_json()
    assert 'error' in json_data
    assert 'name' in json_data.get('details', {})

def test_create_resource_with_too_long_name_returns_400(client, auth_headers):
    """Test name length validation."""
    data = {'name': 'x' * 101}  # Exceeds 100 char limit
    response = client.post('/api/v1/resources',
                          json=data,
                          headers=auth_headers)

    assert response.status_code == 400

# Test business logic
def test_create_resource_with_duplicate_name_returns_409(client, auth_headers):
    """Test duplicate name is rejected."""
    data = {'name': 'Unique Name'}

    # Create first resource
    response1 = client.post('/api/v1/resources',
                           json=data,
                           headers=auth_headers)
    assert response1.status_code == 201

    # Try to create duplicate
    response2 = client.post('/api/v1/resources',
                           json=data,
                           headers=auth_headers)
    assert response2.status_code == 409
    assert 'already exists' in response2.get_json()['error'].lower()

# Test list endpoint
def test_list_resources_returns_paginated_results(client, auth_headers):
    """Test listing resources with pagination."""
    # Create test resources
    for i in range(25):
        Resource.create(name=f'Resource {i}', user_id=current_user.id)

    # Request first page
    response = client.get('/api/v1/resources?page=1&per_page=10',
                         headers=auth_headers)

    assert response.status_code == 200
    json_data = response.get_json()
    assert len(json_data['items']) == 10
    assert json_data['pagination']['page'] == 1
    assert json_data['pagination']['total_items'] == 25
    assert json_data['pagination']['has_next'] is True
    assert json_data['pagination']['has_prev'] is False
```

**Test Coverage Requirements:**
- Happy path (valid data)
- Authentication (with/without auth)
- Authorization (sufficient/insufficient permissions)
- Validation (missing, invalid, edge cases)
- Business logic (duplicates, conflicts)
- Error handling (database errors, etc.)
- Pagination (if applicable)
</step>

<step>
<name>Document with OpenAPI</name>

Create OpenAPI documentation:

**OpenAPI Specification:**
```yaml
openapi: 3.0.0
paths:
  /api/v1/resources:
    post:
      summary: Create a new resource
      description: Creates a new resource for the authenticated user
      tags:
        - Resources
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - name
              properties:
                name:
                  type: string
                  maxLength: 100
                  example: "My Resource"
                description:
                  type: string
                  maxLength: 1000
                  example: "A detailed description"
                tags:
                  type: array
                  items:
                    type: string
                  example: ["important", "project-alpha"]
      responses:
        '201':
          description: Resource created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Resource'
        '400':
          description: Invalid request data
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '401':
          description: Authentication required
        '403':
          description: Permission denied
        '409':
          description: Resource already exists

    get:
      summary: List resources
      description: Retrieve a paginated list of resources
      tags:
        - Resources
      security:
        - BearerAuth: []
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: per_page
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
        - name: sort
          in: query
          schema:
            type: string
            enum: [created_at, updated_at, name]
            default: created_at
        - name: order
          in: query
          schema:
            type: string
            enum: [asc, desc]
            default: desc
      responses:
        '200':
          description: List of resources
          content:
            application/json:
              schema:
                type: object
                properties:
                  items:
                    type: array
                    items:
                      $ref: '#/components/schemas/Resource'
                  pagination:
                    $ref: '#/components/schemas/Pagination'

components:
  schemas:
    Resource:
      type: object
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: "My Resource"
        description:
          type: string
          example: "A detailed description"
        tags:
          type: array
          items:
            type: string
          example: ["important", "project-alpha"]
        user_id:
          type: integer
          example: 42
        created_at:
          type: string
          format: date-time
          example: "2025-01-20T10:30:00Z"
        updated_at:
          type: string
          format: date-time
          example: "2025-01-20T10:30:00Z"

    Error:
      type: object
      properties:
        error:
          type: string
          example: "Validation failed"
        details:
          type: object
          example: {"name": ["This field is required"]}
        code:
          type: string
          example: "VALIDATION_ERROR"
        timestamp:
          type: string
          format: date-time

    Pagination:
      type: object
      properties:
        page:
          type: integer
        per_page:
          type: integer
        total_pages:
          type: integer
        total_items:
          type: integer
        has_next:
          type: boolean
        has_prev:
          type: boolean

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

**Python Automatic Documentation:**
```python
# Using flask-apispec for automatic OpenAPI generation
from flask_apispec import use_kwargs, marshal_with, doc

@api_bp.route('/api/v1/resources', methods=['POST'])
@doc(description='Create a new resource', tags=['Resources'])
@use_kwargs(CreateResourceSchema)
@marshal_with(ResourceSchema, code=201)
@require_auth
def create_resource():
    # Implementation
    pass
```
</step>
</workflow>

<best_practices>
<practice>
<title>Use Consistent URL Patterns</title>

Follow REST conventions for predictability.
</practice>

<practice>
<title>Version Your API</title>

Use `/api/v1/` prefix to allow future breaking changes without affecting existing clients.
</practice>

<practice>
<title>Return Appropriate Status Codes</title>

Status codes provide semantic meaning; use them correctly.
</practice>

<practice>
<title>Validate Early</title>

Validate input as early as possible to fail fast and provide clear errors.
</practice>

<practice>
<title>Degree of Freedom</title>

**Medium Freedom**: Core patterns (auth, validation, error format, documentation) must be followed, but implementation details can vary based on framework and requirements.
</practice>

<practice>
<title>Token Efficiency</title>

This skill uses approximately **3,200 tokens** when fully loaded.
</practice>
</best_practices>

<common_pitfalls>
<pitfall>
<name>Insufficient Validation</name>

**What Happens:** Invalid data reaches database or business logic, causing errors or security issues.

**How to Avoid:**
- Validate all input at the API boundary
- Use schema validation libraries
- Validate types, formats, lengths, and business rules
</pitfall>

<pitfall>
<name>Inconsistent Error Responses</name>

**What Happens:** Different endpoints return errors in different formats, making client integration difficult.

**How to Avoid:**
- Use standard error response format across all endpoints
- Create helper functions for error responses
- Document error format in API spec
</pitfall>

<pitfall>
<name>Missing Authentication/Authorization</name>

**What Happens:** Security vulnerability allowing unauthorized access.

**How to Avoid:**
- Always add authentication to non-public endpoints
- Check authorization (not just authentication)
- Test with and without auth credentials
</pitfall>
</common_pitfalls>

<examples>
<example>
<title>Simple CRUD Endpoint</title>

**Context:** Create endpoints for managing user profiles.

**Implementation:**
```python
# GET /api/v1/profiles/{id}
@api_bp.route('/api/v1/profiles/<int:profile_id>', methods=['GET'])
@require_auth
def get_profile(profile_id):
    profile = Profile.query.get_or_404(profile_id)

    # Check authorization
    if profile.user_id != current_user.id and not current_user.is_admin:
        return handle_api_error('Permission denied', 403)

    return jsonify(profile.to_dict()), 200

# PUT /api/v1/profiles/{id}
@api_bp.route('/api/v1/profiles/<int:profile_id>', methods=['PUT'])
@require_auth
def update_profile(profile_id):
    profile = Profile.query.get_or_404(profile_id)

    if profile.user_id != current_user.id:
        return handle_api_error('Permission denied', 403)

    try:
        data = update_profile_schema.load(request.get_json())
    except ValidationError as e:
        return handle_api_error('Validation failed', 400, details=e.messages)

    profile.update(**data)
    db.session.commit()

    return jsonify(profile.to_dict()), 200

# DELETE /api/v1/profiles/{id}
@api_bp.route('/api/v1/profiles/<int:profile_id>', methods=['DELETE'])
@require_auth
def delete_profile(profile_id):
    profile = Profile.query.get_or_404(profile_id)

    if profile.user_id != current_user.id:
        return handle_api_error('Permission denied', 403)

    db.session.delete(profile)
    db.session.commit()

    return '', 204
```

**Outcome:** Complete CRUD operations following team conventions.
</example>
</examples>

<related_skills>
- **api-design**: General REST API design principles
- **authentication-patterns**: Detailed auth implementation
- **database-design**: Database schema for API resources
- **integration-testing**: Testing API endpoints end-to-end
</related_skills>

<notes>
<version_history>
### Version 1.0.0 (2025-01-20)
- Initial creation
- Standard patterns for REST API endpoints
- Comprehensive examples and testing guidance
</version_history>

<additional_resources>
- [REST API Design Best Practices](https://restfulapi.net/)
- [OpenAPI Specification](https://swagger.io/specification/)
- Internal: API Style Guide at [internal wiki]
</additional_resources>
</notes>

<success_criteria>
API endpoint creation is considered successful when:

1. **Specification Defined**
   - Clear HTTP method and path
   - Request/response schema documented
   - Authentication/authorization requirements specified
   - Rate limiting defined if applicable

2. **Implementation Complete**
   - Request parsing and validation implemented
   - Authentication/authorization checks in place
   - Business logic properly handled
   - Error handling comprehensive
   - Appropriate status codes returned

3. **Error Handling Consistent**
   - Standard error format used
   - All error cases covered
   - Appropriate HTTP status codes
   - Helpful error messages

4. **Pagination Added (if collection endpoint)**
   - Page and per_page parameters supported
   - Sorting options available
   - Pagination metadata in response
   - SQL injection protection for sort fields

5. **Tests Written and Passing**
   - Happy path tested
   - Authentication/authorization tested
   - Validation tested (all edge cases)
   - Business logic tested
   - Error cases tested
   - Test coverage meets threshold

6. **Documentation Complete**
   - OpenAPI specification created
   - Request/response examples provided
   - Authentication requirements documented
   - Error responses documented
   - Code has appropriate docstrings

7. **Review Passed**
   - Code review completed
   - Security review passed
   - Performance acceptable
   - Team conventions followed
</success_criteria>
