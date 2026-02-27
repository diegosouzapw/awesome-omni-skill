---
name: code-documentation
description: Code documentation standards, patterns, and best practices
license: MIT
compatibility: opencode
---

# Code Documentation Skill

## Overview

This skill provides guidelines for writing effective code documentation that improves maintainability, onboarding, and collaboration.

## Documentation Levels

```
1. Code Comments (inline)
2. Function/Method Documentation
3. Module/File Documentation
4. Architecture Documentation
5. API Documentation
6. README and Guides
```

## Code Comments

### 1. When to Comment

```go
// ✅ DO: Explain WHY, not WHAT
// Cache for 5 minutes to reduce database load during high traffic
var userCache = cache.New(5*time.Minute, 10*time.Minute)

// ❌ DON'T: State the obvious
// This function adds two numbers
func add(a, b int) int {
    return a + b
}

// ✅ DO: Explain complex logic
// Using two-phase commit to ensure consistency between
// payment service and inventory service. If payment succeeds
// but inventory fails, we rollback both.
func processOrder(order Order) error {
    // ...
}

// ✅ DO: Document workarounds and hacks
// TODO: Remove this workaround once API v2 is deployed
// See: https://github.com/org/repo/issues/123
func legacyWorkaround() {
    // ...
}
```

### 2. Comment Formats

```go
// Single line comment for brief explanations

/*
Multi-line comment for longer explanations
or temporarily disabled code
*/

// Section comments for organizing code
// ============================================
// SECTION: User Management
// ============================================

// MARK: - Validation (Swift style)

// region Authentication (C# style)
// endregion
```

## Function/Method Documentation

### 1. Go (GoDoc)

```go
// Package user provides user management functionality.
//
// This package handles user creation, authentication,
// and profile management.
package user

// CreateUser creates a new user in the system.
//
// Parameters:
//   - ctx: Context for cancellation and timeouts
//   - req: User creation request with validation
//
// Returns:
//   - *User: Created user with generated ID
//   - error: ValidationError if input is invalid,
//            ConflictError if email already exists
//
// Example:
//
//	user, err := CreateUser(ctx, CreateUserRequest{
//	    Email: "user@example.com",
//	    Name:  "John Doe",
//	})
//
// See: https://docs.example.com/api/users
func CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    // Implementation
}

// IsValidEmail checks if the given string is a valid email address.
// It follows RFC 5322 standards.
func IsValidEmail(email string) bool {
    // Implementation
}
```

### 2. TypeScript/JSDoc

```typescript
/**
 * Creates a new user in the system.
 * 
 * @param {CreateUserRequest} params - User creation parameters
 * @param {string} params.email - User's email address
 * @param {string} params.name - User's full name
 * @param {number} [params.age] - User's age (optional)
 * @returns {Promise<User>} The created user
 * @throws {ValidationError} When input is invalid
 * @throws {ConflictError} When email already exists
 * 
 * @example
 * ```typescript
 * const user = await createUser({
 *   email: 'user@example.com',
 *   name: 'John Doe',
 *   age: 25
 * });
 * ```
 * 
 * @see {@link https://docs.example.com/api/users|API Documentation}
 */
async function createUser(params: CreateUserRequest): Promise<User> {
    // Implementation
}

/**
 * User role in the system.
 * @readonly
 * @enum {string}
 */
const UserRole = {
    /** Regular user with standard permissions */
    USER: 'user',
    /** Administrator with full access */
    ADMIN: 'admin',
    /** Moderator with content management rights */
    MODERATOR: 'moderator'
} as const;
```

### 3. Rust (RustDoc)

```rust
//! User management module
//!
//! This module provides functionality for creating,
//! updating, and deleting users.

/// Creates a new user in the system.
///
/// # Arguments
///
/// * `email` - User's email address
/// * `name` - User's full name
///
/// # Returns
///
/// Returns `Ok(User)` on success, or `Err(UserError)` if:
/// - Email is invalid
/// - Email already exists
/// - Name is empty
///
/// # Examples
///
/// ```
/// use myapp::user::create_user;
///
/// let user = create_user("user@example.com", "John Doe")?;
/// assert_eq!(user.name, "John Doe");
/// ```
///
/// # Errors
///
/// This function will return an error if the email is invalid
/// or already exists in the database.
pub fn create_user(email: &str, name: &str) -> Result<User, UserError> {
    // Implementation
}

/// Checks if the given email is valid.
///
/// Returns `true` if the email follows RFC 5322 standards.
pub fn is_valid_email(email: &str) -> bool {
    // Implementation
}
```

## Documentation Standards

### 1. README Template

```markdown
# Project Name

Brief description of what this project does.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

