---
name: spring-validation
description: Bean Validation (Jakarta Validation) with Spring Boot. Custom validators, validation groups, cross-field validation, and internationalized error messages.
version: "1.0.0"
author: GazApps
tags: [validation, bean-validation, jakarta, constraints, spring-boot, forms]
dependencies: [java-fundamentals, spring-boot-core]
compatibility: [antigravity, claude-code, gemini-cli]
---

# Spring Validation

Bean Validation (Jakarta Validation) patterns for Spring Boot applications.

## Use this skill when

- Adding input validation to DTOs / request bodies
- Creating custom constraint annotations
- Implementing cross-field validation (e.g., password confirmation, date ranges)
- Setting up validation groups (different rules for create vs update)
- Customizing validation error messages or adding i18n
- Implementing method-level validation on service classes
- Building structured API error responses for validation failures
- User mentions "validation", "constraints", "@Valid", "@NotNull", "@NotBlank", "BindingResult"

## Do not use this skill when

- User needs authorization rules (use spring-security-oauth2)
- User wants business rule validation that depends on database state (use service-layer logic)
- User needs client-side / JavaScript validation only
- User asks about OpenAPI schema validation (use springdoc annotations)

## Dependencies

Add to `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

This transitively brings in:
- `jakarta.validation:jakarta.validation-api` (constraint annotations)
- `org.hibernate.validator:hibernate-validator` (reference implementation)

> **Note:** `spring-boot-starter-web` does NOT include validation automatically since Spring Boot 2.3+. You must add `spring-boot-starter-validation` explicitly.

## Built-in Constraints Quick Reference

| Annotation | Applies to | Description |
|---|---|---|
| `@NotNull` | Any type | Must not be `null` (empty string `""` passes) |
| `@NotBlank` | `CharSequence` | Must not be `null`, must contain at least one non-whitespace character |
| `@NotEmpty` | `CharSequence`, `Collection`, `Map`, `Array` | Must not be `null` or empty (blank string `"  "` passes) |
| `@Size(min, max)` | `CharSequence`, `Collection`, `Map`, `Array` | Length/size must be within bounds. `null` is valid. |
| `@Min(value)` | Numeric types | Must be >= value |
| `@Max(value)` | Numeric types | Must be <= value |
| `@Positive` | Numeric types | Must be > 0 |
| `@PositiveOrZero` | Numeric types | Must be >= 0 |
| `@Negative` | Numeric types | Must be < 0 |
| `@NegativeOrZero` | Numeric types | Must be <= 0 |
| `@DecimalMin(value)` | `BigDecimal`, `BigInteger`, `CharSequence`, numeric | Must be >= value (string comparison) |
| `@DecimalMax(value)` | `BigDecimal`, `BigInteger`, `CharSequence`, numeric | Must be <= value (string comparison) |
| `@Email` | `CharSequence` | Must be a valid email format. `null` is valid. |
| `@Pattern(regexp)` | `CharSequence` | Must match the regex. `null` is valid. |
| `@Past` | Date/time types | Must be a date in the past |
| `@PastOrPresent` | Date/time types | Must be a date in the past or present |
| `@Future` | Date/time types | Must be a date in the future |
| `@FutureOrPresent` | Date/time types | Must be a date in the future or present |
| `@Digits(integer, fraction)` | Numeric types | Must have at most N integer digits and F fraction digits |
| `@AssertTrue` | `boolean` | Must be `true` |
| `@AssertFalse` | `boolean` | Must be `false` |

> **Critical:** Most constraints pass on `null`. Combine with `@NotNull` if the field is required. For example, `@Email` alone allows `null`; use `@NotNull @Email` to make it mandatory.

See `references/builtin-constraints.md` for the complete reference with parameters, examples, and common gotchas.

## @Valid vs @Validated

| Feature | `@Valid` (Jakarta) | `@Validated` (Spring) |
|---|---|---|
| Package | `jakarta.validation` | `org.springframework.validation.annotation` |
| Validation groups | No | Yes |
| Method-level validation | No | Yes (on class) |
| Nested object cascade | Yes | Yes |
| Use on parameter | Yes | Yes |
| Use on class | No | Yes |

**Rule of thumb:**
- Use `@Valid` on `@RequestBody` parameters and nested fields when you do not need groups.
- Use `@Validated` on classes (controllers, services) to enable method-level validation and on parameters when you need validation groups.

## Validation in Controllers

### Basic validation with @Valid

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> create(@Valid @RequestBody CreateUserRequest request) {
        // If validation fails, MethodArgumentNotValidException is thrown automatically
        UserResponse response = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

### Using BindingResult for manual error handling

```java
@PostMapping
public ResponseEntity<?> create(
        @Valid @RequestBody CreateUserRequest request,
        BindingResult bindingResult) {

    if (bindingResult.hasErrors()) {
        Map<String, String> errors = bindingResult.getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                fe -> fe.getDefaultMessage() != null ? fe.getDefaultMessage() : "Invalid value",
                (a, b) -> a  // keep first error per field
            ));
        return ResponseEntity.badRequest().body(errors);
    }

    return ResponseEntity.status(HttpStatus.CREATED).body(userService.create(request));
}
```

> **Warning:** When you declare `BindingResult`, Spring will NOT throw `MethodArgumentNotValidException`. You must check errors manually. If you forget, invalid data silently passes through.

### Validation with groups

```java
@RestController
@RequestMapping("/api/users")
@Validated  // Required for group-based validation on controller methods
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> create(
            @Validated(OnCreate.class) @RequestBody CreateUserRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.create(request));
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> update(
            @PathVariable UUID id,
            @Validated(OnUpdate.class) @RequestBody UpdateUserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }
}
```

## Validation Groups

Define marker interfaces to apply different validation rules for create vs update:

```java
// Marker interfaces
public interface OnCreate {}
public interface OnUpdate {}
```

```java
public record CreateUserRequest(
    @NotBlank(groups = OnCreate.class)
    @Size(min = 2, max = 100, groups = {OnCreate.class, OnUpdate.class})
    String name,

    @NotBlank(groups = OnCreate.class)
    @Email(groups = {OnCreate.class, OnUpdate.class})
    String email,

    @NotBlank(groups = OnCreate.class)
    @Size(min = 8, max = 72, groups = OnCreate.class)
    String password
) {}
```

- On **create**: all fields required, all constraints enforced.
- On **update**: `@NotBlank` is skipped (field can be `null` meaning "don't change"), but format constraints still apply if a value is provided.

## Custom Constraint Annotations

### Step 1: Define the annotation

```java
@Documented
@Constraint(validatedBy = StrongPasswordValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface StrongPassword {
    String message() default "{validation.strongPassword}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    int minLength() default 8;
}
```

### Step 2: Implement the validator

```java
public class StrongPasswordValidator implements ConstraintValidator<StrongPassword, String> {

    private int minLength;

    @Override
    public void initialize(StrongPassword annotation) {
        this.minLength = annotation.minLength();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) return true;  // Let @NotNull handle null checks

        if (value.length() < minLength) return false;
        if (!value.matches(".*[A-Z].*")) return false;   // uppercase
        if (!value.matches(".*[a-z].*")) return false;   // lowercase
        if (!value.matches(".*\\d.*")) return false;     // digit
        if (!value.matches(".*[!@#$%^&*].*")) return false; // special char

        return true;
    }
}
```

### Cross-field validation (class-level constraint)

```java
@Documented
@Constraint(validatedBy = ValidDateRangeValidator.class)
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    String startField() default "startDate";
    String endField() default "endDate";
}
```

```java
public class ValidDateRangeValidator implements ConstraintValidator<ValidDateRange, Object> {

