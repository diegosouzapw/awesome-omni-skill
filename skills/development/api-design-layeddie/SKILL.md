---
name: api-design
description: REST vs GraphQL, API versioning, rate limiting, and documentation for Elixir/BEAM APIs
---

# API Design Skill

Use this skill when:
- Designing API endpoints
- Choosing between REST and GraphQL
- Implementing API versioning strategies
- Setting up rate limiting
- Creating API documentation
- Defining pagination and filtering

## API Architecture Patterns

### REST vs GraphQL

### REST API

```elixir
# ✅ Good: Resource-based routing
defmodule MyAppWeb.API.UsersController do
  use MyAppWeb, :controller

  action :index, %{"page" => page} do
    users = MyApp.Users.list_users(page: page, per_page: 20)
    render(conn, "index.json", %{users: users, page: page})
  end

  action :show, %{"id" => id} do
    user = MyApp.Users.get(id)
    render(conn, "show.json", %{user: user})
  end
  end
end

# ❌ Bad: Over-fetching N+1
defmodule MyAppWeb.API.BadUsersController do
  use MyAppWeb, :controller

  action :show, %{"id" => id} do
    user = MyApp.Users.get(id)  # Single query
    comments = MyApp.Comments.by_user(id)  # N+1 query!
    render(conn, "show.json", %{user: user, comments: comments})
  end
  end
end
```

### GraphQL API

```elixir
# ✅ Good: Type-safe schema with Absinthe
defmodule MyAppWeb.Schema do
  use Absinthe.Schema

  @desc "A user object"
  object :user do
    field :id, non_null(:id), :id
    field :name, non_null(:name), :string
    field :email, non_null(:email), :string
    field :comments, list_of(:comment)
  end

  @desc "All users query"
  query :users, type: :user do
    arg :limit, :integer
    resolve fn %{limit: limit}, _, _ do
      MyApp.Users.list_users(limit: limit)
    end
  end
end
```

### Decision Criteria

| Requirement | REST | GraphQL |
|-----------|------|----------|
| **Simple CRUD** | ✅ Better | ⚠️ Overkill |
| **Complex queries** | ⚠️ Multiple requests | ✅ Single request |
| **Type safety** | ⚠️ Runtime | ✅ Compile-time |
| **Over-fetching** | ✅ Explicit queries | ⚠️ Requires careful design |
| **Caching** | ✅ Easy (HTTP) | ⚠️ Requires field-level |
| **Introspection** | ✅ Easy (OpenAPI) | ⚠️ Requires tools |
| **Mobile** | ✅ Mature | ⚠️ Growing |
| **Real-time** | ⚠️ Polling | ✅ Subscriptions |

### Recommendation

**Use REST when:**
- Simple CRUD operations
- File uploads
- Mobile clients
- Need type safety not critical
- Caching at HTTP level

**Use GraphQL when:**
- Complex nested queries
- Multiple data sources in single request
- Type safety critical
- Real-time subscriptions needed
- Field-level caching beneficial

## API Versioning

### URL Versioning

```elixir
# Version in URL path
defmodule MyAppWeb.Router do
  scope "/api/v1", MyAppWeb do
    pipe_through :api

    resources "/users", UserController
  end

  scope "/api/v2", MyAppWeb do
    pipe_through :api

    resources "/users", UserControllerV2
  end
end
```

### Header Versioning

```elixir
defmodule MyAppWeb.APIVersionMiddleware do
  @behaviour Plug

  def init(opts), do: opts

  def call(conn, opts) do
    version = Application.get_env(:my_app, :api_version, "1.0")
    conn
    |> put_resp_header("api-version", version)
    |> put_resp_header("content-type", "application/json")
    |> assign(:api_version, version)
  end
end
```

### Semantic Versioning

```elixir
# Major version for breaking changes
defmodule MyAppWeb.Schemas.UserV1 do
  use Ecto.Schema

  embedded_schema, primary_key: false do
    field :name, :string
    field :email, :string
  end

  embedded_schema, primary_key: true do
    field :id, :id
    field :name, :string
    field :email, :string
    field :preferences, :map
  end
end

defmodule MyAppWeb.Schemas.UserV2 do
  use Ecto.Schema

  embedded_schema, primary_key: true do
    field :id, :id
    field :name, :string
    field :email, :string
    field :preferences, :map
    field :avatar_url, :string
  end
end

defmodule MyAppWeb.API.UsersController do
  def index(conn, params) do
    version = get_req_header(conn, "api-version")
    users = case version do
      "1.0" -> MyApp.Users.list_users_v1()
      "2.0" -> MyApp.Users.list_users_v2()
      _ -> MyApp.Users.list_users_v1()
    end
    render(conn, "index.json", %{users: users, version: version})
  end
end
```

### Deprecation Strategy

