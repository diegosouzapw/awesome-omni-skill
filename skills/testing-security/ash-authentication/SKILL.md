---
name: ash-authentication
description: AshAuthentication guidelines for implementing authentication in Ash Framework. Use when adding password, magic link, API key, or OAuth2 authentication strategies. Covers token configuration, UserIdentity resources, confirmation add-ons, and customizing authentication actions. Never hardcode credentials.
---

# AshAuthentication Guidelines

## Core Concepts

- **Strategies**: password, magic_link, api_key, OAuth2 (github, google, auth0, apple, oidc, slack)
- **Tokens**: JWT for stateless authentication
- **UserIdentity**: Links users to OAuth2 providers (optional, required for multiple providers per user)
- **Add-ons**: confirmation, logout-everywhere

## Key Principles

- **Never hardcode credentials** - always use secrets management
- Enable tokens for magic_link, confirmation, OAuth2
- Check existing strategies: `AshAuthentication.Info.strategies(MyApp.User)`

## Password Strategy

```elixir
authentication do
  strategies do
    password :password do
      identity_field :email
      hashed_password_field :hashed_password
      resettable do
        sender MyApp.PasswordResetSender
      end
    end
  end
end

# Required attributes
attributes do
  attribute :email, :ci_string, allow_nil?: false, public?: true
  attribute :hashed_password, :string, allow_nil?: false, sensitive?: true
end

identities do
  identity :unique_email, [:email]
end
```

## Magic Link Strategy

```elixir
authentication do
  strategies do
    magic_link do
      identity_field :email
      sender MyApp.MagicLinkSender
    end
  end
end

# Sender implementation required
defmodule MyApp.MagicLinkSender do
  use AshAuthentication.Sender

  def send(user_or_email, token, _opts) do
    MyApp.Emails.deliver_magic_link(user_or_email, token)
  end
end
```

## API Key Strategy

```elixir
# 1. Create API key resource
defmodule MyApp.Accounts.ApiKey do
  use Ash.Resource,
    data_layer: AshPostgres.DataLayer,
    authorizers: [Ash.Policy.Authorizer]

  actions do
    defaults [:read, :destroy]

    create :create do
      primary? true
      accept [:user_id, :expires_at]
      change {AshAuthentication.Strategy.ApiKey.GenerateApiKey,
        prefix: :myapp, hash: :api_key_hash}
    end
  end

  attributes do
    uuid_primary_key :id
    attribute :api_key_hash, :binary, allow_nil?: false, sensitive?: true
    attribute :expires_at, :utc_datetime_usec, allow_nil?: false
  end

  relationships do
    belongs_to :user, MyApp.Accounts.User, allow_nil?: false
  end

  calculations do
    calculate :valid, :boolean, expr(expires_at > now())
  end

  identities do
    identity :unique_api_key, [:api_key_hash]
  end

  policies do
    bypass AshAuthentication.Checks.AshAuthenticationInteraction do
      authorize_if always()
    end
  end
end

# 2. Add strategy to user resource
authentication do
  strategies do
    api_key do
      api_key_relationship :valid_api_keys
      api_key_hash_attribute :api_key_hash
    end
  end
end

# 3. Add relationship to user
relationships do
  has_many :valid_api_keys, MyApp.Accounts.ApiKey do
    filter expr(valid)
  end
end

# 4. Add sign-in action to user
actions do
  read :sign_in_with_api_key do
    argument :api_key, :string, allow_nil?: false
    prepare AshAuthentication.Strategy.ApiKey.SignInPreparation
  end
end
```

**API Key Security:**
- Keys are hashed for storage
- Check `user.__metadata__[:using_api_key?]` for API key auth detection
- Access key via `user.__metadata__[:api_key]` for permission checks
- Use prefixes for secret scanning compliance

## OAuth2 Strategies

Providers: github, google, auth0, apple, oidc, slack

```elixir
authentication do
  strategies do
    github do
      client_id MyApp.Secrets
      client_secret MyApp.Secrets
      redirect_uri MyApp.Secrets
      identity_resource MyApp.Accounts.UserIdentity  # Optional
    end
  end
end

# Required action (replace 'github' with provider)
actions do
  create :register_with_github do
    argument :user_info, :map, allow_nil?: false
    argument :oauth_tokens, :map, allow_nil?: false
    upsert? true
    upsert_identity :unique_email

    change AshAuthentication.GenerateTokenChange
    change AshAuthentication.Strategy.OAuth2.IdentityChange  # If using UserIdentity

    change fn changeset, _ctx ->
      user_info = Ash.Changeset.get_argument(changeset, :user_info)
      Ash.Changeset.change_attributes(changeset, Map.take(user_info, ["email"]))
    end
  end
end
```

**Provider-specific requirements:**
- auth0: also needs `base_url`
- apple: also needs `team_id`, `private_key_id`, `private_key_path`
- oidc: also needs `openid_configuration_uri`

## Add-ons

### Confirmation

```elixir
authentication do
  tokens do
    enabled? true
    token_resource MyApp.Accounts.Token
  end

  add_ons do
    confirmation :confirm do
      monitor_fields [:email]
      sender MyApp.ConfirmationSender
    end
  end
end
```

### Log Out Everywhere

```elixir
authentication do
  tokens do
    store_all_tokens? true
  end

  add_ons do
    log_out_everywhere do
      apply_on_password_change? true
    end
  end
end
```

## Token Configuration

```elixir
authentication do
  tokens do
    enabled? true
    token_resource MyApp.Accounts.Token
    signing_secret MyApp.Secrets
    token_lifetime {24, :hours}
    store_all_tokens? true  # For logout-everywhere
    require_token_presence_for_authentication? false
  end
end
```

## Strategy Protocol

```elixir
# Get and use strategies
strategy = AshAuthentication.Info.strategy!(MyApp.User, :password)
{:ok, user} = AshAuthentication.Strategy.action(strategy, :sign_in, params)

# Token operations
subject = AshAuthentication.user_to_subject(user)
{:ok, user} = AshAuthentication.subject_to_user(subject, MyApp.User)
AshAuthentication.TokenResource.revoke(MyApp.Token, token)
```

## Customizing Authentication Actions

**Security Rules:**
- Mark credentials with `sensitive?: true` (passwords, API keys, tokens)
- Use `public?: false` for internal fields and sensitive PII
- Use `public?: true` for identity fields and UI display data
- Include required changes (`GenerateTokenChange`, `HashPasswordChange`)

```elixir
create :register_with_password do
  argument :password, :string, allow_nil?: false, sensitive?: true
  argument :first_name, :string, allow_nil?: false

  accept [:email, :first_name]

  change AshAuthentication.GenerateTokenChange
  change AshAuthentication.Strategy.Password.HashPasswordChange
end
```

## Policies

Always bypass for authentication interactions:

```elixir
policies do
  bypass AshAuthentication.Checks.AshAuthenticationInteraction do
    authorize_if always()
  end
end
```
