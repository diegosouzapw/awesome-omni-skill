---
name: standalone-developer-agent
description: Expert developer in 24+ programming languages that generates production-ready code leveraging unique language features and idiomatic patterns.
---

# Standalone Developer Agent

## Purpose

The **Standalone Developer Agent** is an **EXPERT developer in ALL 24 supported languages**, implementing features with two distinct personas:
- **Conservative Developer** - Prioritizes stability, proven patterns, comprehensive testing
- **Aggressive Developer** - Prioritizes innovation, modern patterns, cutting-edge techniques

### Language Expertise (24 Languages)

The agent is an expert in:
**Systems:** C, C++, Rust, Go, Fortran
**Application:** Python, Java, Groovy, Kotlin, C#, Scala, Ruby, Perl
**Functional:** Haskell, Erlang, Scala
**Web:** JavaScript, TypeScript
**Mobile:** Swift (iOS), Objective-C (iOS), Kotlin (Android)
**Data Science:** R, MATLAB, Python
**Query:** SQL, GraphQL
**Other:** Forth

### Unique Feature Utilization

**CRITICAL:** The agent **leverages unique language features** rather than just translating Python patterns:
- **Perl** - Context-aware operations, powerful regex (PCRE), CPAN ecosystem
- **Fortran** - Whole-array operations, coarrays for HPC, intrinsic functions
- **R** - Vectorization, data.table/dplyr, tidyverse, functional apply family
- **MATLAB** - Matrix operations, vectorization, parfor for parallelism
- **Haskell** - Type system, monads, lazy evaluation, pure functions
- **Erlang** - Actor model, supervision trees, hot code reloading
- **Rust** - Ownership system, zero-cost abstractions, fearless concurrency

The agent uses **IDIOMATIC patterns native to each language**, not generic translations.

## When to Use This Skill

Invoke the developer agent:

1. **Feature Implementation** - Convert ADRs into production code
2. **Bug Fixes** - Implement fixes for reported defects
3. **Refactoring** - Modernize or clean up existing code
4. **Code Review Response** - Address feedback from Code Review Agent
5. **Prototyping** - Quick proof-of-concept implementations

## Responsibilities

### 1. Code Generation

Generates high-quality, production-ready code following SOLID principles:

**Mandatory Standards:**
- ✅ **Custom Exception Wrappers** - Never use raw exceptions
- ✅ **SOLID Principles** - Strictly enforced
- ✅ **Anti-Pattern Avoidance** - No nested loops, no nested ifs, no if-elif chains
- ✅ **Functional Patterns** - Comprehensions, map/filter/reduce over loops
- ✅ **Explicit Comments** - Explain WHAT and WHY (not just how)
- ✅ **Method Size Limits** - <50 lines per method, <200 per class
- ✅ **Memoization** - Use wherever applicable for performance

**Example Output (Python):**
```python
from functools import lru_cache
from typing import List, Optional
from dataclasses import dataclass

# Custom Exception Wrapper
class UserValidationError(Exception):
    """Custom exception for user validation failures"""
    def __init__(self, field: str, reason: str, user_id: Optional[int] = None):
        self.field = field
        self.reason = reason
        self.user_id = user_id
        super().__init__(f"Validation failed for {field}: {reason}")

@dataclass
class UserConfig:
    """Configuration constants for user validation (no magic numbers)"""
    MIN_AGE: int = 18
    MAX_AGE: int = 120
    MIN_NAME_LENGTH: int = 2
    MAX_NAME_LENGTH: int = 100

class UserValidator:
    """
    Validates user data according to business rules.

    Follows SRP: Only responsible for validation logic.
    Uses functional patterns and explicit error handling.
    """

    def __init__(self, config: UserConfig = UserConfig()):
        """
        Initialize validator with configuration.

        Args:
            config: Validation configuration (supports DI for testing)
        """
        self.config = config

    def validate_age(self, age: int) -> None:
        """
        Validate user age is within acceptable range.

        Why: Business rule requires users to be legal adults but realistic age.

        Args:
            age: User's age in years

        Raises:
            UserValidationError: If age outside valid range
        """
        # Use rule-based validation (no nested ifs)
        validation_rules = [
            (age < self.config.MIN_AGE, f"Must be at least {self.config.MIN_AGE}"),
            (age > self.config.MAX_AGE, f"Must be at most {self.config.MAX_AGE}")
        ]

        # Find first failing rule using next() with generator
        error_message = next(
            (msg for condition, msg in validation_rules if condition),
            None
        )

        if error_message:
            raise UserValidationError(field="age", reason=error_message)

    def validate_users(self, users: List[dict]) -> List[dict]:
        """
        Validate multiple users using functional patterns.

        Why: Bulk validation is common operation, must be efficient.
        Uses filter + map pattern instead of loops for clarity.

        Args:
            users: List of user dictionaries

        Returns:
            List of validated user dictionaries

        Raises:
            UserValidationError: If any user fails validation
        """
        # Use comprehension instead of loop
        validated_users = [
            self._validate_single_user(user)
            for user in users
        ]

        return validated_users

    @lru_cache(maxsize=128)
    def _validate_single_user(self, user_tuple: tuple) -> dict:
        """
        Validate single user with memoization for performance.

        Why: Same users validated repeatedly in bulk operations.
        Uses LRU cache to avoid redundant validation.

        Note: Takes tuple instead of dict for hashability (cache requirement)
        """
        user = dict(user_tuple)
        self.validate_age(user['age'])
        # ... other validations
        return user
```

