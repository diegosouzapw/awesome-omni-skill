---
name: ActiveRecord Patterns
description: This skill should be used when the user asks about "ActiveRecord", "database queries", "query optimization", "N+1 queries", "eager loading", "associations", "migrations", "database indexes", "SQL performance", "ActiveRecord callbacks", "scopes", or needs guidance on efficient database access patterns in Rails 7+.
version: 1.0.0
---

# ActiveRecord Patterns for Production Rails

Production-focused guidance for ActiveRecord query optimization, associations, migrations, and database best practices for Rails 7+.

## Query Optimization

### Avoiding N+1 Queries

Always use `includes`, `preload`, or `eager_load` when accessing associations:

```ruby
# Bad - N+1 query
Order.all.each { |o| puts o.user.email }

# Good - eager loading
Order.includes(:user).each { |o| puts o.user.email }

# For nested associations
Order.includes(line_items: :product).each do |order|
  order.line_items.each { |li| puts li.product.name }
end
```

**When to use each:**
- `includes` - Rails decides (usually `preload`)
- `preload` - Separate queries, works with `limit`
- `eager_load` - LEFT OUTER JOIN, needed for filtering

```ruby
# Filtering on association - must use eager_load
Order.eager_load(:line_items)
     .where(line_items: { product_id: 123 })
```

### Select Only Needed Columns

```ruby
# Bad - loads all columns
User.all.map(&:email)

# Good - loads only needed columns
User.pluck(:email)

# When you need objects with limited columns
User.select(:id, :email, :name)
```

### Batch Processing

```ruby
# Bad - loads all records into memory
User.all.each { |u| u.update(last_contacted_at: Time.current) }

# Good - processes in batches
User.find_each(batch_size: 1000) do |user|
  user.update(last_contacted_at: Time.current)
end

# For parallelization
User.in_batches(of: 1000) do |batch|
  batch.update_all(last_contacted_at: Time.current)
end
```

### Counting and Existence

```ruby
# Bad - loads records to count
User.all.count        # Loads all, then counts
User.all.length       # Same problem

# Good - database count
User.count            # SELECT COUNT(*)

# For checking existence
User.any?             # Avoid - can be slow
User.exists?          # SELECT 1 LIMIT 1 - fast
User.where(admin: true).exists?
```

## Associations

### Association Options

```ruby
class Order < ApplicationRecord
  belongs_to :user, counter_cache: true
  belongs_to :created_by, class_name: "User", optional: true

  has_many :line_items, dependent: :destroy
  has_many :products, through: :line_items

  has_one :invoice, dependent: :destroy

  # Polymorphic
  has_many :comments, as: :commentable

  # Self-referential
  belongs_to :parent_order, class_name: "Order", optional: true
  has_many :child_orders, class_name: "Order", foreign_key: :parent_order_id
end
```

### Counter Cache

```ruby
# Migration
add_column :users, :orders_count, :integer, default: 0, null: false

# Reset existing counts
User.find_each do |user|
  User.reset_counters(user.id, :orders)
end

# Model
class Order < ApplicationRecord
  belongs_to :user, counter_cache: true
end

# Usage - no query needed
user.orders_count
```

### Inverse Of

Define explicitly for complex associations:

```ruby
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end

class LineItem < ApplicationRecord
  belongs_to :order, inverse_of: :line_items
end

# Prevents extra queries when building
order = Order.new
order.line_items.build(quantity: 1)
order.line_items.first.order  # Same object, no query
```

## Scopes and Queries

### Named Scopes

```ruby
class Order < ApplicationRecord
  scope :recent, -> { where("created_at > ?", 30.days.ago) }
  scope :pending, -> { where(status: :pending) }
  scope :completed, -> { where(status: :completed) }
  scope :for_user, ->(user) { where(user: user) }

  # Composable scopes
  scope :recent_pending, -> { recent.pending }

  # Scope with default
  scope :by_status, ->(status = :pending) { where(status: status) }
end

# Chaining
Order.recent.pending.for_user(current_user)
```

### Class Methods vs Scopes

Use class methods for complex logic:

```ruby
class Order < ApplicationRecord
  def self.search(query)
    return all if query.blank?

    where("order_number ILIKE ? OR notes ILIKE ?",
          "%#{query}%", "%#{query}%")
  end

  def self.total_revenue
    completed.sum(:total)
  end
end
```

