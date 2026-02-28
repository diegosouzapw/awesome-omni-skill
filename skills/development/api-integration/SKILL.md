---
name: api-integration
description: "Expert API integration decisions for iOS/tvOS: REST vs GraphQL trade-offs, API versioning strategies, caching layer design, and offline-first architecture choices. Use when designing network architecture, implementing offline support, or choosing between API patterns. Trigger keywords: REST, GraphQL, API versioning, caching, offline-first, URLSession, background fetch, ETag, pagination, rate limiting"
version: "3.0.0"
---

# API Integration — Expert Decisions

Expert decision frameworks for API integration choices. Claude knows URLSession and Codable — this skill provides judgment calls for architecture decisions and caching strategies.

---

## Decision Trees

### REST vs GraphQL

```
What's your data access pattern?
├─ Fixed, well-defined endpoints
│  └─ REST
│     Simpler caching, HTTP semantics
│
├─ Flexible queries, varying data needs per screen
│  └─ GraphQL
│     Single endpoint, client specifies shape
│
├─ Real-time subscriptions needed?
│  └─ GraphQL subscriptions or WebSocket + REST
│     GraphQL has built-in subscription support
│
└─ Offline-first with sync?
   └─ REST is simpler for conflict resolution
      GraphQL mutations harder to replay
```

**The trap**: Choosing GraphQL because it's trendy. If your API has stable endpoints and you control both client and server, REST is simpler.

### API Versioning Strategy

```
How stable is your API?
├─ Stable, rarely changes
│  └─ No versioning needed initially
│     Add when first breaking change occurs
│
├─ Breaking changes expected
│  └─ URL path versioning (/v1/, /v2/)
│     Most explicit, easiest to manage
│
├─ Gradual migration needed
│  └─ Header versioning (Accept-Version: v2)
│     Same URL, version negotiated
│
└─ Multiple versions simultaneously
   └─ Consider if you really need this
      Maintenance burden is high
```

### Caching Strategy Selection

```
What type of data?
├─ Static/rarely changes (images, config)
│  └─ Aggressive cache (URLCache + ETag)
│     Cache-Control: max-age=86400
│
├─ User-specific, changes occasionally
│  └─ Cache with validation (ETag/Last-Modified)
│     Always validate, but use cached if unchanged
│
├─ Frequently changing (feeds, notifications)
│  └─ No cache or short TTL
│     Cache-Control: no-cache or max-age=60
│
└─ Critical real-time data (payments, inventory)
   └─ No cache, always fetch
      Cache-Control: no-store
```

### Offline-First Architecture

```
How important is offline?
├─ Nice to have (can show empty state)
│  └─ Simple memory cache
│     Clear on app restart
│
├─ Must show stale data when offline
│  └─ Persistent cache (Core Data, Realm, SQLite)
│     Fetch fresh when online, fallback to cache
│
├─ Must sync user changes when back online
│  └─ Full offline-first architecture
│     Local-first writes, sync queue, conflict resolution
│
└─ Real-time collaboration
   └─ Consider CRDTs or operational transforms
      Complex — usually overkill for mobile
```

---

## NEVER Do

### Service Layer Design

**NEVER** call NetworkManager directly from ViewModels:
```swift
// ❌ ViewModel knows about network layer
@MainActor
final class UserViewModel: ObservableObject {
    func loadUser() async {
        let response = try await NetworkManager.shared.request(APIRouter.getUser(id: "123"))
        // ViewModel parsing network response directly
    }
}

// ✅ Service layer abstracts network details
@MainActor
final class UserViewModel: ObservableObject {
    private let userService: UserServiceProtocol

    func loadUser() async {
        user = try await userService.getUser(id: "123")
        // ViewModel works with domain models
    }
}
```

**NEVER** expose DTOs beyond service layer:
```swift
// ❌ DTO leaks to ViewModel
func getUser() async throws -> UserDTO {
    try await networkManager.request(...)
}

// ✅ Service maps DTO to domain model
func getUser() async throws -> User {
    let dto: UserDTO = try await networkManager.request(...)
    return User(from: dto)  // Mapping happens here
}
```

**NEVER** hardcode base URLs:
```swift
// ❌ Can't change per environment
static let baseURL = "https://api.production.com"

// ✅ Environment-driven configuration
static var baseURL: String {
    #if DEBUG
    return "https://api.staging.com"
    #else
    return "https://api.production.com"
    #endif
}

// Better: Inject via configuration
```

### Error Handling

**NEVER** show raw error messages to users:
```swift
// ❌ Exposes technical details
errorMessage = error.localizedDescription  // "JSON decoding error at keyPath..."

// ✅ Map to user-friendly messages
errorMessage = mapToUserMessage(error)

func mapToUserMessage(_ error: Error) -> String {
    switch error {
    case let networkError as NetworkError:
        return networkError.userMessage
    case is DecodingError:
        return "Unable to process response. Please try again."
    default:
        return "Something went wrong. Please try again."
    }
}
```

**NEVER** treat all errors the same:
```swift
// ❌ Same handling for all errors
catch {
    showErrorAlert(error.localizedDescription)
}

// ✅ Different handling per error type
catch is CancellationError {
    return  // User navigated away — not an error
}
catch NetworkError.unauthorized {
    await sessionManager.logout()  // Force re-auth
}
catch NetworkError.noConnection {
    showOfflineBanner()  // Different UI treatment
}
catch {
    showErrorAlert("Something went wrong")
}
```

