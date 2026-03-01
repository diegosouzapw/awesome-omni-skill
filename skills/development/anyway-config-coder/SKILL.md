---
name: anyway-config-coder
description: Implement type-safe configuration with anyway_config gem. Use when creating configuration classes, replacing ENV access, or managing application settings. Triggers on configuration, environment variables, settings, secrets, or ENV patterns.
---

# Anyway Config Coder

Implement type-safe configuration management using the `anyway_config` gem. Never access ENV directly - wrap all configuration in typed classes.

## Core Principle

**Never use ENV directly or Rails credentials.** Create typed configuration classes instead.

```ruby
# WRONG - scattered ENV access
api_key = ENV["GEMINI_API_KEY"]
timeout = ENV.fetch("API_TIMEOUT", 30).to_i

# RIGHT - typed configuration class
class GeminiConfig < Anyway::Config
  attr_config :api_key,
              timeout: 30

  required :api_key
end

# Usage
GeminiConfig.new.api_key
```

## Setup

```ruby
# Gemfile
gem "anyway_config", "~> 2.6"
```

## Configuration Class Structure

### Basic Configuration

```ruby
# config/configs/gemini_config.rb
class GeminiConfig < Anyway::Config
  # Define attributes with defaults
  attr_config :api_key,
              model: "gemini-pro",
              timeout: 30,
              max_retries: 3

  # Mark required attributes
  required :api_key

  # Computed helpers
  def configured?
    api_key.present?
  end

  def base_url
    "https://generativelanguage.googleapis.com/v1beta"
  end
end
```

### Environment Variable Mapping

Anyway Config automatically maps environment variables:

```ruby
class GeminiConfig < Anyway::Config
  attr_config :api_key    # GEMINI_API_KEY
  attr_config :model      # GEMINI_MODEL
  attr_config :timeout    # GEMINI_TIMEOUT
end

# Custom prefix
class StripeConfig < Anyway::Config
  config_name :payment    # Uses PAYMENT_* prefix instead of STRIPE_*

  attr_config :api_key,   # PAYMENT_API_KEY
              :webhook_secret
end
```

### Nested Configuration

```ruby
class AppConfig < Anyway::Config
  attr_config :name,
              :environment,
              database: {
                host: "localhost",
                port: 5432,
                pool: 5
              },
              redis: {
                url: "redis://localhost:6379"
              }
end

# Access nested values
AppConfig.new.database.host
AppConfig.new.redis.url
```

## Directory Structure

```
config/
├── configs/
│   ├── gemini_config.rb
│   ├── stripe_config.rb
│   ├── storage_config.rb
│   └── app_config.rb
└── settings/           # YAML files (optional)
    ├── gemini.yml
    └── storage.yml
```

## YAML Configuration Files

```yaml
# config/settings/gemini.yml
default: &default
  model: gemini-pro
  timeout: 30

development:
  <<: *default
  api_key: <%= ENV["GEMINI_API_KEY"] %>

test:
  <<: *default
  api_key: test-key

production:
  <<: *default
  timeout: 60
```

## Using Configurations

### Direct Instantiation

```ruby
config = GeminiConfig.new
config.api_key
config.timeout
```

### Singleton Pattern (Recommended)

```ruby
class GeminiConfig < Anyway::Config
  attr_config :api_key, :model

  class << self
    def instance
      @instance ||= new
    end
  end
end

# Usage anywhere
GeminiConfig.instance.api_key
```

### Memoized Helper Method

```ruby
# app/models/concerns/gemini_client.rb
module GeminiClient
  extend ActiveSupport::Concern

  private

  def gemini_config
    @gemini_config ||= GeminiConfig.new
  end
end
```

### In Jobs/Services

```ruby
class Cloud::CardGenerator
  def initialize(cloud)
    @cloud = cloud
    @config = GeminiConfig.new
  end

  def generate
    return unless @config.configured?

    client = Gemini::Client.new(
      api_key: @config.api_key,
      timeout: @config.timeout
    )
    # ...
  end
end
```

