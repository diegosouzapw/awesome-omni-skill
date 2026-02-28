---
name: "ActiveRecord Query Patterns"
description: "Complete guide to ActiveRecord query optimization, associations, scopes, and PostgreSQL-specific patterns. Use this skill when writing database queries, designing model associations, creating migrations, optimizing query performance, or debugging N+1 queries and grouping errors."
---

# ActiveRecord Query Patterns Skill

This skill provides comprehensive guidance for writing efficient, correct ActiveRecord queries in Rails applications with PostgreSQL.

## When to Use This Skill

- Writing complex ActiveRecord queries
- Designing model associations
- Creating database migrations
- Optimizing query performance
- Debugging N+1 queries
- Working with GROUP BY operations
- Implementing scopes and query objects

## Model Structure

### Standard Model Template

```ruby
# app/models/task.rb
class Task < ApplicationRecord
  # == Constants ============================================================
  STATUSES = %w[pending in_progress completed failed cancelled].freeze

  # == Associations =========================================================
  belongs_to :account
  belongs_to :merchant
  belongs_to :carrier, optional: true
  belongs_to :recipient
  belongs_to :zone, optional: true
  
  has_many :timelines, dependent: :destroy
  has_many :task_actions, dependent: :destroy
  has_many :photos, dependent: :destroy

  # == Validations ==========================================================
  validates :status, presence: true, inclusion: { in: STATUSES }
  validates :tracking_number, presence: true, uniqueness: { scope: :account_id }

  # == Scopes ===============================================================
  scope :active, -> { where.not(status: %w[completed failed cancelled]) }
  scope :completed, -> { where(status: 'completed') }
  scope :for_carrier, ->(carrier) { where(carrier: carrier) }
  scope :created_between, ->(start_date, end_date) { where(created_at: start_date..end_date) }
  scope :by_status, ->(status) { where(status: status) if status.present? }

  # == Callbacks ============================================================
  before_validation :generate_tracking_number, on: :create
  after_commit :notify_recipient, on: :create

  # == Class Methods ========================================================
  def self.search(query)
    where("tracking_number ILIKE :q OR description ILIKE :q", q: "%#{query}%")
  end

  # == Instance Methods =====================================================
  def completable?
    %w[pending in_progress].include?(status)
  end

  def complete!
    update!(status: 'completed', completed_at: Time.current)
  end

  private

  def generate_tracking_number
    self.tracking_number ||= SecureRandom.hex(8).upcase
  end

  def notify_recipient
    TaskNotificationJob.perform_later(id)
  end
end
```

## Association Patterns

### Basic Associations

```ruby
# One-to-Many
class Account < ApplicationRecord
  has_many :users, dependent: :destroy
  has_many :tasks, dependent: :destroy
end

class User < ApplicationRecord
  belongs_to :account
end

# Many-to-Many (with join table)
class Task < ApplicationRecord
  has_many :task_tags, dependent: :destroy
  has_many :tags, through: :task_tags
end

class Tag < ApplicationRecord
  has_many :task_tags, dependent: :destroy
  has_many :tasks, through: :task_tags
end

class TaskTag < ApplicationRecord
  belongs_to :task
  belongs_to :tag
end

# Polymorphic
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

class Task < ApplicationRecord
  has_many :comments, as: :commentable
end

class Invoice < ApplicationRecord
  has_many :comments, as: :commentable
end
```

### Association Options

```ruby
class Task < ApplicationRecord
  # Foreign key specification
  belongs_to :creator, class_name: 'User', foreign_key: 'created_by_id'
  
  # Optional association
  belongs_to :carrier, optional: true
  
  # Counter cache
  belongs_to :merchant, counter_cache: true
  
  # Dependent options
  has_many :photos, dependent: :destroy      # Delete associated records
  has_many :logs, dependent: :nullify        # Set foreign key to NULL
  has_many :exports, dependent: :restrict_with_error  # Prevent deletion
  
  # Scoped association
  has_many :active_timelines, -> { where(active: true) }, class_name: 'Timeline'
  
  # Touch parent on update
  belongs_to :bundle, touch: true
end
```

