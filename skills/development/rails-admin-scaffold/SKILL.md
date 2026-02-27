---
name: rails-admin-scaffold
description: Generate a full-featured CRUD admin panel for Rails 6.1+ applications with auto-detection of CSS frameworks, pagination gems, and smart field mapping
disable-model-invocation: true
---

# Admin Scaffold Generator

Generate a complete admin panel with CRUD operations, filtering, pagination, dashboard, exports, and bulk actions for any Rails 6.1+ application.

## Overview

This skill creates a fully functional admin interface by:
1. Auto-detecting your project's CSS framework, pagination gem, and test framework
2. Analyzing all models for fields, associations, enums, and attachments
3. Asking targeted questions to customize the output
4. Generating controllers, views, routes, and optional tests
5. Providing clear next steps for verification

---

## Phase 1: Detection

Before asking any questions, automatically detect the project configuration.

### 1.1 Detect Existing Admin Dashboards

Scan for existing admin implementations and identify which paths/namespaces they use.

**Read `config/routes.rb` and extract:**

```ruby
# 1. Rails Admin - extract mount path
# mount RailsAdmin::Engine => '/admin', as: 'rails_admin'
# mount RailsAdmin::Engine, at: '/backend'
# → Extract path: "/admin" or "/backend" etc.

# 2. ActiveAdmin - extract namespace from config or routes
# ActiveAdmin.routes(self)  → default namespace is "admin"
# Also check config/initializers/active_admin.rb for:
#   config.default_namespace = :backend
# → Extract namespace: "admin" or custom

# 3. Administrate - check for namespace block
# namespace :admin do
#   resources :users, controller: "users"
# → Usually uses "admin" namespace

# 4. Custom admin - check for namespace blocks
# namespace :admin do ... end
# namespace :backend do ... end
# → Extract all admin-like namespaces

# 5. Check controller directories
# app/controllers/admin/ → "admin"
# app/controllers/backend/ → "backend"
```

**Store results as:**
```ruby
existing_admins = [
  { type: "rails_admin", path: "/admin" },
  { type: "activeadmin", path: "/backend" },
  { type: "custom", path: "/management" }
]

# List of taken namespaces (for Question 1)
taken_namespaces = ["admin", "backend", "management"]
```

**Common namespace options to check:**
- `admin`, `backend`, `dashboard`, `management`, `panel`, `console`, `staff`

### 1.2 Detect CSS Framework

Read these files to determine CSS framework:

```ruby
# Priority order:
# 1. Tailwind: tailwind.config.js or config/tailwind.config.js exists
# 2. Bootstrap: Gemfile contains 'bootstrap' or package.json contains 'bootstrap'
# 3. Bulma: Gemfile contains 'bulma' or package.json contains 'bulma'
# 4. Default: Tailwind (will need to be added)
```

**Detection logic:**
- Read `tailwind.config.js` or `config/tailwind.config.js` → **Tailwind**
- Read `Gemfile` for `gem 'bootstrap'` or `gem 'cssbundling-rails'` + `package.json` for `bootstrap` → **Bootstrap**
- Read `Gemfile` or `package.json` for `bulma` → **Bulma**

Store result as: `css_framework = "tailwind" | "bootstrap" | "bulma"`

### 1.3 Detect Pagination Gem

Read `Gemfile`:

```ruby
# Look for:
gem 'pagy'           # → pagination_gem = "pagy" (recommended, ~40x faster)
gem 'kaminari'       # → pagination_gem = "kaminari"
gem 'will_paginate'  # → pagination_gem = "will_paginate"
# None found         # → pagination_gem = "pagy" (default, fastest option)
```

### 1.4 Detect Test Framework

```bash
# Check directory structure:
# spec/ exists → test_framework = "rspec"
# test/ exists → test_framework = "minitest"
```

### 1.5 Detect Stimulus

```ruby
# Check for Stimulus:
# package.json contains "@hotwired/stimulus" → has_stimulus = true
# config/importmap.rb contains "stimulus" → has_stimulus = true
# Otherwise → has_stimulus = false
```

### 1.6 Detect File Uploader

Check Gemfile and model files for file upload solution:

```ruby
# Check Gemfile:
gem 'activestorage'    # → file_uploader = "activestorage" (Rails 5.2+ built-in)
gem 'carrierwave'      # → file_uploader = "carrierwave"
gem 'shrine'           # → file_uploader = "shrine"
gem 'paperclip'        # → file_uploader = "paperclip" (deprecated)
gem 'dragonfly'        # → file_uploader = "dragonfly"

# Also check model files for:
has_one_attached       # → Active Storage
has_many_attached      # → Active Storage
mount_uploader         # → CarrierWave
include ImageUploader  # → Shrine
has_attached_file      # → Paperclip
dragonfly_accessor     # → Dragonfly
```

**Preview syntax by uploader:**

