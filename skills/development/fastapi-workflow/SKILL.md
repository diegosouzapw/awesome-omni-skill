---
name: fastapi-workflow
description: Docs-first development workflow for Python + FastAPI + Pydantic v2 projects with async APIs, dependency injection, and SQLAlchemy. Fetches current documentation via MCP before any implementation. Use when building or modifying FastAPI backends, API endpoints, Pydantic models, or database operations. Trigger phrases - "fastapi", "python api", "backend api", "pydantic", "sqlalchemy", "async api", "dependency injection". NOT for frontend work (use frontend-app/frontend-lp) or non-Python backends.
metadata:
  author: Petr
  version: 1.0.0
hooks:
  PostToolUse:
    - matcher: "tool == \"Edit\" && tool_input.file_path matches \"\\\\.py$\""
      hooks:
        - type: command
          command: 'bash -c ''cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)" && python -m mypy --ignore-missing-imports "$TOOL_INPUT_FILE_PATH" 2>&1 | head -20'''
          timeout: 30
---

# FastAPI Workflow-Driven Development Skill

## CRITICAL PRINCIPLE

**DOCS FIRST, CODE SECOND**

You MUST NEVER generate FastAPI code without first fetching relevant documentation via MCP tools.

This is NON-NEGOTIABLE. Every workflow below has MANDATORY MCP fetch steps that MUST be completed before implementation.

## When to Use This Skill

Use this skill for ALL FastAPI development tasks:

- Creating API endpoints (routers)
- Defining Pydantic models (request/response)
- Implementing dependency injection
- Database operations (SQLAlchemy)
- Authentication/authorization
- Background tasks
- WebSocket endpoints
- Any Python backend code with FastAPI

## Current Stack Requirements

**Required Versions:**

- Python 3.11+ (3.12+ recommended)
- FastAPI 0.124+
- Pydantic 2.12+ (v2 is default, v1 support is DEPRECATED)
- SQLAlchemy 2.0.44+ (async support)
- Uvicorn (ASGI server)
- scalar-fastapi (API docs — replaces Swagger UI and ReDoc)

**DEPRECATED:**

- Pydantic v1 compatibility will be removed in future FastAPI versions

**Key Pydantic v2 Changes:**

- `model_validator` instead of `validator`
- `field_validator` instead of `validator`
- `ConfigDict` instead of `class Config`
- Performance boost (Rust core)
- Strict mode available

## API Documentation Setup (Scalar)

**ALWAYS use Scalar instead of Swagger UI / ReDoc.** Scalar provides modern, interactive API docs with dark mode, search, and code snippets.

**Setup pattern:**

```python
# requirements.txt
scalar-fastapi>=1.6.0

# main.py
from scalar_fastapi import get_scalar_api_reference

app = FastAPI(
    title="My API",
    docs_url=None,   # Disable Swagger UI
    redoc_url=None,  # Disable ReDoc
)

@app.get("/docs", include_in_schema=False)
async def scalar_docs():
    return get_scalar_api_reference(
        openapi_url=app.openapi_url,
        title="My API",
        dark_mode=True,
    )
```

**Key points:**

- Disable built-in `docs_url` and `redoc_url` in FastAPI constructor
- Serve Scalar on `/docs` as a custom route
- `include_in_schema=False` hides the docs endpoint from the API schema
- OpenAPI JSON stays at `/openapi.json` (FastAPI default)

## Core Workflows

### Workflow 1: Creating API Endpoints

**Use for:** Creating new API routes/endpoints

**MANDATORY CHECKLIST:**

