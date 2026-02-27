---
name: rails-architecture
description: Guides modern Rails 8 code architecture decisions and patterns. Use when deciding where to put code, choosing between patterns (service objects vs concerns vs query objects), designing feature architecture, refactoring for better organization, or when user mentions architecture, code organization, design patterns, or layered design.
allowed-tools: Read, Glob, Grep
---

# Modern Rails 8 Architecture Patterns

## Project Conventions
- **Testing:** Minitest + fixtures (NEVER RSpec or FactoryBot)
- **Components:** ViewComponents for reusable UI (partials OK for simple one-offs)
- **Authorization:** Pundit policies (deny by default)
- **Jobs:** Solid Queue, shallow jobs, `_later`/`_now` naming
- **Frontend:** Hotwire (Turbo + Stimulus) + Tailwind CSS
- **State:** State-as-records for business state (booleans only for technical flags)
- **Architecture:** Rich models first, service objects for multi-model orchestration
- **Routing:** Everything-is-CRUD (new resource over new action)
- **Quality:** RuboCop (omakase) + Brakeman

## Architecture Decision Tree

```
Where should this code go?
│
├─ Is it data validation, associations, or simple business logic?
│   └─ → Model (rich models first!)
│
├─ Is it shared behavior across models?
│   └─ → Concern
│
├─ Is it business state tracking (who/when/why)?
│   └─ → State Record (see: state-records pattern)
│
├─ Does it orchestrate 3+ models or call external APIs?
│   └─ → Service Object (with Result pattern)
│
├─ Is it a complex database query (3+ joins, aggregations)?
│   └─ → Query Object
│
├─ Is it view/display formatting?
│   └─ → Presenter (SimpleDelegator)
│
├─ Is it authorization logic?
│   └─ → Pundit Policy
│
├─ Is it reusable UI with logic?
│   └─ → ViewComponent
│
├─ Is it async/background work?
│   └─ → Shallow Job (Solid Queue)
│
├─ Is it a complex form (multi-model, wizard)?
│   └─ → Form Object
│
├─ Is it a transactional email?
│   └─ → Mailer
│
└─ Is it HTTP request/response handling only?
    └─ → Controller (keep it thin!)
```

## Hybrid Philosophy: Models First, Services When Needed

### The Rule of Three
- **1 model affected** → Keep logic in the model
- **2 models affected** → Consider a concern or model method
- **3+ models affected** → Extract to a service object

### Rich Models (Default)
Models handle validations, associations, scopes, simple derived attributes, and single-model business logic. This is where most code belongs.

```ruby
class Order < ApplicationRecord
  include Closeable  # State-as-records concern

  belongs_to :user
  has_many :line_items, dependent: :destroy

  validates :total_cents, presence: true, numericality: { greater_than: 0 }

  scope :recent, -> { order(created_at: :desc) }
  scope :pending, -> { where.missing(:closure) }

  def add_item(product, quantity: 1)
    line_items.create!(product: product, quantity: quantity, price_cents: product.price_cents)
    recalculate_total!
  end

  private

  def recalculate_total!
    update!(total_cents: line_items.sum("price_cents * quantity"))
  end
end
```

### Service Objects (When Justified)
Use only when logic spans 3+ models, calls external APIs, or orchestrates complex workflows.

```ruby
module Orders
  class CheckoutService
    def call(user:, cart:, payment_method_id:)
      order = nil

      ActiveRecord::Base.transaction do
        order = user.orders.create!(total_cents: cart.total_cents)
        cart.items.each { |item| order.add_item(item.product, quantity: item.quantity) }
        Inventory::ReserveService.new.call(order: order)
      end

      Payments::ChargeService.new.call(order: order, payment_method_id: payment_method_id)
      OrderMailer.confirmation(order).deliver_later
      Result.new(success: true, data: order)
    rescue ActiveRecord::RecordInvalid => e
      Result.new(success: false, error: e.message)
    end
  end
end
```

## Everything-is-CRUD Routing

Prefer creating a new resource over adding custom actions:

```ruby
# GOOD: New resource for publishing
resources :posts do
  resource :publication, only: [:create, :destroy]
end
# POST /posts/:post_id/publication → Publications#create
# DELETE /posts/:post_id/publication → Publications#destroy

# BAD: Custom action
resources :posts do
  member do
    post :publish
    post :unpublish
  end
end
```