\`\`\`bash
# Clone the repository
git clone https://github.com/org/repo.git
cd repo

# Install dependencies
npm install

# Set up environment
cp .env.example .env
# Edit .env with your configuration
\`\`\`

## Usage

\`\`\`bash
# Start development server
npm run dev

# Build for production
npm run build

# Run tests
npm test
\`\`\`

## API Documentation

See [API Docs](./docs/API.md) for detailed API documentation.

## Architecture

See [Architecture Guide](./docs/ARCHITECTURE.md) for system design details.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for contribution guidelines.

## License

[MIT](./LICENSE)
```

### 2. Architecture Decision Records (ADRs)

```markdown
# ADR 001: Use PostgreSQL as Primary Database

## Status

Accepted

## Context

We need to choose a database for storing user data and application state.
Requirements:
- ACID compliance
- JSON support for flexible schemas
- Full-text search
- Geospatial queries

## Decision

We will use PostgreSQL as our primary database.

## Consequences

### Positive
- Strong data integrity
- Rich feature set (JSON, FTS, GIS)
- Excellent documentation
- Wide tool support

### Negative
- More complex setup than SQLite
- Requires connection pooling
- Horizontal scaling is complex

## Alternatives Considered

- MySQL: Good, but lacks some advanced features
- MongoDB: Flexible schema, but weaker consistency
- SQLite: Great for development, not for production scale

## References

- [PostgreSQL Documentation](https://postgresql.org/docs)
- [Comparison with MySQL](https://example.com/comparison)
```

### 3. Inline Documentation Standards

```go
// Constants should document their purpose and values
const (
    // DefaultTimeout is the default HTTP request timeout.
    // It balances responsiveness with reliability.
    DefaultTimeout = 30 * time.Second
    
    // MaxRetries is the maximum number of retry attempts.
    // This prevents infinite loops while allowing recovery
    // from transient failures.
    MaxRetries = 3
)

// Variables should document their use
var (
    // ErrNotFound is returned when a requested resource
    // does not exist in the database.
    ErrNotFound = errors.New("resource not found")
    
    // dbPool is the global database connection pool.
    // It is initialized once in init() and should not
    // be reassigned.
    dbPool *sql.DB
)

// Types should document their purpose and usage
type UserRepository interface {
    // Create persists a new user to the database.
    // It returns ErrDuplicateEmail if the email already exists.
    Create(ctx context.Context, user *User) error
    
    // GetByID retrieves a user by their unique identifier.
    // Returns ErrNotFound if no user exists with the given ID.
    GetByID(ctx context.Context, id string) (*User, error)
    
    // Update modifies an existing user's data.
    // All fields except ID can be updated.
    Update(ctx context.Context, user *User) error
    
    // Delete permanently removes a user from the database.
    // This operation cannot be undone.
    Delete(ctx context.Context, id string) error
}
```

## Code Examples in Documentation

### 1. Runnable Examples (Go)

```go
package user_test

import (
    "context"
    "fmt"
    "log"
    
    "github.com/org/repo/user"
)

// ExampleCreateUser demonstrates creating a new user.
func ExampleCreateUser() {
    ctx := context.Background()
    
    newUser, err := user.CreateUser(ctx, user.CreateUserRequest{
        Email: "user@example.com",
        Name:  "John Doe",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Created user: %s\n", newUser.Name)
    // Output: Created user: John Doe
}
```

### 2. Doctests (Python)

```python
def calculate_total(items):
    """
    Calculate the total price of a list of items.
    
    Args:
        items: List of items with price and quantity
        
    Returns:
        Total price as a float
        
    Example:
        >>> items = [
        ...     {'price': 10.0, 'quantity': 2},
        ...     {'price': 5.0, 'quantity': 3}
        ... ]
        >>> calculate_total(items)
        35.0
    """
    return sum(item['price'] * item['quantity'] for item in items)
```

## Documentation Tools

### 1. Static Documentation Generators

```bash
# Go
# Generate documentation
godoc -http=:6060

# TypeScript
# Generate TypeDoc
typedoc --out docs src

# Rust
# Generate RustDoc
cargo doc --open

# Python
# Generate Sphinx docs
sphinx-build -b html docs docs/_build
```

### 2. Documentation Testing

```bash
# Go: Test examples
go test -v -run Example

# Rust: Test documentation
cargo test --doc

# Python: Test docstrings
python -m doctest myfile.py
```

## Best Practices

### DO:
- ✅ Document public APIs thoroughly
- ✅ Include code examples
- ✅ Document parameters and return values
- ✅ Document error conditions
- ✅ Keep documentation up-to-date
- ✅ Use consistent formatting
- ✅ Document why, not just what
- ✅ Cross-reference related documentation

### DON'T:
- ❌ Document private/internal functions (unless complex)
- ❌ Repeat code in comments
- ❌ Use outdated examples
- ❌ Write novels in comments
- ❌ Skip error documentation
- ❌ Use inconsistent terminology

## When to Use

Use this skill when:
- Writing function/method documentation
- Creating README files
- Documenting architecture decisions
- Writing code comments
- Generating API documentation
- Onboarding new team members