```
☐ 1. Determine endpoint type
     - GET (retrieve data)
     - POST (create data)
     - PUT/PATCH (update data)
     - DELETE (remove data)

☐ 2. ⚠️ MANDATORY: Fetch FastAPI routing docs
     Call MCP tool: fetch_fastapi_docs(topic: "routing")
     Wait for response before continuing

☐ 3. ⚠️ MANDATORY: Fetch Pydantic model docs
     Call MCP tool: fetch_pydantic_docs(topic: "models")
     Wait for response before continuing

☐ 4. Define Pydantic models:
     - Request body model (for POST/PUT)
     - Response model
     - Use Pydantic v2 syntax

☐ 5. Implement endpoint:
     - Use APIRouter for organization
     - Proper HTTP status codes
     - Response model in decorator
     - Async def for async operations

☐ 6. Add dependencies:
     - Database session
     - Authentication
     - Rate limiting (if needed)

☐ 7. VERIFY against quality checklist:
     ✓ Uses FastAPI 0.115+ patterns
     ✓ Pydantic v2 models
     ✓ Proper error handling
     ✓ OpenAPI docs generated
     ✓ Type hints everywhere
```

**Example Execution:**

```python
# WRONG: Generating code without fetching docs
@app.get("/products")
def get_products():
    return products

# CORRECT: Workflow followed
# 1. Determined: GET endpoint for products
# 2. Called: fetch_fastapi_docs(topic: "routing")
# 3. Called: fetch_pydantic_docs(topic: "models")
# 4. Reviewed current patterns from docs
# 5. Now implementing with current API:

from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, ConfigDict
from typing import Annotated

router = APIRouter(prefix="/products", tags=["products"])

class ProductResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    name: str
    price: float
    description: str | None = None

@router.get(
    "/",
    response_model=list[ProductResponse],
    summary="Get all products",
)
async def get_products(
    db: Annotated[AsyncSession, Depends(get_db)],
    skip: int = 0,
    limit: int = 100,
) -> list[ProductResponse]:
    """Retrieve all products with pagination."""
    products = await db.execute(
        select(Product).offset(skip).limit(limit)
    )
    return products.scalars().all()
```

### Workflow 2: Pydantic Models

**Use for:** Defining request/response schemas

**MANDATORY CHECKLIST:**

```
☐ 1. Identify model purpose:
     - Request body (input validation)
     - Response model (output serialization)
     - Database model (ORM mapping)
     - Internal model (business logic)

☐ 2. ⚠️ MANDATORY: Fetch Pydantic v2 documentation
     Call MCP tool: fetch_pydantic_docs(topic: "models")
     For validators: fetch_pydantic_docs(topic: "validators")
     Wait for response before continuing

☐ 3. Define model with Pydantic v2 syntax:
     - Use ConfigDict instead of class Config
     - Use field_validator instead of validator
     - Use model_validator for cross-field validation
     - Proper type hints (str | None, not Optional[str])

☐ 4. Add validation:
     - Field constraints (min_length, ge, le)
     - Custom validators
     - Computed fields if needed

☐ 5. Handle serialization:
     - from_attributes=True for ORM
     - alias for field name mapping
     - exclude for sensitive fields

☐ 6. VERIFY model:
     ✓ Pydantic v2 syntax (not v1)
     ✓ All fields typed
     ✓ Validation covers edge cases
     ✓ Serialization correct
```

**Example - Pydantic v2 Models:**

```python
# CORRECT: Pydantic v2 patterns
from pydantic import BaseModel, ConfigDict, Field, field_validator, model_validator
from datetime import datetime

class ProductCreate(BaseModel):
    """Request model for creating a product."""
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0, description="Price must be positive")
    description: str | None = None
    category_id: int

    @field_validator('name')
    @classmethod
    def name_must_be_title_case(cls, v: str) -> str:
        return v.strip().title()

class ProductResponse(BaseModel):
    """Response model for product data."""
    model_config = ConfigDict(
        from_attributes=True,  # Enable ORM mode
        json_schema_extra={
            "example": {
                "id": 1,
                "name": "Example Product",
                "price": 29.99,
            }
        }
    )

    id: int
    name: str
    price: float
    description: str | None = None
    created_at: datetime

class ProductUpdate(BaseModel):
    """Request model for updating a product."""
    name: str | None = None
    price: float | None = Field(default=None, gt=0)
    description: str | None = None

    @model_validator(mode='after')
    def check_at_least_one_field(self) -> 'ProductUpdate':
        if all(v is None for v in [self.name, self.price, self.description]):
            raise ValueError('At least one field must be provided')
        return self
```