## Layer Responsibilities

| Layer | Responsibility | Should NOT contain |
|-------|---------------|-------------------|
| **Controller** | HTTP, params, authorize, render | Business logic, queries |
| **Model** | Data, validations, relations, scopes | Display logic, HTTP |
| **Concern** | Shared model/controller behavior | Unrelated cross-cutting logic |
| **Service** | Multi-model orchestration, external APIs | HTTP, display logic |
| **Query** | Complex database queries, reports | Business logic |
| **Presenter** | View formatting, badges | Business logic, queries |
| **Policy** | Authorization rules | Business logic |
| **Component** | Reusable UI encapsulation | Business logic |
| **Job** | Async delegation (shallow!) | Business logic |

## Project Directory Structure

```
app/
├── channels/            # Action Cable channels
├── components/          # ViewComponents (UI + logic)
├── controllers/
│   └── concerns/        # Shared controller behavior
├── forms/               # Form objects
├── jobs/                # Background jobs (Solid Queue)
├── mailers/             # Action Mailer classes
├── models/
│   └── concerns/        # Shared model behavior
├── policies/            # Pundit authorization
├── presenters/          # View formatting
├── queries/             # Complex queries
├── services/            # Business logic (use sparingly)
│   └── result.rb        # Shared Result class
└── views/
    └── components/      # ViewComponent templates
```

## When NOT to Abstract

| Situation | Keep It Simple | Don't Create |
|-----------|----------------|--------------|
| Simple CRUD (< 10 lines) | Keep in controller | Service object |
| Used only once | Inline the code | Abstraction |
| Simple query with 1-2 conditions | Model scope | Query object |
| Basic text formatting | Helper method | Presenter |
| Single model form | `form_with model:` | Form object |
| Simple partial without logic | Partial | ViewComponent |

## When TO Abstract

| Signal | Action |
|--------|--------|
| Same code in 3+ places | Extract to concern/service |
| Controller action > 15 lines | Extract to service |
| Model > 300 lines | Extract concerns |
| Complex conditionals | Extract to policy/service |
| Query joins 3+ tables | Extract to query object |
| Form spans multiple models | Extract to form object |
| Partial has > 5 lines of logic | Use ViewComponent |

## Result Object Pattern

All services return a consistent Result:

```ruby
# app/services/result.rb
class Result
  attr_reader :data, :error, :code

  def initialize(success:, data: nil, error: nil, code: nil)
    @success = success
    @data = data
    @error = error
    @code = code
  end

  def success? = @success
  def failure? = !@success

  def self.success(data = nil) = new(success: true, data: data)
  def self.failure(error, code: nil) = new(success: false, error: error, code: code)
end
```

## Testing Strategy by Layer

| Layer | Test Type | Location | Focus |
|-------|-----------|----------|-------|
| Model | Unit | `test/models/` | Validations, scopes, methods |
| Service | Unit | `test/services/` | Business logic, edge cases |
| Query | Unit | `test/queries/` | Query results, correctness |
| Presenter | Unit | `test/presenters/` | Formatting, HTML output |
| Controller | Integration | `test/controllers/` | HTTP flow, authorization |
| Component | Component | `test/components/` | Rendering, variants |
| Policy | Unit | `test/policies/` | Authorization rules |
| System | E2E | `test/system/` | Critical user paths |

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| God Model | Model > 500 lines | Extract concerns |
| Fat Controller | Logic in controllers | Move to models/services |
| Premature Service | Service for 3 lines | Keep in model |
| Callback Hell | Complex model callbacks | Use services for orchestration |
| Boolean State | `approved: true` | State-as-records |
| N+1 Queries | Unoptimized queries | Use `.includes()` |

## References

- See [layer-interactions.md](reference/layer-interactions.md) for layer communication patterns
- See [service-patterns.md](reference/service-patterns.md) for service object patterns
- See [query-patterns.md](reference/query-patterns.md) for query object patterns
- See [error-handling.md](reference/error-handling.md) for error handling strategies
- See [testing-strategy.md](reference/testing-strategy.md) for comprehensive testing
- See [multi-tenancy.md](reference/multi-tenancy.md) for multi-tenant patterns
- See [event-tracking.md](reference/event-tracking.md) for domain event patterns
- See [state-records.md](reference/state-records.md) for state-as-records patterns
