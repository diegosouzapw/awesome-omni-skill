---
name: rails-dhh
description: DHH/37signals Rails conventions — CRUD controllers, rich models with concerns, Hotwire, vanilla CSS, Solid Stack, Minitest.
---

# Rails the DHH/37signals Way

This skill encodes the complete DHH/37signals Rails philosophy extracted from The Rails Doctrine, production patterns from 37signals' open-source codebases (Fizzy, Campfire, Writebook), DHH's Rails World 2025 keynote, and 265 analyzed pull requests.

**Core thesis: "Vanilla Rails is plenty."** Maximize what Rails gives out of the box. Minimize dependencies. Resist abstractions until absolutely necessary. Delete more than you add.

## Reference Guides

For detailed patterns on specific topics, read the companion files in this skill directory:

- **`hotwire.md`** — Turbo Drive/Frames/Streams/Morphing decision framework, Stimulus controller patterns, anti-patterns, gotchas
- **`css.md`** — Vanilla CSS with `@layer`, OKLCH color system, `:has()`, container queries, `@starting-style`
- **`testing.md`** — Minitest with fixtures, integration tests, DHH's testing philosophy
- **`deployment.md`** — Kamal 2, Solid Queue production config, safe migrations, Rails 8.1 features

---

## Quick Reference — Before Writing Code

Ask yourself:

1. **Can I use an existing Rails convention?** If yes, use it.
2. **Am I creating a new abstraction?** Does it earn its keep? Can you point to 3+ variations?
3. **Am I adding a dependency?** Can you build it yourself in fewer lines than the gem's README?
4. **Is this a new action on an existing controller?** Make it a new resource instead.
5. **Am I using a boolean column for state?** Use a record instead.
6. **Am I putting business logic in the controller?** Move it to the model.
7. **Am I writing a service object?** Put the logic in a concern on the model.
8. **Am I reaching for Turbo Streams?** Could morphing or a simple redirect handle it?
9. **Am I writing a Stimulus controller?** Could Turbo handle this without JS?
10. **Am I writing a system test?** Write an integration test instead.
11. **Am I writing CSS with JavaScript?** Could `:has()`, container queries, or `@starting-style` handle it?
12. **Is this migration safe?** Would it lock a large table? Use `strong_migrations`.
13. **Does this code earn its place?** Would DHH call it "anemic"?

---

## Philosophy — The Eight Pillars

1. **Rich domain models** over service objects
2. **CRUD controllers** over custom actions
3. **Concerns** for horizontal code sharing
4. **Records as state** over boolean columns
5. **Database-backed everything** (Solid Stack, no Redis)
6. **Build it yourself** before reaching for gems
7. **Ship, Validate, Refine** — deploy prototype quality, iterate on real feedback
8. **Delete more than you add** — fight "anemic" code (thin wrappers that explain nothing)

---

## The Stack

| Component        | Tool                          | NOT This              |
|------------------|-------------------------------|-----------------------|
| Framework        | Rails 8.x                    |                       |
| Ruby             | 3.2+ (3.3+ for YJIT)        |                       |
| Frontend         | Hotwire (Turbo 8 + Stimulus) | React, Vue, Svelte   |
| JavaScript       | Importmap-rails (no Node.js) | Webpack, esbuild, Vite |
| CSS              | Vanilla CSS with `@layer`    | Tailwind, Sass, PostCSS |
| Background Jobs  | Solid Queue                  | Sidekiq, Resque       |
| Caching          | Solid Cache                  | Redis, Memcached      |
| WebSockets       | Solid Cable                  | Redis-backed ActionCable |
| Asset Pipeline   | Propshaft                    | Sprockets             |
| Testing          | Minitest + fixtures          | RSpec + FactoryBot    |
| Auth             | `rails generate authentication` | Devise             |
| Deployment       | Kamal 2 + Thruster           | Heroku, Fly.io        |
| Server           | Puma                         |                       |

---

## Architecture Rules

### Everything is CRUD

Every action maps to a CRUD verb. When something doesn't fit, create a new resource:

```ruby
# ❌ BAD: Custom actions on existing resource
resources :cards do
  post :close
  post :reopen
end

# ✅ GOOD: New resources for each state change
resources :cards do
  resource :closure      # POST to close, DELETE to reopen
  resource :pin          # POST to pin, DELETE to unpin
  resource :watch        # POST to watch, DELETE to unwatch
end
```

Controllers have only the 7 RESTful actions: `index`, `show`, `new`, `create`, `edit`, `update`, `destroy`. If you need something else, you need a new controller.

### Thin Controllers, Rich Models

Controllers orchestrate. Models contain business logic:

```ruby
class Cards::ClosuresController < ApplicationController
  include CardScoped

  def create
    @card.close  # All logic lives in the model
    respond_to do |format|
      format.turbo_stream { render_card_replacement }
      format.html { redirect_to @card.board }
    end
  end

  def destroy
    @card.reopen
    respond_to do |format|
      format.turbo_stream { render_card_replacement }
      format.html { redirect_to @card.board }
    end
  end
end
```