```erb
<%# Active Storage %>
<%% if record.avatar.attached? %>
  <%%= image_tag record.avatar.variant(resize_to_limit: [100, 100]) %>
<%% end %>

<%# CarrierWave %>
<%% if record.avatar.present? %>
  <%%= image_tag record.avatar.thumb.url %>
<%% end %>

<%# Shrine %>
<%% if record.avatar_data.present? %>
  <%%= image_tag record.avatar_url(:thumb) %>
<%% end %>

<%# Paperclip (legacy) %>
<%% if record.avatar.present? %>
  <%%= image_tag record.avatar.url(:thumb) %>
<%% end %>

<%# Dragonfly %>
<%% if record.avatar_stored? %>
  <%%= image_tag record.avatar.thumb('100x100#').url %>
<%% end %>
```

**Form input syntax by uploader:**

```erb
<%# Active Storage %>
<%%= form.file_field :avatar, accept: "image/*", class: "{CSS: file-input}" %>
<%% if record.avatar.attached? %>
  <div class="{CSS: file-preview}">
    <%%= image_tag record.avatar.variant(resize_to_limit: [200, 200]) %>
    <%%= form.check_box :remove_avatar, label: "Remove" %>
  </div>
<%% end %>

<%# CarrierWave %>
<%%= form.file_field :avatar, class: "{CSS: file-input}" %>
<%% if record.avatar.present? %>
  <div class="{CSS: file-preview}">
    <%%= image_tag record.avatar.thumb.url %>
    <%%= form.check_box :remove_avatar %>
  </div>
<%% end %>

<%# Shrine %>
<%%= form.hidden_field :avatar, value: record.cached_avatar_data %>
<%%= form.file_field :avatar, class: "{CSS: file-input}" %>

<%# Paperclip %>
<%%= form.file_field :avatar, class: "{CSS: file-input}" %>

<%# Dragonfly %>
<%%= form.file_field :avatar, class: "{CSS: file-input}" %>
<%%= form.hidden_field :retained_avatar %>
```

### 1.7 Discover Models

```bash
# List all model files
ls app/models/*.rb
```

Parse each model file to extract:
- Model name (from filename and class definition)
- Skip abstract models, concerns, ApplicationRecord

Store as: `models = ["User", "Post", "Comment", ...]`

### 1.7 Detect Existing Admin Panels

Check for existing admin implementations to understand current patterns:

```ruby
# Check Gemfile for admin gems:
gem 'activeadmin'    # → existing_admin = "activeadmin"
gem 'rails_admin'    # → existing_admin = "rails_admin"
gem 'administrate'   # → existing_admin = "administrate"
```

**If existing admin gem found:**

1. **ActiveAdmin** - Check `app/admin/*.rb` for:
   - Custom form blocks
   - Custom show blocks
   - Filters configuration
   - Batch actions
   - Sidebar sections

2. **RailsAdmin** - Check `config/initializers/rails_admin.rb` for:
   - Custom actions
   - Field configurations
   - Navigation labels

3. **Administrate** - Check `app/dashboards/*_dashboard.rb` for:
   - COLLECTION_ATTRIBUTES
   - SHOW_PAGE_ATTRIBUTES
   - FORM_ATTRIBUTES

**If custom admin exists** (`app/controllers/admin/` without gems):

Analyze existing controllers and views:
```bash
# List existing admin controllers
ls app/controllers/admin/*_controller.rb

# List existing admin views
ls -la app/views/admin/*/
```

For each existing admin resource, extract:
- Custom actions (beyond standard CRUD)
- Custom filters or scopes
- Special display logic in show views
- Custom form fields or nested forms
- Bulk actions
- Export functionality

Store as:
```ruby
existing_admin_features = {
  "User" => {
    custom_actions: ["impersonate", "ban", "export_csv"],
    custom_filters: ["by_role", "by_status", "date_range"],
    nested_forms: ["profile", "addresses"],
    show_sections: ["activity_log", "permissions"],
    bulk_actions: ["activate", "deactivate"]
  },
  # ... other models
}
```

---

## Phase 2: Model Analysis

For each discovered model, extract detailed information.

### 2.1 Schema Information

Look for Annotate gem comments at top of model:

```ruby
# == Schema Information
#
# Table name: users
#
#  id                     :bigint           not null, primary key
#  email                  :string           default(""), not null
#  encrypted_password     :string           default(""), not null
#  created_at             :datetime         not null
#  updated_at             :datetime         not null
```

If no annotation, read `db/schema.rb` for the table definition.

### 2.2 Extract Field Types

Map each column to form input and display type using `reference/field-mappings.md`.

### 2.3 Associations

Look for:
```ruby
belongs_to :author, class_name: 'User'
has_many :comments
has_one :profile
has_and_belongs_to_many :tags
```

### 2.4 Enums

```ruby
enum status: { draft: 0, published: 1, archived: 2 }
enum :role, { admin: 0, user: 1 }, prefix: true  # Rails 7+
```

