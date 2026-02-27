---
name: slack-auth-security
description: OAuth flows, token management, and security best practices for Slack apps. Use when implementing app distribution, multi-workspace installations, token storage and rotation, managing scopes and permissions, or securing production Slack applications.
keywords: [slack, oauth2, authentication, security, token-management]
---

# Slack Auth & Security

## OAuth 2.0 Flow

### Bot Token vs User Token

**Bot Token** (`xoxb-`):
- Performs actions as the bot user
- Requires bot scopes
- Recommended for most operations

**User Token** (`xoxp-`):
- Acts on behalf of specific user
- Requires user scopes
- Used for user-specific operations

## Basic OAuth Flow

```go
import "github.com/slack-go/slack"

// Step 1: Generate authorization URL
state := generateRandomState()
authURL := fmt.Sprintf(
    "https://slack.com/oauth/v2/authorize?client_id=%s&scope=%s&state=%s",
    clientID,
    "chat:write,channels:read",
    state,
)
```

See [oauth-flow.md](../../references/oauth-flow.md) for complete OAuth implementation.

## Token Management

### Storing Tokens Securely

```go
// NEVER hardcode tokens
// Use environment variables or secrets manager
token := os.Getenv("SLACK_BOT_TOKEN")
api := slack.New(token)
```

### Token Rotation

```go
// Rotate tokens periodically
newToken, err := api.RotateTokens(refreshToken)
if err != nil {
    return err
}

// Update stored token
storeToken(newToken)
```

See [token-management.md](../../references/token-management.md) for storage strategies and rotation patterns.

## Scopes and Permissions

### Required Scopes by Operation

**Messaging**:
- `chat:write` - Send messages
- `chat:write.public` - Post to channels bot isn't in

**Channels**:
- `channels:read` - View public channels
- `channels:manage` - Create/manage channels
- `groups:read` - View private channels

**Users**:
- `users:read` - View users
- `users:read.email` - View user emails

See [scopes-permissions.md](../../references/scopes-permissions.md) for comprehensive scope guide.

## Security Best Practices

### 1. Request Verification

Always verify requests from Slack:

```go
import "github.com/slack-go/slack"

func verifySlackRequest(r *http.Request, signingSecret string) bool {
    verifier, err := slack.NewSecretsVerifier(r.Header, signingSecret)
    if err != nil {
        return false
    }

    body, _ := ioutil.ReadAll(r.Body)
    verifier.Write(body)

    return verifier.Ensure() == nil
}
```

### 2. HTTPS Only

Never use HTTP endpoints for webhooks:
- ✅ `https://your-app.com/slack/events`
- ❌ `http://your-app.com/slack/events`

### 3. Token Storage

- Use environment variables for development
- Use secrets managers (AWS Secrets Manager, HashiCorp Vault) for production
- Encrypt tokens at rest
- Never commit tokens to version control

### 4. Rate Limiting

Implement rate limiting to avoid abuse:

```go
type RateLimiter struct {
    requests map[string][]time.Time
    mu       sync.Mutex
}

func (rl *RateLimiter) Allow(userID string, maxRequests int, window time.Duration) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()

    now := time.Now()
    cutoff := now.Add(-window)

    // Remove old requests
    var validRequests []time.Time
    for _, t := range rl.requests[userID] {
        if t.After(cutoff) {
            validRequests = append(validRequests, t)
        }
    }

    if len(validRequests) >= maxRequests {
        return false
    }

    rl.requests[userID] = append(validRequests, now)
    return true
}
```

## Multi-Workspace Installations

### Token Per Workspace

Store tokens separately for each workspace:

```go
type WorkspaceToken struct {
    TeamID      string
    BotToken    string
    BotUserID   string
    InstalledAt time.Time
}

func getAPIForTeam(teamID string) (*slack.Client, error) {
    token, err := loadTokenForTeam(teamID)
    if err != nil {
        return nil, err
    }
    return slack.New(token.BotToken), nil
}
```

## App Manifest API

Programmatically configure apps:

```go
manifest := &slack.Manifest{
    DisplayInformation: slack.ManifestDisplayInformation{
        Name: "My Bot",
        Description: "Helpful bot",
    },
    Features: slack.ManifestFeatures{
        BotUser: &slack.ManifestBotUser{
            DisplayName: "mybot",
            AlwaysOnline: true,
        },
    },
    OAuthConfig: slack.ManifestOAuthConfig{
        Scopes: slack.ManifestOAuthScopes{
            Bot: []string{"chat:write", "channels:read"},
        },
    },
}

_, err := api.CreateManifest(manifest)
```

See [manifest-api.md](../../references/manifest-api.md) for manifest patterns.

## Production Checklist

See [security-checklist.md](../../references/security-checklist.md) for comprehensive security audit:
- ✅ HTTPS endpoints
- ✅ Request signature verification
- ✅ Token encryption at rest
- ✅ Rate limiting
- ✅ Audit logging
- ✅ Error handling (don't leak sensitive info)
- ✅ Regular token rotation

## Common Pitfalls

- Hardcoding tokens in code
- Not verifying request signatures
- Using HTTP instead of HTTPS
- Storing tokens in plain text
- Not implementing rate limiting
- Exposing sensitive errors to users
