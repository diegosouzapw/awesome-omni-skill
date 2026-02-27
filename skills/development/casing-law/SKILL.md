---
name: casing-law
description: "This skill should be used when the user is writing code with mixed casing conventions, choosing casing for identifiers, enforcing casing consistency across a codebase, or when camelCase and snake_case appear in the same file. Covers per-context casing rules, acronym handling, and the absolute prohibition of mixed conventions."
version: 1.0.0
---

# Casing Is Law, Not Preference

Mixed casing in a codebase is not a style disagreement. It is a defect. When `getUserName` sits next to `get_user_email` in the same file, the reader cannot predict the shape of the next function. Predictability is the foundation of readable code. Every language has established casing conventions. Follow them absolutely, enforce them mechanically, and treat violations the same way you treat failing tests.

## The Universal Casing Table

Every identifier in your codebase falls into one of these categories. The casing is determined by the language and the identifier's role. There is no room for personal preference.

### Python

| Identifier Type | Convention | Example |
|----------------|------------|---------|
| Variables | `snake_case` | `user_count`, `max_retries` |
| Functions | `snake_case` | `fetch_active_users`, `calculate_tax_rate` |
| Method names | `snake_case` | `def validate_email(self)` |
| Function parameters | `snake_case` | `def create_order(customer_id, line_items)` |
| Classes | `PascalCase` | `UserProfile`, `PaymentProcessor` |
| Exceptions | `PascalCase` | `InvalidTokenError`, `RateLimitExceeded` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_SECONDS` |
| Modules (files) | `snake_case` | `user_profile.py`, `payment_processor.py` |
| Packages (directories) | `snake_case` (short, no underscores preferred) | `payments`, `userprofiles` |
| Type aliases | `PascalCase` | `UserId = str`, `OrderItems = list[LineItem]` |
| Enum members | `UPPER_SNAKE_CASE` | `class Status(Enum): ACTIVE = "active"` |

**Python enforces this by convention (PEP 8), and tools like `ruff` flag violations automatically.**

### TypeScript

| Identifier Type | Convention | Example |
|----------------|------------|---------|
| Variables | `camelCase` | `userCount`, `maxRetries` |
| Functions | `camelCase` | `fetchActiveUsers`, `calculateTaxRate` |
| Method names | `camelCase` | `validateEmail()` |
| Function parameters | `camelCase` | `function createOrder(customerId, lineItems)` |
| Classes | `PascalCase` | `UserProfile`, `PaymentProcessor` |
| Interfaces | `PascalCase` | `UserProfile`, `OrderRequest` |
| Type aliases | `PascalCase` | `UserId`, `OrderItems` |
| Enums | `PascalCase` (name), `PascalCase` (members) | `enum OrderStatus { Pending, Shipped }` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |
| React components | `PascalCase` | `UserProfileCard`, `PaymentForm` |
| Hooks | `camelCase` with `use` prefix | `useUserProfile`, `usePaymentForm` |
| Files (general) | `kebab-case` | `user-profile.ts`, `payment-processor.ts` |
| Files (React components) | `PascalCase` | `UserProfileCard.tsx` |
| Files (tests) | Match source + `.test` | `user-profile.test.ts` |

**Do not prefix interfaces with `I`.** `IUserProfile` is a C# convention that has no place in TypeScript. Use `UserProfile` for the interface. If you need to distinguish an interface from a class, the class gets the more specific name: `UserProfileImpl`, `InMemoryUserProfile`.

### Go

| Identifier Type | Convention | Example |
|----------------|------------|---------|
| Exported (public) anything | `PascalCase` | `UserProfile`, `FetchActiveUsers`, `MaxRetries` |
| Unexported (private) anything | `camelCase` | `userProfile`, `fetchActiveUsers`, `maxRetries` |
| Constants | `PascalCase` (exported) / `camelCase` (unexported) | `MaxRetryCount` / `maxRetryCount` |
| Packages | `lowercase` (single word, no underscores, no hyphens) | `payments`, `auth`, `httputil` |
| Files | `snake_case` | `user_profile.go`, `payment_processor.go` |
| Interfaces | `PascalCase`, often `-er` suffix | `Reader`, `Validator`, `OrderProcessor` |
| Struct fields | `PascalCase` (exported) / `camelCase` (unexported) | `UserID`, `createdAt` |
| Receivers | 1-2 letter abbreviation of the type | `func (u *User) Validate()` |
| Test files | Match source + `_test` | `user_profile_test.go` |

**Go has one absolute rule: no underscores in identifiers (except file names and test functions).** `max_retries` is wrong in Go. Always. `maxRetries` or `MaxRetries` depending on export visibility.

## Acronym Handling

Acronyms are the most common source of casing inconsistency. Each language handles them differently, and those differences are non-negotiable.

### Python: Acronyms Follow Normal Casing

In Python, treat acronyms like regular words:

| Context | Correct | Wrong |
|---------|---------|-------|
| Variable | `http_client` | `HTTP_client`, `hTTPClient` |
| Function | `parse_json_response` | `parse_JSON_response` |
| Class | `HttpClient` | `HTTPClient` |
| Class | `JsonParser` | `JSONParser` |
| Class | `UrlValidator` | `URLValidator` |
| Constant | `DEFAULT_HTTP_TIMEOUT` | `DEFAULT_Http_TIMEOUT` |

**Exception:** Two-letter acronyms in class names stay uppercase: `IOError`, `OSError`. This follows stdlib convention. But for your own classes, prefer `IoHandler` for consistency.

### TypeScript: Acronyms Follow Normal Casing

Same as Python -- treat acronyms as regular words in PascalCase and camelCase:

| Context | Correct | Wrong |
|---------|---------|-------|
| Variable | `httpClient` | `HTTPClient`, `hTTPClient` |
| Function | `parseJsonResponse` | `parseJSONResponse` |
| Class/Interface | `HttpClient` | `HTTPClient` |
| Class/Interface | `JsonParser` | `JSONParser` |
| Type | `UrlConfig` | `URLConfig` |
| Constant | `DEFAULT_HTTP_TIMEOUT` | `DEFAULT_Http_TIMEOUT` |

**Why not `HTTPClient`?** Because when two acronyms collide, readability collapses: `XMLHTTPRequest` vs `XmlHttpRequest`. The latter is instantly parseable. The former requires the reader to mentally split `XMLHTTP`.

### Go: Acronyms Stay Uppercase

Go is the exception. Go's official convention keeps well-known acronyms fully uppercase:

| Context | Correct | Wrong |
|---------|---------|-------|
| Exported | `HTTPClient` | `HttpClient` |
| Exported | `JSONParser` | `JsonParser` |
| Exported | `URLValidator` | `UrlValidator` |
| Exported | `UserID` | `UserId` |
| Unexported | `httpClient` | `hTTPClient` |
| Unexported | `userID` | `userId` |

**Go recognizes these acronyms:** `API`, `ASCII`, `CPU`, `CSS`, `DNS`, `EOF`, `GUID`, `HTML`, `HTTP`, `HTTPS`, `ID`, `IP`, `JSON`, `LHS`, `QPS`, `RAM`, `RHS`, `RPC`, `SLA`, `SMTP`, `SQL`, `SSH`, `TCP`, `TLS`, `TTL`, `UDP`, `UI`, `UID`, `UUID`, `URI`, `URL`, `UTF8`, `VM`, `XML`, `XMPP`, `XSRF`, `XSS`.

When an unexported identifier starts with an acronym, the entire acronym is lowercase: `httpClient`, `jsonParser`, `urlValidator`. Never `hTTPClient`.

## File Naming Casing

File names follow language-specific conventions. Mixing file name casing within a project is a codebase smell visible from the directory listing.

| Language | Convention | Example |
|----------|-----------|---------|
| Python | `snake_case.py` | `user_profile.py`, `payment_processor.py` |
| TypeScript (general) | `kebab-case.ts` | `user-profile.ts`, `payment-processor.ts` |
| TypeScript (React) | `PascalCase.tsx` | `UserProfileCard.tsx`, `PaymentForm.tsx` |
| Go | `snake_case.go` | `user_profile.go`, `payment_processor.go` |

**Never mix.** If your project has `userProfile.ts`, `payment-processor.ts`, and `OrderService.ts`, the project has three conventions and zero discipline.

## The Cardinal Sin: Mixed Conventions in One File

This is the single most important rule in this entire skill. **Never mix casing conventions within a file.** A file that contains both `getUserName` and `get_user_email` is broken. It does not matter which convention is "better." Pick one. Apply it everywhere. Enforce it with a linter.

```typescript
// CARDINAL SIN: Three conventions in one file