### 2.5 Active Storage Attachments

```ruby
has_one_attached :avatar
has_many_attached :images
```

### 2.6 Ransack Methods (if existing)

```ruby
def self.ransackable_attributes(auth_object = nil)
  ["email", "name", "created_at"]
end

def self.ransackable_associations(auth_object = nil)
  ["posts", "comments"]
end
```

### 2.7 Soft Delete Detection

```ruby
# Paranoia gem:
acts_as_paranoid

# Discard gem:
include Discard::Model
```

Store soft_delete info: `soft_delete = "paranoia" | "discard" | nil`

---

## Phase 3: Interactive Questions

Use the AskUserQuestion tool with these questions:

### Question 1: Namespace

Build the question dynamically based on detected existing admins:

**If existing admin dashboards were detected:**

First, inform the user what was found:
```
Detected existing admin dashboard(s):
- Rails Admin at /admin
- Custom admin at /backend
```

Then ask with only available (non-conflicting) options:

```json
{
  "question": "Which namespace should the new admin panel use?",
  "header": "Namespace",
  "options": [
    // Only include namespaces NOT in taken_namespaces
    // First available option from this priority list gets "(Recommended)":
    // 1. "admin" (if available)
    // 2. "backend" (if available)
    // 3. "dashboard" (if available)
    // 4. "management" (if available)
    // 5. "panel" (if available)

    // Example if "admin" and "backend" are taken:
    {"label": "dashboard (Recommended)", "description": "Use /dashboard path"},
    {"label": "management", "description": "Use /management path"},
    {"label": "panel", "description": "Use /panel path"}
  ],
  "multiSelect": false
}
```

**If no existing admin dashboards detected:**

```json
{
  "question": "What namespace should the admin panel use?",
  "header": "Namespace",
  "options": [
    {"label": "admin (Recommended)", "description": "Standard /admin path"},
    {"label": "backend", "description": "Alternative /backend path"},
    {"label": "dashboard", "description": "Use /dashboard path"},
    {"label": "management", "description": "Use /management path"}
  ],
  "multiSelect": false
}
```

**Available namespace pool (in priority order):**
`admin`, `backend`, `dashboard`, `management`, `panel`, `console`, `staff`

If user selects "Other", use their custom namespace.

### Question 2: Models to Exclude

Before asking, analyze models and categorize them:

```ruby
# Auto-detect model categories:

# 1. Technical models (always exclude)
technical_models = [
  # Abstract/base classes
  "ApplicationRecord",
  # Models matching patterns: *Record, *Base, Abstract*
]

# 2. Join table models (likely exclude)
# Models with only foreign keys + timestamps, used for has_many :through
join_models = models.select do |m|
  columns = m.column_names - %w[id created_at updated_at]
  columns.all? { |c| c.ends_with?('_id') }
end

# 3. Empty tables (likely exclude)
# Models with 0 records - often internal registries, configs
empty_models = models.select { |m| m.count == 0 }

# 4. Nested/child models (suggest exclude, manage via parent)
# Naming patterns: Parent* (CandidateEmailAddress, UserPhone, etc.)
# Models that only belong_to one parent and have no independent meaning
nested_models = models.select do |m|
  # Has belongs_to and name starts with parent model name
  # e.g., CandidatePhone belongs_to :candidate
end

# 5. Primary models (include by default)
primary_models = models - technical_models - join_models - empty_models - nested_models
```

**Build dynamic options based on detection:**

```json
{
  "question": "Which models should be EXCLUDED from the admin panel?",
  "header": "Exclude",
  "options": [
    {
      "label": "Smart exclude (Recommended)",
      "description": "Include {N} primary models, exclude {M} technical/join/empty models"
    },
    {
      "label": "Include all {total} models",
      "description": "Generate admin for every model including join tables"
    },
    {
      "label": "Select specific models",
      "description": "I'll review and choose which to exclude"
    }
  ],
  "multiSelect": false
}
```

**If "Smart exclude" selected, show summary:**
```
Including {N} primary models:
  User, Post, Comment, Order, Product, ...

Excluding {M} models:
  Technical: ApplicationRecord
  Join tables: PostTag, UserRole
  Empty tables: LiquidTemplate, FeatureFlag
  Nested (manage via parent): CandidateEmail, CandidatePhone, UserAddress

You can manage nested models through their parent's edit form with nested attributes.
```

**If "Select specific models"**, show multiSelect with categories:
```json
{
  "question": "Select models to EXCLUDE from admin:",
  "header": "Exclude",
  "options": [
    // Group by category, pre-select recommended exclusions
    {"label": "ApplicationRecord", "description": "technical ✓"},
    {"label": "PostTag", "description": "join table ✓"},
    {"label": "LiquidTemplate", "description": "empty (0 records) ✓"},
    {"label": "CandidatePhone", "description": "nested → Candidate ✓"},
    {"label": "User", "description": "primary (1,234 records)"},
    {"label": "Post", "description": "primary (567 records)"},
    // ...
  ],
  "multiSelect": true
}
```