## Validation

```ruby
class StorageConfig < Anyway::Config
  attr_config :bucket,
              :region,
              :access_key_id,
              :secret_access_key

  # Required attributes
  required :bucket, :region

  # Conditional requirements
  required :access_key_id, :secret_access_key, env: :production

  # Custom validation
  def validate!
    super
    raise_validation_error("Invalid region") unless valid_regions.include?(region)
  end

  private

  def valid_regions
    %w[us-east-1 us-west-2 eu-west-1]
  end
end
```

## Type Coercion

```ruby
class ApiConfig < Anyway::Config
  # Automatic coercion
  attr_config timeout: 30       # Integer
  attr_config enabled: true     # Boolean
  attr_config rate: 1.5         # Float

  # Coerce arrays from comma-separated strings
  coerce_types allowed_origins: {
    type: :string,
    array: true
  }
  # ALLOWED_ORIGINS="example.com,other.com" => ["example.com", "other.com"]
end
```

## Testing Configurations

```ruby
# spec/configs/gemini_config_spec.rb
RSpec.describe GeminiConfig do
  subject(:config) { described_class.new }

  describe "defaults" do
    it "has default timeout" do
      expect(config.timeout).to eq(30)
    end

    it "has default model" do
      expect(config.model).to eq("gemini-pro")
    end
  end

  describe "validation" do
    it "requires api_key" do
      expect { described_class.new(api_key: nil) }
        .to raise_error(Anyway::Config::ValidationError)
    end
  end

  describe "#configured?" do
    context "with api_key" do
      subject(:config) { described_class.new(api_key: "test") }

      it "returns true" do
        expect(config.configured?).to be true
      end
    end
  end
end
```

### Override in Tests

```ruby
# spec/support/anyway_config.rb
RSpec.configure do |config|
  config.around(:each) do |example|
    # Override config for test
    with_env(
      "GEMINI_API_KEY" => "test-key",
      "GEMINI_TIMEOUT" => "5"
    ) do
      example.run
    end
  end
end
```

## Common Patterns

### Feature Flags

```ruby
class FeaturesConfig < Anyway::Config
  attr_config dark_mode: false,
              beta_features: false,
              maintenance_mode: false

  def maintenance?
    maintenance_mode
  end

  def beta?
    beta_features
  end
end
```

### External API Client

```ruby
class OpenAIConfig < Anyway::Config
  attr_config :api_key,
              :organization_id,
              model: "gpt-4",
              max_tokens: 1000,
              temperature: 0.7

  required :api_key

  def client_options
    {
      access_token: api_key,
      organization_id: organization_id
    }.compact
  end
end
```

### Multi-Environment Storage

```ruby
class StorageConfig < Anyway::Config
  attr_config provider: "local",
              bucket: nil,
              endpoint: nil,
              credentials: {}

  def s3?
    provider == "s3"
  end

  def r2?
    provider == "r2"
  end

  def local?
    provider == "local"
  end

  def service_options
    case provider
    when "s3" then s3_options
    when "r2" then r2_options
    else local_options
    end
  end
end
```

## Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Direct `ENV["KEY"]` | No type safety, scattered | Config class |
| `ENV.fetch` everywhere | Duplication, no validation | Centralized config |
| Rails credentials | Complex, hard to test | anyway_config classes |
| Hardcoded secrets | Security risk | Environment variables |
| Magic strings | Typos, no IDE support | Config constants |

## Quick Reference

```ruby
# Create config class
class MyConfig < Anyway::Config
  attr_config :required_key,    # Required
              optional: "default" # With default

  required :required_key
end

# Environment variables
MY_REQUIRED_KEY=value  # Mapped automatically
MY_OPTIONAL=override   # Overrides default

# Usage
config = MyConfig.new
config.required_key    # => "value"
config.optional        # => "override"
```

## Detailed References

- `references/advanced-patterns.md` - Dynamic configs, callbacks, inheritance