### State as Records, Not Booleans

Instead of `closed: boolean`, create a separate model. This gives you who did it, when, and easy scoping for free:

```ruby
# Migration
create_table :closures do |t|
  t.references :card, null: false, foreign_key: true
  t.references :user, foreign_key: true
  t.timestamps
end

# Model
class Closure < ApplicationRecord
  belongs_to :card, touch: true
  belongs_to :user, optional: true
end

# Concern on parent
module Card::Closeable
  extend ActiveSupport::Concern

  included do
    has_one :closure, dependent: :destroy
    scope :closed, -> { joins(:closure) }
    scope :open, -> { where.missing(:closure) }
  end

  def close(user: Current.user)
    unless closed?
      transaction do
        create_closure!(user: user)
        track_event(:closed, creator: user)
      end
    end
  end

  def closed?
    closure.present?
  end

  def reopen
    closure&.destroy!
  end
end
```

### Concerns for Horizontal Sharing

Models include many small, self-contained concerns:

```ruby
class Card < ApplicationRecord
  include Assignable, Closeable, Eventable, Pinnable,
    Searchable, Taggable, Watchable

  belongs_to :board
  belongs_to :creator, class_name: "User", default: -> { Current.user }
end
```

Each concern owns its associations, scopes, validations, and methods. A concern should be understandable in isolation.

Controller concerns share behavior across controllers:

```ruby
module CardScoped
  extend ActiveSupport::Concern

  included do
    before_action :set_card
  end

  private
    def set_card
      @card = Card.find(params[:card_id])
    end
end
```

### Database-Backed Everything

No Redis. Use the Solid Stack:
- `solid_queue` for background jobs
- `solid_cache` for caching
- `solid_cable` for WebSockets

Configure separate databases in `config/database.yml`:

```yaml
production:
  primary:
    <<: *default
    database: app_production
  queue:
    <<: *default
    database: app_production_queue
    migrations_paths: db/queue_migrate
  cache:
    <<: *default
    database: app_production_cache
    migrations_paths: db/cache_migrate
  cable:
    <<: *default
    database: app_production_cable
    migrations_paths: db/cable_migrate
```

### Fight Anemic Code

DHH's test: **Does this abstraction earn its keep?** If you can't point to 3+ variations that need an abstraction, inline it.

- If a private method just wraps one line, inline it
- If a concern has one method, put it on the model
- If a helper reformats one value, do it in the view
- If a service object just calls one model method, remove it

### Data Integrity at the Database Level

Enforce constraints in the database, not just application validations:

```ruby
create_table :cards do |t|
  t.string :title, null: false
  t.references :board, null: false, foreign_key: true
  t.integer :position, null: false, default: 0
  t.timestamps
end

add_index :cards, [:board_id, :position], unique: true
```

---

## What NOT to Do

| Pattern                 | Why It's Avoided                                    | Instead                          |
|-------------------------|-----------------------------------------------------|----------------------------------|
| Service objects         | Business logic belongs in models                    | Rich models with concerns        |
| Form objects            | Rarely needed                                       | ActiveModel when absolutely needed |
| Decorators/Presenters   | Extra indirection                                   | Helpers + partials               |
| GraphQL                 | Over-engineered for most apps                       | REST + Turbo                     |
| Devise                  | Auth is ~150 lines of custom code                   | `rails generate authentication`  |
| RSpec                   | Minitest is simpler and faster                      | Minitest + fixtures              |
| FactoryBot             | Fixtures are faster and simpler                     | YAML fixtures                    |
| Tailwind               | Vanilla CSS with modern features is sufficient      | `@layer`, nesting, OKLCH, `:has()` |
| Redis                  | Database-backed everything                          | Solid Stack                      |
| ViewComponent          | Extra abstraction over ERB                          | Partials                         |
| Sidekiq                | Requires Redis                                      | Solid Queue                      |
| `dry-rb` gems          | Over-engineered for most Rails apps                 | Plain Ruby                       |
| Interactor/Command     | Service objects by another name                     | Model methods                    |
| `pundit`/`cancancan`   | Authorization logic belongs in models               | Model methods + concerns         |
| React/Vue/Svelte       | Conflicts with Hotwire server-first approach        | Turbo + Stimulus                 |
| Sass/PostCSS           | Native CSS is powerful enough now                   | Vanilla CSS                      |
| System tests           | "Always brittle, always broken, always slow" — DHH  | Controller integration tests     |

---

## Code Style

### Ruby Conventions (from Fizzy)

