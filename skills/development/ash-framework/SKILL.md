---
name: ash-framework
description: Comprehensive Ash framework guidelines for Elixir applications. Use when working with Ash resources, domains, actions, queries, changesets, policies, calculations, or aggregates. Covers code interfaces, error handling, validations, changes, relationships, and authorization. Read documentation before using Ash features - do not assume prior knowledge.
---

# Ash Framework Guidelines

Ash is a declarative framework for modeling domains with resources. **Read documentation before using features.**

## Code Interfaces

Define code interfaces on domains - avoid direct `Ash.get!/2` calls in LiveViews/Controllers:

```elixir
# In domain
resource Post do
  define :get_post, action: :read, get_by: [:id]
  define :list_posts, action: :read
  define :create_post, action: :create, args: [:title]
end

# Usage - prefer query option over manual Ash.Query building
posts = MyApp.Blog.list_posts!(
  query: [filter: [status: :published], sort: [published_at: :desc], limit: 10],
  load: [author: :profile, comments: [:author]]
)

post = MyApp.Blog.get_post!(id, load: [comments: [:author]])
```

**Authorization functions** are auto-generated: `can_create_post?(actor)`, `can_update_post?(actor, post)`.

**Using scopes**: Pass `scope: socket.assigns.scope` in LiveViews; use `context` parameter in hooks.

## Actions

- Create specific, well-named actions (not generic CRUD)
- Put business logic inside action definitions
- Use `before_action/after_action` for same-transaction logic
- Use `before_transaction/after_transaction` for external calls

```elixir
actions do
  create :sign_up do
    argument :invite_code, :string, allow_nil?: false
    change set_attribute(:joined_at, &DateTime.utc_now/0)
    change relate_actor(:creator)
  end
end
```

## Querying

**Important**: `Ash.Query.filter/2` is a macro - you must `require Ash.Query`:

```elixir
require Ash.Query

Post
|> Ash.Query.filter(status == :published)
|> Ash.Query.sort(published_at: :desc)
|> Ash.Query.load([:author, :comments])
|> Ash.Query.limit(10)
|> Ash.read!()
```

## Error Handling

- Use `!` variations (`Ash.create!`, `Domain.action!`) when expecting success
- Prefer `!` over pattern matching `{:ok, result} = ...`
- Error classes: `Forbidden` > `Invalid` > `Framework` > `Unknown`

## Validations

```elixir
# Built-in validations
validate compare(:age, greater_than_or_equal_to: 18)
validate match(:email, ~r/@/)
validate one_of(:status, [:active, :pending])

# Conditional validation
validate present(:phone) do
  where eq(:contact_method, "phone")
end

# Skip if prior validations failed
validate expensive_check() do
  only_when_valid? true
end
```

**Avoid redundant validations** - don't duplicate attribute constraints (`allow_nil? false` already validates presence).

## Changes

```elixir
# Built-in changes
change set_attribute(:status, "pending")
change relate_actor(:creator)
change atomic_update(:counter, expr(counter + 1))

# Custom change module
defmodule MyApp.Changes.Slugify do
  use Ash.Resource.Change

  def change(changeset, _opts, _context) do
    title = Ash.Changeset.get_attribute(changeset, :title)
    slug = title |> String.downcase() |> String.replace(~r/[^a-z0-9]+/, "-")
    Ash.Changeset.change_attribute(changeset, :slug, slug)
  end
end
```

**Prefer custom modules** over anonymous functions for changes, validations, preparations.

## Preparations

Modify queries before execution:

```elixir
prepare build(sort: [created_at: :desc])
prepare build(filter: [deleted: false])

# Conditional preparation
prepare build(filter: [visible: true]) do
  where argument_equals(:include_hidden, false)
end
```

## Data Layers

```elixir
use Ash.Resource,
  domain: MyApp.Blog,
  data_layer: AshPostgres.DataLayer  # or :embedded, Ash.DataLayer.Ets

postgres do
  table "posts"
  repo MyApp.Repo
end
```

## Migrations

Run `mix ash.codegen <name>` after modifying resources. Use `--dev` during development, final name at end.

## Testing

- Test through code interfaces
- Use `authorize?: false` when auth isn't the focus
- Use `Ash.can?` to test policies
- Use raising `!` functions

**Prevent deadlocks** - use unique values for identity fields in concurrent tests:

```elixir
%{email: "test-#{System.unique_integer([:positive])}@example.com"}
```

## References

- **Authorization & Policies**: See [references/policies.md](references/policies.md)
- **Relationships**: See [references/relationships.md](references/relationships.md)
- **Calculations & Aggregates**: See [references/calculations.md](references/calculations.md)