### Question 3: Model Configuration Mode

```json
{
  "question": "How do you want to configure fields and views for each model?",
  "header": "Config",
  "options": [
    {"label": "Smart defaults (Recommended)", "description": "Auto-detect fields, hide sensitive data, full CRUD for all"},
    {"label": "Configure each model", "description": "I'll specify fields and views for each model individually"}
  ],
  "multiSelect": false
}
```

**If "Smart defaults":**
- All included models get full CRUD (index, show, new, edit, delete)
- Show all fields except sensitive ones (passwords, tokens, secrets)
- Auto-generate filters for searchable fields
- Skip to Question 4 (Authentication)

**If "Configure each model":**
For each included model, ask the following questions:

#### 3a. Model View Type

```json
{
  "question": "What views should {Model} have?",
  "header": "{Model}",
  "options": [
    {"label": "Full CRUD", "description": "Index, show, new, edit, delete"},
    {"label": "Read-only", "description": "Index and show only, no editing"},
    {"label": "Skip", "description": "Don't generate admin for this model"}
  ],
  "multiSelect": false
}
```

#### 3b. Index/Table Columns

```json
{
  "question": "Which fields should appear in the {Model} table (index view)?",
  "header": "Table",
  "options": [
    // List all non-sensitive fields, pre-select recommended ones
    // Pre-select: id, name/title (if exists), key identifiers, status, created_at
    {"label": "id", "description": "bigint"},
    {"label": "name", "description": "string ✓"},
    {"label": "email", "description": "string ✓"},
    {"label": "status", "description": "enum ✓"},
    {"label": "created_at", "description": "datetime ✓"},
    {"label": "bio", "description": "text"},
    // ... (exclude sensitive fields entirely)
  ],
  "multiSelect": true
}
```

#### 3c. Show Page Fields

```json
{
  "question": "Which fields should appear on the {Model} show page?",
  "header": "Show",
  "options": [
    // List all non-sensitive fields, pre-select all by default
    {"label": "id", "description": "bigint ✓"},
    {"label": "name", "description": "string ✓"},
    {"label": "email", "description": "string ✓"},
    {"label": "bio", "description": "text ✓"},
    {"label": "created_at", "description": "datetime ✓"},
    {"label": "updated_at", "description": "datetime ✓"},
    // ...
  ],
  "multiSelect": true
}
```

#### 3d. Form Fields (only if Full CRUD selected)

```json
{
  "question": "Which fields should be editable in the {Model} form?",
  "header": "Form",
  "options": [
    // List editable fields (exclude id, timestamps, computed fields)
    // Pre-select all editable fields
    {"label": "name", "description": "string ✓"},
    {"label": "email", "description": "string ✓"},
    {"label": "bio", "description": "text ✓"},
    {"label": "status", "description": "enum ✓"},
    {"label": "role", "description": "enum ✓"},
    // ...
  ],
  "multiSelect": true
}
```

**Sensitive fields (always excluded from options):**
```ruby
SENSITIVE_FIELDS = %w[
  encrypted_password password_digest password
  reset_password_token confirmation_token unlock_token
  api_token access_token refresh_token auth_token secret_token
  otp_secret encrypted_otp_secret
  secret_key api_key private_key
]
```

**Note:** If existing admin panel was detected (Section 1.7), show detected custom features and ask if they should be replicated.

---

### Question 4: Authentication

```json
{
  "question": "How should admin authentication work?",
  "header": "Auth",
  "options": [
    {"label": "Devise AdminUser (Recommended)", "description": "Create separate AdminUser model with Devise"},
    {"label": "Existing User model", "description": "Use current User model with admin role/flag"},
    {"label": "Skip authentication", "description": "No auth - secure with other means"},
    {"label": "HTTP Basic Auth", "description": "Simple username/password protection"}
  ],
  "multiSelect": false
}
```

### Question 5: Internationalization

```json
{
  "question": "Do you want internationalization (i18n) support?",
  "header": "i18n",
  "options": [
    {"label": "No i18n", "description": "English-only, hardcoded strings"},
    {"label": "Yes - English", "description": "i18n with English locale file"},
    {"label": "Yes - Russian", "description": "i18n with Russian locale file"},
    {"label": "Yes - Other language", "description": "I'll specify the language"}
  ],
  "multiSelect": false
}
```

### Question 6: Tests

```json
{
  "question": "Should tests be generated for the admin controllers?",
  "header": "Tests",
  "options": [
    {"label": "Yes (Recommended)", "description": "Generate controller tests"},
    {"label": "No", "description": "Skip test generation"}
  ],
  "multiSelect": false
}
```

### Question 7: Export

