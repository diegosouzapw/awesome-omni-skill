---
name: aspnet-core-fundamentals
version: "2.0.0"
description: Master ASP.NET Core fundamentals including C#, project structure, routing, middleware, and basic API development. Essential skills for all ASP.NET Core developers.
sasmp_version: "1.3.0"
bonded_agent: aspnet-core-backend
bond_type: PRIMARY_BOND

# Skill Configuration
skill_type: atomic
responsibility: single  # Single-responsibility design

# Parameter Validation Schema
parameters:
  dotnet_version:
    type: string
    required: false
    default: "9.0"
    validation:
      pattern: "^[6-9]\\.0$"
      allowed_values: ["6.0", "7.0", "8.0", "9.0"]
  project_type:
    type: string
    required: false
    default: "webapi"
    validation:
      allowed_values: ["webapi", "mvc", "razor", "minimal"]
  database_provider:
    type: string
    required: false
    default: "sqlserver"
    validation:
      allowed_values: ["sqlserver", "postgresql", "mysql", "sqlite", "inmemory"]

# Retry Logic Configuration
retry_config:
  enabled: true
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 10000
  jitter: true
  retryable_errors:
    - NETWORK_ERROR
    - TIMEOUT
    - RATE_LIMIT

# Observability Hooks
observability:
  logging:
    enabled: true
    level: Information
    include_parameters: true
    sensitive_fields: ["connectionString", "password", "apiKey"]
  metrics:
    enabled: true
    namespace: skill.aspnetcore.fundamentals
    dimensions:
      - operation
      - status
      - duration_bucket
  tracing:
    enabled: true
    span_name: aspnetcore-fundamentals
    attributes:
      - skill.version
      - skill.operation
      - user.intent

# Unit Test Templates
test_templates:
  framework: xunit
  mocking: moq
  assertions: fluent_assertions
  coverage_target: 80
---

# ASP.NET Core Fundamentals

## Skill Overview

Production-grade fundamentals skill for ASP.NET Core 8.0/9.0 development. Implements atomic, single-responsibility design with comprehensive validation, retry logic, and observability.

## Core Skills

### C# Essentials (C# 12/13)
```yaml
fundamentals:
  variables_and_types:
    - Primitive types (int, string, bool, decimal)
    - Reference vs value types
    - Nullable reference types (NRT)
    - var and target-typed new
    - Records and record structs

  control_flow:
    - if/else, switch expressions
    - Pattern matching
    - for, foreach, while loops
    - LINQ query syntax
    - Exception handling (try/catch/finally)

  functions_and_methods:
    - Method signatures and overloading
    - Optional and named parameters
    - ref, out, in parameters
    - Local functions
    - Expression-bodied members

  oop_principles:
    - Classes and inheritance
    - Interfaces and abstract classes
    - Encapsulation (access modifiers)
    - Polymorphism
    - Composition over inheritance

  modern_csharp:
    - Primary constructors (C# 12)
    - Collection expressions (C# 12)
    - Raw string literals
    - Required members
    - File-scoped types

  async_programming:
    - async/await fundamentals
    - Task and ValueTask
    - Cancellation tokens
    - Async streams (IAsyncEnumerable)
    - ConfigureAwait considerations
```

### ASP.NET Core Project Setup
```yaml
project_creation:
  commands:
    webapi: dotnet new webapi -n MyApi --use-controllers
    minimal_api: dotnet new webapi -n MyApi
    mvc: dotnet new mvc -n MyApp
    razor: dotnet new razor -n MyApp

  project_structure:
    root:
      - Program.cs (entry point, DI, middleware)
      - appsettings.json (configuration)
      - appsettings.Development.json
    controllers:
      - Controller classes
    models:
      - Entity classes
      - DTOs
    services:
      - Business logic
    data:
      - DbContext
      - Repositories

configuration:
  appsettings_structure:
    ConnectionStrings: Database connections
    Logging: Log level configuration
    AllowedHosts: CORS settings
    CustomSettings: Application-specific

  environment_variables:
    ASPNETCORE_ENVIRONMENT: Development/Staging/Production
    ASPNETCORE_URLS: Binding URLs

  configuration_sources:
    - appsettings.json
    - appsettings.{Environment}.json
    - Environment variables
    - User secrets (development)
    - Azure Key Vault (production)
```