### Caching

**NEVER** cache sensitive data without encryption:
```swift
// ❌ Tokens cached in plain URLCache
URLCache.shared.storeCachedResponse(response, for: request)
// Login response with tokens now on disk!

// ✅ Exclude sensitive endpoints from cache
if endpoint.isSensitive {
    request.cachePolicy = .reloadIgnoringLocalCacheData
}
```

**NEVER** assume cached data is fresh:
```swift
// ❌ Trusts cache blindly
if let cached = cache.get(key) {
    return cached  // May be hours/days old
}

// ✅ Validate cache or show stale indicator
if let cached = cache.get(key) {
    if cached.isStale {
        Task { await refreshInBackground(key) }
    }
    return (data: cached.data, isStale: cached.isStale)
}
```

### Pagination

**NEVER** load all pages at once:
```swift
// ❌ Memory explosion, slow initial load
func loadAllUsers() async throws -> [User] {
    var all: [User] = []
    var page = 1
    while true {
        let response = try await fetchPage(page)
        all.append(contentsOf: response.users)
        if response.users.isEmpty { break }
        page += 1
    }
    return all  // May be thousands of items!
}

// ✅ Paginate on demand
func loadNextPage() async throws {
    guard hasMorePages, !isLoading else { return }
    isLoading = true
    let response = try await fetchPage(currentPage + 1)
    users.append(contentsOf: response.users)
    currentPage += 1
    hasMorePages = !response.users.isEmpty
    isLoading = false
}
```

**NEVER** use offset pagination for mutable lists:
```swift
// ❌ Items shift during pagination
// User scrolls, new item added, page 2 now has duplicate
GET /users?offset=20&limit=20

// ✅ Cursor-based pagination
GET /users?after=abc123&limit=20
// Cursor is stable reference point
```

---

## Essential Patterns

### Service Layer with Caching

```swift
protocol UserServiceProtocol {
    func getUser(id: String, forceRefresh: Bool) async throws -> User
}

final class UserService: UserServiceProtocol {
    private let networkManager: NetworkManagerProtocol
    private let cache: CacheProtocol

    func getUser(id: String, forceRefresh: Bool = false) async throws -> User {
        let cacheKey = "user_\(id)"

        // Return cached if valid and not forcing refresh
        if !forceRefresh, let cached: User = cache.get(cacheKey), !cached.isExpired {
            return cached
        }

        // Fetch fresh
        let dto: UserDTO = try await networkManager.request(APIRouter.getUser(id: id))
        let user = User(from: dto)

        // Cache with TTL
        cache.set(cacheKey, value: user, ttl: .minutes(5))

        return user
    }
}
```

### Stale-While-Revalidate Pattern

```swift
func getData() async -> (data: Data?, isStale: Bool) {
    // Immediately return cached (possibly stale)
    let cached = cache.get(key)

    // Refresh in background
    Task {
        do {
            let fresh = try await fetchFromNetwork()
            cache.set(key, value: fresh)
            // Notify UI of fresh data (publisher, callback, etc.)
            await MainActor.run { self.data = fresh }
        } catch {
            // Keep showing stale data
        }
    }

    return (data: cached?.data, isStale: cached?.isExpired ?? true)
}
```

### Retry with Idempotency Key

```swift
struct CreateOrderRequest {
    let items: [OrderItem]
    let idempotencyKey: String  // Client-generated UUID
}

func createOrder(items: [OrderItem]) async throws -> Order {
    let idempotencyKey = UUID().uuidString

    return try await withRetry(maxAttempts: 3) {
        try await orderService.create(
            CreateOrderRequest(items: items, idempotencyKey: idempotencyKey)
        )
        // Server uses idempotencyKey to prevent duplicate orders
    }
}
```

### Background Fetch Configuration

```swift
// In AppDelegate
func application(_ application: UIApplication, performFetchWithCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    Task {
        do {
            let hasNewData = try await syncService.performBackgroundSync()
            completionHandler(hasNewData ? .newData : .noData)
        } catch {
            completionHandler(.failed)
        }
    }
}

// In Info.plist: UIBackgroundModes = ["fetch"]
// Call: UIApplication.shared.setMinimumBackgroundFetchInterval(...)
```

---

## Quick Reference

### API Architecture Decision Matrix

| Factor | REST | GraphQL |
|--------|------|---------|
| Fixed endpoints | ✅ Best | Overkill |
| Flexible queries | Verbose | ✅ Best |
| Caching | ✅ HTTP native | Complex |
| Real-time | WebSocket addition | ✅ Subscriptions |
| Offline sync | ✅ Easier | Harder |
| Learning curve | Lower | Higher |

### Caching Strategy by Data Type

| Data Type | Strategy | TTL |
|-----------|----------|-----|
| App config | Aggressive | 24h |
| User profile | Validate | 5-15m |
| Feed/timeline | Short or none | 1-5m |
| Payments | None | 0 |
| Images | Aggressive | 7d |

### Red Flags

| Smell | Problem | Fix |
|-------|---------|-----|
| DTO in ViewModel | Coupling to API | Map to domain model |
| Hardcoded base URL | Can't switch env | Configuration |
| Showing DecodingError | Leaks internals | User-friendly messages |
| Loading all pages | Memory explosion | Paginate on demand |
| Offset pagination for mutable data | Duplicates/gaps | Cursor pagination |
| Caching auth responses | Security risk | Exclude sensitive |