### Workflow 3: Dependency Injection

**Use for:** Reusable dependencies (database, auth, etc.)

**MANDATORY CHECKLIST:**

```
☐ 1. Identify dependency type:
     - Database session
     - Current user (authentication)
     - Configuration settings
     - External service client

☐ 2. ⚠️ MANDATORY: Fetch FastAPI dependency docs
     Call MCP tool: fetch_fastapi_docs(topic: "dependencies")
     Wait for response before continuing

☐ 3. Implement dependency function:
     - Use Annotated for type hints
     - Yield for cleanup (database sessions)
     - Async for async operations

☐ 4. Use in endpoints:
     - Depends() for injection
     - Annotated[Type, Depends(dep)] pattern

☐ 5. VERIFY dependency:
     ✓ Proper cleanup (yield)
     ✓ Error handling
     ✓ Reusable across endpoints
```

**Example - Dependencies:**

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Annotated

# Database dependency
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

# Type alias for reuse
DbSession = Annotated[AsyncSession, Depends(get_db)]

# Auth dependency
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token")

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: DbSession,
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = await db.get(User, user_id)
    if user is None:
        raise credentials_exception
    return user

# Type alias for authenticated endpoints
CurrentUser = Annotated[User, Depends(get_current_user)]

# Usage in endpoint
@router.get("/me")
async def get_me(current_user: CurrentUser) -> UserResponse:
    return current_user
```

### Workflow 4: Database Operations (SQLAlchemy 2.0)

**Use for:** Database models and queries

**MANDATORY CHECKLIST:**

```
☐ 1. Determine database operation:
     - Model definition
     - CRUD operations
     - Complex queries
     - Migrations

☐ 2. ⚠️ MANDATORY: Fetch SQLAlchemy 2.0 docs
     Call MCP tool: fetch_sqlalchemy_docs(topic: "async")
     Wait for response before continuing

☐ 3. Define SQLAlchemy model:
     - Use Mapped and mapped_column (SQLAlchemy 2.0)
     - Relationships with proper loading
     - Indexes for performance

☐ 4. Implement repository/service:
     - Async operations
     - Proper transaction handling
     - Error handling

☐ 5. VERIFY database code:
     ✓ SQLAlchemy 2.0 syntax
     ✓ Async operations
     ✓ N+1 queries avoided
     ✓ Transactions handled
```

**Example - SQLAlchemy 2.0:**

```python
from sqlalchemy import ForeignKey, String, select
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship

class Base(DeclarativeBase):
    pass

class Product(Base):
    __tablename__ = "products"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))
    price: Mapped[float]
    description: Mapped[str | None]
    category_id: Mapped[int] = mapped_column(ForeignKey("categories.id"))

    category: Mapped["Category"] = relationship(back_populates="products")

class Category(Base):
    __tablename__ = "categories"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)

    products: Mapped[list["Product"]] = relationship(back_populates="category")

# Repository pattern
class ProductRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_all(self, skip: int = 0, limit: int = 100) -> list[Product]:
        result = await self.session.execute(
            select(Product)
            .options(selectinload(Product.category))
            .offset(skip)
            .limit(limit)
        )
        return list(result.scalars().all())

    async def get_by_id(self, product_id: int) -> Product | None:
        return await self.session.get(Product, product_id)

    async def create(self, product_data: ProductCreate) -> Product:
        product = Product(**product_data.model_dump())
        self.session.add(product)
        await self.session.flush()
        return product
```

### Workflow 5: Error Handling

**Use for:** API error responses

**MANDATORY CHECKLIST:**

```
☐ 1. ⚠️ MANDATORY: Fetch FastAPI error handling docs
     Call MCP tool: fetch_fastapi_docs(topic: "error-handling")
     Wait for response before continuing