### Routing & Controllers
```yaml
attribute_routing:
  controller_level: "[Route(\"api/[controller]\")]"
  action_level: "[HttpGet(\"{id}\")]"
  route_constraints:
    - "{id:int}" (integer)
    - "{name:alpha}" (letters only)
    - "{date:datetime}" (date)
    - "{id:min(1)}" (minimum value)

http_methods:
  - "[HttpGet]" - Retrieve resource
  - "[HttpPost]" - Create resource
  - "[HttpPut]" - Replace resource
  - "[HttpPatch]" - Partial update
  - "[HttpDelete]" - Remove resource

action_results:
  success:
    - Ok(data) - 200
    - Created(uri, data) - 201
    - NoContent() - 204
    - Accepted() - 202
  client_errors:
    - BadRequest(error) - 400
    - Unauthorized() - 401
    - Forbidden() - 403
    - NotFound() - 404
    - Conflict() - 409
  server_errors:
    - StatusCode(500) - Internal error

model_binding:
  sources:
    - "[FromRoute]" - URL path
    - "[FromQuery]" - Query string
    - "[FromBody]" - Request body (JSON)
    - "[FromHeader]" - HTTP headers
    - "[FromForm]" - Form data
    - "[FromServices]" - DI container
```

### Middleware Pipeline
```yaml
middleware_order:
  1: Exception handling
  2: HTTPS redirection
  3: Static files
  4: Routing
  5: CORS
  6: Authentication
  7: Authorization
  8: Custom middleware
  9: Endpoints

built_in_middleware:
  - UseExceptionHandler()
  - UseHttpsRedirection()
  - UseStaticFiles() / MapStaticAssets() (.NET 9)
  - UseRouting()
  - UseCors()
  - UseAuthentication()
  - UseAuthorization()
  - UseRateLimiter()

custom_middleware:
  inline: app.Use(async (context, next) => { ... })
  class_based: app.UseMiddleware<CustomMiddleware>()
  convention: Must have Invoke/InvokeAsync method
```

### Models & Data Binding
```yaml
model_classes:
  entities:
    purpose: Database representation
    features:
      - Navigation properties
      - Data annotations
      - Fluent configuration

  dtos:
    purpose: API contracts
    best_practices:
      - Separate from entities
      - Use records for immutability
      - Include only needed fields

validation:
  data_annotations:
    - "[Required]"
    - "[StringLength(100)]"
    - "[Range(1, 100)]"
    - "[EmailAddress]"
    - "[RegularExpression(pattern)]"
    - "[Compare(\"OtherProperty\")]"

  fluent_validation:
    purpose: Complex validation rules
    example: |
      RuleFor(x => x.Email)
        .NotEmpty()
        .EmailAddress()
        .Must(BeUniqueEmail);

model_binding_validation:
  automatic: ModelState.IsValid
  problem_details: Automatic 400 response
  custom_response: Override with filters
```

### Dependency Injection
```yaml
service_lifetimes:
  singleton:
    description: Single instance for application lifetime
    use_cases:
      - Configuration
      - Caching services
      - Logging
    caution: Thread-safety required

  scoped:
    description: New instance per request
    use_cases:
      - DbContext
      - Request-specific services
      - Unit of Work

  transient:
    description: New instance every time
    use_cases:
      - Lightweight stateless services
      - Factory-created services
    caution: Memory allocation overhead

registration_patterns:
  interface_based: |
    services.AddScoped<IProductService, ProductService>();

  concrete_type: |
    services.AddSingleton<MyConfiguration>();

  factory: |
    services.AddScoped<IService>(sp =>
        new MyService(sp.GetRequiredService<IDependency>()));

  keyed_services: |  # .NET 8+
    services.AddKeyedSingleton<ICache>("memory", new MemoryCache());
    services.AddKeyedSingleton<ICache>("redis", new RedisCache());
```

## Code Examples

### Production-Ready Minimal API
```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration
builder.Configuration
    .AddJsonFile("appsettings.json", optional: false)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
    .AddEnvironmentVariables();

// Services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// Add validation
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// Add problem details
builder.Services.AddProblemDetails();

var app = builder.Build();

// Middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseExceptionHandler();

// Endpoints
var products = app.MapGroup("/api/products")
    .WithTags("Products")
    .WithOpenApi();

products.MapGet("/", async (IProductService service, CancellationToken ct) =>
{
    var result = await service.GetAllAsync(ct);
    return Results.Ok(result);
})
.WithName("GetProducts")
.Produces<IEnumerable<ProductDto>>(StatusCodes.Status200OK);

products.MapGet("/{id:int}", async (int id, IProductService service, CancellationToken ct) =>
{
    var product = await service.GetByIdAsync(id, ct);
    return product is null
        ? Results.NotFound()
        : Results.Ok(product);
})
.WithName("GetProduct")
.Produces<ProductDto>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound);

products.MapPost("/", async (
    CreateProductRequest request,
    IValidator<CreateProductRequest> validator,
    IProductService service,
    CancellationToken ct) =>
{
    var validation = await validator.ValidateAsync(request, ct);
    if (!validation.IsValid)
        return Results.ValidationProblem(validation.ToDictionary());

    var id = await service.CreateAsync(request, ct);
    return Results.Created($"/api/products/{id}", new { id });
})
.WithName("CreateProduct")
.Produces<object>(StatusCodes.Status201Created)
.ProducesValidationProblem();

app.Run();
```

