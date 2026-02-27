---
name: tapestry
description: Social graph protocol integration for Copium app using Tapestry API. Use when building social features - profiles, follows, content/posts, comments, likes, activity feeds, identity resolution, or trades. Covers all 41 Tapestry API endpoints.
argument-hint: [feature-or-endpoint]
---

# Tapestry Social Graph Integration

You are building social features for a React Native (Expo) mobile app called **Copium** - a social trading app using Tapestry and Polymarket.

## Project Context

- **Framework**: React Native with Expo Router (file-based routing)
- **Auth**: Privy (OAuth + embedded Solana wallet) + Mobile Wallet Adapter (MWA)
- **Wallet hook**: `useUnifiedWallet()` from `@/hooks/use-wallet` provides `address`, `walletType`, `provider`, `isConnected`, `logout`
- **State**: React Query (`@tanstack/react-query`) for server state
- **Styling**: StyleSheet with `#0a0a0a` bg, `#232323` borders, `PlusJakartaSans` font
- **API Key**: `process.env.TAPESTRY_API_KEY` (server-side only, stored in `.env.local`)
- **Base URL**: `https://api.usetapestry.dev/api/v1`

## API Authentication

All endpoints require `apiKey` as a query parameter:
```
GET /profiles/{id}?apiKey=YOUR_API_KEY
```

The API key is in `.env.local` as `TAPESTRY_API_KEY`. Since this is a React Native app (no server-side routes), call the Tapestry API directly from a utility module, keeping the API key secure via `process.env.TAPESTRY_API_KEY` (bundled at build time but not exposed to web).

## Execution Methods (for write operations)

| Method | Speed | Use When |
|--------|-------|----------|
| `FAST_UNCONFIRMED` | <1s | Default. Returns after graph DB write, on-chain tx in background |
| `QUICK_SIGNATURE` | ~5s | Need tx signature but don't need confirmation |
| `CONFIRMED_AND_PARSED` | ~15s | Need full on-chain confirmation |

## API Client Pattern

Create API calls in `lib/tapestry/` directory. Use this pattern:

```typescript
// lib/tapestry/client.ts
const TAPESTRY_BASE = 'https://api.usetapestry.dev/api/v1';
const API_KEY = process.env.TAPESTRY_API_KEY!;

export async function tapestryFetch<T>(
  endpoint: string,
  options?: { method?: string; body?: any; params?: Record<string, string> }
): Promise<T> {
  const { method = 'GET', body, params = {} } = options ?? {};
  const url = new URL(`${TAPESTRY_BASE}${endpoint}`);
  url.searchParams.set('apiKey', API_KEY);
  Object.entries(params).forEach(([k, v]) => v && url.searchParams.set(k, v));

  const res = await fetch(url.toString(), {
    method,
    headers: body ? { 'Content-Type': 'application/json' } : undefined,
    body: body ? JSON.stringify(body) : undefined,
  });

  if (!res.ok) {
    const err = await res.json().catch(() => ({}));
    throw new Error(err.error || `Tapestry API error: ${res.status}`);
  }
  return res.json();
}
```

## React Hook Pattern

```typescript
// hooks/use-tapestry-profile.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { tapestryFetch } from '@/lib/tapestry/client';

export function useTapestryProfile(username: string) {
  return useQuery({
    queryKey: ['tapestry-profile', username],
    queryFn: () => tapestryFetch<ProfileResponse>(`/profiles/${username}`),
    enabled: !!username,
  });
}
```

## All Endpoints Reference

For detailed endpoint specs, see [reference.md](reference.md).

### Quick Endpoint Map

**Profiles**
- `GET /profiles/` - List/search profiles (by wallet, phone, twitter, email)
- `POST /profiles/findOrCreate` - Find or create profile
- `GET /profiles/{id}` - Get profile by ID or username
- `PUT /profiles/{id}` - Update profile
- `GET /profiles/{id}/followers` - Get followers
- `GET /profiles/{id}/following` - Get following
- `GET /profiles/{id}/followers/global` - Cross-namespace followers
- `GET /profiles/{id}/following/global` - Cross-namespace following
- `GET /profiles/{id}/following-who-follow` - Mutual connections
- `GET /profiles/{id}/wallets` - Get linked wallets
- `PATCH /profiles/{id}/wallets` - Link wallets
- `DELETE /profiles/{id}/wallets` - Unlink wallets
- `PATCH /profiles/{id}/contacts` - Link contacts
- `DELETE /profiles/{id}/contacts` - Unlink contacts
- `POST /profiles/{id}/notification` - Send notification
- `GET /profiles/{id}/referrals` - Get referral tree
- `GET /profiles/{id}/suggested-profiles` - Suggested follows
- `GET /profiles/suggested/{identifier}` - Suggestions by wallet/contact
- `GET /profiles/suggested/{identifier}/global` - Cross-namespace suggestions
- `GET /profiles/token-owners/{tokenAddress}` - Token holder profiles
- `GET /search/profiles` - Search profiles by text

**Social Graph (Followers)**
- `POST /followers/add` - Follow `{ startId, endId }`
- `POST /followers/remove` - Unfollow `{ startId, endId }`
- `GET /followers/state` - Check follow state `?startId=&endId=`

**Content (Posts)**
- `GET /contents/` - List content (with filters, ordering, pagination)
- `GET /contents/aggregation` - Aggregate content properties
- `POST /contents/batch/read` - Batch get by IDs (max 20)
- `POST /contents/findOrCreate` - Create content `{ id, profileId, properties: [{key,value}] }`
- `GET /contents/{id}` - Get content details
- `PUT /contents/{id}` - Update content properties
- `DELETE /contents/{id}` - Delete content