const user_name = "Alice";          // snake_case variable in TypeScript
const userEmail = "alice@test.com"; // camelCase variable
const UserAge = 30;                 // PascalCase variable (not a class)

function get_profile() { ... }      // snake_case function in TypeScript
function getUserOrders() { ... }    // camelCase function
```

Every one of these is a defect. In TypeScript, variables and functions are `camelCase`. Period.

## Cross-Boundary Consistency

When data crosses a boundary -- API to frontend, database to application, service to service -- casing conventions collide. The rule: **map at the boundary, never mix in the interior.**

### API Returns snake_case, Frontend Uses camelCase

```typescript
// The API response (snake_case -- standard for JSON APIs)
// {
//   "user_id": "usr_abc123",
//   "first_name": "Alice",
//   "created_at": "2024-01-15T10:30:00Z"
// }

// BAD: Mixing snake_case from the API with camelCase in application code
function UserCard({ user }: { user: any }) {
  return (
    <div>
      <h2>{user.first_name}</h2>         {/* snake_case leaked into component */}
      <span>{user.created_at}</span>      {/* snake_case leaked into component */}
      <span>{user.phoneNumber}</span>     {/* camelCase from somewhere else */}
    </div>
  );
}

// GOOD: Map at the boundary, use camelCase everywhere internally
interface User {
  userId: string;
  firstName: string;
  createdAt: string;
}