```ruby
# Symbol arrays with spaces inside brackets
before_action :set_message, only: %i[ show edit update destroy ]

# Private method indentation — 2 spaces after `private`
private
  def set_message
    @message = Message.find(params[:id])
  end

# Bang methods for fail-fast
@message = Message.create!(message_params)

# Ternaries for simple conditionals
@room.direct? ? "DM" : @room.name

# Default values via lambdas
belongs_to :creator, class_name: "User", default: -> { Current.user }

# Expression-less case for complex conditionals
case
when params[:before].present?
  messages.page_before(params[:before])
else
  messages.last_page
end

# Current for request context
Current.user
Current.session
Current.account
```

### Naming Conventions

- **Verb methods for actions:** `card.close`, `card.reopen`, `card.gild`
- **Predicate methods for state:** `card.closed?`, `card.golden?`, `card.pinned?`
- **Concerns are adjectives:** `Closeable`, `Publishable`, `Watchable`, `Taggable`
- **Controllers are nouns:** `Cards::ClosuresController`, `Cards::PinsController`
- **`_later` and `_now` for async patterns:**

```ruby
def mark_as_read_later(user:)
  MarkCardAsReadJob.perform_later(self, user)
end

def mark_as_read_now(user:)
  readings.find_or_create_by!(user: user).touch
end
```

### Background Jobs — Shallow Wrappers

Jobs just call model methods. All logic stays in the model:

```ruby
class NotifyRecipientsJob < ApplicationJob
  def perform(notifiable)
    notifiable.notify_recipients
  end
end
```

Recurring jobs in `config/recurring.yml`:

```yaml
deliver_bundled_notifications:
  command: "Notification::Bundle.deliver_all_later"
  schedule: every 30 minutes

cleanup_magic_links:
  command: "MagicLink.cleanup"
  schedule: every 4 hours
```

### HTTP Caching

Use ETags and `fresh_when` aggressively:

```ruby
class ApplicationController < ActionController::Base
  etag { "v1" }  # Bump to bust all caches on deploy
end

# In specific controllers
fresh_when etag: [@board, @page.records]
```

---

## Hotwire Decision Framework (Summary)

> **Full reference with code examples and anti-patterns:** Read `hotwire.md` in this directory.

```
1. Turbo Drive (free, zero code — handles all page navigation)
        ↓ not enough?
2. Morphing Page Refreshes (same-URL redirects, broadcasts_refreshes)
        ↓ not enough?
3. Turbo Frames (one self-contained region: inline edit, lazy load, modal)
        ↓ not enough?
4. Turbo Streams (multiple unrelated DOM updates from one response)
        ↓ not enough?
5. Stimulus (client-side interactivity: toggles, clipboard, animations)
        ↓ not enough?
6. Custom JS (last resort, wrap in Stimulus controller)
```

- **Different URL?** → Turbo Drive handles it. Do nothing.
- **Same-page redirect after form?** → Morphing: `turbo_refreshes_with method: :morph, scroll: :preserve`
- **Real-time multi-user?** → `broadcasts_refreshes` on model + `touch: true` on children
- **One self-contained region?** → Turbo Frame
- **Multiple unrelated DOM updates?** → Turbo Streams (HTTP response)
- **Client-side interactivity only?** → Stimulus controller

---

## File Organization

```
app/
├── controllers/
│   ├── concerns/             # Authentication, CardScoped, etc.
│   ├── cards/
│   │   ├── closures_controller.rb
│   │   ├── pins_controller.rb
│   │   └── watches_controller.rb
│   ├── cards_controller.rb
│   └── application_controller.rb
├── models/
│   ├── concerns/             # Closeable, Pinnable, Watchable, etc.
│   ├── card/
│   │   └── closeable.rb      # Namespaced concerns for complex models
│   ├── card.rb
│   ├── closure.rb
│   ├── current.rb            # CurrentAttributes
│   └── application_record.rb
├── views/
│   ├── cards/
│   │   ├── _card.html.erb
│   │   ├── index.html.erb
│   │   └── show.html.erb
│   ├── cards/closures/
│   │   └── create.turbo_stream.erb
│   ├── layouts/
│   └── shared/               # Shared partials
├── javascript/
│   └── controllers/          # Stimulus controllers
├── assets/
│   └── stylesheets/          # Flat structure, one file per concept
│       ├── application.css   # @layer imports only
│       ├── _reset.css
│       ├── base.css
│       ├── colors.css        # OKLCH + dark mode
│       ├── buttons.css
│       ├── inputs.css
│       └── utilities.css
├── jobs/                     # Shallow wrappers
└── mailers/
config/
├── database.yml              # PostgreSQL + Solid databases
├── deploy.yml                # Kamal 2
├── recurring.yml             # Solid Queue recurring jobs
├── solid_queue.yml           # Worker/dispatcher config
└── importmap.rb              # JS dependencies (no Node.js)
test/
├── fixtures/                 # YAML fixtures
├── models/
├── controllers/              # Integration tests (NOT system tests)
└── test_helper.rb
```