## Query Patterns

### Basic Queries

```ruby
# Find
Task.find(1)                    # Raises RecordNotFound
Task.find_by(id: 1)             # Returns nil if not found
Task.find_by!(id: 1)            # Raises RecordNotFound

# Where
Task.where(status: 'pending')
Task.where(status: %w[pending in_progress])  # IN query
Task.where.not(status: 'completed')
Task.where(created_at: 1.week.ago..)         # Range (>= date)
Task.where(created_at: ..1.week.ago)         # Range (<= date)
Task.where(created_at: 1.month.ago..1.week.ago)  # Between

# Order
Task.order(created_at: :desc)
Task.order(:status, created_at: :desc)

# Limit & Offset
Task.limit(10).offset(20)

# Distinct
Task.distinct.pluck(:status)
```

### Avoiding N+1 Queries

```ruby
# WRONG - N+1 query
tasks = Task.all
tasks.each { |t| puts t.carrier.name }  # Query per task!

# CORRECT - Eager loading
tasks = Task.includes(:carrier)
tasks.each { |t| puts t.carrier.name }  # Single query

# Multiple associations
Task.includes(:carrier, :merchant, :recipient)

# Nested associations
Task.includes(merchant: :branches)

# With conditions on association (use joins or references)
Task.includes(:carrier).where(carriers: { active: true }).references(:carriers)
# OR
Task.joins(:carrier).where(carriers: { active: true })
```

### Choosing Loading Strategy

```ruby
# includes - Smart loading (preload or eager_load based on usage)
Task.includes(:carrier)

# preload - Separate queries (can't filter on association)
Task.preload(:carrier)
# SELECT * FROM tasks
# SELECT * FROM carriers WHERE id IN (...)

# eager_load - Single LEFT JOIN query
Task.eager_load(:carrier)
# SELECT tasks.*, carriers.* FROM tasks LEFT JOIN carriers...

# joins - INNER JOIN (no loading, just filtering)
Task.joins(:carrier).where(carriers: { active: true })
```

### GROUP BY Queries (Critical for PostgreSQL)

**Rule**: Every non-aggregated column in SELECT must appear in GROUP BY.

```ruby
# CORRECT - Only grouped columns and aggregates
Task.group(:status).count
# => { "pending" => 10, "completed" => 25 }

Task.group(:status).sum(:amount)
# => { "pending" => 1000, "completed" => 5000 }

# CORRECT - Multiple GROUP BY columns
Task
  .group(:status, :task_type)
  .count
# => { ["pending", "express"] => 5, ["completed", "standard"] => 10 }

# CORRECT - Explicit select with aggregates
Task
  .select(:status, 'COUNT(*) as task_count', 'AVG(amount) as avg_amount')
  .group(:status)

# CORRECT - Date grouping
Task
  .group("DATE(created_at)")
  .count

# WRONG - includes with group
Task.includes(:carrier).group(:status).count  # ERROR!

# CORRECT - Separate queries if you need associated data
status_counts = Task.group(:status).count
tasks_by_status = status_counts.keys.each_with_object({}) do |status, hash|
  hash[status] = Task.where(status: status).includes(:carrier).limit(5)
end
```

### Subqueries

```ruby
# Subquery in WHERE
active_carrier_ids = Carrier.where(active: true).select(:id)
Task.where(carrier_id: active_carrier_ids)
# SELECT * FROM tasks WHERE carrier_id IN (SELECT id FROM carriers WHERE active = true)

# Subquery with join
Task.where(carrier_id: Carrier.active.select(:id))
    .where(merchant_id: Merchant.premium.select(:id))
```

### Raw SQL (When Needed)

```ruby
# Safe with sanitization
Task.where("created_at > ?", 1.week.ago)
Task.where("description ILIKE ?", "%#{query}%")

# Named bindings
Task.where("status = :status AND amount > :min", status: 'pending', min: 100)

# Select with raw SQL
Task.select("*, amount * 0.1 as commission")

# Find by SQL
Task.find_by_sql(["SELECT * FROM tasks WHERE status = ?", 'pending'])
```

