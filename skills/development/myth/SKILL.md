---
name: myth
description: Complete guide for using the Myth .NET ecosystem - enterprise-grade libraries for building scalable applications with SOLID principles, clean architecture, CQRS, validation, pipelines, and DDD patterns
args:
  library:
    description: "Specific Myth library to use (Flow, Flow.Actions, Guard, Morph, Repository, Specification, etc.)"
    required: false
  operation:
    description: "Operation to perform (setup, configure, validate, transform, query, command, etc.)"
    required: false
  context:
    description: "Application context (ASP.NET Core, Console App, Test, etc.)"
    required: false
---

# Myth Ecosystem Skill Documentation

This skill provides complete guidance for using the **Myth** ecosystem - a comprehensive collection of .NET libraries designed for enterprise applications following SOLID principles, Clean Code, and Domain-Driven Design.

## Table of Contents

1. [Overview](#overview)
2. [Architecture & Integration](#architecture--integration)
3. [Library-Specific Guides](#library-specific-guides)
4. [Common Workflows](#common-workflows)
5. [Best Practices](#best-practices)
6. [Examples by Scenario](#examples-by-scenario)

---

## Overview

### What is Myth?

Myth is a modular collection of .NET libraries that work together to provide:

- **Myth.Commons**: Base utilities, ValueObjects, global ServiceProvider management, JSON extensions
- **Myth.DependencyInjection**: Auto-discovery and convention-based service registration
- **Myth.Flow**: Pipeline pattern with Result, retry policies, telemetry
- **Myth.Flow.Actions**: CQRS, event-driven architecture, message brokers (Kafka, RabbitMQ)
- **Myth.Guard**: Fluent validation with 100+ rules, context-aware, async service integration
- **Myth.Morph**: Object transformation and mapping with schema-based bindings
- **Myth.Specification**: Query specification pattern for encapsulating business rules
- **Myth.Repository**: Generic repository interfaces with read/write separation
- **Myth.Repository.EntityFramework**: EF Core implementation with Unit of Work
- **Myth.Rest**: Fluent REST client with circuit breaker and retry
- **Myth.Testing**: Testing utilities, mocks, base test classes
- **Myth.Tool**: CLI tool for code generation with CQRS, DDD, and Clean Architecture patterns

### Key Architectural Concepts

#### MythServiceProvider (Global Service Provider)

All Myth libraries use a centralized, thread-safe service provider for dependency resolution:

```csharp
// ASP.NET Core - use BuildApp() instead of Build()
var app = builder.BuildApp(); // Initializes MythServiceProvider.Current

// Console Apps - use BuildWithGlobalProvider()
var serviceProvider = services.BuildWithGlobalProvider();

// Accessing global provider
var provider = MythServiceProvider.Current;
var service = MythServiceProvider.GetRequired();
```

#### IScopedService<T>

Allows transient services (like handlers) to use scoped services safely:

```csharp
public class CreateOrderHandler : ICommandHandler<CreateOrderCommand> {
    private readonly IScopedService<IOrderRepository> _repository;

    public async Task<CommandResult> HandleAsync(CreateOrderCommand command, CancellationToken ct) {
        return await _repository.ExecuteAsync(async repo => {
            var order = await repo.CreateAsync(command, ct);
            return CommandResult.Success();
        });
    }
}
```

---

## Architecture & Integration

### Setup for ASP.NET Core Applications

```csharp
using Myth.Extensions;

var builder = WebApplication.CreateBuilder(args);

// 1. Add core libraries
builder.Services.AddFlow(config => config
    .UseTelemetry()
    .UseRetry(maxAttempts: 3, backoffMs: 1000)
    .UseActions(actions => actions
        .UseInMemory() // or .UseKafka() / .UseRabbitMQ()
        .UseCaching()
        .ScanAssemblies(typeof(Program).Assembly)
        .EnableAutoSubscription()));

builder.Services.AddGuard();
builder.Services.AddMorph();

// 2. Register repositories and services
builder.Services.AddDbContext<AppDbContext>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IUnitOfWorkRepository, UnitOfWorkRepository>();

// 3. Build with global service provider (CRITICAL!)
var app = builder.BuildApp(); // NOT builder.Build()

// 4. Add middleware
app.UseGuard(); // Validation exception handling
app.MapControllers();

app.Run();
```

### Setup for Console Applications

```csharp
var services = new ServiceCollection();

services.AddFlow(config => config.UseTelemetry().UseRetry(3, 100));
services.AddGuard();
services.AddMorph();

// Register your services
services.AddScoped<IDataService, DataService>();

// Build with global provider
var serviceProvider = services.BuildWithGlobalProvider();

// Use services
var dataService = serviceProvider.GetRequiredService<IDataService>();
await dataService.ProcessDataAsync();
```

---

## Library-Specific Guides

### 1. Myth.Commons

#### Global Service Provider Management

```csharp
// Initialize (done automatically by BuildApp/BuildWithGlobalProvider)
MythServiceProvider.Initialize(serviceProvider);

// Check if initialized
if (MythServiceProvider.IsInitialized) {
    var current = MythServiceProvider.Current;
}

// Get required (throws if not initialized)
var provider = MythServiceProvider.GetRequired();

// Get with fallback
var provider = MythServiceProvider.GetOrFallback(fallbackProvider);

// Reset (for testing)
MythServiceProvider.Reset();
```

#### JSON Extensions

```csharp
// Global configuration
JsonExtensions.Configure(settings => {
    settings.CaseStrategy = CaseStrategy.SnakeCase;
    settings.IgnoreNullValues = true;
    settings.MinifyResult = true;
});

// Serialize to JSON
var json = user.ToJson();
var minifiedJson = user.ToJson(s => s.MinifyResult = true);

// Deserialize from JSON
var user = json.FromJson<User>();
var safeUser = json.SafeFromJson<User>(); // Returns null if fails

// Validate JSON
if (content.IsValidJson()) {
    var data = content.FromJson<Data>();
}

// Deserialize or throw with HTTP context
var user = json.FromJsonOrThrow<User>(HttpStatusCode.BadRequest);
```

#### ValueObjects

```csharp
// Define ValueObject
public class Address : ValueObject {
    public string Street { get; }
    public string City { get; }
    public string Country { get; }

    public Address(string street, string city, string country) {
        Street = street;
        City = city;
        Country = country;
    }

    protected override IEnumerable<object> GetAtomicValues() {
        yield return Street;
        yield return City;
        yield return Country;
    }
}

// Usage
var addr1 = new Address("123 Main St", "New York", "USA");
var addr2 = new Address("123 Main St", "New York", "USA");
Console.WriteLine(addr1.Equals(addr2)); // True (value equality)
```

#### Constants (Type-Safe Enums)

```csharp
// Define constant
public class OrderStatus : Constant<OrderStatus, string> {
    public static readonly OrderStatus Pending = CreateWithCallerName("P");
    public static readonly OrderStatus Active = CreateWithCallerName("A");
    public static readonly OrderStatus Completed = CreateWithCallerName("C");
    public static readonly OrderStatus Cancelled = CreateWithCallerName("X");

    private OrderStatus(string name, string value) : base(name, value) { }
}

// Usage
var status = OrderStatus.FromValue("A"); // Returns Active
var all = OrderStatus.GetAll(); // All instances
var allValues = OrderStatus.Values.All; // ["P", "A", "C", "X"]
string options = OrderStatus.GetOptions(); // "P: Pending | A: Active | ..."

// Try get
if (OrderStatus.TryFromValue("A", out var result)) {
    Console.WriteLine(result.Name); // "Active"
}
```

### 2. Myth.Flow - Pipeline Pattern

#### Basic Pipeline

```csharp
public class OrderService {
    public async Task<Result<OrderDto>> ProcessOrder(CreateOrderRequest request) {
        return await Pipeline.Start(request)
            .WithTelemetry("ProcessOrder")
            .WithRetry(maxAttempts: 3, backoffMs: 100)
            .StepResultAsync<ValidationService>((svc, ctx) => svc.ValidateAsync(ctx))
            .StepResultAsync<OrderCreationService>((svc, ctx) => svc.CreateAsync(ctx))
            .TapAsync<EventService>((svc, ctx) => svc.PublishOrderCreatedAsync(ctx))
            .Transform<OrderDto>(ctx => new OrderDto {
                OrderId = ctx.OrderId,
                Total = ctx.Total
            })
            .ExecuteAsync();
    }
}
```

#### Step Types

```csharp
// Synchronous step
.Step(ctx => {
    ctx.ProcessedData = ProcessData(ctx.RawData);
    return ctx;
})

// Async step
.StepAsync(async ctx => {
    ctx.Data = await LoadDataAsync(ctx.Id);
    return ctx;
})

// Step with Result pattern
.StepResultAsync(async ctx => {
    if (ctx.IsValid) {
        return Result<Context>.Success(ctx);
    }
    return Result<Context>.Failure("Invalid context");
})

// Side effects (tap)
.Tap(ctx => Console.WriteLine($"Processing: {ctx.Id}"))
.TapAsync(async ctx => await LogAsync(ctx))

// Conditional execution
.When(
    ctx => ctx.Amount > 1000,
    pipeline => pipeline
        .StepAsync(ctx => FraudCheckAsync(ctx))
        .StepAsync(ctx => ApprovalAsync(ctx)))

// Transform to different type
.Transform<OutputDto>(ctx => new OutputDto {
    Id = ctx.Id,
    Name = ctx.Name
})
```

#### Configuration

```csharp
// Global configuration
builder.Services.AddFlow(config => config
    .UseTelemetry()
    .UseRetry(maxAttempts: 3, backoffMs: 100)
    .PropagateExceptions(typeof(ValidationException)));

// Per-pipeline configuration
Pipeline.Start(context, config => {
    config.EnableTelemetry = true;
    config.DefaultRetryAttempts = 5;
    config.DefaultBackoffMs = 200;
})
```

### 3. Myth.Flow.Actions - CQRS & Event-Driven

#### Commands

```csharp
// Define command
public record CreateOrderCommand(CreateOrderDto Data) : ICommand<Guid>;

// Command handler
public class CreateOrderHandler : ICommandHandler<CreateOrderCommand, Guid> {
    private readonly IScopedService<IOrderRepository> _repository;
    private readonly IValidator _validator;
    private readonly IDispatcher _dispatcher;

    public async Task<CommandResult<Guid>> HandleAsync(
        CreateOrderCommand command,
        CancellationToken ct) {

        // Validate
        await _validator.ValidateAsync(command.Data, ValidationContextKey.Create, ct);

        // Process with pipeline
        var result = await Pipeline.Start(command.Data)
            .StepResultAsync(async dto => {
                var order = dto.To<Order>();
                await _repository.ExecuteAsync(repo => repo.AddAsync(order, ct));
                return Result<Order>.Success(order);
            })
            .TapAsync(async order => {
                await _dispatcher.PublishEventAsync(
                    new OrderCreatedEvent(order.Id), ct);
            })
            .ExecuteAsync(ct);

        if (result.IsFailure) {
            return CommandResult<Guid>.Failure(result.ErrorMessage);
        }

        return CommandResult<Guid>.Success(result.Value.Id);
    }
}

// Usage in controller
[HttpPost]
public async Task<IActionResult> Create(CreateOrderDto dto) {
    var command = new CreateOrderCommand(dto);
    var result = await _dispatcher.DispatchCommandAsync<CreateOrderCommand, Guid>(command);

    return result.IsSuccess
        ? CreatedAtAction(nameof(Get), new { id = result.Data }, result.Data)
        : BadRequest(result.ErrorMessage);
}
```

#### Queries with Caching

```csharp
// Define query
public record GetOrderQuery(Guid OrderId) : IQuery<OrderDto>;

// Query handler
public class GetOrderHandler : IQueryHandler<GetOrderQuery, OrderDto> {
    private readonly IScopedService<IOrderRepository> _repository;

    public async Task<QueryResult<OrderDto>> HandleAsync(
        GetOrderQuery query,
        CancellationToken ct) {

        var spec = SpecBuilder<Order>.Create()
            .And(o => o.Id == query.OrderId);

        var order = await _repository.ExecuteAsync(repo =>
            repo.FirstOrDefaultAsync(spec, ct));

        if (order == null) {
            return QueryResult<OrderDto>.Failure("Order not found");
        }

        var dto = order.To<OrderDto>();
        return QueryResult<OrderDto>.Success(dto);
    }
}

// Usage with cache
[HttpGet("{id}")]
public async Task<IActionResult> Get(Guid id) {
    var query = new GetOrderQuery(id);

    var cacheOptions = new CacheOptions {
        Enabled = true,
        Ttl = TimeSpan.FromMinutes(5),
        CacheKey = $"order:{id}"
    };

    var result = await _dispatcher.DispatchQueryAsync<GetOrderQuery, OrderDto>(
        query, cacheOptions);

    return result.IsSuccess ? Ok(result.Data) : NotFound();
}
```

#### Events

```csharp
// Define event
public record OrderCreatedEvent(Guid OrderId, decimal Total, string CustomerEmail) : IEvent;

// Event handler
public class OrderCreatedHandler : IEventHandler<OrderCreatedEvent> {
    private readonly IScopedService<IEmailService> _emailService;
    private readonly IScopedService<IInventoryService> _inventoryService;

    public async Task HandleAsync(OrderCreatedEvent @event, CancellationToken ct) {
        await Pipeline.Start(@event)
            .TapAsync(async evt => {
                await _emailService.ExecuteAsync(svc =>
                    svc.SendOrderConfirmationAsync(evt.CustomerEmail, evt.OrderId, ct));
            })
            .TapAsync(async evt => {
                await _inventoryService.ExecuteAsync(svc =>
                    svc.UpdateStockAsync(evt.OrderId, ct));
            })
            .ExecuteAsync(ct);
    }
}

// Publish event
await _dispatcher.PublishEventAsync(new OrderCreatedEvent(orderId, total, email));
```

#### Configuration

```csharp
builder.Services.AddFlow(config => config
    .UseTelemetry()
    .UseRetry(3, 1000)
    .UseActions(actions => actions
        // Message Broker
        .UseInMemory(opts => {
            opts.UseCircuitBreaker = true;
            opts.UseRetryPolicy = true;
            opts.UseDeadLetterQueue = true;
        })
        // or
        .UseKafka(kafka => {
            kafka.BootstrapServers = "localhost:9092";
            kafka.GroupId = "my-app";
            kafka.Topic = "events";
        })
        // or
        .UseRabbitMQ(rabbit => {
            rabbit.HostName = "localhost";
            rabbit.UserName = "guest";
            rabbit.Password = "guest";
            rabbit.ExchangeName = "my-exchange";
        })

        // Caching
        .UseCaching(cache => {
            cache.ProviderType = CacheProviderType.Memory;
            cache.DefaultTtl = TimeSpan.FromMinutes(5);
        })

        // Handler scanning
        .ScanAssemblies(typeof(Program).Assembly)
        .EnableAutoSubscription()));
```

### 4. Myth.Guard - Validation

#### Basic Validation

```csharp
public class CreateUserDto : IValidatable<CreateUserDto> {
    public string Name { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
    public List<string> Tags { get; set; }

    public void Validate(ValidationBuilder<CreateUserDto> builder,
        ValidationContextKey? context = null) {

        // String rules
        builder.For(Name, x => x
            .NotEmpty()
            .MinimumLength(2)
            .MaximumLength(100)
            .OnlyLetters());

        // Email validation
        builder.For(Email, x => x
            .NotEmpty()
            .Email());

        // Numeric rules
        builder.For(Age, x => x
            .GreaterThan(0)
            .LessThan(150));

        // Collection rules
        builder.For(Tags, x => x
            .NotEmpty()
            .CountBetween(1, 10)
            .All(tag => !string.IsNullOrWhiteSpace(tag))
            .Distinct());
    }
}
```

#### Context-Aware Validation

```csharp
public void Validate(ValidationBuilder<CreateUserDto> builder,
    ValidationContextKey? context = null) {

    // Global rules (always applied)
    builder.For(Email, x => x.NotEmpty().Email());
    builder.For(Age, x => x.GreaterThan(0).LessThan(150));

    // Create-specific rules
    builder.InContext(ValidationContextKey.Create, b => {
        b.For(Email, x => x
            .RespectAsync(async (email, ct, sp) => {
                var userService = sp.GetRequiredService<IUserService>();
                return await userService.IsEmailAvailableAsync(email, ct);
            })
            .WithMessage("Email already exists")
            .WithStatusCode(HttpStatusCode.Conflict));

        b.For(Password, x => x
            .NotEmpty()
            .MinimumLength(8)
            .Matches(new Regex(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).*$")));
    });

    // Update-specific rules
    builder.InContext(ValidationContextKey.Update, b => {
        b.For(Age, x => x.GreaterOrEquals(18));
    });
}
```

#### Async Validation with Services

```csharp
builder.For(ProductId, x => x
    .GreaterThan(0)
    .RespectAsync(async (productId, ct, sp) => {
        var productService = sp.GetRequiredService<IProductService>();
        return await productService.ExistsAsync(productId, ct);
    })
    .WithMessage("Product does not exist")
    .WithStatusCode(HttpStatusCode.NotFound));
```

#### Cross-Property Validation

```csharp
// Access entire entity in validation
builder.For(Amount, x => x
    .GreaterThan(0)
    .Respect<OrderDto>((amount, order) => {
        return order.CustomerType switch {
            "Premium" => amount <= 50000m,
            "Gold" => amount <= 25000m,
            "Silver" => amount <= 10000m,
            _ => amount <= 5000m
        };
    })
    .WithMessage("Amount exceeds limit for customer type"));

// Async with entity access
builder.For(Email, x => x
    .RespectAsync<LoginDto>(async (email, login, ct, sp) => {
        var userService = sp.GetRequiredService<IUserService>();
        return await userService.ValidateCredentialsAsync(login.Email, login.Password, ct);
    })
    .WithMessage("Invalid email and password combination"));
```

#### Usage in Controllers

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateUserDto dto) {
    // Validate and throw on failure
    await _validator.ValidateAsync(dto, ValidationContextKey.Create);

    // Or validate without throwing
    var result = await _validator.ValidateAndReturnAsync(dto, ValidationContextKey.Create);
    if (!result.IsValid) {
        return BadRequest(new { errors = result.Errors });
    }

    // Process...
    return Ok();
}
```

#### All Available Rules

**String Rules:**
- `NotEmpty()`, `MinimumLength(int)`, `MaximumLength(int)`, `LengthBetween(int, int)`
- `Email()`, `Url()`
- `OnlyLetters()`, `OnlyNumbers()`, `Alphanumeric()`
- `StartsWith(string)`, `EndsWith(string)`, `Contains(string)`
- `Matches(Regex)`, `BeOneOf(params string[])`
- `AvailableCharacters(params char[])`, `ForbiddenCharacters(params char[])`
- `NoSymbols(char[]?)`

**Numeric Rules (int, long, decimal, double, float):**
- `GreaterThan(T)`, `GreaterOrEquals(T)`, `LessThan(T)`, `LessOrEquals(T)`
- `Between(T, T)`, `Positive()`, `Negative()`, `Zero()`, `NotZero()`

**Collection Rules:**
- `NotEmpty()`, `CountBetween(int, int)`, `CountGreaterThan(int)`, `CountLessThan(int)`
- `All(Func<T, bool>)`, `Any(Func<T, bool>)`, `None(Func<T, bool>)`
- `Distinct()`, `DistinctBy<TKey>(Func<T, TKey>)`

**DateTime/DateOnly Rules:**
- `Past()`, `Future()`, `Today()`
- `After(DateTime)`, `Before(DateTime)`, `Between(DateTime, DateTime)`
- `AfterOrEquals(DateTime)`, `BeforeOrEquals(DateTime)`

**Boolean & Enum Rules:**
- `IsTrue()`, `IsFalse()`
- `BeInEnum<TEnum>()`, `BeNotInEnum<TEnum>()`

**Generic Rules (all types):**
- `NotNull()`, `BeNull()`, `EqualsTo(T)`, `NotEqualsTo(T)`
- `BeDefault()`, `NotDefault()`
- `Respect(Func<T, bool>)`, `RespectAsync(Func<T, CT, SP, Task<bool>>)`
- `Respect<TEntity>(Func<T, TEntity, bool>)`, `RespectAsync<TEntity>(Func<T, TEntity, CT, SP, Task<bool>>)`

**Rule Modifiers:**
- `.WithMessage(string)` - Custom error message
- `.WithMessage(Func<T, string>)` - Dynamic error message using field value
- `.WithStatusCode(int | HttpStatusCode)` - Custom HTTP status
- `.WithOptions(IReadOnlyList<string>)` - Valid options list for error response
- `.WithOptions(OptionsType)` - Auto-generate options from Constant<T,V> types
- `.When(Func<bool>)` - Conditional execution
- `.Unless(Func<bool>)` - Inverse conditional
- `.SetStopOnFailure(bool)` - Stop on first failure

#### Error Response Format (RFC 9457 Problem Details)

When using `app.UseGuard()` middleware, validation exceptions are automatically formatted following [RFC 9457 Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html):

```json
{
    "type": "https://github.com/paulaolileal/myth/blob/main/docs/errors/validation.md",
    "title": "One or more validation errors occurred",
    "status": 400,
    "instance": "/api/users",
    "traceId": "00-abc123...",
    "errors": {
        "email": ["Email already exists"],
        "password": ["Password must be at least 8 characters"]
    }
}
```

**Key Features:**
- Content-Type: `application/problem+json`
- `status`: HTTP status code (highest from all validation errors)
- `errors`: Grouped by field name with all error messages
- `traceId`: Correlation ID for debugging
- No custom error codes per field (use `status` and `message` instead)

**ValidationError Structure:**
```csharp
public sealed class ValidationError {
    public string Field { get; init; }
    public string Message { get; init; }
    public HttpStatusCode StatusCode { get; init; }
    public IReadOnlyList<string>? Options { get; init; }
}
```

### 5. Myth.Morph - Object Transformation

#### IMorphableTo (Source → Destination)

```csharp
// DTO to Entity transformation
public class UserDto : IMorphableTo<User> {
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime JoinDate { get; set; }

    public void MorphTo(Schema<User> schema) {
        // Auto-mapping (same names)
        // Email is automatically mapped

        // Custom mapping
        schema.Bind(u => u.FirstName, () => Name.Split(' ')[0]);
        schema.Bind(u => u.LastName, () => Name.Split(' ').Last());
        schema.Bind(u => u.CreatedAt, () => JoinDate);

        // With service provider
        schema.Bind(u => u.Profile, sp => {
            var service = sp.GetRequiredService<IProfileService>();
            return service.GetDefaultProfile();
        });

        // Async binding
        schema.BindAsync(u => u.Avatar, async sp => {
            var service = sp.GetRequiredService<IAvatarService>();
            return await service.GetDefaultAvatarAsync();
        });

        // Ignore properties
        schema.Ignore(u => u.Id);
        schema.Ignore(u => u.InternalField);
    }
}

// Usage
var userDto = new UserDto { Name = "John Doe", Email = "john@example.com" };
var user = userDto.To<User>(serviceProvider);
```

#### IMorphableFrom (Destination ← Source)

```csharp
// Entity to DTO transformation
public class OrderDto : IMorphableFrom<Order> {
    public Guid Id { get; set; }
    public string Customer { get; set; }
    public decimal Total { get; set; }
    public string CreatedDate { get; set; }
    public int ItemCount { get; set; }

    public void MorphFrom(Schema<Order> schema) {
        // Reverse mapping (Order → OrderDto)
        schema.Bind(() => Id, order => order.Id);
        schema.Bind(() => Customer, order => order.CustomerName);
        schema.Bind(() => Total, order => order.TotalAmount);
        schema.Bind(() => CreatedDate, order => order.CreatedAt.ToString("yyyy-MM-dd"));
        schema.Bind(() => ItemCount, order => order.Items.Count);

        // Async reverse mapping
        schema.BindAsync(() => Customer, async (order, sp) => {
            var customerService = sp.GetRequiredService<ICustomerService>();
            var customer = await customerService.GetByIdAsync(order.CustomerId);
            return customer?.Name ?? "Unknown";
        });
    }
}

// Usage
var order = await _repository.GetByIdAsync(orderId);
var orderDto = order.To<OrderDto>(serviceProvider);
```

#### Collections

```csharp
// Transform list
var users = await _repository.GetAllAsync();
var userDtos = users.To<User, UserDto>(serviceProvider);

// Async transformation
var productDtos = await products.ToAsync<Product, ProductDto>(serviceProvider);
```

#### Check Mapping Availability

```csharp
if (source.CanBindTo<Destination>(serviceProvider)) {
    var result = source.To<Destination>(serviceProvider);
}
```

### 6. Myth.Specification - Query Specifications

#### Basic Specifications

```csharp
// Define reusable specifications as extension methods
public static class ProductSpecifications {
    public static ISpec<Product> IsActive(this ISpec<Product> spec) {
        return spec.And(p => p.IsActive);
    }

    public static ISpec<Product> InCategory(this ISpec<Product> spec, string category) {
        return spec.And(p => p.Category == category);
    }

    public static ISpec<Product> PriceRange(this ISpec<Product> spec, decimal min, decimal max) {
        return spec.And(p => p.Price >= min && p.Price <= max);
    }

    public static ISpec<Product> SearchByName(this ISpec<Product> spec, string searchTerm) {
        return spec.AndIf(
            !string.IsNullOrEmpty(searchTerm),
            p => p.Name.Contains(searchTerm));
    }

    public static ISpec<Product> OrderByPrice(this ISpec<Product> spec, bool descending = false) {
        return descending
            ? spec.OrderDescending(p => p.Price)
            : spec.Order(p => p.Price);
    }
}
```

#### Usage in Repository

```csharp
public async Task<List<Product>> SearchProducts(ProductSearchDto search) {
    var spec = SpecBuilder<Product>.Create()
        .IsActive()
        .InCategory(search.Category)
        .PriceRange(search.MinPrice, search.MaxPrice)
        .SearchByName(search.SearchTerm)
        .OrderByPrice(search.SortDescending)
        .Skip((search.Page - 1) * search.PageSize)
        .Take(search.PageSize);

    return await _context.Products
        .Specify(spec)
        .ToListAsync();
}
```

#### Logical Composition

```csharp
var spec = SpecBuilder<Order>.Create()
    .And(o => o.CustomerId == customerId)
    .Or(o => o.Status == OrderStatus.Pending)
    .AndIf(fromDate.HasValue, o => o.CreatedAt >= fromDate.Value)
    .OrIf(toDate.HasValue, o => o.CreatedAt <= toDate.Value)
    .Not(); // Negate entire spec
```

#### Pagination & Ordering

```csharp
var spec = SpecBuilder<Product>.Create()
    .IsActive()
    .Order(p => p.Name) // Ascending
    .OrderDescending(p => p.Price) // Descending
    .Skip(20) // Offset
    .Take(10) // Limit
    .WithPagination(new Pagination { PageNumber = 2, PageSize = 10 });
```

#### Advanced Features

```csharp
// Distinct
var spec = SpecBuilder<Product>.Create()
    .DistinctBy(p => p.Brand);

// Get single item
var product = spec.SatisfyingItemFrom(query);

// Get all matching items
var products = spec.SatisfyingItemsFrom(query);

// In-memory validation
if (spec.IsSatisfiedBy(entity)) {
    // Entity matches specification
}

// Prepare full query (filter + sort + post-process)
var preparedQuery = spec.Prepare(query);

// Apply only filters
var filteredQuery = spec.Filtered(query);

// Apply only sorting
var sortedQuery = spec.Sorted(query);
```

### 7. Myth.Repository - Data Access

#### Basic Repository

```csharp
// Interface
public interface IProductRepository : IReadWriteRepositoryAsync<Product> { }

// Implementation
public class ProductRepository : ReadWriteRepositoryAsync<Product>, IProductRepository {
    public ProductRepository(AppDbContext context) : base(context) { }
}

// Registration
services.AddScoped<IProductRepository, ProductRepository>();

// Usage
public class ProductService {
    private readonly IProductRepository _repository;

    public async Task<Product> CreateAsync(CreateProductDto dto) {
        var product = new Product { Name = dto.Name, Price = dto.Price };
        await _repository.AddAsync(product);
        return product;
    }

    public async Task<IEnumerable<Product>> GetAllAsync() {
        return await _repository.GetAllAsync();
    }
}
```

#### Repository with Specifications

```csharp
public async Task<IPaginated<Order>> SearchOrdersAsync(
    OrderSearchDto search,
    CancellationToken ct = default) {

    var spec = SpecBuilder<Order>.Create()
        .And(o => o.CustomerId == search.CustomerId)
        .And(o => search.Statuses.Contains(o.Status))
        .And(o => o.CreatedAt >= search.FromDate)
        .OrderDescending(o => o.CreatedAt)
        .WithPagination(new Pagination {
            PageNumber = search.Page,
            PageSize = search.PageSize
        });

    return await _repository.GetPaginatedAsync(spec, ct);
}
```

#### Unit of Work

```csharp
public class OrderService {
    private readonly IOrderRepository _orderRepository;
    private readonly IOrderItemRepository _itemRepository;
    private readonly IUnitOfWorkRepository _unitOfWork;

    public async Task<Order> CreateOrderAsync(CreateOrderDto dto) {
        try {
            await _unitOfWork.BeginTransactionAsync();

            var order = new Order {
                CustomerId = dto.CustomerId,
                CreatedAt = DateTime.UtcNow
            };
            await _orderRepository.AddAsync(order);

            foreach (var itemDto in dto.Items) {
                var item = new OrderItem {
                    OrderId = order.Id,
                    ProductId = itemDto.ProductId,
                    Quantity = itemDto.Quantity
                };
                await _itemRepository.AddAsync(item);
            }

            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();

            return order;
        } catch {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
```

#### Read/Write Separation (CQRS)

```csharp
// Read repository for queries
public interface IProductReadRepository : IReadRepositoryAsync<Product> { }

public class ProductReadRepository : ReadRepositoryAsync<Product>, IProductReadRepository {
    public ProductReadRepository(AppDbContext context) : base(context) { }
}

// Write repository for commands
public interface IProductWriteRepository : IWriteRepositoryAsync<Product> { }

public class ProductWriteRepository : WriteRepositoryAsync<Product>, IProductWriteRepository {
    public ProductWriteRepository(AppDbContext context) : base(context) { }
}
```

### 8. Myth.Repository.EntityFramework - EF Core Implementation

Provides Entity Framework Core implementation of repository interfaces with Unit of Work pattern.

#### Basic Setup

```csharp
using Myth.Contexts;
using Myth.Repository.EntityFramework.Repositories;

// Define DbContext
public class AppDbContext : BaseContext {
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder) {
        base.OnModelCreating(modelBuilder);
        // Configure entities
    }
}

// Implement Repository
public class ProductRepository : ReadWriteRepositoryAsync<Product>, IProductRepository {
    public ProductRepository(AppDbContext context) : base(context) { }
}

// Register
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
services.AddScoped<IProductRepository, ProductRepository>();
services.AddScoped<IUnitOfWorkRepository, UnitOfWorkRepository<AppDbContext>>();
```

#### Unit of Work Pattern

```csharp
public class OrderService {
    private readonly IUnitOfWorkRepository _unitOfWork;

    public async Task ProcessOrderAsync(Order order, CancellationToken ct) {
        await _unitOfWork.BeginTransactionAsync(ct);
        try {
            await _unitOfWork.AddAsync(order, ct);
            await _unitOfWork.SaveChangesAsync(ct);
            await _unitOfWork.CommitTransactionAsync(ct);
        } catch {
            await _unitOfWork.RollbackTransactionAsync(ct);
            throw;
        }
    }
}
```

---

### 9. Myth.Rest - HTTP Client

Fluent REST client with retry policies, circuit breaker, and certificate support.

#### Basic Usage

```csharp
using Myth.Rest;

// Register
services.AddRest();

// Inject factory
public class ExternalApiService {
    private readonly IRestFactory _restFactory;

    public async Task<UserDto> GetUserAsync(int id) {
        var result = await _restFactory
            .Create("https://api.example.com")
            .Get($"/users/{id}")
            .WithRetry(maxAttempts: 3, backoffMs: 1000)
            .WithCircuitBreaker(failureThreshold: 5, breakDurationMs: 30000)
            .WithTimeout(TimeSpan.FromSeconds(10))
            .ExecuteAsync<UserDto>();

        return result.Data;
    }
}
```

#### POST Request with JSON

```csharp
var createUserDto = new CreateUserDto { Name = "John", Email = "john@example.com" };

var result = await _restFactory
    .Create("https://api.example.com")
    .Post("/users")
    .WithJsonBody(createUserDto)
    .WithHeader("Authorization", $"Bearer {token}")
    .WithRetry(3, 500)
    .ExecuteAsync<UserDto>();
```

---

### 10. Myth.Testing - Testing Utilities

Provides base classes, mocks, and utilities for comprehensive testing.

#### Base Test Class

```csharp
using Myth.Testing.Repositories;
using Myth.Testing.Extensions;

public class ProductServiceTests : BaseDatabaseTests<AppDbContext> {
    private readonly ProductService _service;

    public ProductServiceTests() : base() {
        AddServices(services => {
            services.AddScoped<IProductRepository, ProductRepository>();
            services.AddScoped<ProductService>();
            services.AddGuard();
            services.AddMorph();
        });

        _service = CreateInstance<ProductService>();
    }

    [Fact]
    public async Task CreateProduct_ShouldSucceed_WhenValid() {
        // Arrange
        var dto = new CreateProductDto { Name = "Test", Price = 10.00m };

        // Act
        var result = await _service.CreateProductAsync(dto);

        // Assert
        result.Should().NotBeNull();
        result.Name.Should().Be("Test");
    }
}
```

#### HTTP Client Mock

```csharp
using Myth.Testing.Mocks;

var httpClientMock = new HttpClientMock()
    .SetupGet("/users/1")
    .ReturnsJson(new UserDto { Id = 1, Name = "John" })
    .WithStatusCode(HttpStatusCode.OK);

var httpClient = httpClientMock.CreateClient();
```

---

### 11. Myth.Tool - Code Generation CLI

CLI tool for generating Myth architecture code with CQRS, DDD, and Clean Architecture patterns.

#### Installation

```bash
dotnet tool install -g Myth.Tool
```

#### Project Setup

```bash
# Setup new project with clean structure
myth setup MyProject --clean

# Setup project keeping examples
myth setup MyProject
```

#### Generate Domain Model

```bash
myth create model User \
  -p Id:Guid \
  -p Name:string:required \
  -p Email:string:required \
  --validate
```

#### Generate CQRS Command

```bash
myth create command User CreateUser \
  -p Name:string:required \
  -p Email:string:required \
  --return Guid \
  --validate \
  --events UserCreated
```

#### Generate Query

```bash
myth create query User GetUser \
  -p Id:Guid:required \
  --return GetUserResponse
```

#### Generate Repository

```bash
myth create repository User --type readwrite
```

#### Generate Controller

```bash
myth create controller User
```

#### Generate Tests

```bash
myth create test UserController
```

---

## Common Workflows

### Workflow 1: Complete CQRS with Validation, Pipeline, and Repository

```csharp
// 1. Define DTOs with validation
public class CreateOrderDto : IValidatable<CreateOrderDto> {
    public Guid CustomerId { get; set; }
    public List<CreateOrderItemDto> Items { get; set; }

    public void Validate(ValidationBuilder<CreateOrderDto> builder,
        ValidationContextKey? context = null) {

        builder.For(CustomerId, x => x.NotDefault());
        builder.For(Items, x => x
            .NotEmpty()
            .CountBetween(1, 100)
            .All(item => item.Quantity > 0));

        builder.InContext(ValidationContextKey.Create, b => {
            b.For(CustomerId, x => x
                .RespectAsync(async (id, ct, sp) => {
                    var customerService = sp.GetRequiredService<ICustomerService>();
                    return await customerService.ExistsAsync(id, ct);
                })
                .WithMessage("Customer not found")
                .WithStatusCode(HttpStatusCode.NotFound));
        });
    }
}

public class OrderDto : IMorphableFrom<Order> {
    public Guid Id { get; set; }
    public string Customer { get; set; }
    public decimal Total { get; set; }
    public int ItemCount { get; set; }

    public void MorphFrom(Schema<Order> schema) {
        schema.Bind(() => Id, o => o.Id);
        schema.Bind(() => Customer, o => o.CustomerName);
        schema.Bind(() => Total, o => o.TotalAmount);
        schema.Bind(() => ItemCount, o => o.Items.Count);
    }
}

// 2. Define Command
public record CreateOrderCommand(CreateOrderDto Data) : ICommand<Guid>;

// 3. Define Event
public record OrderCreatedEvent(Guid OrderId, decimal Total) : IEvent;

// 4. Command Handler with full pipeline
public class CreateOrderHandler : ICommandHandler<CreateOrderCommand, Guid> {
    private readonly IScopedService<IOrderRepository> _repository;
    private readonly IScopedService<IUnitOfWorkRepository> _unitOfWork;
    private readonly IValidator _validator;
    private readonly IDispatcher _dispatcher;

    public async Task<CommandResult<Guid>> HandleAsync(
        CreateOrderCommand command,
        CancellationToken ct) {

        var result = await Pipeline.Start(command.Data)
            .WithTelemetry("CreateOrder")
            .WithRetry(maxAttempts: 3, backoffMs: 100)

            // Validate
            .StepResultAsync(async dto => {
                var validationResult = await _validator.ValidateAndReturnAsync(
                    dto, ValidationContextKey.Create, ct);

                if (!validationResult.IsValid) {
                    return Result<CreateOrderDto>.Failure(
                        validationResult.Errors.First().Message);
                }

                return Result<CreateOrderDto>.Success(dto);
            })

            // Create order
            .StepResultAsync(async dto => {
                var order = new Order {
                    CustomerId = dto.CustomerId,
                    Items = dto.Items.Select(i => new OrderItem {
                        ProductId = i.ProductId,
                        Quantity = i.Quantity,
                        Price = i.Price
                    }).ToList(),
                    CreatedAt = DateTime.UtcNow
                };

                order.TotalAmount = order.Items.Sum(i => i.Price * i.Quantity);

                await _repository.ExecuteAsync(repo => repo.AddAsync(order, ct));
                await _unitOfWork.ExecuteAsync(uow => uow.SaveChangesAsync(ct));

                return Result<Order>.Success(order);
            })

            // Publish event
            .TapAsync(async order => {
                await _dispatcher.PublishEventAsync(
                    new OrderCreatedEvent(order.Id, order.TotalAmount), ct);
            })

            .ExecuteAsync(ct);

        if (result.IsFailure) {
            return CommandResult<Guid>.Failure(result.ErrorMessage);
        }

        return CommandResult<Guid>.Success(result.Value.Id);
    }
}

// 5. Query Handler with Specification
public record GetOrderQuery(Guid OrderId) : IQuery<OrderDto>;

public class GetOrderHandler : IQueryHandler<GetOrderQuery, OrderDto> {
    private readonly IScopedService<IOrderRepository> _repository;

    public async Task<QueryResult<OrderDto>> HandleAsync(
        GetOrderQuery query,
        CancellationToken ct) {

        var spec = SpecBuilder<Order>.Create()
            .And(o => o.Id == query.OrderId);

        var order = await _repository.ExecuteAsync(repo =>
            repo.FirstOrDefaultAsync(spec, ct));

        if (order == null) {
            return QueryResult<OrderDto>.Failure("Order not found");
        }

        var dto = order.To<OrderDto>();
        return QueryResult<OrderDto>.Success(dto);
    }
}

// 6. Event Handler
public class OrderCreatedHandler : IEventHandler<OrderCreatedEvent> {
    private readonly IScopedService<IEmailService> _emailService;

    public async Task HandleAsync(OrderCreatedEvent @event, CancellationToken ct) {
        await _emailService.ExecuteAsync(svc =>
            svc.SendOrderConfirmationAsync(@event.OrderId, ct));
    }
}

// 7. Controller
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase {
    private readonly IDispatcher _dispatcher;

    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderDto dto) {
        var command = new CreateOrderCommand(dto);
        var result = await _dispatcher.DispatchCommandAsync<CreateOrderCommand, Guid>(command);

        return result.IsSuccess
            ? CreatedAtAction(nameof(Get), new { id = result.Data }, result.Data)
            : BadRequest(result.ErrorMessage);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(Guid id) {
        var query = new GetOrderQuery(id);

        var cacheOptions = new CacheOptions {
            Enabled = true,
            Ttl = TimeSpan.FromMinutes(5)
        };

        var result = await _dispatcher.DispatchQueryAsync<GetOrderQuery, OrderDto>(
            query, cacheOptions);

        return result.IsSuccess ? Ok(result.Data) : NotFound();
    }
}
```

---

## Best Practices

### 1. Always Use BuildApp() or BuildWithGlobalProvider()

```csharp
// ✅ CORRECT - ASP.NET Core
var app = builder.BuildApp();

// ❌ INCORRECT
var app = builder.Build();

// ✅ CORRECT - Console App
var serviceProvider = services.BuildWithGlobalProvider();

// ❌ INCORRECT
var serviceProvider = services.BuildServiceProvider();
```

### 2. Use IScopedService<T> for Transient Handlers

```csharp
// ✅ CORRECT - Handlers are transient but need scoped repos
public class CreateOrderHandler : ICommandHandler<CreateOrderCommand, Guid> {
    private readonly IScopedService<IOrderRepository> _repository;

    public async Task<CommandResult<Guid>> HandleAsync(...) {
        return await _repository.ExecuteAsync(async repo => {
            var order = await repo.CreateAsync(...);
            return CommandResult<Guid>.Success(order.Id);
        });
    }
}

// ❌ INCORRECT - Injecting scoped service directly into transient handler
public class CreateOrderHandler : ICommandHandler<CreateOrderCommand, Guid> {
    private readonly IOrderRepository _repository; // ❌ Will cause issues!
}
```

### 3. Always Validate Before Processing

```csharp
// ✅ CORRECT
public async Task<CommandResult> HandleAsync(CreateOrderCommand command, CancellationToken ct) {
    await _validator.ValidateAsync(command.Data, ValidationContextKey.Create, ct);
    // Process...
}

// ❌ INCORRECT - No validation
public async Task<CommandResult> HandleAsync(CreateOrderCommand command, CancellationToken ct) {
    // Directly process without validation
}
```

### 4. Use Telemetry and Retry for Production

```csharp
// ✅ CORRECT
return await Pipeline.Start(context)
    .WithTelemetry("OperationName")
    .WithRetry(maxAttempts: 3, backoffMs: 100)
    .StepAsync(...)
    .ExecuteAsync();

// ❌ INCORRECT - No observability or resilience
return await Pipeline.Start(context)
    .StepAsync(...)
    .ExecuteAsync();
```

### 5. Use Specifications for Queries

```csharp
// ✅ CORRECT - Reusable, testable, encapsulated
var spec = SpecBuilder<Order>.Create()
    .ForCustomer(customerId)
    .WithStatus(OrderStatus.Pending)
    .Recent(TimeSpan.FromDays(30))
    .OrderDescending(o => o.CreatedAt);

var orders = await _repository.FindAsync(spec);

// ❌ INCORRECT - Query logic in repository
var orders = await _context.Orders
    .Where(o => o.CustomerId == customerId)
    .Where(o => o.Status == OrderStatus.Pending)
    .Where(o => o.CreatedAt >= DateTime.UtcNow.AddDays(-30))
    .OrderByDescending(o => o.CreatedAt)
    .ToListAsync();
```

### 6. Use Context-Aware Validation

```csharp
// ✅ CORRECT - Different rules per operation
public void Validate(ValidationBuilder<UserDto> builder, ValidationContextKey? context = null) {
    // Global rules
    builder.For(Email, x => x.NotEmpty().Email());

    // Create-specific
    builder.InContext(ValidationContextKey.Create, b => {
        b.For(Password, x => x.NotEmpty().MinimumLength(8));
    });

    // Update-specific
    builder.InContext(ValidationContextKey.Update, b => {
        b.For(Age, x => x.GreaterOrEquals(18));
    });
}

// ❌ INCORRECT - Same rules for all operations
public void Validate(ValidationBuilder<UserDto> builder, ValidationContextKey? context = null) {
    builder.For(Email, x => x.NotEmpty().Email());
    builder.For(Password, x => x.NotEmpty().MinimumLength(8)); // Always required!
}
```

### 7. Use Result Pattern Consistently

```csharp
// ✅ CORRECT
.StepResultAsync(async ctx => {
    if (ctx.IsValid) {
        return Result<Context>.Success(ctx);
    }
    return Result<Context>.Failure("Invalid context");
})

// ❌ INCORRECT - Throwing exceptions for flow control
.StepAsync(async ctx => {
    if (!ctx.IsValid) {
        throw new InvalidOperationException("Invalid context");
    }
    return ctx;
})
```

---

## Examples by Scenario

### E-Commerce Order Processing

```csharp
// Complete e-commerce order flow
public class OrderProcessingWorkflow {
    public async Task<Result<OrderConfirmationDto>> ProcessOrderAsync(
        CreateOrderRequest request) {

        return await Pipeline.Start(request)
            .WithTelemetry("ProcessOrder")
            .WithRetry(maxAttempts: 3, backoffMs: 100)

            // 1. Validate order
            .StepResultAsync<IValidator>((validator, req) =>
                validator.ValidateAndReturnAsync(req, ValidationContextKey.Create))

            // 2. Check inventory
            .StepResultAsync<InventoryService>((svc, req) =>
                svc.CheckAvailabilityAsync(req.Items))

            // 3. Calculate pricing
            .StepResultAsync<PricingService>((svc, req) =>
                svc.CalculateTotalAsync(req.Items, req.CouponCode))

            // 4. Process payment
            .StepResultAsync<PaymentService>((svc, ctx) =>
                svc.ProcessPaymentAsync(ctx.Total, ctx.PaymentMethod))

            // 5. Reserve inventory
            .StepResultAsync<InventoryService>((svc, ctx) =>
                svc.ReserveItemsAsync(ctx.Items))

            // 6. Create order
            .StepResultAsync<OrderService>((svc, ctx) =>
                svc.CreateOrderAsync(ctx))

            // 7. Publish events
            .TapAsync<IDispatcher>((dispatcher, ctx) =>
                dispatcher.PublishEventAsync(new OrderCreatedEvent(ctx.OrderId)))

            // 8. Send notifications
            .TapAsync<EmailService>((svc, ctx) =>
                svc.SendOrderConfirmationAsync(ctx.CustomerEmail, ctx.OrderId))

            .TapAsync<SmsService>((svc, ctx) =>
                svc.SendOrderSmsAsync(ctx.CustomerPhone, ctx.OrderId))

            // 9. Transform to response
            .Transform<OrderConfirmationDto>(ctx => new OrderConfirmationDto {
                OrderId = ctx.OrderId,
                Total = ctx.Total,
                EstimatedDelivery = DateTime.UtcNow.AddDays(5)
            })

            .ExecuteAsync();
    }
}
```

### User Registration with Email Verification

```csharp
// Complete user registration workflow
public class UserRegistrationWorkflow {
    public async Task<Result<UserDto>> RegisterUserAsync(
        RegisterUserRequest request) {

        return await Pipeline.Start(request)
            .WithTelemetry("RegisterUser")

            // 1. Validate
            .StepResultAsync<IValidator>((validator, req) =>
                validator.ValidateAndReturnAsync(req, ValidationContextKey.Create))

            // 2. Hash password
            .StepAsync<PasswordHasher>((hasher, req) => {
                req.PasswordHash = hasher.Hash(req.Password);
                return Task.FromResult(req);
            })

            // 3. Create user
            .StepResultAsync<UserService>((svc, req) =>
                svc.CreateUserAsync(req))

            // 4. Generate verification token
            .StepAsync<TokenService>((svc, ctx) => {
                ctx.VerificationToken = svc.GenerateEmailVerificationToken(ctx.UserId);
                return Task.FromResult(ctx);
            })

            // 5. Send verification email
            .TapAsync<EmailService>((svc, ctx) =>
                svc.SendVerificationEmailAsync(ctx.Email, ctx.VerificationToken))

            // 6. Publish event
            .TapAsync<IDispatcher>((dispatcher, ctx) =>
                dispatcher.PublishEventAsync(new UserRegisteredEvent(ctx.UserId)))

            // 7. Transform to DTO
            .Transform<UserDto>(ctx => ctx.User.To<UserDto>())

            .ExecuteAsync();
    }
}
```

### Complex Search with Filters, Sorting, and Pagination

```csharp
public class ProductSearchService {
    private readonly IProductRepository _repository;

    public async Task<IPaginated<ProductDto>> SearchAsync(ProductSearchRequest request) {
        // Build specification
        var spec = SpecBuilder<Product>.Create();

        // Apply filters conditionally
        if (!string.IsNullOrEmpty(request.Category)) {
            spec = spec.InCategory(request.Category);
        }

        if (request.MinPrice.HasValue || request.MaxPrice.HasValue) {
            spec = spec.PriceRange(
                request.MinPrice ?? 0,
                request.MaxPrice ?? decimal.MaxValue);
        }

        if (!string.IsNullOrEmpty(request.SearchTerm)) {
            spec = spec.SearchByName(request.SearchTerm);
        }

        if (request.InStock) {
            spec = spec.And(p => p.StockQuantity > 0);
        }

        if (request.Tags?.Any() == true) {
            spec = spec.And(p => p.Tags.Any(t => request.Tags.Contains(t)));
        }

        // Apply sorting
        spec = request.SortBy switch {
            "price_asc" => spec.Order(p => p.Price),
            "price_desc" => spec.OrderDescending(p => p.Price),
            "name" => spec.Order(p => p.Name),
            "newest" => spec.OrderDescending(p => p.CreatedAt),
            _ => spec.Order(p => p.Relevance)
        };

        // Apply pagination
        spec = spec.WithPagination(new Pagination {
            PageNumber = request.Page,
            PageSize = request.PageSize
        });

        // Execute query
        var paginatedProducts = await _repository.GetPaginatedAsync(spec);

        // Transform to DTOs
        var productDtos = paginatedProducts.Items.To<Product, ProductDto>();

        return new PaginatedResult<ProductDto> {
            Items = productDtos,
            CurrentPage = paginatedProducts.CurrentPage,
            TotalPages = paginatedProducts.TotalPages,
            PageSize = paginatedProducts.PageSize,
            TotalCount = paginatedProducts.TotalCount
        };
    }
}
```

---

## How to Use This Skill

### Prompting Examples for AI Assistants

**Setup a new project:**
```
"Use the myth skill to setup a new ASP.NET Core application with CQRS, validation, and pipelines"
```

**Add validation to DTO:**
```
"Use the myth skill with library=Guard to add validation rules for a CreateUserDto with email, age, and password fields"
```

**Create a command handler:**
```
"Use the myth skill with library=Flow.Actions and operation=command to create a CreateOrderCommand handler with validation, repository, and event publishing"
```

**Setup specifications:**
```
"Use the myth skill with library=Specification to create reusable query specifications for Product entity with filters for category, price range, and active status"
```

**Add object transformation:**
```
"Use the myth skill with library=Morph to create bidirectional mapping between Order entity and OrderDto"
```

**Configure complete CQRS:**
```
"Use the myth skill to configure a complete CQRS setup with Flow.Actions, Guard validation, Repository pattern, and Specifications for an e-commerce order system"
```

### Response Template

When asked to use the Myth skill, an AI assistant should:

1. **Identify the specific library/libraries needed** from the request
2. **Determine the operation type** (setup, configure, create handler, add validation, etc.)
3. **Provide complete, working code** following Myth conventions:
   - Use `BuildApp()` for ASP.NET Core
   - Use `IScopedService<T>` for handlers
   - Include validation, telemetry, and retry where appropriate
   - Follow Result pattern
   - Use specifications for queries
   - Apply SOLID principles
4. **Explain the code** with comments and context
5. **Suggest related patterns** or next steps

---

## Additional Resources

- **SOLID Principles**: All code should follow Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion
- **Clean Code**: Use meaningful names, small functions, clear abstractions
- **DDD**: Use ValueObjects, Entities, Aggregates, and Specifications
- **CQRS**: Separate read and write operations for scalability
- **Event-Driven**: Use events for cross-cutting concerns and integration

---

**End of Myth Skill Documentation**