```elixir
defmodule MyAppWeb.DeprecationMiddleware do
  @behaviour Plug

  def init(opts), do: opts

  def call(conn, opts) do
    path = request_path(conn)
    version = get_req_header(conn, "api-version")
    
    if path in ["/api/v1/old_endpoint"] do
      conn
      |> put_resp_header("x-api-deprecation-date", "2024-01-01")
      |> put_resp_header("x-api-deprecated", "true")
      |> put_resp_header("x-api-replaced-by", "api/v2")
      |> put_resp_header("x-api-replacement-date", "2024-06-01")
    end
    
    assign(conn, :deprecation_warning, version && path in deprecation_paths())
  end

  defp deprecation_paths do
    Application.get_env(:my_app, :deprecation_paths, [])
  end
end
```

## Rate Limiting

### Token Bucket Algorithm

```elixir
defmodule MyApp.RateLimiter do
  use GenServer

  def start_link(_opts) do
    GenServer.start_link(__MODULE__, %{})
  end

  @impl true
  def init(state) do
    {:ok, state}
  end

  @impl true
  def handle_call({:check_rate_limit, user_id}, _from, state) do
    key = "user:#{user_id}"
    case :ets.lookup_element(:rate_limiter_table, key) do
      {:miss, count} -> {:reply, {:ok, true, count + 1}, state}
      {:hit, count} when count < 10 -> {:reply, {:ok, false, count + 1}, state}
      {:hit, count} -> {:reply, {:ok, false, count + 1}, state}  # Rate limited
    end
  end

  @impl true
  def handle_cast({:reset, user_id}, _from, state) do
    key = "user:#{user_id}"
    :ets.delete_element(:rate_limiter_table, key)
    {:noreply, state}
  end

  defp schedule_reset(user_id, delay_ms) do
    Process.send_after(self(), {:reset, user_id}, delay_ms)
  end
end
```

### Plug Integration

```elixir
defmodule MyAppWeb.RateLimiterPlug do
  use Plug

  def init(opts) do
    opts
  end

  def call(conn, opts) do
    user_id = get_user_id(conn)
    
    case MyApp.RateLimiter.check_rate_limit(user_id) do
      {:ok, :allowed, remaining} ->
        conn
        |> put_resp_header("x-ratelimit-remaining", to_string(remaining))
        |> put_resp_header("x-ratelimit-reset", to_string(60))
        assign(conn, :rate_limited, false)
      
      {:ok, :limited, _remaining} ->
        conn
        |> put_resp_header("retry-after", to_string(60))
        |> assign(:rate_limited, true)
        |> halt()
    end
  end
end
```

## API Documentation

### OpenAPI Integration

```elixir
# Use Absinthe.Plug for auto-documentation
defmodule MyAppWeb.APIRouter do
  use Absinthe.Plug,
      spec: "priv/api_spec.json",
      for: MyAppWeb

  pipeline :api do
    plug :accepts, ["application/json"]
    plug :rate_limiter
  end

  scope "/api", MyAppWeb do
    resources "/users", UsersController
  end
end
```

### API Spec Example (priv/api_spec.json)

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "My App API",
    "version": "1.0.0"
  },
  "paths": {
    "/users": {
      "get": {
        "summary": "List all users",
        "operationId": "listUsers",
        "responses": {
          "200": {
            "description": "Users list response",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/User"
                }
              }
            }
          }
        }
      },
      "post": {
        "summary": "Create new user",
        "operationId": "createUser",
        "requestBody": {
          "required": ["name", "email"],
          "schema": {
            "type": "object",
            "properties": {
              "name": {"type": "string"},
              "email": {"type": "string"}
            }
          }
        }
      }
    }
  }
}
```

### ExDoc Documentation

```elixir
# Document public modules
defmodule MyAppWeb.API do
  @moduledoc """
  API module for user management.
  
  ## Endpoints
  
  ### GET /api/users
  Lists all users with pagination.
  
  ### GET /api/users/:id
  Get a specific user by ID.
  
  ### POST /api/users
  Creates a new user.
  
  ### PUT /api/users/:id
  Updates an existing user.
  
  ### DELETE /api/users/:id
  Deletes a user.
  
  ## Authentication
  
  All endpoints require a valid JWT token.
  Use the Authorization header with format: `Bearer <token>`.
  
  ## Rate Limiting
  
  Default rate limits:
  - 10 requests per minute per user
  - 100 requests per hour per user
  
  ## Errors
  
  Standard error responses:
  - 400: Bad Request - Invalid input
  - 401: Unauthorized - Missing or invalid token
  - 403: Forbidden - Insufficient permissions
  - 404: Not Found - Resource not found
  - 429: Too Many Requests - Rate limit exceeded
  - 500: Internal Server Error
  """
end
```

## Best Practices

### 1. Consistent Response Format
```elixir
# ✅ Good: Consistent structure
def render_success(conn, data) do
  conn
  |> put_status(200)
  |> json(%{success: true, data: data})