## Scope Patterns

### Simple Scopes

```ruby
class Task < ApplicationRecord
  scope :active, -> { where.not(status: %w[completed cancelled]) }
  scope :completed, -> { where(status: 'completed') }
  scope :recent, -> { order(created_at: :desc) }
  scope :today, -> { where(created_at: Time.current.all_day) }
end
```

### Parameterized Scopes

```ruby
class Task < ApplicationRecord
  scope :by_status, ->(status) { where(status: status) }
  scope :created_after, ->(date) { where('created_at >= ?', date) }
  scope :for_carrier, ->(carrier_id) { where(carrier_id: carrier_id) }
  
  # With default
  scope :recent, ->(limit = 10) { order(created_at: :desc).limit(limit) }
  
  # Conditional scope
  scope :by_status_if_present, ->(status) { where(status: status) if status.present? }
end
```

### Chainable Scopes

```ruby
# All scopes are chainable
Task.active.recent.by_status('pending').for_carrier(123)

# Combine with where
Task.active.where(merchant_id: 456)
```

## Query Objects

```ruby
# app/queries/tasks/pending_delivery_query.rb
module Tasks
  class PendingDeliveryQuery
    def initialize(relation = Task.all)
      @relation = relation
    end

    def call(zone_id: nil, since: 24.hours.ago)
      result = @relation
        .where(status: 'pending')
        .where('created_at >= ?', since)
        .includes(:carrier, :recipient)
      
      result = result.where(zone_id: zone_id) if zone_id.present?
      result.order(created_at: :asc)
    end
  end
end

# Usage
Tasks::PendingDeliveryQuery.new.call(zone_id: 123)
Tasks::PendingDeliveryQuery.new(account.tasks).call(since: 1.hour.ago)
```

## Migration Patterns

### Create Table

```ruby
class CreateTasks < ActiveRecord::Migration[7.1]
  def change
    create_table :tasks do |t|
      t.references :account, null: false, foreign_key: true
      t.references :merchant, null: false, foreign_key: true
      t.references :carrier, foreign_key: true  # nullable
      
      t.string :tracking_number, null: false
      t.string :status, null: false, default: 'pending'
      t.decimal :amount, precision: 10, scale: 2
      t.jsonb :metadata, default: {}
      
      t.datetime :completed_at
      t.timestamps
      
      t.index :tracking_number, unique: true
      t.index :status
      t.index [:account_id, :status]
      t.index [:merchant_id, :created_at]
      t.index :metadata, using: :gin  # For JSONB queries
    end
  end
end
```

### Safe Migrations

```ruby
# Add column with default (safe in PostgreSQL 11+)
class AddPriorityToTasks < ActiveRecord::Migration[7.1]
  def change
    add_column :tasks, :priority, :integer, default: 0, null: false
  end
end

# Add index concurrently (for large tables)
class AddIndexToTasksStatus < ActiveRecord::Migration[7.1]
  disable_ddl_transaction!

  def change
    add_index :tasks, :status, algorithm: :concurrently
  end
end

# Remove column safely
class RemoveOldColumnFromTasks < ActiveRecord::Migration[7.1]
  def change
    safety_assured { remove_column :tasks, :old_column, :string }
  end
end
```

### JSONB Columns

```ruby
# Migration
add_column :tasks, :metadata, :jsonb, default: {}
add_index :tasks, :metadata, using: :gin

# Model
class Task < ApplicationRecord
  # Using jsonb_accessor gem
  jsonb_accessor :metadata,
    priority: :integer,
    tags: [:string, array: true],
    notes: :string
end

# Queries
Task.where("metadata @> ?", { priority: 1 }.to_json)
Task.where("metadata->>'priority' = ?", '1')
Task.where("metadata ? 'special_flag'")
```

## Performance Optimization

### Batch Processing