### Controller-Based API
```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductService service,
        ILogger<ProductsController> logger)
    {
        _service = service;
        _logger = logger;
    }

    /// <summary>
    /// Get all products with optional filtering
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] ProductQueryParameters query,
        CancellationToken ct)
    {
        var result = await _service.GetProductsAsync(query, ct);

        Response.Headers.Append("X-Total-Count", result.TotalCount.ToString());

        return Ok(result);
    }

    /// <summary>
    /// Get product by ID
    /// </summary>
    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetProduct(
        int id,
        CancellationToken ct)
    {
        var product = await _service.GetByIdAsync(id, ct);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }

    /// <summary>
    /// Create a new product
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> CreateProduct(
        [FromBody] CreateProductRequest request,
        CancellationToken ct)
    {
        var product = await _service.CreateAsync(request, ct);

        return CreatedAtAction(
            nameof(GetProduct),
            new { id = product.Id },
            product);
    }

    /// <summary>
    /// Update existing product
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateProduct(
        int id,
        [FromBody] UpdateProductRequest request,
        CancellationToken ct)
    {
        var success = await _service.UpdateAsync(id, request, ct);

        if (!success)
            return NotFound();

        return NoContent();
    }

    /// <summary>
    /// Delete product
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteProduct(int id, CancellationToken ct)
    {
        var success = await _service.DeleteAsync(id, ct);

        if (!success)
            return NotFound();

        return NoContent();
    }
}
```

### Custom Middleware
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault()
            ?? Guid.NewGuid().ToString();

        context.Response.Headers.Append("X-Correlation-ID", correlationId);

        using var scope = _logger.BeginScope(new Dictionary<string, object>
        {
            ["CorrelationId"] = correlationId,
            ["RequestPath"] = context.Request.Path,
            ["RequestMethod"] = context.Request.Method
        });

        var stopwatch = Stopwatch.StartNew();

        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();

            _logger.LogInformation(
                "{Method} {Path} completed in {ElapsedMs}ms with status {StatusCode}",
                context.Request.Method,
                context.Request.Path,
                stopwatch.ElapsedMilliseconds,
                context.Response.StatusCode);
        }
    }
}

// Registration
app.UseMiddleware<RequestLoggingMiddleware>();
```

### Configuration with Options Pattern
```csharp
// appsettings.json
{
    "EmailSettings": {
        "SmtpServer": "smtp.example.com",
        "SmtpPort": 587,
        "SenderEmail": "noreply@example.com",
        "EnableSsl": true
    }
}

// Options class
public class EmailSettings
{
    public const string SectionName = "EmailSettings";

    public string SmtpServer { get; init; } = string.Empty;
    public int SmtpPort { get; init; } = 587;
    public string SenderEmail { get; init; } = string.Empty;
    public bool EnableSsl { get; init; } = true;
}

// Registration with validation
builder.Services.AddOptions<EmailSettings>()
    .BindConfiguration(EmailSettings.SectionName)
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Usage with IOptions
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
}

// Usage with IOptionsSnapshot (reloads on change)
public class EmailService
{
    private readonly IOptionsSnapshot<EmailSettings> _options;

    public EmailSettings Settings => _options.Value;
}
```

## Unit Test Templates

### Controller Unit Test
```csharp
public class ProductsControllerTests
{
    private readonly Mock<IProductService> _serviceMock;
    private readonly Mock<ILogger<ProductsController>> _loggerMock;
    private readonly ProductsController _controller;

    public ProductsControllerTests()
    {
        _serviceMock = new Mock<IProductService>();
        _loggerMock = new Mock<ILogger<ProductsController>>();
        _controller = new ProductsController(_serviceMock.Object, _loggerMock.Object);
    }

