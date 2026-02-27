---
name: api-scaffolding
description: Quickly scaffold production-ready FastAPI endpoints for client workflows. Use when building new APIs for QuickBooks, ShipStation, or webhook integrations.
---

# API Scaffolding for SMB Automation

Spin up professional REST APIs in minutes for client deliverables.

## When to Use This Skill

- Creating new API endpoints for client workflows
- Building webhook receivers (Stripe, ShipStation, etc.)
- Setting up FastAPI projects from scratch
- Adding endpoints to existing `execution/` scripts

## Quick Start: New Endpoint

### 1. Define the Pydantic Models
```python
# execution/schemas/invoice.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from datetime import datetime

class LineItem(BaseModel):
    description: str
    quantity: int = Field(gt=0)
    unit_price: float = Field(gt=0)

class CreateInvoiceRequest(BaseModel):
    customer_email: EmailStr
    customer_name: str = Field(min_length=1, max_length=100)
    line_items: list[LineItem] = Field(min_length=1)
    due_date: datetime
    memo: Optional[str] = None

class InvoiceResponse(BaseModel):
    invoice_id: str
    qbo_id: Optional[str] = None
    status: str
    total: float
    created_at: datetime
```

### 2. Create the Endpoint
```python
# execution/api/invoices.py
from fastapi import APIRouter, HTTPException, Depends
from ..schemas.invoice import CreateInvoiceRequest, InvoiceResponse
from ..services.qbo_service import qbo_service
import logging

router = APIRouter(prefix="/invoices", tags=["invoices"])
log = logging.getLogger(__name__)

@router.post("/", response_model=InvoiceResponse, status_code=201)
async def create_invoice(request: CreateInvoiceRequest):
    """Create a new invoice in QuickBooks."""
    try:
        # Calculate total
        total = sum(item.quantity * item.unit_price for item in request.line_items)

        # Create in QBO
        qbo_invoice = await qbo_service.create_invoice(
            customer_email=request.customer_email,
            customer_name=request.customer_name,
            line_items=[item.model_dump() for item in request.line_items],
            due_date=request.due_date,
            memo=request.memo
        )

        log.info("Invoice created", extra={
            "invoice_id": qbo_invoice.id,
            "customer": request.customer_email,
            "total": total
        })

        return InvoiceResponse(
            invoice_id=qbo_invoice.doc_number,
            qbo_id=qbo_invoice.id,
            status="created",
            total=total,
            created_at=datetime.utcnow()
        )

    except QBOAuthError:
        raise HTTPException(status_code=401, detail="QuickBooks auth expired")
    except QBOValidationError as e:
        raise HTTPException(status_code=422, detail=str(e))
    except Exception as e:
        log.exception("Invoice creation failed")
        raise HTTPException(status_code=500, detail="Internal error")
```

### 3. Wire Up the Router
```python
# execution/main.py
from fastapi import FastAPI
from .api import invoices, webhooks, inventory

app = FastAPI(
    title="SMB Automation API",
    version="1.0.0"
)

app.include_router(invoices.router, prefix="/api/v1")
app.include_router(webhooks.router, prefix="/api/v1")
app.include_router(inventory.router, prefix="/api/v1")

@app.get("/health")
async def health():
    return {"status": "ok"}
```

## Common Endpoint Patterns

### Webhook Receiver
```python
# execution/api/webhooks.py
from fastapi import APIRouter, Request, HTTPException, Header
import hmac
import hashlib

router = APIRouter(prefix="/webhooks", tags=["webhooks"])

@router.post("/stripe")
async def stripe_webhook(
    request: Request,
    stripe_signature: str = Header(alias="Stripe-Signature")
):
    """Handle Stripe webhook events."""
    payload = await request.body()

    # Verify signature
    try:
        event = stripe.Webhook.construct_event(
            payload, stripe_signature, WEBHOOK_SECRET
        )
    except stripe.error.SignatureVerificationError:
        raise HTTPException(status_code=400, detail="Invalid signature")

    # Process idempotently
    event_id = event["id"]
    if await is_processed(event_id):
        return {"status": "duplicate"}

    # Handle event types
    match event["type"]:
        case "payment_intent.succeeded":
            await handle_payment_success(event["data"]["object"])
        case "invoice.paid":
            await handle_invoice_paid(event["data"]["object"])

    await mark_processed(event_id)
    return {"status": "processed"}
```