```ruby
# WRONG - Loads all records into memory
Task.all.each { |task| process(task) }

# CORRECT - Batches of 1000
Task.find_each(batch_size: 1000) { |task| process(task) }

# With specific order
Task.order(:id).find_each { |task| process(task) }

# In batches (for batch operations)
Task.in_batches(of: 1000) do |batch|
  batch.update_all(processed: true)
end
```

### Select Only Needed Columns

```ruby
# WRONG - Loads all columns
users = User.all
users.each { |u| puts u.email }

# CORRECT - Only needed columns
users = User.select(:id, :email)
users.each { |u| puts u.email }

# With pluck (returns arrays, not AR objects)
emails = User.pluck(:email)
```

### Counter Caches

```ruby
# Migration
add_column :merchants, :tasks_count, :integer, default: 0

# Model
class Task < ApplicationRecord
  belongs_to :merchant, counter_cache: true
end

# Now merchant.tasks_count doesn't query
merchant.tasks_count  # Uses cached count
```

### Exists? vs Any? vs Present?

```ruby
# EFFICIENT - Stops at first match
Task.where(status: 'pending').exists?
# SELECT 1 FROM tasks WHERE status = 'pending' LIMIT 1

# LESS EFFICIENT - Loads records
Task.where(status: 'pending').any?
# May load records depending on implementation

# INEFFICIENT - Loads all records
Task.where(status: 'pending').present?
# SELECT * FROM tasks WHERE status = 'pending'
```

### Explain & Analyze

```ruby
# In Rails console
Task.where(status: 'pending').explain
Task.where(status: 'pending').explain(:analyze)

# Check for sequential scans on large tables
# Look for "Seq Scan" - may need index
```

## Debugging Queries

```bash
# In Rails console, enable query logging
ActiveRecord::Base.logger = Logger.new(STDOUT)

# Or in development.rb
config.active_record.verbose_query_logs = true

# Using bullet gem for N+1 detection
# Gemfile: gem 'bullet', group: :development
```

## Rails 7.x/8.x Modern Features

### Composite Primary Keys (Rails 7.1+)

```ruby
# Migration
class CreateBookOrders < ActiveRecord::Migration[7.1]
  def change
    create_table :book_orders, primary_key: [:shop_id, :id] do |t|
      t.integer :shop_id
      t.integer :id
      t.string :status
      t.timestamps
    end
  end
end

# Model
class BookOrder < ApplicationRecord
  self.primary_key = [:shop_id, :id]

  belongs_to :shop
  has_many :line_items, foreign_key: [:shop_id, :order_id]
end

# Usage
order = BookOrder.find([shop_id: 1, id: 100])
order.id # => { shop_id: 1, id: 100 }
```

### ActiveRecord::Encryption (Rails 7+)

**For encrypting sensitive data at rest:**

```ruby
# config/credentials.yml.enc
active_record_encryption:
  primary_key: <%= ENV['AR_ENCRYPTION_PRIMARY_KEY'] %>
  deterministic_key: <%= ENV['AR_ENCRYPTION_DETERMINISTIC_KEY'] %>
  key_derivation_salt: <%= ENV['AR_ENCRYPTION_KEY_DERIVATION_SALT'] %>

# Model
class User < ApplicationRecord
  encrypts :email               # Non-deterministic (can't query)
  encrypts :ssn, deterministic: true  # Deterministic (can query equality)
  encrypts :credit_card, ignore_case: true
end

# Queries with deterministic encryption
User.where(ssn: '123-45-6789')  # Works with deterministic: true
User.where(email: 'user@example.com')  # Doesn't work without deterministic

# Unencrypted reads (for migration)
class User < ApplicationRecord
  encrypts :email, ignore_case: true, previous: { ignore_case: false }
end
```

### Multi-Database Configuration (Rails 6.1+)