### 2. Anti-Pattern Avoidance

Strictly avoids common code anti-patterns:

| Anti-Pattern | Solution | Example |
|-------------|----------|---------|
| **Nested Loops** | Use comprehensions, map/filter | `[x*y for x in list1 for y in list2]` |
| **Nested Ifs** | Use guard clauses, rule-based validation | Early returns, rule tables |
| **If-Elif Chains** | Use dict dispatch, strategy pattern | `actions[action_type]()` |
| **God Classes** | Split by SRP | UserService, AuthService, EmailService |
| **Magic Numbers** | Use config classes | `EstimationConfig.HIGH_RISK_THRESHOLD` |
| **Mutable Global State** | Use DI, immutable config | Pass config to constructors |

### 3. Design Patterns

Applies appropriate design patterns:

**Creational:**
- Factory Pattern (object creation)
- Builder Pattern (complex object construction)
- Singleton Pattern (global state, used sparingly)

**Structural:**
- Adapter Pattern (interface compatibility)
- Decorator Pattern (extending behavior)
- Repository Pattern (data access abstraction)

**Behavioral:**
- Strategy Pattern (algorithm selection)
- Observer Pattern (event handling)
- Command Pattern (action encapsulation)
- State Machine (workflow management)

**Concurrency:**
- ThreadPoolExecutor (parallel processing)
- Actor Model (Erlang, Scala/Akka)
- async/await (Python, JS, C#)

### 4. Test Generation

Generates comprehensive test suites (85%+ coverage target):

**Test Types:**
- Unit tests (pytest, JUnit, RSpec)
- Property-based tests (Hypothesis, QuickCheck)
- Integration tests
- Contract tests (for APIs)

**Example Test:**
```python
import pytest
from hypothesis import given, strategies as st

class TestUserValidator:
    """
    Comprehensive test suite for UserValidator.

    Uses property-based testing for edge cases.
    """

    def test_validate_age_valid(self):
        """Test that valid ages pass validation"""
        validator = UserValidator()
        # Should not raise
        validator.validate_age(25)

    def test_validate_age_too_young(self):
        """Test that underage users are rejected"""
        validator = UserValidator()
        with pytest.raises(UserValidationError) as exc_info:
            validator.validate_age(16)

        assert exc_info.value.field == "age"
        assert "at least 18" in exc_info.value.reason

    @given(age=st.integers(min_value=18, max_value=120))
    def test_validate_age_property_based(self, age):
        """Property-based test: all ages 18-120 should be valid"""
        validator = UserValidator()
        validator.validate_age(age)  # Should not raise

    @pytest.fixture
    def mock_config(self):
        """Fixture providing test configuration (DI example)"""
        return UserConfig(MIN_AGE=21, MAX_AGE=100)

    def test_validate_age_custom_config(self, mock_config):
        """Test that custom config is respected (DI testing)"""
        validator = UserValidator(config=mock_config)
        with pytest.raises(UserValidationError):
            validator.validate_age(20)  # Should fail with MIN_AGE=21
```

## Developer Personas

### Conservative Developer

**Characteristics:**
- 15+ years of experience
- Stability and reliability over clever tricks
- Proven patterns over experimental approaches
- Comprehensive testing and error handling
- Production-ready code (no TODOs or placeholders)
- Defensive programming

**When to Use:**
- Production-critical features
- Security-sensitive code
- Financial/healthcare applications
- Legacy system integration
- High-reliability requirements

**Example Approach:**
```python
# Conservative: Explicit error handling, defensive checks
def process_payment(amount: Decimal, user_id: int) -> PaymentResult:
    """
    Process payment with comprehensive validation and error handling.

    Why: Financial transaction requires maximum reliability.
    """
    # Validate all inputs explicitly
    if amount <= 0:
        raise PaymentValidationError("Amount must be positive")

    if amount > Decimal('10000.00'):
        raise PaymentValidationError("Amount exceeds daily limit")

    # Check user exists
    user = self.user_repo.get_by_id(user_id)
    if user is None:
        raise UserNotFoundError(f"User {user_id} not found")

    # Transaction with explicit rollback
    try:
        with self.db.transaction():
            payment = self.create_payment(amount, user_id)
            self.update_balance(user_id, -amount)
            self.log_transaction(payment.id)
            return PaymentResult(success=True, payment_id=payment.id)
    except Exception as e:
        self.logger.error(f"Payment failed: {e}", exc_info=True)
        raise PaymentProcessingError(f"Failed to process payment") from e
```

### Aggressive/Innovative Developer

**Characteristics:**
- Focus on modern patterns and performance
- Latest language features and frameworks
- Innovation and extensibility
- Performance and scalability as first-class concerns
- Cutting-edge but production-ready

**When to Use:**
- Greenfield projects
- Performance-critical features
- Scalability requirements
- Modern tech stack
- Innovation-focused teams

**Example Approach:**
```python
# Aggressive: Modern patterns, async, performance optimized
async def process_payments_batch(
    payments: List[PaymentRequest]
) -> List[PaymentResult]:
    """
    Process multiple payments concurrently with circuit breaker.

    Why: High-throughput payment processing requires async + resilience.
    Uses modern Python 3.11+ features for performance.
    """
    # Use async comprehension for concurrent processing
    results = await asyncio.gather(
        *[self._process_single_payment(p) for p in payments],
        return_exceptions=True
    )

    # Use pattern matching (Python 3.10+) for result handling
    return [
        match result:
            case PaymentResult() as r: r
            case Exception() as e: PaymentResult(success=False, error=str(e))
            case _: PaymentResult(success=False, error="Unknown error")
        for result in results
    ]

async def _process_single_payment(
    self, payment: PaymentRequest
) -> PaymentResult:
    """Process single payment with circuit breaker and retry"""
    # Use circuit breaker pattern for resilience
    async with self.circuit_breaker:
        return await self._execute_payment(payment)
```

## Multi-Language Support

The developer agent follows comprehensive standards across all languages:

### Language-Specific Examples

**Rust (Conservative):**
```rust
// Custom error type (never use .unwrap())
#[derive(Debug)]
enum ValidationError {
    AgeOutOfRange { age: u8, min: u8, max: u8 },
    InvalidName { reason: String },
}

impl std::fmt::Display for ValidationError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            Self::AgeOutOfRange { age, min, max } =>
                write!(f, "Age {} not in range {}-{}", age, min, max),
            Self::InvalidName { reason } =>
                write!(f, "Invalid name: {}", reason),
        }
    }
}

// Result<T, E> pattern (explicit error handling)
fn validate_age(age: u8) -> Result<(), ValidationError> {
    const MIN_AGE: u8 = 18;
    const MAX_AGE: u8 = 120;

    match age {
        a if a < MIN_AGE => Err(ValidationError::AgeOutOfRange {
            age, min: MIN_AGE, max: MAX_AGE
        }),
        a if a > MAX_AGE => Err(ValidationError::AgeOutOfRange {
            age, min: MIN_AGE, max: MAX_AGE
        }),
        _ => Ok(()),
    }
}

// Functional pattern: filter + map instead of loops
fn validate_users(users: Vec<User>) -> Result<Vec<User>, ValidationError> {
    users.into_iter()
        .map(|user| validate_user(user))
        .collect()  // Collects Result<Vec<T>, E>, short-circuits on error
}
```

**C# (Aggressive):**
```csharp
// Use records for immutable data (C# 9.0+)
public record UserValidationConfig(
    int MinAge = 18,
    int MaxAge = 120,
    int MinNameLength = 2
);

// Nullable reference types (C# 8.0+)
public class UserValidator
{
    private readonly UserValidationConfig _config;

    public UserValidator(UserValidationConfig? config = null)
    {
        _config = config ?? new UserValidationConfig();
    }

    // Pattern matching with switch expressions (C# 8.0+)
    public ValidationResult ValidateAge(int age) => age switch
    {
        < 0 => ValidationResult.Error("Age cannot be negative"),
        var a when a < _config.MinAge =>
            ValidationResult.Error($"Must be at least {_config.MinAge}"),
        var a when a > _config.MaxAge =>
            ValidationResult.Error($"Must be at most {_config.MaxAge}"),
        _ => ValidationResult.Success()
    };

    // LINQ for collection operations (functional pattern)
    public async Task<List<User>> ValidateUsersAsync(List<User> users)
    {
        // Parallel processing with LINQ
        var validatedUsers = await Task.WhenAll(
            users.Select(async user => await ValidateSingleUserAsync(user))
        );

        return validatedUsers.ToList();
    }
}
```

**Haskell (Pure Functional):**
```haskell
-- Algebraic data type for errors
data ValidationError
    = AgeOutOfRange Int Int Int  -- age, min, max
    | InvalidName String
    deriving (Show, Eq)

-- Type class for validation
class Validatable a where
    validate :: a -> Either ValidationError a

-- User type
data User = User
    { userId :: Int
    , userName :: String
    , userAge :: Int
    } deriving (Show, Eq)

-- Validation instance using Either monad
instance Validatable User where
    validate user =
        validateAge (userAge user) >>
        validateName (userName user) >>
        Right user

-- Pure function with pattern matching
validateAge :: Int -> Either ValidationError ()
validateAge age
    | age < 18  = Left $ AgeOutOfRange age 18 120
    | age > 120 = Left $ AgeOutOfRange age 18 120
    | otherwise = Right ()

-- Functor/Applicative for bulk validation
validateUsers :: [User] -> Either ValidationError [User]
validateUsers = traverse validate  -- Short-circuits on first error
```

## Integration with Pipeline

### Placement in Pipeline

```
Requirements → Sprint Planning → [Development Stage] → Code Review
                                        ↓
                           ┌────────────┴────────────┐
                           │                         │
                   Developer A              Developer B
                 (Conservative)            (Aggressive)
                           │                         │
                           └────────────┬────────────┘
                                        ↓
                                  Code Review
                                        ↓
                                  Arbitration
```

### Communication

**Receives:**
- Task title and description
- Architecture Decision Record (ADR)
- Code review feedback (for revisions)
- Sprint context

**Sends:**
- Implementation files (.py, .rs, .cs, etc.)
- Test files
- Documentation
- Implementation summary (JSON)

## Usage Examples

### Standalone Usage

```bash
python3 standalone_developer_agent.py \
  --developer-name "developer-a" \
  --persona conservative \
  --task-title "User Authentication" \
  --task-description "Implement JWT-based auth with bcrypt" \
  --adr-file /tmp/adr-auth.md \
  --output-dir /tmp/developer-a/
```

### Programmatic Usage

```python
from standalone_developer_agent import StandaloneDeveloperAgent

agent = StandaloneDeveloperAgent(
    developer_name="developer-a",
    persona="conservative",
    llm_provider="openai"
)

result = agent.implement_task(
    task_title="User Authentication",
    task_description="Implement JWT auth with bcrypt password hashing",
    adr_content=adr_text,
    output_dir="/tmp/developer-a/"
)

print(f"Files created: {result['files']}")
print(f"Tests coverage: {result['test_coverage']}%")
```

## Configuration

### Environment Variables

```bash
# Developer persona (default: conservative)
ARTEMIS_DEVELOPER_PERSONA=conservative|aggressive

# LLM Provider
ARTEMIS_LLM_PROVIDER=openai
ARTEMIS_LLM_MODEL=gpt-4o

# Coding standards enforcement
ARTEMIS_ENFORCE_SOLID=true
ARTEMIS_MAX_METHOD_LINES=50
ARTEMIS_MAX_CLASS_LINES=200
ARTEMIS_REQUIRE_TESTS=true
ARTEMIS_MIN_TEST_COVERAGE=85
```

## Cost Considerations

Typical implementation costs:

| Task Size | Tokens | Cost (GPT-4o) | Duration |
|-----------|--------|---------------|----------|
| Small (1-2 files) | 5K-10K | $0.10-0.20 | 30-60s |
| Medium (3-5 files) | 15K-30K | $0.30-0.60 | 2-5 min |
| Large (6-10 files) | 40K-80K | $0.80-1.60 | 5-10 min |

## Best Practices

1. **Choose Persona Wisely** - Conservative for prod-critical, Aggressive for innovation
2. **Provide Clear ADRs** - Better ADRs = better implementations
3. **Iterate on Feedback** - Use code review feedback to improve
4. **Monitor Quality** - Track test coverage and code quality metrics
5. **Language-Specific** - Specify target language in task description
6. **Version Control** - Always use git for developer outputs

## Limitations

- **Static Analysis** - Cannot test runtime behavior
- **Context Limits** - Large codebases may exceed context window
- **No Debugging** - Cannot debug failing tests interactively
- **No Deployment** - Generates code but doesn't deploy
- **Language Expertise** - Quality varies by language (best for Python, JS, Rust, Java)

## References

- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [Clean Code - Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

---

**Version:** 1.0.0
**Last Updated:** October 24, 2025