function mapApiUserToUser(apiUser: Record<string, unknown>): User {
  return {
    userId: apiUser.user_id as string,
    firstName: apiUser.first_name as string,
    createdAt: apiUser.created_at as string,
  };
}

function UserCard({ user }: { user: User }) {
  return (
    <div>
      <h2>{user.firstName}</h2>
      <span>{user.createdAt}</span>
    </div>
  );
}
```

### Database Uses snake_case, Go Uses PascalCase/camelCase

```go
// GOOD: Struct tags map the boundary, Go code uses Go conventions
type UserProfile struct {
    UserID    string    `db:"user_id" json:"user_id"`
    FirstName string    `db:"first_name" json:"first_name"`
    CreatedAt time.Time `db:"created_at" json:"created_at"`
}

// BAD: Leaking database casing into Go code
type UserProfile struct {
    user_id    string    // Wrong: underscores in Go identifiers
    first_name string    // Wrong: underscores in Go identifiers
    created_at time.Time // Wrong: underscores in Go identifiers
}
```

## Good/Bad Examples

### Python

```python
# BAD: Mixed casing, inconsistent conventions
class userProfile:                          # class should be PascalCase
    firstName = ""                          # attribute should be snake_case
    LastName = ""                           # attribute should be snake_case
    def GetFullName(self):                  # method should be snake_case
        return f"{self.firstName} {self.LastName}"

maxRetries = 3                              # constant should be UPPER_SNAKE_CASE
default_timeout = 30                        # constant should be UPPER_SNAKE_CASE

# GOOD: Consistent Python conventions
class UserProfile:
    first_name = ""
    last_name = ""
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}"

MAX_RETRIES = 3
DEFAULT_TIMEOUT_SECONDS = 30
```

### TypeScript

```typescript
// BAD: Mixed casing nightmare
interface user_profile {                     // interface should be PascalCase
  First_Name: string;                        // field should be camelCase
  last_name: string;                         // field should be camelCase
  Email: string;                             // field should be camelCase
}

const MAX_RETRIES = 3;                       // OK: constant
let User_Name = "Alice";                     // variable should be camelCase
function Process_Order(order_id: string) {}  // function and param should be camelCase

// GOOD: Consistent TypeScript conventions
interface UserProfile {
  firstName: string;
  lastName: string;
  email: string;
}

const MAX_RETRIES = 3;
let userName = "Alice";
function processOrder(orderId: string) {}
```

### Go

```go
// BAD: Underscores and wrong casing
type user_profile struct {                   // No underscores in Go identifiers
    first_name string                        // No underscores
    user_Id    string                        // Inconsistent: should be userID
}

func get_user_name(user_id string) string {  // No underscores in function names
    return ""
}

const max_retries = 3                        // No underscores in constants

// GOOD: Go conventions followed exactly
type userProfile struct {
    firstName string
    userID    string                          // ID is a recognized acronym: fully uppercase
}

func getUserName(userID string) string {
    return ""
}

const maxRetries = 3
```

## Examples

Working implementations in `examples/`:
- **`examples/casing-conventions.md`** -- Comprehensive multi-language examples showing correct casing conventions in Python, TypeScript, and Go, including acronym handling, file naming, and cross-boundary mapping

## Review Checklist

When reviewing code for casing consistency:

- [ ] Every identifier follows the language's casing convention (see Universal Casing Table)
- [ ] No mixed casing conventions within any single file
- [ ] Acronyms follow language-specific rules (lowercase in Python/TypeScript PascalCase, uppercase in Go)
- [ ] File names follow language conventions: `snake_case.py`, `kebab-case.ts`, `PascalCase.tsx`, `snake_case.go`
- [ ] No `I` prefix on TypeScript interfaces (`UserProfile` not `IUserProfile`)
- [ ] Constants use `UPPER_SNAKE_CASE` in Python and TypeScript, `PascalCase`/`camelCase` in Go
- [ ] Cross-boundary casing is mapped at the boundary (API snake_case to frontend camelCase, database to application)
- [ ] No snake_case leaking into TypeScript/Go application code from API responses or database columns
- [ ] Enum members follow language conventions: `UPPER_SNAKE_CASE` in Python, `PascalCase` in TypeScript, `PascalCase` in Go
- [ ] React components use `PascalCase` file names; non-component TypeScript files use `kebab-case`
- [ ] Go identifiers contain zero underscores (except file names and test function names)
- [ ] Go struct tags handle the mapping between Go casing and JSON/database casing
