---
name: 37signals-rails-style
description: Apply 37signals/DHH Rails conventions when writing Ruby on Rails code. Use when building Rails applications, reviewing Rails code, or making architectural decisions. Covers various aspects of Rails application architecture, design and dependencies.
---

# 37signals/DHH Rails Style Guide

## Core Philosophy

- **"Vanilla Rails is plenty."** Maximize what Rails gives you, minimize dependencies, resist abstractions until necessary.
- **Rich domain models** over service objects
- **CRUD controllers** over custom actions
- **Concerns** for horizontal code sharing
- **Records as state** over boolean columns
- **Database-backed everything** (no Redis)
- **Build it yourself** before reaching for gems

---

## Dependencies

### Use
- Rails (edge), turbo-rails, stimulus-rails, importmap-rails, propshaft, solid_queue, solid_cache, solid_cable (database-backed, NO Redis), geared_pagination, bcrypt, rqrcode, redcarpet

### Avoid

| Gem/Pattern            | Why                               |
|------------------------|-----------------------------------|
| `devise`               | Auth is ~150 lines of custom code |
| `pundit`/`cancancan`   | Authorization lives in models     |
| `dry-rb`, `interactor` | Over-engineered                   |
| `view_component`       | ERB partials are fine             |
| `sidekiq`, `redis`     | Use Solid Queue (database-backed) |
| `graphql`              | REST with Turbo is sufficient     |

---

## Routing: Everything is CRUD

Every action maps to a CRUD verb. Create new resources instead of custom actions:

```ruby
# Avoid: Custom actions
resources :cards do
  post :close
end

# Good: State changes as resources
resources :cards do
  resource :closure      # POST to close, DELETE to reopen
  resource :pin          # POST to pin, DELETE to unpin
  resource :watch        # POST to watch, DELETE to unwatch
end
```

Use `scope module:` for namespaced nested resources. Use `resolve` for custom polymorphic URL generation.

---

## Controllers

### Thin Controllers, Rich Models

Controllers orchestrate; business logic lives in models.

```ruby
def create
  @card.close  # All logic in model
  respond_to do |format|
    format.turbo_stream { render_card_replacement }
    format.json { head :no_content }
  end
end
```

### Controller Concerns

Use concerns for shared behavior:
- **Resource scoping**: `CardScoped`, `BoardScoped` - load parent resources via `before_action`
- **Request context**: `CurrentRequest` - populate `Current` with request data
- **Security**: `BlockSearchEngineIndexing`, `RequestForgeryProtection`
- **Turbo helpers**: `TurboFlash` - flash messages via Turbo Stream

### Authorization

Check permissions in controller, define permission logic in model:

```ruby
# Controller
before_action :ensure_permission_to_administer_card, only: [:destroy]

# Model
def can_administer_card?(card)
  admin? || card.creator == self
end
```

---

## Models & Concerns

### Heavy Use of Concerns

Each concern is self-contained with associations, scopes, and methods:

```ruby
class Card < ApplicationRecord
  include Assignable, Closeable, Eventable, Pinnable, Watchable
end
```

### Concern Structure

```ruby
module Card::Closeable
  extend ActiveSupport::Concern

  included do
    has_one :closure, dependent: :destroy
    scope :closed, -> { joins(:closure) }
    scope :open, -> { where.missing(:closure) }
  end

  def closed? = closure.present?

  def close(user: Current.user)
    create_closure!(user: user) unless closed?
  end
end
```

### Default Values via Lambdas

```ruby
belongs_to :account, default: -> { board.account }
belongs_to :creator, class_name: "User", default: -> { Current.user }
```

### Current for Request Context

Use `ActiveSupport::CurrentAttributes` for session, user, identity, account, and request metadata.

### POROs (Plain Old Ruby Objects)

Namespace under parent model: `Event::Description`, `Card::Eventable::SystemCommenter`

Use for:
- **Presentation logic** - formatting for display
- **Complex operations** - multi-step processes
- **View context bundling** - collecting UI state

POROs are model-adjacent, NOT controller-adjacent (that would be a service object).

---

## State as Records, Not Booleans

Create separate records instead of boolean columns:

```ruby
# Separate record gives you: timestamp, who did it, easy scoping
class Closure < ApplicationRecord
  belongs_to :card, touch: true
  belongs_to :user, optional: true
end

card.closure.present?     # Is it closed?
card.closure.user         # Who closed it?
card.closure.created_at   # When?

# Scoping
Card.closed  # joins(:closure)
Card.open    # where.missing(:closure)
```