    [Fact]
    public async Task GetProduct_WhenExists_ReturnsOk()
    {
        // Arrange
        var productId = 1;
        var expectedProduct = new ProductDto { Id = productId, Name = "Test" };

        _serviceMock
            .Setup(s => s.GetByIdAsync(productId, It.IsAny<CancellationToken>()))
            .ReturnsAsync(expectedProduct);

        // Act
        var result = await _controller.GetProduct(productId, CancellationToken.None);

        // Assert
        var okResult = result.Result.Should().BeOfType<OkObjectResult>().Subject;
        var product = okResult.Value.Should().BeOfType<ProductDto>().Subject;
        product.Id.Should().Be(productId);
    }

    [Fact]
    public async Task GetProduct_WhenNotFound_ReturnsNotFound()
    {
        // Arrange
        var productId = 999;

        _serviceMock
            .Setup(s => s.GetByIdAsync(productId, It.IsAny<CancellationToken>()))
            .ReturnsAsync((ProductDto?)null);

        // Act
        var result = await _controller.GetProduct(productId, CancellationToken.None);

        // Assert
        result.Result.Should().BeOfType<NotFoundResult>();
    }

    [Fact]
    public async Task CreateProduct_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new CreateProductRequest { Name = "New Product", Price = 99.99m };
        var createdProduct = new ProductDto { Id = 1, Name = request.Name, Price = request.Price };

        _serviceMock
            .Setup(s => s.CreateAsync(request, It.IsAny<CancellationToken>()))
            .ReturnsAsync(createdProduct);

        // Act
        var result = await _controller.CreateProduct(request, CancellationToken.None);

        // Assert
        var createdResult = result.Result.Should().BeOfType<CreatedAtActionResult>().Subject;
        createdResult.ActionName.Should().Be(nameof(ProductsController.GetProduct));
        createdResult.RouteValues!["id"].Should().Be(1);
    }
}
```

### Integration Test
```csharp
public class ProductsApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    private readonly WebApplicationFactory<Program> _factory;

    public ProductsApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace database with in-memory
                services.RemoveAll<DbContextOptions<AppDbContext>>();
                services.AddDbContext<AppDbContext>(options =>
                    options.UseInMemoryDatabase("TestDb"));
            });
        });

        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_ReturnsSuccessStatusCode()
    {
        // Act
        var response = await _client.GetAsync("/api/products");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }

    [Fact]
    public async Task CreateProduct_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new { Name = "Test Product", Price = 99.99 };
        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json");

        // Act
        var response = await _client.PostAsync("/api/products", content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        response.Headers.Location.Should().NotBeNull();
    }

    [Fact]
    public async Task CreateProduct_WithInvalidData_ReturnsBadRequest()
    {
        // Arrange
        var request = new { Name = "", Price = -1 }; // Invalid
        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json");

        // Act
        var response = await _client.PostAsync("/api/products", content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
    }
}
```

## Troubleshooting Guide

### Common Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| 404 Not Found | Route not matching | Check route template, HTTP method |
| 415 Unsupported Media Type | Content-Type missing | Add `Content-Type: application/json` |
| 500 Internal Error | Unhandled exception | Check logs, add exception middleware |
| Model binding fails | Null values | Check property names, [FromBody] attribute |
| DI resolution fails | Service not registered | Add service to DI container |

### Debug Checklist

```yaml
step_1_routing:
  - Verify controller has [ApiController] attribute
  - Check route template matches URL
  - Confirm HTTP method matches action attribute
  - Validate route constraints

step_2_model_binding:
  - Check JSON property names match
  - Verify Content-Type header
  - Inspect ModelState errors
  - Check for [FromBody], [FromQuery] attributes

step_3_di_issues:
  - Verify service is registered
  - Check service lifetime compatibility
  - Look for circular dependencies
  - Inspect exception details

step_4_configuration:
  - Verify appsettings.json syntax
  - Check environment name
  - Confirm configuration binding
  - Inspect IConfiguration values
```

## Assessment Criteria

- [ ] Can create a new ASP.NET Core project
- [ ] Understand request/response pipeline
- [ ] Write basic REST APIs with proper HTTP methods
- [ ] Use dependency injection correctly
- [ ] Apply validation to models
- [ ] Configure middleware pipeline
- [ ] Handle errors with ProblemDetails
- [ ] Use async/await correctly
- [ ] Write unit and integration tests
- [ ] Apply configuration with Options pattern

## References

- [ASP.NET Core Documentation](https://learn.microsoft.com/aspnet/core)
- [C# Language Reference](https://learn.microsoft.com/dotnet/csharp)
- [Minimal APIs Tutorial](https://learn.microsoft.com/aspnet/core/tutorials/min-web-api)
- [Controller-based APIs](https://learn.microsoft.com/aspnet/core/web-api)