```json
{
  "question": "What export functionality do you need?",
  "header": "Export",
  "options": [
    {"label": "CSV with field selection (Recommended)", "description": "Export to CSV, admin chooses which fields"},
    {"label": "CSV + Excel with field selection", "description": "Both formats, admin chooses fields"},
    {"label": "Simple CSV (all fields)", "description": "Quick export without field picker"},
    {"label": "No export", "description": "Skip export functionality"}
  ],
  "multiSelect": false
}
```

---

## Phase 4: Generation

Generate files in this specific order. Use templates from `templates/` directory.

### 4.1 Add Gems (if needed)

If pagination gem not found, add to Gemfile:
```ruby
gem 'pagy', '~> 6.0'
gem 'ransack', '~> 4.0'
```

If Excel export requested:
```ruby
gem 'caxlsx', '~> 4.0'
gem 'caxlsx_rails', '~> 0.6'
```

### 4.2 Create Concerns

Create `app/controllers/concerns/{namespace}/`:

**date_range_filterable.rb** - See `templates/concerns/date_range_filterable.rb`

**exportable.rb** - Advanced export concern with field selection:

```ruby
# app/controllers/concerns/{namespace}/exportable.rb
module {Namespace}
  module Exportable
    extend ActiveSupport::Concern

    def export
      @model_class = controller_name.classify.constantize
      @exportable_fields = exportable_fields_for(@model_class)

      if request.get?
        # Show export modal/form with field selection
        render "#{namespace}/shared/export_modal", locals: {
          fields: @exportable_fields,
          record_count: filtered_scope.count,
          selected_ids: params[:ids]
        }
      else
        # Perform export with selected fields
        selected_fields = params[:export_fields] || @exportable_fields.keys
        records = export_scope(params[:ids])

        respond_to do |format|
          format.csv { send_csv(records, selected_fields) }
          format.xlsx { send_xlsx(records, selected_fields) }
        end
      end
    end

    private

    def exportable_fields_for(model_class)
      # Returns hash of field_name => human_label
      model_class.column_names.each_with_object({}) do |col, hash|
        next if col.in?(%w[encrypted_password reset_password_token])
        hash[col] = col.humanize
      end
    end

    def export_scope(selected_ids = nil)
      scope = filtered_scope  # Uses current ransack filters
      scope = scope.where(id: selected_ids) if selected_ids.present?
      scope
    end

    def filtered_scope
      # Reuse ransack query from index
      model_class = controller_name.classify.constantize
      model_class.ransack(params[:q]).result
    end

    def send_csv(records, fields)
      csv_data = CSV.generate(headers: true) do |csv|
        csv << fields.map { |f| f.humanize }
        records.find_each do |record|
          csv << fields.map { |f| format_field(record, f) }
        end
      end
      send_data csv_data, filename: "\#{controller_name}-\#{Date.current}.csv"
    end

    def format_field(record, field)
      value = record.send(field)
      case value
      when Time, DateTime then value.strftime("%Y-%m-%d %H:%M")
      when Date then value.strftime("%Y-%m-%d")
      when true then "Yes"
      when false then "No"
      else value.to_s
      end
    end
  end
end
```

**bulk_actions.rb** - See `templates/features/bulk_actions.rb`

### 4.3 Create Base Controller

Use `templates/controllers/base_controller.rb` as template.

Create `app/controllers/{namespace}/base_controller.rb`:

```ruby
module {Namespace}
  class BaseController < ApplicationController
    include Pagy::Backend
    include {Namespace}::DateRangeFilterable
    include {Namespace}::Exportable
    include {Namespace}::BulkActions

    before_action :authenticate_{auth_method}!  # Based on auth choice

    layout "{namespace}"

    private

    def pagy_get_vars(collection, vars)
      vars[:items] ||= 25
      vars[:count] ||= collection.count(:all)
      vars
    end
  end
end
```

### 4.4 Create Layout and Shared Partials

Create `app/views/layouts/{namespace}.html.erb` using `templates/shared/layout.html.erb`.

Create `app/views/{namespace}/shared/` partials:
- `_sidebar.html.erb` - Navigation with all resources
- `_flash.html.erb` - Flash message display
- `_pagination.html.erb` - Pagination controls
- `_table_header.html.erb` - Sortable column headers
- `_bulk_actions.html.erb` - Bulk action dropdown
- `_export_modal.html.erb` - Export dialog with field selection (if export enabled)

**Export Modal** (`_export_modal.html.erb`):