Examples: `Closure`, `Pin`, `Watch`, `Publication`, `Goldness`

---

## Authentication

Custom passwordless magic link auth (~150 lines). No Devise.

Key components:
- `Authentication` concern with `require_authentication`, `resume_session`, `start_new_session_for`
- `Session` model (belongs_to identity)
- `MagicLink` model with expiration and consumption
- Bearer token authentication for API access

---

## Views & Turbo/Hotwire

- **Turbo Streams** for partial updates (`turbo_stream.replace`, `turbo_stream.before`)
- **Morphing** for complex updates (`method: :morph`)
- **Partials over ViewComponents** - standard ERB partials with caching
- **Stimulus controllers** - single-purpose, small (~50 lines), `static values`/`static classes` for config, `this.dispatch()` for events, `this.#privateMethod()` for private methods

---

## Background Jobs

- **Shallow jobs, rich models** - jobs just call model methods
- **`_later` and `_now` convention** - `mark_as_read_later` queues job, `mark_as_read_now` executes immediately
- **Solid Queue** - database-backed, no Redis
- **Recurring jobs** via `config/recurring.yml`

---

## Testing

- **Request specs** for controllers (not controller specs)
- **Ship tests with features** in the same commit
- Use `change { }`, `as: :turbo_stream`, `as: :json`

---

## What They Avoid

- **No service objects** - use model methods
- **No form objects** (usually) - exception: `Signup` as ActiveModel
- **No decorators/presenters** - use view helpers
- **No GraphQL** - REST with Turbo

---

## Naming Conventions

### Methods
- **Verbs for actions**: `close`, `reopen`, `publish`
- **Predicates for state**: `closed?`, `published?`

### Concerns
Adjectives describing capability: `Closeable`, `Publishable`, `Watchable`, `Searchable`

### Controllers
Nouns matching the resource: `Cards::ClosuresController`, `Boards::PublicationsController`

### Scopes
- **Ordering**: `chronologically`, `reverse_chronologically`, `alphabetically`, `latest`
- **Preloading**: `preloaded` as standard name for eager loading
- **Parameterized**: `indexed_by(index)`, `sorted_by(sort)`

---

## Caching

### HTTP Caching
- `fresh_when etag: [...]` for conditional GET
- Global `etag { "v1" }` in ApplicationController (bump to bust caches)
- Concern-level ETags for timezone, authentication

### Fragment Caching
- `cache card do` in views
- `cached: true` for collection rendering
- `touch: true` on associations for cache invalidation

---

## Database Patterns

- **UUIDs** for primary keys
- **Every model has `account_id`** for multi-tenancy
- **URL-based multi-tenancy**: `/{account_id}/boards/...`
- **No foreign key constraints** - removed for flexibility

---

## CSS Architecture

- **Vanilla CSS** no Sass, PostCSS, or Tailwind.
- **CSS Cascade Layers** `@layer reset, base, components, modules, utilities`
- **OKLCH color system** with CSS variables
- **Modern features** `@starting-style`, `color-mix()`, `:has()`, native nesting, container queries

---

## API Design

- Same controllers, different format via `respond_to`
- Response codes: Create → `201 Created` + Location, Update/Delete → `204 No Content`
- Bearer token authentication

---

## Callbacks

Use sparingly:
- `after_commit :relay_later, on: :create` for async work
- `before_save :set_defaults` for derived data
- Avoid complex chains, avoid synchronous external calls

---

## Summary

1. **Start with vanilla Rails** - Don't add abstractions until you feel the pain
2. **Models are rich** - Business logic lives in models, not services
3. **Controllers are thin** - Just orchestration and response formatting
4. **Everything is CRUD** - New resource over new action
5. **State is records** - Not boolean columns
6. **Concerns are compositions** - Horizontal behavior sharing
7. **Build before buying** - Auth, search, jobs - all custom
8. **Database is king** - No Redis, no Elasticsearch
9. **Test with fixtures** - Deterministic, fast, simple
10. **Ship incrementally** - Many small commits
11. **Tests ship with features** - Not TDD, not afterthought, but together
12. **Refactor toward consistency** - Establish patterns, then update old code
13. **CSS uses the platform** - Native layers, nesting, OKLCH - no preprocessors
14. **Design tokens everywhere** - CSS variables for colors, spacing, typography

The best code is the code you don't write. The second best is the code that's obviously correct.
