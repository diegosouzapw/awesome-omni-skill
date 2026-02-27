---
name: oauth-platform-integration
description: Add new OAuth platform connectors to Twin Me Soul Signature Platform. Use when the user wants to integrate a new platform (Spotify, Netflix, LinkedIn, TikTok, etc.) for soul signature data extraction.
---

# OAuth Platform Integration Skill

This skill streamlines adding new OAuth-enabled platforms to the Twin Me platform for authentic soul signature data collection.

## When to Use This Skill

Trigger this skill when:
- User requests adding a new platform connector (e.g., "Add Spotify integration")
- Implementing OAuth flow for entertainment or professional platforms
- Setting up data extraction for a new service

## Prerequisites

Before adding a platform, verify:
1. Platform has a public OAuth 2.0 API
2. OAuth credentials obtained (CLIENT_ID, CLIENT_SECRET)
3. Platform API documentation accessible

## Integration Checklist

### 1. OAuth App Configuration (External)

For each platform, configure OAuth app settings:

**Required redirect URIs:**
- Local development: `http://localhost:8086/oauth/callback`
- Production: `https://twin-ai-learn.vercel.app/oauth/callback`

**Required OAuth scopes** (platform-specific):
- Spotify: `user-read-recently-played`, `user-top-read`, `user-library-read`
- Discord: `identify`, `guilds`, `messages.read`
- GitHub: `user`, `repo` (read-only)
- LinkedIn: `r_liteprofile`, `r_emailaddress`
- Netflix: (No official API - requires browser extension approach)

### 2. Environment Variables

Add to `.env` file:

```env
# [PLATFORM_NAME] OAuth
[PLATFORM]_CLIENT_ID=your-client-id
[PLATFORM]_CLIENT_SECRET=your-client-secret
```

**Example:**
```env
SPOTIFY_CLIENT_ID=abc123def456
SPOTIFY_CLIENT_SECRET=xyz789uvw012
```

### 3. Platform API Configuration

Add platform config to `api/services/platformAPIMappings.js`:

```javascript
const PLATFORM_CONFIGS = {
  // ... existing platforms

  [newPlatform]: {
    name: 'Platform Display Name',
    authUrl: 'https://platform.com/oauth/authorize',
    tokenUrl: 'https://platform.com/oauth/token',
    scopes: ['scope1', 'scope2', 'scope3'],
    apiBaseUrl: 'https://api.platform.com/v1',

    // OAuth endpoints
    endpoints: {
      userProfile: '/me',
      recentActivity: '/me/recent',
      topItems: '/me/top',
    },

    // Token handling
    tokenType: 'Bearer', // or 'OAuth'
    refreshable: true, // Does platform support token refresh?

    // Rate limiting
    rateLimit: {
      requests: 100,
      window: 3600, // seconds
    },
  },
};
```

### 4. OAuth Route Implementation

Add OAuth routes to `api/routes/entertainment-connectors.js` (or `mcp-connectors.js` for professional platforms):

```javascript
// Initiate OAuth flow
router.get('/connect/:platform', async (req, res) => {
  const { platform } = req.params;
  const userId = req.user.id; // From JWT auth middleware

  const config = PLATFORM_CONFIGS[platform];
  if (!config) {
    return res.status(404).json({ error: 'Platform not supported' });
  }

  // Generate OAuth state for security
  const state = crypto.randomBytes(16).toString('hex');
  await saveOAuthState(userId, platform, state);

  // Build authorization URL
  const authUrl = new URL(config.authUrl);
  authUrl.searchParams.append('client_id', process.env[`${platform.toUpperCase()}_CLIENT_ID`]);
  authUrl.searchParams.append('redirect_uri', `${process.env.CLIENT_URL}/oauth/callback`);
  authUrl.searchParams.append('response_type', 'code');
  authUrl.searchParams.append('scope', config.scopes.join(' '));
  authUrl.searchParams.append('state', state);

  res.json({ authUrl: authUrl.toString() });
});

// Handle OAuth callback
router.get('/callback/:platform', async (req, res) => {
  const { platform } = req.params;
  const { code, state } = req.query;

  try {
    // Verify state to prevent CSRF
    const validState = await verifyOAuthState(state);
    if (!validState) {
      throw new Error('Invalid OAuth state');
    }

    // Exchange code for tokens
    const tokens = await exchangeCodeForTokens(platform, code);

    // Encrypt and store tokens
    await savePlatformConnection(validState.userId, platform, {
      accessToken: tokens.access_token,
      refreshToken: tokens.refresh_token,
      expiresAt: Date.now() + (tokens.expires_in * 1000),
    });

    // Redirect back to frontend
    res.redirect(`${process.env.CLIENT_URL}/get-started?connected=${platform}`);
  } catch (error) {
    console.error(`OAuth callback error for ${platform}:`, error);
    res.redirect(`${process.env.CLIENT_URL}/get-started?error=${platform}`);
  }
});
```

### 5. Data Extraction Service

Add extraction logic to `api/services/dataExtraction.js`:

