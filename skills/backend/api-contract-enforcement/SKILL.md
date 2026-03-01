---
name: api-contract-enforcement
description: Validates REST API implementations against OpenAPI 3.x specifications. Use when (1) Implementing or reviewing API endpoints, (2) Before deploying API changes to production, (3) Ensuring contract compliance between spec and implementation, or (4) Debugging API integration issues caused by contract mismatches. Works with Python (FastAPI/Flask) backends.
---

# API Contract Enforcement Skill

Ensures REST API implementations strictly match OpenAPI 3.x specifications for routes, methods, payloads, and responses.

## Quick Start

Validate API implementation against spec:
```bash
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py <openapi-spec.yaml> <backend-module>
```

Example:
```bash
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main
```

## When to Use This Skill

Trigger this skill when:
- Implementing new REST endpoints (validate against spec before PR)
- Reviewing API changes or pull requests
- Before deploying to production (contract compliance gate)
- Debugging API integration failures (check for mismatches)
- Refactoring API endpoints (ensure contract unchanged)
- Updating OpenAPI spec (verify all endpoints match)

## Validation Process

### Step 1: Locate Artifacts

Ensure you have:
- **OpenAPI spec**: `openapi.yaml` or `openapi.json`
- **Backend module**: Python module containing the API app (e.g., `myapp.main`)

Common locations:
- OpenAPI spec: `api/openapi.yaml`, `docs/openapi.yaml`, or root
- Backend module: `main.py`, `app/main.py`, or similar entry point

### Step 2: Run Validation

Execute the validation script:
```bash
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py <openapi-spec> <backend-module>
```

The script performs these validations:
- **Route/method/path validation**: Each OpenAPI path+method exists in implementation
- **Request payload validation**: Request body fields, types, and required constraints match
- **Response payload validation**: Response structure, types, and status codes match
- **Parameter validation**: Query/path/header parameters are correctly implemented

### Step 3: Review Results

Validation outputs:
```
PASS/FAIL: <overall status>
✓ GET /api/users - routes match
✗ POST /api/users - request body missing field 'email'
✓ GET /api/users/{id} - responses match

Contract Violations:
[Details of each mismatch]

Recommendations:
[Actionable fixes]
```

### Step 4: Fix Contract Violations

For each violation:

**Missing endpoint**: Implement the missing route/method in your backend
```python
@app.post("/api/users")  # Missing in implementation
def create_user(user: UserCreate):
    return user
```

**Request body mismatch**: Update implementation to match spec
```python
# Spec requires: email (required), name (optional)
class UserCreate(BaseModel):
    email: str  # This was missing
    name: Optional[str]
```

**Response mismatch**: Fix response structure or update spec
```python
# Spec expects {id, email, name}, implementation returns {id, email}
def get_user(user_id: int):
    return {"id": user_id, "email": "...", "name": "..."}  # Added 'name'
```

**Parameter mismatch**: Add or correct parameters in route definition
```python
@app.get("/api/users/{id}")  # Spec has required path parameter 'id'
def get_user(id: int):
    ...
```

### Step 5: Revalidate

Run validation again after fixes until all checks pass.

## Validation Rules

### Route/Method/Path Validation

Checks that every endpoint defined in OpenAPI spec is implemented:
- HTTP methods match (GET, POST, PUT, DELETE, etc.)
- URL paths match exactly (including parameters like `{id}`)
- Path parameters are properly declared in function signatures

**Example violation:**
- Spec: `GET /api/users/{id}`
- Impl: `GET /api/users/{user_id}`
- Issue: Path parameter name mismatch

### Request Payload Validation

Compares request body schemas with implementation:
- All required fields present in request model
- Field types match (string, integer, boolean, array, object)
- Nested structures correctly defined
- Validation rules (min/max length, patterns) considered warnings

**Example violation:**
- Spec requires: `email: string (required)`
- Impl: `email: string (optional)`
- Issue: Required field not enforced in implementation

### Response Payload Validation

Verifies response structure matches OpenAPI definitions:
- Response status codes documented
- Response body structure matches schema
- Field types consistent
- Nested objects correctly structured

**Example violation:**
- Spec returns: `{id, email, name, created_at}`
- Impl returns: `{id, email}`
- Issue: Missing fields in response

### Parameter Validation

Ensures query, path, and header parameters are correct:
- Required parameters present
- Optional parameters have defaults or nullable types
- Parameter types match

**Example violation:**
- Spec: `GET /api/items?limit=10&offset=0`
- Impl: `GET /api/items`
- Issue: Query parameters not implemented

## Common Issues and Solutions

### Missing Endpoint
**Error**: `Endpoint GET /api/missing not found in implementation`
**Fix**: Implement the endpoint or remove from spec

### Path Parameter Mismatch
**Error**: `Path parameter 'id' in spec vs 'user_id' in implementation`
**Fix**: Rename parameter to match spec or update spec

### Required Field Missing
**Error**: `Request body missing required field 'email'`
**Fix**: Add field to request model with required type

### Response Structure Mismatch
**Error**: `Response missing field 'created_at'`
**Fix**: Add field to response or update spec if field not needed

### Status Code Not Documented
**Error**: `Endpoint returns 401 but not in OpenAPI spec`
**Fix**: Add response for 401 status code in spec or fix auth

## Integration with Development Workflow

### Before PR Submission
```bash
# Run validation in CI/CD
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main
# Exit code 0 on success, 1 on contract violation
```

### During Implementation
```bash
# Validate frequently to catch issues early
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main
```

### After Spec Updates
```bash
# When OpenAPI spec changes, verify all endpoints still match
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main
```

## Advanced Usage

### Specific Endpoints Only
Validate specific endpoint groups:
```python
# Modify script to filter by path prefix
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main --path-prefix "/api/v2"
```

### Strict Mode
Fail on warnings (e.g., validation rules not implemented):
```bash
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main --strict
```

### Output Format
Generate JSON for CI/CD integration:
```bash
python .claude/skills/api-contract-enforcement/scripts/validate_api_contract.py openapi.yaml myapp.main --format json
```

## Reference Materials

- [OPENAPI_BEST_PRACTICES.md](references/OPENAPI_BEST_PRACTICES.md): Guidelines for writing clear OpenAPI specs
- [FASTAPI_CONTRACT_PATTERNS.md](references/FASTAPI_CONTRACT_PATTERNS.md): Common patterns for contract-compliant FastAPI endpoints
- [VALIDATION_RULES.md](references/VALIDATION_RULES.md): Detailed explanation of all validation rules

## Framework Support

Currently supports:
- **FastAPI**: Automatic schema extraction from Pydantic models
- **Flask**: Manual route inspection (limited automatic validation)

For other frameworks, see [VALIDATION_RULES.md](references/VALIDATION_RULES.md) for manual validation guidance.