end

# ❌ Bad: Inconsistent responses
def render_success(conn, data) do
  case data do
    %{users: users} -> json(conn, %{status: "success", users: users})
    %{error: error} -> json(conn, %{error: error})
  end
end
```

### 2. Proper HTTP Status Codes

| Code | Usage |
|------|-------|
| **200** | ✅ Success |
| **201** | ✅ Created |
| **204** | ✅ No Content |
| **400** | ✅ Bad Request |
| **401** | ✅ Unauthorized |
| **403** | ✅ Forbidden |
| **404** | ✅ Not Found |
| **409** | ✅ Conflict |
| **429** | ✅ Too Many Requests |
| **500** | ✅ Internal Error |

### 3. Pagination

```elixir
# ✅ Good: Cursor-based pagination
defmodule MyApp.Users do
  alias MyApp.Repo

  def list_users(page: page, per_page: 20) do
    offset = (page - 1) * per_page
    query = from(u in User, order_by: [asc: u.inserted_at], limit: ^per_page, offset: ^offset)
    Repo.all(query)
  end
end

# ❌ Bad: Offset-based pagination (easier for unlimited scrolling)
defmodule MyApp.Users do
  alias MyApp.Repo

  def list_users(offset: 0, limit: 20) do
    query = from(u in User, order_by: [asc: u.inserted_at], limit: ^limit, offset: ^offset)
    Repo.all(query)
  end
end
```

### 4. Filtering and Sorting

```elixir
# ✅ Good: Parameterized filters
defmodule MyApp.Users do
  alias MyApp.Repo

  def list_users(filters: %{"search" => search, "role" => role}) do
  query = from(u in User,
      where: [ilike: ^filters.search, u.role == ^filters.role],
      order_by: [asc: u.name])
    Repo.all(query)
  end
end

# ❌ Bad: String interpolation (SQL injection risk)
defmodule MyApp.Users do
  alias MyApp.Repo

  def list_users(search: search, role: role) do
    query = "SELECT * FROM users WHERE name LIKE '%#{search}%' AND role = '#{role}'"
    Repo.query(query)
  end
end
```

### 5. Error Handling

```elixir
# ✅ Good: Structured errors with typespecs
defmodule MyApp.API.Error do
  @moduledoc """
  Structured API error responses.
  """
  
  @type t :: :error
  @type reason :: :string

  @enforce_keys [type: :atom]
  
  @spec new(t, reason, changeset) :: t()
  def new(type, reason, changeset) do
    %__MODULE__{
      type: type,
      reason: reason,
      changeset: changeset
    }
  end
end

# ❌ Bad: Generic error responses
defmodule MyAppWeb.API do
  def render_error(message) do
    json(conn, %{error: message})
  end
end
```

## Token Efficiency

Use API design patterns for:
- **Single request optimization** (~30% token savings vs multiple requests)
- **Explicit data selection** (~40% token savings vs returning all fields)
- **Cursor-based pagination** (~50% token savings vs offset-based)
- **Efficient filtering** (~35% token savings vs returning all results)

## Tools to Use

- **Absinthe**: Automatic OpenAPI spec generation
- **ExDoc**: Comprehensive documentation generation
- **Phoenix.Swagger**: Alternative API documentation
- **Plug**: Request/response processing
- **Plug.CSRFProtection**: CSRF token validation
- **Joken**: JWT authentication and validation
- **Ecto**: Database queries with parameterization

### Ash Igniter Integration

**Critical**: Nix environment must match what Igniter expects

When using Igniter with Ash:
- **Architect Phase**: Consult Nix specialist for environment setup
- **Orchestrator Phase**: Set up Nix devshell with correct Elixir version
- **Implementation Phase**: Ensure Ash packages work with Nix-provided Elixir
- **Review Phase**: Verify Ash package works with selected version

**Nix specialist** provides:
- Guidance on version selection for Igniter compatibility
- Multiple environment configurations (stable vs testing)
- Version testing strategies before full implementation

**Workflow**:
```bash
# 1. Nix Specialist (Planning)
nix-specialist:
  "We need Ash 3.4+ for this project"
  "Nix 1.17+ supports this"

# 2. Architect (Design with Igniter guidance)
architect:
  "Use Igniter to explore Ash 3.4+"
  "Nix specialist confirmed 1.17+ compatibility"

# 3. Orchestrator (Implementation)
orchestrator:
  "Setting up Nix devshell"
  "Use Ash 3.4+ from Nix"

# 4. Reviewer (Verify)
reviewer:
  "Check Ash package compatibility"
  "Verify Nix environment"
```

**Resources**:
- Ash Igniter: https://github.com/ash-project/igniter
- Ash Official Docs: https://hexdocs.pm/ash
- Nix Specialist: `roles/nix-specialist.md`
- Phoenix Storybook: https://github.com/phenixdigital/phoenix_storybook/fork