```javascript
async function extract[PlatformName]Data(userId) {
  const connection = await getPlatformConnection(userId, '[platform]');
  if (!connection || !connection.accessToken) {
    throw new Error('Platform not connected');
  }

  try {
    // Decrypt access token
    const accessToken = decryptToken(connection.accessToken);

    // Fetch data from platform API
    const config = PLATFORM_CONFIGS['[platform]'];
    const response = await axios.get(
      `${config.apiBaseUrl}${config.endpoints.recentActivity}`,
      {
        headers: {
          'Authorization': `${config.tokenType} ${accessToken}`,
        },
      }
    );

    // Transform platform data to soul signature format
    const extractedData = transformPlatformData(response.data, '[platform]');

    // Save to database
    await saveSoulData(userId, {
      platform: '[platform]',
      dataType: 'recent_activity',
      rawData: response.data,
      extractedPatterns: extractedData,
      extractedAt: new Date(),
    });

    return {
      success: true,
      itemsExtracted: extractedData.length,
      platform: '[platform]',
    };
  } catch (error) {
    // Handle token expiration
    if (error.response?.status === 401) {
      await markConnectionRequiresReauth(userId, '[platform]');
      return {
        success: false,
        requiresReauth: true,
        error: 'Token expired - reconnection required',
      };
    }

    throw error;
  }
}
```

### 6. Frontend Platform Card

Add platform card to `src/pages/GetStarted.tsx`:

```typescript
const platformConfig = {
  // ... existing platforms

  [newPlatform]: {
    name: 'Platform Name',
    icon: PlatformIcon, // Import from lucide-react or custom SVG
    description: 'What this platform reveals about your soul signature',
    setupTime: '10 seconds setup',
    category: 'essential' | 'optional', // Determines which section
    insights: [
      'Primary Insight Type',
      'Secondary Insight Type',
      '+N more',
    ],
    color: '#BRAND_COLOR', // Platform brand color for UI
  },
};
```

### 7. Database Schema Update

Ensure `data_connectors` table supports the new platform:

```sql
-- Platform connection already exists in schema
-- Just verify the platform name is consistent:

SELECT * FROM data_connectors
WHERE provider = '[platform_name]';

-- If adding custom metadata fields:
ALTER TABLE data_connectors
ADD COLUMN IF NOT EXISTS platform_metadata JSONB DEFAULT '{}'::jsonb;
```

### 8. Soul Signature Mapping

Define how platform data maps to personality traits in `api/services/soulSignatureAnalyzer.js`:

```javascript
const PLATFORM_TRAIT_MAPPINGS = {
  // ... existing mappings

  [newPlatform]: {
    // Big Five personality traits
    traits: {
      openness: (data) => calculateOpenness(data.genres, data.diversity),
      conscientiousness: (data) => calculateConscientiousness(data.organizationLevel),
      extraversion: (data) => calculateExtraversion(data.socialActivity),
      agreeableness: (data) => calculateAgreeableness(data.interactions),
      neuroticism: (data) => calculateNeuroticism(data.emotionalContent),
    },

    // Communication style
    communication: (data) => analyzeCommunicationStyle(data.messages),

    // Interests and preferences
    interests: (data) => extractInterests(data.content),
  },
};
```

## Testing Checklist

After implementation, verify:

- [ ] OAuth flow redirects correctly (local + production)
- [ ] Tokens encrypted/decrypted successfully
- [ ] Connection shows "Connected" status in UI
- [ ] Data extraction endpoint returns items
- [ ] No token decryption errors in logs
- [ ] Platform appears in Soul Signature dashboard
- [ ] Personality traits updated with platform data

## Common Pitfalls

### Redirect URI Mismatch
**Error:** "redirect_uri did not match"
**Solution:** Ensure OAuth app has BOTH localhost AND production URLs configured

### Token Encryption Key Mismatch
**Error:** "Unsupported state or unable to authenticate data"
**Solution:** Use current `ENCRYPTION_KEY` from `.env`, clear old tokens before reconnecting

### Rate Limiting
**Error:** 429 Too Many Requests
**Solution:** Implement exponential backoff, respect platform rate limits in `PLATFORM_CONFIGS`

### Scope Permissions
**Error:** 403 Forbidden on specific endpoints
**Solution:** Verify OAuth scopes match required API permissions

## Platform-Specific Notes

### Spotify
- Requires both user authorization AND token refresh flow
- Rate limit: 180 requests per minute
- Best practice: Batch requests for user's top tracks/artists

### Discord
- Guilds endpoint requires `guilds` scope
- Message access limited to servers bot has joined
- Rate limit: 50 requests per second globally

### GitHub
- Only allows ONE callback URL per OAuth app
- Recommendation: Create separate dev OAuth app for localhost
- Rate limit: 5,000 requests/hour for authenticated requests

### LinkedIn
- V2 API requires separate permissions for profile vs email
- Limited data access compared to V1 (deprecated)
- Rate limit: varies by endpoint

### Netflix / Streaming Platforms
**Note:** Most streaming platforms (Netflix, HBO, Disney+) have NO public API

**Alternative Approach:** Browser extension for data extraction
- See `browser-extension/` directory
- Uses content scripts to capture viewing history
- Sends data to backend via secure API

## File References

For detailed implementation examples, see:
- **OAuth Config:** `./oauth-config-examples.md`
- **Extraction Patterns:** `./extraction-patterns.md`
- **API Rate Limiting:** `./rate-limiting-guide.md`

## Quick Start Template

Use the provided script to scaffold a new platform integration:

```bash
cd twin-me
node skills/oauth-platform-integration/scripts/scaffold-platform.js <platform-name>
```

This generates:
- Environment variable template
- Platform config skeleton
- OAuth route boilerplate
- Extraction service stub
- Frontend platform card
- Test suite

## Success Criteria

Platform integration is complete when:

1. ✅ OAuth flow works end-to-end (local + production)
2. ✅ Tokens stored encrypted in database
3. ✅ Data extraction returns > 0 items
4. ✅ Platform appears as "Connected" in UI
5. ✅ Soul signature updates with new data
6. ✅ No console errors or token decryption issues

---

**Skill Version:** 1.0.0
**Last Updated:** January 18, 2025
**Maintainer:** Twin Me Development Team