```erb
<%# Export modal with field selection %>
<div id="export-modal" class="{CSS: modal}" data-controller="export-modal">
  <div class="{CSS: modal-content}">
    <div class="{CSS: modal-header}">
      <h3>Export <%= controller_name.humanize %></h3>
      <button type="button" data-action="export-modal#close">&times;</button>
    </div>

    <%= form_tag export_path(format: :csv), method: :post, data: { export_modal_target: "form" } do %>
      <%# Pass current filters %>
      <% params[:q]&.each do |key, value| %>
        <%= hidden_field_tag "q[#{key}]", value %>
      <% end %>

      <%# Pass selected IDs if bulk export %>
      <% if local_assigns[:selected_ids].present? %>
        <% selected_ids.each do |id| %>
          <%= hidden_field_tag "ids[]", id %>
        <% end %>
      <% end %>

      <div class="{CSS: modal-body}">
        <p class="{CSS: text-muted}">
          Exporting <strong><%= record_count %></strong> records
          <% if selected_ids.present? %>
            (<%= selected_ids.size %> selected)
          <% else %>
            (filtered)
          <% end %>
        </p>

        <div class="{CSS: form-group}">
          <label class="{CSS: label}">Select fields to export:</label>

          <div class="{CSS: checkbox-group}">
            <label class="{CSS: checkbox}">
              <%= check_box_tag "select_all", "1", true, data: { action: "export-modal#toggleAll" } %>
              <strong>Select All</strong>
            </label>
          </div>

          <div class="{CSS: checkbox-grid}" data-export-modal-target="fields">
            <% fields.each do |field_name, field_label| %>
              <label class="{CSS: checkbox}">
                <%= check_box_tag "export_fields[]", field_name, true, data: { export_modal_target: "field" } %>
                <%= field_label %>
              </label>
            <% end %>
          </div>
        </div>
      </div>

      <div class="{CSS: modal-footer}">
        <button type="button" class="{CSS: btn-secondary}" data-action="export-modal#close">
          Cancel
        </button>
        <button type="submit" name="format" value="csv" class="{CSS: btn-primary}">
          Export CSV
        </button>
        <%# If Excel enabled %>
        <button type="submit" name="format" value="xlsx" class="{CSS: btn-primary}">
          Export Excel
        </button>
      </div>
    <% end %>
  </div>
</div>
```

**Export Modal Stimulus Controller** (if has_stimulus):

```javascript
// app/javascript/controllers/{namespace}/export_modal_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["form", "fields", "field"]

  open(event) {
    event.preventDefault()
    // If triggered from bulk actions, collect selected IDs
    const selectedIds = this.getSelectedIds()
    if (selectedIds.length > 0) {
      this.addSelectedIds(selectedIds)
    }
    this.element.classList.remove("hidden")
  }

  close() {
    this.element.classList.add("hidden")
  }

  toggleAll(event) {
    const checked = event.target.checked
    this.fieldTargets.forEach(field => field.checked = checked)
  }

  getSelectedIds() {
    // Get IDs from bulk select checkboxes
    const checkboxes = document.querySelectorAll('[data-bulk-select-target="checkbox"]:checked')
    return Array.from(checkboxes).map(cb => cb.value)
  }

  addSelectedIds(ids) {
    // Remove existing hidden ID fields
    this.formTarget.querySelectorAll('input[name="ids[]"]').forEach(el => el.remove())
    // Add new ones
    ids.forEach(id => {
      const input = document.createElement('input')
      input.type = 'hidden'
      input.name = 'ids[]'
      input.value = id
      this.formTarget.appendChild(input)
    })
  }
}
```

**IMPORTANT:** Use CSS classes from `css/{css_framework}.md` for all styling.

### 4.5 Create Dashboard

Create `app/controllers/{namespace}/dashboard_controller.rb`:

```ruby
module {Namespace}
  class DashboardController < BaseController
    def index
      @stats = {
        # For each model:
        users: User.count,
        posts: Post.count,
        # etc.
      }

      @recent_activity = [
        # Last 10 records from each model with timestamps
      ]
    end
  end
end
```

Create `app/views/{namespace}/dashboard/index.html.erb` with:
- Stat cards for each model (count, recent count)
- Recent activity feed
- Quick links to each resource

### 4.6 Create Resource Controllers

For each included model, create `app/controllers/{namespace}/{model_plural}_controller.rb`:

Use `templates/controllers/resource_controller.rb` as base.

**Based on CRUD configuration (Question 3):**

**If model has full CRUD:**
- `index` with ransack filtering and pagy pagination
- `show` with association preloading
- `new/create` with strong parameters
- `edit/update` with strong parameters
- `destroy` (or soft delete if detected)
- `export` action for CSV/Excel
- `bulk_destroy` action
- `restore` action (if soft delete)

**If model is read-only:**
- `index` with ransack filtering and pagy pagination
- `show` with association preloading
- `export` action for CSV/Excel
- NO new/create/edit/update/destroy actions

**If existing admin features detected (Section 1.7):**

Replicate custom functionality from existing admin:

```ruby
# Example: If ActiveAdmin had custom action
member_action :impersonate, method: :post do
  # Replicate as:
end

# Becomes in new controller:
def impersonate
  @user = User.find(params[:id])
  # ... implementation
end
```

