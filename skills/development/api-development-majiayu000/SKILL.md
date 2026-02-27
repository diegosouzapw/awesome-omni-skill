---
name: api-development
description: Use when creating new API endpoints, working with async operations, implementing long-running tasks with progress tracking, adding Pydantic models, or designing request/response schemas.
---

# API Development

Load this skill when:
- Creating new API endpoints
- Working with async operations
- Implementing long-running tasks with progress tracking
- Adding Pydantic models or request/response schemas

---

## API Structure

```
ktrdr/api/
├── endpoints/      # Route handlers
├── models/         # Pydantic request/response models
├── services/       # Business logic
└── main.py         # Router registration
```

## Adding New Endpoints

1. **Create endpoint** in `ktrdr/api/endpoints/`
2. **Define Pydantic models** in `ktrdr/api/models/`
3. **Implement business logic** in `ktrdr/api/services/`
4. **Register router** in `ktrdr/api/main.py`
5. **Add tests** in `tests/api/`

---

## Async Operation Pattern

For long-running tasks (training, backtesting, data downloads):

```python
from ktrdr.api.services.operations_service import OperationsService

@router.post("/long-operation")
async def start_operation(
    background_tasks: BackgroundTasks,
    operations_service: OperationsService = Depends(get_operations_service)
):
    # Register operation
    operation_id = await operations_service.register_operation(
        operation_type=OperationType.TRAINING,
        description="Training model..."
    )

    # Start background task
    background_tasks.add_task(
        run_operation,
        operation_id,
        operations_service
    )

    return {"operation_id": operation_id}
```

### Key Components

- **OperationsService**: Tracks all operations, progress, and status
- **BackgroundTasks**: FastAPI's mechanism for fire-and-forget tasks
- **operation_id**: Returned immediately so client can poll for status

---

## Progress Tracking

Operations should report progress for long-running tasks:

```python
async def run_operation(operation_id: str, ops_service: OperationsService):
    try:
        await ops_service.update_status(operation_id, OperationStatus.RUNNING)
        
        for i, step in enumerate(steps):
            # Do work
            await process_step(step)
            
            # Update progress
            await ops_service.update_progress(
                operation_id,
                percentage=(i + 1) / len(steps) * 100,
                phase=f"Processing step {i + 1}"
            )
        
        await ops_service.update_status(operation_id, OperationStatus.COMPLETED)
    except Exception as e:
        await ops_service.update_status(
            operation_id, 
            OperationStatus.FAILED,
            error=str(e)
        )
        raise
```

---

## Operation Status Endpoints

Standard endpoints for operation management:

```python
@router.get("/operations/{operation_id}")
async def get_operation_status(operation_id: str):
    """Get current status of an operation."""
    
@router.get("/operations/{operation_id}/metrics")
async def get_operation_metrics(operation_id: str):
    """Get detailed metrics for an operation."""

@router.delete("/operations/{operation_id}/cancel")
async def cancel_operation(operation_id: str):
    """Request cancellation of a running operation."""
```

---

## Pydantic Models

### Request Models

```python
from pydantic import BaseModel, Field

class TrainingRequest(BaseModel):
    strategy_path: str = Field(..., description="Path to strategy YAML")
    symbol: str = Field(..., description="Trading symbol")
    
    class Config:
        json_schema_extra = {
            "example": {
                "strategy_path": "config/strategies/example.yaml",
                "symbol": "AAPL"
            }
        }
```

### Response Models

```python
class OperationResponse(BaseModel):
    operation_id: str
    status: OperationStatus
    progress: float = 0.0
    phase: str | None = None
    error: str | None = None
    created_at: datetime
    updated_at: datetime
```

---

## Error Handling

Use HTTPException for API errors:

```python
from fastapi import HTTPException

@router.get("/resource/{id}")
async def get_resource(id: str):
    resource = await fetch_resource(id)
    if not resource:
        raise HTTPException(
            status_code=404,
            detail=f"Resource {id} not found"
        )
    return resource
```

---

## Documentation

Once server is running:
- **Swagger UI**: http://localhost:8000/api/v1/docs
- **ReDoc**: http://localhost:8000/api/v1/redoc

---

## Testing API Endpoints

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_operation(client: AsyncClient):
    response = await client.post(
        "/api/v1/operations",
        json={"type": "training", "params": {...}}
    )
    assert response.status_code == 200
    assert "operation_id" in response.json()
```

---

## Key Files

- `ktrdr/api/main.py` — App setup and router registration
- `ktrdr/api/services/operations_service.py` — Operation tracking
- `ktrdr/api/dependencies.py` — Dependency injection
