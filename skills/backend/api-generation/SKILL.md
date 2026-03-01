---
name: api-generation
description: Generate TypeScript API client from Swagger/Go comments. Use when updating API endpoints, adding new routes, or regenerating the frontend API client after backend changes.
allowed-tools: Read, Edit, Write, Bash, Glob, Grep
---

# API Generation

Generate typed TypeScript API client from Go Swagger comments using swag + Orval.

## Workflow

```
Go Handler → swag init → swagger.json → Orval → TypeScript Hooks
```

## One Command for Everything

```bash
make api
```

This executes:
1. `swag init` → Generates `backend/docs/swagger.json` from Go comments
2. `orval` → Generates TypeScript Hooks in `frontend/src/shared/api/`

## Adding a New Endpoint

### Step 1: Go Handler with Swagger Comments

```go
// internal/handler/product.go

// GetProducts godoc
// @Summary List all products
// @Description Get all products for the authenticated user
// @Tags products
// @Accept json
// @Produce json
// @Success 200 {array} ProductResponse
// @Failure 401 {object} ErrorResponse
// @Security BearerAuth
// @Router /products [get]
func (h *ProductHandler) GetProducts(w http.ResponseWriter, r *http.Request) {
    // Implementation
}

// CreateProduct godoc
// @Summary Create a product
// @Description Create a new product
// @Tags products
// @Accept json
// @Produce json
// @Param request body CreateProductRequest true "Product data"
// @Success 201 {object} ProductResponse
// @Failure 400 {object} ErrorResponse
// @Failure 401 {object} ErrorResponse
// @Security BearerAuth
// @Router /products [post]
func (h *ProductHandler) CreateProduct(w http.ResponseWriter, r *http.Request) {
    // Implementation
}
```

### Step 2: Define Response/Request Types

```go
// In handler or separate types.go

type ProductResponse struct {
    ID    string  `json:"id" example:"prod_123"`
    Name  string  `json:"name" example:"Widget"`
    Price float64 `json:"price" example:"29.99"`
}

type CreateProductRequest struct {
    Name  string  `json:"name" example:"Widget"`
    Price float64 `json:"price" example:"29.99"`
}
```

### Step 3: Generate API Client

```bash
make api
```

### Step 4: Use in Frontend

```tsx
import { useGetProducts, usePostProducts } from "@/api/endpoints/products/products"

function ProductsPage() {
  const { data, isLoading } = useGetProducts()
  const createProduct = usePostProducts()

  const handleCreate = () => {
    createProduct.mutate({ data: { name: "New", price: 10 } })
  }

  return (...)
}
```

## Swagger Annotation Reference

| Annotation | Description |
|------------|-------------|
| `@Summary` | Short description |
| `@Description` | Detailed description |
| `@Tags` | Grouping (becomes folder in endpoints/) |
| `@Accept` | Request Content-Type |
| `@Produce` | Response Content-Type |
| `@Param` | Parameter (body, query, path) |
| `@Success` | Success response |
| `@Failure` | Error response |
| `@Security` | Auth requirement |
| `@Router` | HTTP path and method |

## Generated Files

```
frontend/src/shared/api/
├── endpoints/
│   ├── products/        # Grouped by @Tags
│   │   └── products.ts  # useGetProducts, usePostProducts, etc.
│   └── users/
│       └── users.ts
├── models/              # TypeScript Types
└── custom-fetch.ts      # Fetch Wrapper with Auth
```

## Important Rules

- **Never** manually edit generated files
- **Always** run `make api` after handler changes
- Tags become folder names → `@Tags products` → `endpoints/products/`
- operationId is automatically generated from Router path