Include detected features:
- Custom actions → add as controller methods
- Custom filters → add to ransack configuration
- Nested forms → include in strong parameters and form
- Custom show sections → add to show view
- Bulk actions → add to bulk_actions concern

### 4.7 Create Resource Views

For each included model, create views in `app/views/{namespace}/{model_plural}/`:

**Based on CRUD configuration (Question 3):**

#### All models (read-only and CRUD):

**index.html.erb** - Use `templates/views/index.html.erb`:
- Filter form with ransack
- Data table with sortable columns
- Pagination
- If CRUD: "New" button, bulk action checkboxes, edit/delete links
- If read-only: only "View" links, no bulk actions

**Export buttons** (if export enabled):
```erb
<div class="{CSS: btn-group}">
  <%# Export filtered records %>
  <%= link_to "Export", "#",
      class: "{CSS: btn-secondary}",
      data: { action: "export-modal#open" } %>

  <%# Export selected records (shown when records selected) %>
  <span data-bulk-select-target="bulkActions" class="hidden">
    <%= link_to "Export Selected", "#",
        class: "{CSS: btn-secondary}",
        data: { action: "export-modal#open" } %>
  </span>
</div>

<%# Include export modal %>
<%= render "{namespace}/shared/export_modal",
    fields: exportable_fields_for(@model_class),
    record_count: @query.result.count,
    selected_ids: nil %>
```

**show.html.erb** - Use `templates/views/show.html.erb`:
- Field display based on type
- Associated records lists
- Pretty JSON for jsonb fields
- Image previews for attachments
- If CRUD: Edit, Delete, Back buttons
- If read-only: only Back button

**_table.html.erb** - Use `templates/views/_table.html.erb`

**_filters.html.erb** - Use `templates/views/_filters.html.erb`

#### Only for models with full CRUD:

**new.html.erb** - Use `templates/views/new.html.erb`

**edit.html.erb** - Use `templates/views/edit.html.erb`

**_form.html.erb** - Use `templates/views/_form.html.erb`:
- Form fields based on column types (see field-mappings.md)
- Association selects
- Enum dropdowns
- File upload fields with previews
- JSON textarea for jsonb

#### If existing admin features detected:

Replicate custom view sections from existing admin:
- Custom show page sections → add to show.html.erb
- Custom index columns → add to _table.html.erb
- Custom form fields → add to _form.html.erb
- Nested resource forms → add accepts_nested_attributes and fields_for

### 4.8 Add Ransackable Methods to Models

For each model without existing ransackable methods, add:

```ruby
# In app/models/{model}.rb

def self.ransackable_attributes(auth_object = nil)
  # List of searchable column names
  ["name", "email", "status", "created_at"]
end

def self.ransackable_associations(auth_object = nil)
  # List of searchable association names
  ["author", "comments"]
end
```

### 4.9 Create Routes

Add to `config/routes.rb`:

**Based on CRUD configuration (Question 3):**

```ruby
namespace :{namespace} do
  root to: "dashboard#index"

  # For models with FULL CRUD:
  resources :users do
    collection do
      get :export      # Show export modal with field selection
      post :export     # Perform export with selected fields
      delete :bulk_destroy
    end
    member do
      patch :restore  # Only if soft delete
      # Custom actions from existing admin:
      post :impersonate  # If detected
      post :ban          # If detected
    end
  end

  # For READ-ONLY models:
  resources :audit_logs, only: [:index, :show] do
    collection do
      get :export
      post :export
    end
  end

  # Repeat for all models...
end
```

**Export route explanation:**
- `GET /admin/users/export` - Opens modal (if JS disabled, shows export page)
- `POST /admin/users/export` - Performs export with selected fields, respects current filters
- `POST /admin/users/export?ids[]=1&ids[]=2` - Exports only selected records

### 4.10 Create Stimulus Controllers (if has_stimulus)

Create `app/javascript/controllers/{namespace}/`:

**bulk_select_controller.js**:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["checkbox", "selectAll", "bulkActions", "selectedCount"]

  connect() {
    this.updateUI()
  }

  toggleAll() {
    const checked = this.selectAllTarget.checked
    this.checkboxTargets.forEach(cb => cb.checked = checked)
    this.updateUI()
  }

  toggle() {
    this.updateUI()
  }

  updateUI() {
    const checked = this.checkboxTargets.filter(cb => cb.checked)
    const count = checked.length

    this.bulkActionsTarget.classList.toggle("hidden", count === 0)
    this.selectedCountTarget.textContent = count

    this.selectAllTarget.checked = count === this.checkboxTargets.length
    this.selectAllTarget.indeterminate = count > 0 && count < this.checkboxTargets.length
  }

  getSelectedIds() {
    return this.checkboxTargets
      .filter(cb => cb.checked)
      .map(cb => cb.value)
  }
}
```

**confirm_controller.js**:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { message: String }

  confirm(event) {
    if (!window.confirm(this.messageValue || "Are you sure?")) {
      event.preventDefault()
    }
  }
}
```