**Comments**
- `GET /comments/` - List comments `?contentId=&profileId=&targetProfileId=`
- `POST /comments/` - Create comment `{ profileId, text, contentId?, commentId?, targetProfileId? }`
- `POST /comments/batch/read` - Batch get (max 20)
- `GET /comments/{id}` - Get comment with replies
- `PUT /comments/{id}` - Update comment
- `DELETE /comments/{id}` - Delete comment
- `GET /comments/{id}/replies` - Get replies

**Likes**
- `GET /likes/{nodeId}` - Get profiles who liked a node
- `POST /likes/{nodeId}` - Like `{ startId }`
- `DELETE /likes/{nodeId}` - Unlike `{ startId }`

**Activity Feeds**
- `GET /activity/feed` - User activity feed `?username=`
- `GET /activity/global` - Global activity feed
- `GET /activity/swap` - Swap activity from followed wallets `?username=&tokenAddress=`

**Identities**
- `GET /identities/{id}` - Resolve wallets/contacts from ID
- `GET /identities/{id}/profiles` - Find profiles across namespaces

**Trades**
- `POST /trades/` - Log a trade
- `GET /trades/all-trades` - Get all trades in time period
- `GET /trades/fetch-transaction-history` - Wallet tx history

**Wallets**
- `POST /wallets/{address}/connect` - Connect two wallets
- `GET /wallets/{address}/socialCounts` - Social counts by wallet

## Key Integration Points for Copium

### 1. Profile Creation (on signup/login)
After wallet connection + X login, call `findOrCreate`:
```typescript
await tapestryFetch('/profiles/findOrCreate', {
  method: 'POST',
  body: {
    username: twitterHandle, // from Privy OAuth
    walletAddress: address,  // from useUnifiedWallet
    blockchain: 'SOLANA',
    image: twitterProfileImage,
    bio: twitterBio,
    contact: { id: twitterHandle, type: 'TWITTER' },
  },
});
```

### 2. Follow/Unfollow
```typescript
await tapestryFetch('/followers/add', {
  method: 'POST',
  body: { startId: myProfileId, endId: targetProfileId },
});
```

### 3. Create a Post (Content)
```typescript
await tapestryFetch('/contents/findOrCreate', {
  method: 'POST',
  body: {
    id: `post-${Date.now()}`,
    profileId: myProfileId,
    properties: [
      { key: 'text', value: 'My post content' },
      { key: 'type', value: 'trade_call' },
    ],
  },
});
```

### 4. Like Content
```typescript
await tapestryFetch(`/likes/${contentId}`, {
  method: 'POST',
  body: { startId: myProfileId },
});
```

### 5. Comment on Content
```typescript
await tapestryFetch('/comments/', {
  method: 'POST',
  body: {
    profileId: myProfileId,
    contentId: contentId,
    text: 'Great trade!',
  },
});
```

### 6. Activity Feed
```typescript
const feed = await tapestryFetch('/activity/feed', {
  params: { username: myUsername },
});
```

### 7. Search Users
```typescript
const results = await tapestryFetch('/search/profiles', {
  params: { query: searchText },
});
```

### 8. Polymarket Integration Pattern
Store Polymarket-related data as content properties:
```typescript
await tapestryFetch('/contents/findOrCreate', {
  method: 'POST',
  body: {
    id: `prediction-${marketId}-${profileId}`,
    profileId: myProfileId,
    properties: [
      { key: 'type', value: 'prediction' },
      { key: 'marketId', value: polymarketConditionId },
      { key: 'position', value: 'YES' },
      { key: 'amount', value: 50 },
      { key: 'odds', value: 0.65 },
    ],
  },
});
```

## Response Types

```typescript
interface TapestryProfile {
  id: string;
  namespace: string;
  created_at: number;
  username: string;
  bio: string | null;
  image: string | null;
}

interface ProfileResponse {
  profile: TapestryProfile;
  walletAddress: string;
  socialCounts: { followers: number; following: number };
  namespace: { name: string | null; readableName: string | null };
}

interface ContentResponse {
  id: string;
  created_at: number;
  properties: Record<string, any>;
}

interface CommentResponse {
  comment: { id: string; created_at: number; text: string };
  author: TapestryProfile;
  socialCounts: { likeCount: number };
  requestingProfileSocialInfo?: { hasLiked: boolean };
}

interface ActivityItem {
  type: 'following' | 'new_content' | 'like' | 'comment' | 'new_follower';
  actor_id: string;
  actor_username: string;
  target_id?: string;
  target_username?: string;
  timestamp: number;
  activity: string;
}

interface PaginatedResponse<T> {
  page: number;
  pageSize: number;
  totalCount: number;
  data?: T[];
  profiles?: T[];
  activities?: T[];
}
```

## File Structure Convention

```
lib/tapestry/
  client.ts          # Base fetch wrapper
  profiles.ts        # Profile API functions
  social.ts          # Follow/unfollow/like functions
  content.ts         # Content/posts CRUD
  comments.ts        # Comments CRUD
  activity.ts        # Activity feed functions
  types.ts           # TypeScript interfaces

hooks/
  use-tapestry-profile.ts
  use-tapestry-feed.ts
  use-tapestry-social.ts
  use-tapestry-content.ts
```

When implementing features with `$ARGUMENTS`, refer to [reference.md](reference.md) for full endpoint details.