```ruby
# config/database.yml
production:
  primary:
    <<: *default
    database: my_primary_database
  analytics:
    <<: *default
    database: my_analytics_database
    replica: true
    migrations_paths: db/analytics_migrate

# Models
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
  connects_to database: { writing: :primary, reading: :primary }
end

class AnalyticsRecord < ActiveRecord::Base
  self.abstract_class = true
  connects_to database: { writing: :analytics, reading: :analytics }
end

class Event < AnalyticsRecord
end

# Switching databases
ActiveRecord::Base.connected_to(role: :reading) do
  # Read from replica
end

ActiveRecord::Base.connected_to(role: :writing) do
  # Write to primary
end

# Prevent writes
ActiveRecord::Base.connected_to(role: :reading, prevent_writes: true) do
  # Raises error on write
end
```

### Horizontal Sharding (Rails 7.1+)

```ruby
# config/database.yml
production:
  primary:
    database: my_primary_database
  shard_one:
    database: my_shard_one_database
  shard_two:
    database: my_shard_two_database

# Model
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
  connects_to shards: {
    shard_one: { writing: :shard_one },
    shard_two: { writing: :shard_two }
  }
end

# Usage
ActiveRecord::Base.connected_to(shard: :shard_one) do
  User.create!(name: "User in shard one")
end

ActiveRecord::Base.connected_to(shard: :shard_two) do
  User.create!(name: "User in shard two")
end

# Switching shards based on data
def with_user_shard(user_id)
  shard = user_id.even? ? :shard_one : :shard_two
  ActiveRecord::Base.connected_to(shard: shard) do
    yield
  end
end
```

### Enum Patterns with i18n

```ruby
# Model
class Task < ApplicationRecord
  enum status: {
    pending: 0,
    in_progress: 1,
    completed: 2,
    failed: 3,
    cancelled: 4
  }, _prefix: true  # status_pending?, status_completed?

  enum priority: {
    low: 0,
    medium: 1,
    high: 2,
    urgent: 3
  }, _suffix: true  # low_priority?, high_priority?

  # Auto-generated methods:
  # task.status               => "pending"
  # task.pending?             => true
  # task.status_pending?      => true (with prefix)
  # task.completed!           => Changes to completed
  # Task.statuses             => {"pending" => 0, "completed" => 2, ...}
  # Task.pending              => Scope for pending tasks
  # Task.not_pending          => Scope for non-pending tasks
end

# i18n
# config/locales/en.yml
en:
  activerecord:
    attributes:
      task:
        status:
          pending: "Pending"
          in_progress: "In Progress"
          completed: "Completed"
          failed: "Failed"
          cancelled: "Cancelled"

# Usage in views
<%= t("activerecord.attributes.task.status.#{task.status}") %>

# Or with enum_help gem
gem 'enum_help'
Task.human_attribute_name("status.#{task.status}")

# Scopes with enums
Task.pending           # SELECT * FROM tasks WHERE status = 0
Task.not_pending       # SELECT * FROM tasks WHERE status != 0
Task.where.not(status: :completed)
```

### Database Views

```ruby
# Migration
class CreateActiveTasksView < ActiveRecord::Migration[7.1]
  def up
    execute <<-SQL
      CREATE VIEW active_tasks AS
      SELECT
        tasks.*,
        merchants.name AS merchant_name,
        carriers.name AS carrier_name
      FROM tasks
      INNER JOIN merchants ON merchants.id = tasks.merchant_id
      LEFT JOIN carriers ON carriers.id = tasks.carrier_id
      WHERE tasks.status IN ('pending', 'in_progress')
    SQL
  end

  def down
    execute "DROP VIEW IF EXISTS active_tasks"
  end
end

# Model
class ActiveTask < ApplicationRecord
  # Read-only model backed by view
  self.primary_key = :id

  def readonly?
    true
  end
end

# Usage
ActiveTask.all
ActiveTask.where(merchant_name: "ACME Corp")

# Materialized Views (faster, but need refresh)
class CreateTaskSummaryView < ActiveRecord::Migration[7.1]
  def up
    execute <<-SQL
      CREATE MATERIALIZED VIEW task_summaries AS
      SELECT
        DATE(created_at) as date,
        status,
        COUNT(*) as count,
        AVG(amount) as average_amount
      FROM tasks
      GROUP BY DATE(created_at), status
    SQL

    add_index :task_summaries, :date
  end

  def down
    execute "DROP MATERIALIZED VIEW IF EXISTS task_summaries"
  end
end

# Refresh materialized view
ActiveRecord::Base.connection.execute("REFRESH MATERIALIZED VIEW task_summaries")

# Concurrent refresh (non-blocking)
ActiveRecord::Base.connection.execute("REFRESH MATERIALIZED VIEW CONCURRENTLY task_summaries")
```