☐ 2. Define custom exceptions:
     - Inherit from Exception
     - Include error details

☐ 3. Create exception handlers:
     - Register with app
     - Return proper HTTP responses

☐ 4. Use HTTPException for simple cases:
     - Proper status codes
     - Clear error messages

☐ 5. VERIFY error handling:
     ✓ All errors return JSON
     ✓ Proper status codes
     ✓ No sensitive info leaked
```

**Example - Error Handling:**

```python
from fastapi import FastAPI, HTTPException, Request, status
from fastapi.responses import JSONResponse

# Custom exception
class ProductNotFoundError(Exception):
    def __init__(self, product_id: int):
        self.product_id = product_id

# Exception handler
@app.exception_handler(ProductNotFoundError)
async def product_not_found_handler(
    request: Request,
    exc: ProductNotFoundError,
) -> JSONResponse:
    return JSONResponse(
        status_code=status.HTTP_404_NOT_FOUND,
        content={
            "detail": f"Product with id {exc.product_id} not found",
            "error_code": "PRODUCT_NOT_FOUND",
        },
    )

# Usage in endpoint
@router.get("/{product_id}")
async def get_product(product_id: int, db: DbSession) -> ProductResponse:
    product = await db.get(Product, product_id)
    if product is None:
        raise ProductNotFoundError(product_id)
    return product
```

### Workflow 6: Testing

**Use for:** API tests with pytest

**MANDATORY CHECKLIST:**

```
☐ 1. ⚠️ MANDATORY: Fetch FastAPI testing docs
     Call MCP tool: fetch_fastapi_docs(topic: "testing")
     Wait for response before continuing

☐ 2. Set up test client:
     - Use TestClient or httpx.AsyncClient
     - Override dependencies for testing

☐ 3. Write tests:
     - Test happy path
     - Test error cases
     - Test edge cases

☐ 4. Use fixtures:
     - Database fixtures
     - Auth fixtures
     - Factory patterns

☐ 5. VERIFY tests:
     ✓ All endpoints tested
     ✓ Error cases covered
     ✓ No real DB in unit tests
```

**Example - Testing:**

```python
import pytest
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

@pytest.fixture
async def client(db_session: AsyncSession):
    """Test client with overridden dependencies."""
    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test",
    ) as client:
        yield client

    app.dependency_overrides.clear()

@pytest.mark.asyncio
async def test_get_products(client: AsyncClient, sample_products):
    response = await client.get("/products/")
    assert response.status_code == 200
    data = response.json()
    assert len(data) == len(sample_products)

@pytest.mark.asyncio
async def test_get_product_not_found(client: AsyncClient):
    response = await client.get("/products/99999")
    assert response.status_code == 404
    assert response.json()["error_code"] == "PRODUCT_NOT_FOUND"

@pytest.mark.asyncio
async def test_create_product(client: AsyncClient, auth_headers):
    payload = {"name": "New Product", "price": 29.99, "category_id": 1}
    response = await client.post(
        "/products/",
        json=payload,
        headers=auth_headers,
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "New Product"
```

## Code Quality Requirements

### Python 3.11+ Features (MANDATORY)

```python
# ✓ CORRECT: Modern Python
# Union types with |
name: str | None = None

# Match statements
match status:
    case "active":
        return "Active"
    case "pending":
        return "Pending"
    case _:
        return "Unknown"

# Exception groups (if needed)
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task1())
        tg.create_task(task2())
except* ValueError as eg:
    handle_errors(eg.exceptions)
```

### FastAPI Patterns (MANDATORY)

```python
# ✓ CORRECT: Annotated dependencies
from typing import Annotated

DbSession = Annotated[AsyncSession, Depends(get_db)]

@router.get("/items")
async def get_items(db: DbSession) -> list[ItemResponse]:
    ...

# ✓ CORRECT: Response model in decorator
@router.post("/items", response_model=ItemResponse, status_code=201)
async def create_item(...):
    ...