### 4.11 Create Tests (if requested)

Use templates from `templates/specs/`:

**RSpec** - Create `spec/controllers/{namespace}/` or `spec/requests/{namespace}/`

**Minitest** - Create `test/controllers/{namespace}/`

### 4.12 Create i18n Files (if requested)

Create `config/locales/{namespace}.{locale}.yml`:

```yaml
{locale}:
  {namespace}:
    shared:
      actions: "Actions"
      edit: "Edit"
      delete: "Delete"
      back: "Back"
      save: "Save"
      cancel: "Cancel"
      search: "Search"
      filter: "Filter"
      export: "Export"
      export_csv: "Export CSV"
      export_excel: "Export Excel"
      confirm_delete: "Are you sure you want to delete this record?"
      bulk_delete: "Delete selected"
      no_records: "No records found"
      # ... more keys

    dashboard:
      title: "Dashboard"
      total_records: "Total records"
      recent_activity: "Recent activity"

    # Per-model translations:
    users:
      title: "Users"
      new: "New User"
      edit: "Edit User"
      # field names...
```

---

## Phase 5: Verification

After generation, output these instructions with SPECIFIC details based on user choices:

### Output Template

```markdown
## ✅ Admin Panel Generated Successfully!

### 🔗 Admin Panel URL

**URL:** http://localhost:3000/{namespace}

(Replace `localhost:3000` with your actual host if different)

---

### 🔐 Authentication

{IF auth_choice == "Devise AdminUser"}
**Method:** Devise with AdminUser model

1. Run migrations:
   ```bash
   rails db:migrate
   ```

2. Create your first admin user:
   ```bash
   rails console
   ```
   ```ruby
   AdminUser.create!(email: 'admin@example.com', password: 'password123')
   ```

3. Login at: http://localhost:3000/{namespace}/login
   - Email: admin@example.com
   - Password: password123

{ELSE IF auth_choice == "Existing User model"}
**Method:** Using existing User model with admin check

Make sure your User model has an `admin?` method or `admin` boolean field.
Users with `admin? == true` can access the admin panel.

Login with any admin user credentials at: http://localhost:3000/{namespace}

{ELSE IF auth_choice == "HTTP Basic Auth"}
**Method:** HTTP Basic Authentication

Credentials are set in `app/controllers/{namespace}/base_controller.rb`:
- Username: admin
- Password: (check the controller file)

You can change credentials in the `authenticate` method.

{ELSE IF auth_choice == "Skip authentication"}
**⚠️ WARNING: No authentication configured!**

Your admin panel is currently **publicly accessible**.
This is fine for development, but **NEVER deploy to production without authentication**.

**Recommended:** Add Devise authentication:

1. Add to Gemfile:
   ```ruby
   gem 'devise'
   ```

2. Run:
   ```bash
   bundle install
   rails generate devise:install
   rails generate devise AdminUser
   rails db:migrate
   ```

3. Create admin user:
   ```ruby
   AdminUser.create!(email: 'admin@example.com', password: 'password123')
   ```

4. Add to `app/controllers/{namespace}/base_controller.rb`:
   ```ruby
   before_action :authenticate_admin_user!
   ```

{END IF}

---

### 📁 Files Created

- `app/controllers/{namespace}/` - Admin controllers
- `app/views/{namespace}/` - Admin views
- `app/views/layouts/{namespace}.html.erb` - Admin layout
- `config/routes.rb` - Updated with admin routes
{if tests}- `{spec|test}/controllers/{namespace}/` - Controller tests{/if}
{if i18n}- `config/locales/{namespace}.{locale}.yml` - Translations{/if}

---

### 🚀 Quick Start

1. Install dependencies:
   ```bash
   bundle install
   ```

2. Start server:
   ```bash
   rails server
   ```

3. Open in browser:
   ```
   http://localhost:3000/{namespace}
   ```

---

### 🎨 Customization

- **Navigation:** Edit `app/views/{namespace}/shared/_sidebar.html.erb`
- **Global settings:** Modify `app/controllers/{namespace}/base_controller.rb`
- **Styles:** Adjust the layout file for your design preferences
```

---

## CSS Framework Reference

When generating views, use the appropriate CSS classes based on detected framework.

### Tailwind (Default)
See `css/tailwind.md` for class mappings.

### Bootstrap
See `css/bootstrap.md` for class mappings.

### Bulma
See `css/bulma.md` for class mappings.

---

## Field Type Mappings

See `reference/field-mappings.md` for complete mapping of:
- Database column types → Form input types
- Database column types → Table display formats
- Special handling for JSON, attachments, enums

---

## Patterns Reference

See `reference/patterns.md` for:
- Ransack query patterns
- Pagy/Kaminari/WillPaginate integration
- Soft delete patterns
- Export patterns
- Bulk action patterns