### CRUD Resource
```python
# execution/api/customers.py
from fastapi import APIRouter, HTTPException, Query
from typing import Optional

router = APIRouter(prefix="/customers", tags=["customers"])

@router.get("/")
async def list_customers(
    skip: int = Query(0, ge=0),
    limit: int = Query(50, ge=1, le=100),
    status: Optional[str] = None
):
    """List customers with pagination."""
    customers = await customer_service.list(skip=skip, limit=limit, status=status)
    return {"customers": customers, "total": len(customers)}

@router.get("/{customer_id}")
async def get_customer(customer_id: str):
    """Get customer by ID."""
    customer = await customer_service.get(customer_id)
    if not customer:
        raise HTTPException(status_code=404, detail="Customer not found")
    return customer

@router.post("/", status_code=201)
async def create_customer(request: CreateCustomerRequest):
    """Create new customer."""
    customer = await customer_service.create(request)
    return customer

@router.patch("/{customer_id}")
async def update_customer(customer_id: str, request: UpdateCustomerRequest):
    """Update customer."""
    customer = await customer_service.update(customer_id, request)
    if not customer:
        raise HTTPException(status_code=404, detail="Customer not found")
    return customer

@router.delete("/{customer_id}", status_code=204)
async def delete_customer(customer_id: str):
    """Delete customer."""
    deleted = await customer_service.delete(customer_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="Customer not found")
```

### Background Task Trigger
```python
# execution/api/jobs.py
from fastapi import APIRouter, BackgroundTasks

router = APIRouter(prefix="/jobs", tags=["jobs"])

@router.post("/sync-inventory")
async def trigger_inventory_sync(background_tasks: BackgroundTasks):
    """Trigger background inventory sync."""
    job_id = generate_job_id()

    background_tasks.add_task(sync_inventory_task, job_id)

    return {
        "job_id": job_id,
        "status": "queued",
        "check_status": f"/api/v1/jobs/{job_id}"
    }

@router.get("/{job_id}")
async def get_job_status(job_id: str):
    """Check job status."""
    job = await job_store.get(job_id)
    if not job:
        raise HTTPException(status_code=404, detail="Job not found")
    return job
```

## Project Structure

```
execution/
├── main.py                 # FastAPI app entry
├── api/
│   ├── __init__.py
│   ├── invoices.py         # Invoice endpoints
│   ├── webhooks.py         # Webhook receivers
│   ├── customers.py        # Customer CRUD
│   └── jobs.py             # Background job triggers
├── schemas/
│   ├── __init__.py
│   ├── invoice.py          # Invoice Pydantic models
│   └── customer.py         # Customer Pydantic models
├── services/
│   ├── __init__.py
│   ├── qbo_service.py      # QuickBooks API client
│   └── stripe_service.py   # Stripe API client
└── core/
    ├── config.py           # Settings from .env
    ├── database.py         # DB connection (if needed)
    └── security.py         # Auth helpers
```

## Essential Dependencies

```txt
# requirements.txt
fastapi>=0.109.0
uvicorn>=0.27.0
pydantic>=2.5.0
pydantic-settings>=2.1.0
httpx>=0.26.0
python-dotenv>=1.0.0
```

## Running Locally

```bash
# Install
pip install -r requirements.txt

# Run
uvicorn execution.main:app --reload --port 8000

# Docs
open http://localhost:8000/docs
```

## Deployment Checklist

- [ ] All endpoints have Pydantic request/response models
- [ ] Error handling returns proper status codes
- [ ] Logging includes context (IDs, user info)
- [ ] Secrets in `.env`, not hardcoded
- [ ] Health endpoint at `/health`
- [ ] CORS configured if needed
- [ ] Rate limiting for public endpoints