# ✓ CORRECT: Tags and summary for OpenAPI
@router.get("/items", tags=["items"], summary="List all items")
async def list_items(...):
    """
    Retrieve all items.

    - **skip**: Number of items to skip
    - **limit**: Maximum number of items to return
    """
    ...
```

### Pydantic v2 Patterns (MANDATORY)

```python
# ✓ CORRECT: Pydantic v2
from pydantic import BaseModel, ConfigDict, Field, field_validator

class ItemModel(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
        str_strip_whitespace=True,
    )

    name: str = Field(min_length=1)
    price: float = Field(gt=0)

    @field_validator('name')
    @classmethod
    def validate_name(cls, v: str) -> str:
        return v.title()

# ✗ WRONG: Pydantic v1 (deprecated)
class ItemModel(BaseModel):
    class Config:  # Old style
        orm_mode = True

    @validator('name')  # Old decorator
    def validate_name(cls, v):
        return v.title()
```

## Decision Trees

### Endpoint Type Selection

```
What operation?
├─ Retrieve data → GET
├─ Create new resource → POST
├─ Full update → PUT
├─ Partial update → PATCH
└─ Remove resource → DELETE
```

### Response Model Selection

```
What are you returning?
├─ Single item → ItemResponse
├─ List of items → list[ItemResponse]
├─ Paginated → PaginatedResponse[ItemResponse]
├─ Nothing → None (with 204)
└─ Created item → ItemResponse (with 201)
```

### Database Pattern Selection

```
Query complexity?
├─ Simple CRUD → Direct SQLAlchemy in endpoint
├─ Complex queries → Repository pattern
├─ Business logic → Service layer + Repository
└─ Multiple models → Unit of Work pattern
```

## Common Mistakes to Avoid

**DON'T:**

- Generate code without fetching docs first
- Use Pydantic v1 syntax (validator, class Config)
- Use sync database operations
- Skip type hints
- Hardcode secrets in code
- Return ORM models directly (use response models)
- Ignore error handling
- Skip tests

**DO:**

- ALWAYS fetch documentation before implementing
- Follow workflows step-by-step
- Use Pydantic v2 patterns
- Use async/await for I/O operations
- Use environment variables for config
- Define response models
- Handle all error cases
- Write tests first (TDD)

## Integration with MCP Server

This skill works with the FastAPI MCP server. MCP provides current docs, this skill enforces the workflow.

**Available MCP Tools:**

1. `fetch_fastapi_docs(topic)` - FastAPI patterns and features
2. `fetch_pydantic_docs(topic)` - Pydantic v2 models and validation
3. `fetch_sqlalchemy_docs(topic)` - SQLAlchemy 2.0 async operations
4. `fetch_python_docs(topic)` - Python stdlib features
5. `search_pypi(query)` - Search Python packages

**You MUST call these tools at the workflow steps marked with ⚠️ MANDATORY**

## Verification Before Completion

Before considering any FastAPI implementation complete:

```
FINAL VERIFICATION CHECKLIST:
☐ All MANDATORY MCP tool calls were made
☐ Implementation follows fetched documentation
☐ Uses FastAPI 0.115+ patterns
☐ Uses Pydantic v2 syntax
☐ Uses SQLAlchemy 2.0 (if database)
☐ Python 3.11+ features used
☐ All endpoints have response models
☐ Error handling implemented
☐ Type hints everywhere
☐ Tests written and passing
☐ No secrets in code
```

## Summary

This skill enforces a DOCS-FIRST development process for FastAPI:

1. **Identify task** → Choose workflow
2. **Fetch docs** → Call MCP tools (MANDATORY)
3. **Review docs** → Understand current API
4. **Implement** → Follow fetched patterns exactly
5. **Verify** → Check against quality checklist

**NEVER skip step 2.** The MCP server provides current, accurate documentation. Use it.

**Your workflows are your process.** Follow them rigorously for consistent, high-quality Python backend code that uses current FastAPI, Pydantic v2, and SQLAlchemy 2.0 patterns.