### Arel for Complex Queries

```ruby
class Order < ApplicationRecord
  def self.with_high_value_items
    line_items_table = LineItem.arel_table
    orders_table = arel_table

    joins(:line_items)
      .where(line_items_table[:price].gt(100))
      .distinct
  end
end
```

## Migrations

### Production-Safe Migrations

```ruby
class AddIndexToOrdersUserIdConcurrently < ActiveRecord::Migration[7.1]
  disable_ddl_transaction!

  def change
    add_index :orders, :user_id, algorithm: :concurrently,
              if_not_exists: true
  end
end
```

### Column Additions with Defaults

```ruby
# Rails 7+ handles this safely
class AddStatusToOrders < ActiveRecord::Migration[7.1]
  def change
    add_column :orders, :status, :string, default: "pending", null: false
  end
end
```

### Safe Column Removal

```ruby
# Step 1: Stop using column in code (deploy first)
# Step 2: Ignore column
class Order < ApplicationRecord
  self.ignored_columns += ["legacy_status"]
end

# Step 3: Remove column (separate deploy)
class RemoveLegacyStatusFromOrders < ActiveRecord::Migration[7.1]
  def change
    safety_assured { remove_column :orders, :legacy_status }
  end
end
```

### Indexing Strategy

```ruby
# Foreign keys - always index
add_index :orders, :user_id

# Composite index for common queries
add_index :orders, [:user_id, :status]

# Partial index for specific conditions
add_index :orders, :created_at,
          where: "status = 'pending'",
          name: "index_orders_pending_created_at"

# Unique constraint
add_index :users, :email, unique: true

# GIN index for array/JSON columns
add_index :products, :tags, using: :gin
```

## Transactions

### Basic Transactions

```ruby
Order.transaction do
  order = Order.create!(order_params)
  order.line_items.create!(line_item_params)
  InventoryService.deduct(order)
end
```

### Nested Transactions

```ruby
Order.transaction do
  order.update!(status: :processing)

  Order.transaction(requires_new: true) do
    # This savepoint can fail independently
    PaymentService.charge(order)
  rescue PaymentError => e
    order.update!(payment_error: e.message)
  end
end
```

### Advisory Locks

```ruby
# Prevent concurrent processing
Order.with_advisory_lock("order_#{order.id}") do
  process_order(order)
end
```

## Callbacks

### Safe Callback Patterns

```ruby
class Order < ApplicationRecord
  # Good: Simple, side-effect free callbacks
  before_validation :normalize_email
  before_save :set_defaults

  private

  def normalize_email
    self.email = email&.downcase&.strip
  end

  def set_defaults
    self.order_number ||= generate_order_number
  end
end
```

### When to Avoid Callbacks

Move business logic to service objects:

```ruby
# Avoid: Complex callback chains
class Order < ApplicationRecord
  after_create :send_confirmation, :update_inventory, :notify_warehouse
end

# Prefer: Explicit service object
class Orders::CreateService
  def call(params)
    Order.transaction do
      order = Order.create!(params)
      OrderMailer.confirmation(order).deliver_later
      Inventory::DeductService.new(order).call
      Warehouse::NotifyJob.perform_later(order.id)
      order
    end
  end
end
```

## Connection Management

### Connection Pool Configuration

```yaml
# config/database.yml
production:
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  checkout_timeout: 5
  reaping_frequency: 10
```

### Read Replicas (Rails 6+)

```ruby
# config/database.yml
production:
  primary:
    database: myapp_production
  primary_replica:
    database: myapp_production
    replica: true

# Usage
ActiveRecord::Base.connected_to(role: :reading) do
  Order.where(status: :pending).count
end

# Automatic switching
class ApplicationController < ActionController::Base
  around_action :switch_to_replica, only: [:index, :show]

  private

  def switch_to_replica
    ActiveRecord::Base.connected_to(role: :reading) { yield }
  end
end
```

## Additional Resources

### Reference Files

For detailed patterns and advanced techniques:
- **`references/query-patterns.md`** - Complex query patterns, CTEs, window functions
- **`references/migration-safety.md`** - Zero-downtime migration strategies