    private String startField;
    private String endField;

    @Override
    public void initialize(ValidDateRange annotation) {
        this.startField = annotation.startField();
        this.endField = annotation.endField();
    }

    @Override
    public boolean isValid(Object obj, ConstraintValidatorContext context) {
        try {
            BeanWrapper wrapper = new BeanWrapperImpl(obj);
            LocalDate start = (LocalDate) wrapper.getPropertyValue(startField);
            LocalDate end = (LocalDate) wrapper.getPropertyValue(endField);

            if (start == null || end == null) return true;

            boolean valid = end.isAfter(start);
            if (!valid) {
                context.disableDefaultConstraintViolation();
                context.buildConstraintViolationWithTemplate(context.getDefaultConstraintMessageTemplate())
                    .addPropertyNode(endField)
                    .addConstraintViolation();
            }
            return valid;
        } catch (Exception e) {
            return false;
        }
    }
}
```

## Nested Object Validation

Use `@Valid` on nested fields to cascade validation:

```java
public record CreateOrderRequest(
    @NotNull
    UUID customerId,

    @NotEmpty(message = "Order must have at least one item")
    @Valid  // Cascade validation into each OrderItemRequest
    List<OrderItemRequest> items,

    @Valid  // Cascade validation into the address object
    @NotNull
    AddressRequest shippingAddress
) {}

public record OrderItemRequest(
    @NotNull UUID productId,
    @Positive int quantity
) {}

public record AddressRequest(
    @NotBlank String street,
    @NotBlank String city,
    @NotBlank @Size(min = 2, max = 2) String state,
    @NotBlank @Pattern(regexp = "\\d{5}(-\\d{4})?") String zipCode
) {}
```

> **Critical:** Without `@Valid` on the nested field, constraints on `OrderItemRequest` and `AddressRequest` will NOT be checked.

## Collection Validation

Validate elements inside collections using `@Valid` and container element constraints:

```java
public record BulkCreateRequest(
    @NotEmpty
    @Size(max = 100, message = "Cannot create more than 100 items at once")
    List<@Valid CreateUserRequest> users
) {}
```

## Method-Level Validation

Apply validation to service method parameters and return values:

```java
@Service
@Validated  // Enables method-level validation for this bean
public class UserService {