### Common Table Expressions (CTEs)

```ruby
# Simple CTE
Task.with(
  active_merchants: Merchant.where(active: true).select(:id)
).joins("INNER JOIN active_merchants ON tasks.merchant_id = active_merchants.id")

# Complex CTE example
Task.with(
  recent_tasks: Task.where('created_at > ?', 30.days.ago).select(:id, :merchant_id),
  active_merchants: Merchant.where(active: true).select(:id)
).from("recent_tasks")
 .joins("INNER JOIN active_merchants ON recent_tasks.merchant_id = active_merchants.id")

# Recursive CTE for hierarchical data
Category.with_recursive(
  category_tree: [
    Category.where(id: 1),  # Base case
    Category.joins("INNER JOIN category_tree ON categories.parent_id = category_tree.id")  # Recursive
  ]
).from(:category_tree)

# Using raw SQL for complex CTEs
sql = <<-SQL
  WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE id = ?

    UNION ALL

    SELECT e.id, e.name, e.manager_id, s.level + 1
    FROM employees e
    INNER JOIN subordinates s ON e.manager_id = s.id
  )
  SELECT * FROM subordinates
SQL

Employee.find_by_sql([sql, manager_id])
```

### Single Table Inheritance (STI) Patterns

```ruby
# Base model
class Vehicle < ApplicationRecord
  # Columns: id, type, name, ...
  validates :name, presence: true

  # Shared behavior
  def describe
    "#{self.class.name}: #{name}"
  end
end

class Car < Vehicle
  # Car-specific behavior
  def drive
    "Driving #{name}"
  end

  # Car-specific validations
  validates :num_doors, presence: true
end

class Motorcycle < Vehicle
  def ride
    "Riding #{name}"
  end
end

# Usage
car = Car.create!(name: "Tesla", num_doors: 4)
motorcycle = Motorcycle.create!(name: "Harley")

Vehicle.all  # Returns mix of cars and motorcycles
Car.all      # Returns only cars
car.type     # => "Car"

# Scopes work with STI
class Vehicle < ApplicationRecord
  scope :recent, -> { where('created_at > ?', 1.week.ago) }
end

Car.recent  # Only recent cars
Vehicle.recent  # All recent vehicles

# Custom type column name
class Vehicle < ApplicationRecord
  self.inheritance_column = 'vehicle_type'
end

# Disable STI (use 'type' column for something else)
class Vehicle < ApplicationRecord
  self.inheritance_column = nil
end
```

**STI Best Practices:**
- Use when subclasses share 80%+ of attributes
- Avoid if types have very different attributes (use polymorphic instead)
- Watch for sparse tables (lots of nulls) - consider delegated_type
- Add database constraint on type column

**STI Anti-patterns:**
```ruby
# BAD - Too many type-specific columns
create_table :vehicles do |t|
  t.string :type
  t.string :name
  # Car-specific
  t.integer :num_doors
  t.string :trunk_type
  # Boat-specific
  t.integer :hull_length
  t.string :sail_type
  # Plane-specific
  t.integer :max_altitude
  t.string :engine_type
end

# GOOD - Use polymorphic or separate tables instead
```

### Generated Columns (PostgreSQL/MySQL)

