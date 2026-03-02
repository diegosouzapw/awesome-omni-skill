---
name: ash-json-api
description: AshJsonApi guidelines for exposing Ash resources as JSON:API compliant REST endpoints. Use when adding JSON:API extensions to domains/resources, configuring routes, or implementing filtering, sorting, pagination, and includes. Supports full JSON:API specification.
---

# AshJsonApi Guidelines

AshJsonApi generates JSON:API compliant endpoints from Ash resources, supporting filtering, sorting, pagination, includes, and relationships.

## Domain Configuration

Add the `AshJsonApi.Domain` extension and define routes:

```elixir
defmodule MyApp.Blog do
  use Ash.Domain,
    extensions: [AshJsonApi.Domain]

  json_api do
    authorize? true

    routes do
      base_route "/posts", MyApp.Blog.Post do
        get :read
        index :read
        post :create
        patch :update
        delete :destroy
      end
    end
  end

  resources do
    resource MyApp.Blog.Post
    resource MyApp.Blog.Comment
  end
end
```

## Resource Configuration

Add the `AshJsonApi.Resource` extension and specify the type:

```elixir
defmodule MyApp.Blog.Post do
  use Ash.Resource,
    domain: MyApp.Blog,
    extensions: [AshJsonApi.Resource]

  attributes do
    uuid_primary_key :id
    attribute :title, :string
    attribute :body, :string
    attribute :published, :boolean
  end

  relationships do
    belongs_to :author, MyApp.Accounts.User
    has_many :comments, MyApp.Blog.Comment
  end

  json_api do
    type "post"  # Required: JSON:API type name
  end

  actions do
    defaults [:create, :read, :update, :destroy]

    read :list_published do
      filter expr(published == true)
    end

    update :publish do
      accept []
      change set_attribute(:published, true)
    end
  end
end
```

## Route Types

| Route | HTTP | Path | Description |
|-------|------|------|-------------|
| `get` | GET | `/posts/:id` | Fetch single resource |
| `index` | GET | `/posts` | List resources (filter/sort/paginate) |
| `post` | POST | `/posts` | Create resource |
| `patch` | PATCH | `/posts/:id` | Update resource |
| `delete` | DELETE | `/posts/:id` | Destroy resource |
| `related` | GET | `/posts/:id/comments` | Fetch related resources |
| `relationship` | GET | `/posts/:id/relationships/comments` | Fetch relationship data |
| `post_to_relationship` | POST | `/posts/:id/relationships/comments` | Add to relationship |
| `patch_relationship` | PATCH | `/posts/:id/relationships/comments` | Replace relationship |
| `delete_from_relationship` | DELETE | `/posts/:id/relationships/comments` | Remove from relationship |

## Query Parameters

Standard JSON:API query parameters:

```
# Filter
?filter[attribute]=value

# Sort (descending with -)
?sort=attribute,-other_attribute

# Pagination
?page[number]=2&page[size]=10

# Includes (sideload related resources)
?include=author,comments.author
```