    public UserResponse createUser(@Valid CreateUserRequest request) {
        // @Valid triggers validation on the request parameter
        // ConstraintViolationException thrown if invalid
        // ...
    }

    @NotNull
    public UserResponse findById(@NotNull UUID id) {
        // Both parameter and return value validated
        // ...
    }

    public void updateEmails(@NotEmpty List<@Email String> emails) {
        // Validates list is not empty and each element is a valid email
        // ...
    }
}
```

> **Note:** Method-level validation throws `ConstraintViolationException` (not `MethodArgumentNotValidException`). Your exception handler must catch both.

## Error Message Customization

### Inline messages

```java
@NotBlank(message = "Name is required")
@Size(min = 2, max = 100, message = "Name must be between {min} and {max} characters")
private String name;
```

### Using messages.properties

```java
@NotBlank(message = "{user.name.required}")
private String name;
```

```properties
# src/main/resources/messages.properties
user.name.required=Name is required
user.email.invalid=Please provide a valid email address
```

### Message interpolation variables

```properties
# {min}, {max}, {value} are automatically available
user.name.size=Name must be between {min} and {max} characters
order.quantity.min=Quantity must be at least {value}
```

See `references/error-message-patterns.md` for complete i18n setup and custom message resolvers.

## Programmatic Validation

Use `Validator` directly when validation must happen outside the annotation flow:

```java
@Service
@RequiredArgsConstructor
public class ImportService {

    private final Validator validator;

    public ImportResult importUsers(List<CreateUserRequest> requests) {
        List<String> errors = new ArrayList<>();

        for (int i = 0; i < requests.size(); i++) {
            Set<ConstraintViolation<CreateUserRequest>> violations =
                validator.validate(requests.get(i), OnCreate.class);

            if (!violations.isEmpty()) {
                for (ConstraintViolation<CreateUserRequest> v : violations) {
                    errors.add("Row %d: %s %s".formatted(
                        i + 1, v.getPropertyPath(), v.getMessage()));
                }
            }
        }

        if (!errors.isEmpty()) {
            return ImportResult.failure(errors);
        }

        // Proceed with import...
        return ImportResult.success(requests.size());
    }
}
```

## Exception Handling for Validation Errors

```java
@RestControllerAdvice
public class ValidationExceptionHandler {

    // Handles @Valid @RequestBody failures
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ProblemDetail> handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Validation Failed");
        problem.setDetail("One or more fields have validation errors");

        Map<String, List<String>> fieldErrors = ex.getBindingResult()
            .getFieldErrors().stream()
            .collect(Collectors.groupingBy(
                FieldError::getField,
                Collectors.mapping(fe ->
                    fe.getDefaultMessage() != null ? fe.getDefaultMessage() : "Invalid",
                    Collectors.toList())
            ));

        problem.setProperty("fieldErrors", fieldErrors);
        return ResponseEntity.badRequest().body(problem);
    }

    // Handles @Validated method-level failures
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ProblemDetail> handleConstraintViolation(ConstraintViolationException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Constraint Violation");

        Map<String, String> errors = ex.getConstraintViolations().stream()
            .collect(Collectors.toMap(
                v -> extractFieldName(v.getPropertyPath()),
                ConstraintViolation::getMessage,
                (a, b) -> a
            ));

        problem.setProperty("errors", errors);
        return ResponseEntity.badRequest().body(problem);
    }

    private String extractFieldName(Path propertyPath) {
        String path = propertyPath.toString();
        return path.contains(".") ? path.substring(path.lastIndexOf('.') + 1) : path;
    }
}
```

## Code Quality Checklist

- [ ] `spring-boot-starter-validation` is in `pom.xml`
- [ ] DTOs use `@NotNull` combined with format constraints (`@Email`, `@Size`, etc.) for required fields
- [ ] Nested objects annotated with `@Valid` to cascade validation
- [ ] Collections use `List<@Valid ItemRequest>` for element validation
- [ ] Validation groups defined for create vs update scenarios
- [ ] Custom constraints return `true` for `null` values (let `@NotNull` handle nulls)
- [ ] Class-level constraints used for cross-field validation
- [ ] `@RestControllerAdvice` handles both `MethodArgumentNotValidException` and `ConstraintViolationException`
- [ ] Error responses use `ProblemDetail` (RFC 7807) format
- [ ] Validation messages externalized to `messages.properties`
- [ ] i18n messages provided for supported locales
- [ ] Method-level validation uses `@Validated` on the class, not `@Valid`
- [ ] `BindingResult` is only used when manual error handling is intentional

## References

- See `references/builtin-constraints.md` for the complete constraint reference
- See `references/error-message-patterns.md` for message customization and i18n
- See `examples/` for complete code examples