```ruby
# PostgreSQL generated columns (Rails 7.0+)
class AddFullNameToUsers < ActiveRecord::Migration[7.1]
  def change
    add_column :users, :full_name, :virtual,
      type: :string,
      as: "first_name || ' ' || last_name",
      stored: true  # Or false for computed on-the-fly

    add_index :users, :full_name
  end
end

# Model (read-only)
class User < ApplicationRecord
  # full_name is automatically calculated
end

user = User.create!(first_name: "Alice", last_name: "Smith")
user.full_name # => "Alice Smith"

# Can query generated columns
User.where("full_name ILIKE ?", "%smith%")
```

### Full-Text Search with pg_search

```ruby
# Gemfile
gem 'pg_search'

# Model
class Article < ApplicationRecord
  include PgSearch::Model

  pg_search_scope :search_full_text,
    against: {
      title: 'A',        # Higher weight
      body: 'B',
      author: 'C'
    },
    using: {
      tsearch: {
        prefix: true,
        dictionary: 'english'
      }
    }

  # Multi-table search
  pg_search_scope :search_with_comments,
    against: [:title, :body],
    associated_against: {
      comments: [:body]
    }

  # Trigram similarity search
  pg_search_scope :fuzzy_search,
    against: [:title, :body],
    using: {
      trigram: { threshold: 0.3 }
    }
end

# Migration for indexes
class AddPgSearchIndexes < ActiveRecord::Migration[7.1]
  def up
    # tsvector column for better performance
    add_column :articles, :tsv, :tsvector
    add_index :articles, :tsv, using: :gin

    execute <<-SQL
      UPDATE articles
      SET tsv = to_tsvector('english', coalesce(title, '') || ' ' || coalesce(body, ''))
    SQL

    # Trigger to keep it updated
    execute <<-SQL
      CREATE TRIGGER articles_tsv_update BEFORE INSERT OR UPDATE
      ON articles FOR EACH ROW EXECUTE FUNCTION
      tsvector_update_trigger(tsv, 'pg_catalog.english', title, body);
    SQL

    # For trigram search
    enable_extension 'pg_trgm'
    add_index :articles, :title, using: :gin, opclass: :gin_trgm_ops
  end

  def down
    execute "DROP TRIGGER IF EXISTS articles_tsv_update ON articles"
    remove_column :articles, :tsv
  end
end

# Usage
Article.search_full_text("rails tutorial")
Article.fuzzy_search("raails")  # Finds "rails"
Article.search_with_comments("ruby")

# With rankings
Article.search_full_text("rails")
  .with_pg_search_rank
  .order('pg_search_rank DESC')
```

### Advanced JSONB Queries

```ruby
# Model with JSONB
class Product < ApplicationRecord
  # Column: specifications (jsonb)

  # Using jsonb_accessor for typed access
  jsonb_accessor :specifications,
    color: :string,
    weight: :float,
    dimensions: [:string, array: true],
    features: [:string, array: true]
end

# Query patterns
# Contains
Product.where("specifications @> ?", { color: 'red' }.to_json)

# Has key
Product.where("specifications ? 'warranty'")

# Array contains element
Product.where("specifications -> 'features' ? 'wireless'")

# Extract and compare
Product.where("specifications ->> 'color' = ?", 'red')
Product.where("(specifications ->> 'weight')::float > ?", 5.0)

# With indexes
add_index :products, :specifications, using: :gin
add_index :products, "(specifications -> 'color')", using: :btree

# Array queries
Product.where("specifications -> 'features' @> ?", ['wireless'].to_json)
```

## Pre-Query Checklist

Before writing any complex query:

```
[ ] What columns am I selecting?
[ ] Am I using GROUP BY? If so, is every SELECT column grouped or aggregated?
[ ] Am I using includes/preload with GROUP BY? (DON'T!)
[ ] Will this query run on a large table? Do indexes exist?
[ ] Am I iterating and accessing associations? Use includes.
[ ] Am I loading more data than needed? Use select/pluck.
[ ] Is this sensitive data? Consider ActiveRecord::Encryption.
[ ] Should this be in a separate database? (multi-database)
[ ] Is this a hierarchical query? Consider CTEs.
[ ] Need full-text search? Use pg_search.
```
