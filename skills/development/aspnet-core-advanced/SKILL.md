---
name: aspnet-core-advanced
version: "2.0.0"
description: Master advanced ASP.NET Core development including Entity Framework Core, authentication, testing, and enterprise patterns for production applications.
sasmp_version: "1.3.0"
bonded_agent: aspnet-core-architecture
bond_type: PRIMARY_BOND

# Skill Configuration
skill_type: atomic
responsibility: single

# Parameter Validation Schema
parameters:
  ef_version:
    type: string
    required: false
    default: "9.0"
    validation:
      allowed_values: ["6.0", "7.0", "8.0", "9.0"]
  auth_type:
    type: string
    required: false
    default: "jwt"
    validation:
      allowed_values: ["jwt", "cookie", "oauth2", "apikey", "certificate"]
  cache_provider:
    type: string
    required: false
    default: "memory"
    validation:
      allowed_values: ["memory", "redis", "sqlserver", "hybrid"]

# Retry Logic Configuration
retry_config:
  enabled: true
  max_attempts: 3
  backoff_type: exponential
  initial_delay_ms: 1000
  max_delay_ms: 15000
  jitter: true
  retryable_errors:
    - DATABASE_TIMEOUT
    - DEADLOCK
    - CONNECTION_FAILURE

# Observability Hooks
observability:
  logging:
    enabled: true
    level: Information
    include_sql_queries: true
    sensitive_fields: ["password", "token", "secret", "connectionString"]
  metrics:
    enabled: true
    namespace: skill.aspnetcore.advanced
    dimensions:
      - operation
      - entity_type
      - cache_hit
  tracing:
    enabled: true
    span_name: aspnetcore-advanced
    include_db_spans: true

# Unit Test Templates
test_templates:
  framework: xunit
  mocking: moq
  assertions: fluent_assertions
  coverage_target: 85
  database_strategy: in_memory
---

# ASP.NET Core Advanced Development

## Skill Overview

Production-grade advanced skill for enterprise ASP.NET Core applications. Covers Entity Framework Core 8/9, authentication, advanced testing, caching, and design patterns with comprehensive observability.

## Advanced Skills

### Entity Framework Core 8/9
```yaml
core_features:
  dbcontext:
    - Lifetime management (scoped)
    - Connection pooling
    - Query tracking behavior
    - Interceptors and events
    - Compiled models

  mapping:
    fluent_api:
      - Entity configuration
      - Relationships (HasOne, HasMany)
      - Complex types (.NET 8+)
      - JSON columns
      - Owned types
    conventions:
      - Table naming
      - Key naming
      - Property discovery

  queries:
    linq:
      - Projection (Select)
      - Filtering (Where)
      - Sorting (OrderBy)
      - Grouping (GroupBy)
      - Joins (Include, ThenInclude)
    raw_sql:
      - FromSqlRaw
      - FromSqlInterpolated
      - ExecuteSql

  performance:
    optimization:
      - No-tracking queries
      - Split queries (AsSplitQuery)
      - Compiled queries
      - Bulk operations
      - Connection resiliency
    monitoring:
      - Query logging
      - Execution plan analysis
      - Missing index detection

new_in_ef9:
  - LINQ improvements
  - Better JSON column support
  - Improved migrations
  - ExecuteUpdate/Delete enhancements
```

### Authentication & Authorization
```yaml
authentication_schemes:
  jwt_bearer:
    setup:
      - Configure token validation
      - Set issuer and audience
      - Configure signing key
    best_practices:
      - Short token expiry
      - Refresh token rotation
      - Secure key storage

  cookie:
    setup:
      - Configure cookie options
      - Set secure and httpOnly
      - Configure SameSite policy
    use_cases:
      - Server-rendered apps
      - MVC applications

  oauth2_openid:
    providers:
      - Microsoft Identity
      - Google
      - GitHub
      - Custom OIDC
    flows:
      - Authorization code
      - Client credentials
      - Device code

authorization:
  role_based:
    - [Authorize(Roles = "Admin")]
    - User.IsInRole("Admin")

  claims_based:
    - [Authorize(Policy = "RequireClaim")]
    - RequireClaim("permission", "read")

  policy_based:
    - Custom requirements
    - Authorization handlers
    - Resource-based

  resource_based:
    - IAuthorizationService
    - Document-level permissions
```

### Testing Strategies
```yaml
unit_testing:
  frameworks:
    - xUnit (recommended)
    - NUnit
    - MSTest
  patterns:
    - Arrange-Act-Assert (AAA)
    - Given-When-Then (BDD)
  mocking:
    - Moq
    - NSubstitute
    - FakeItEasy

integration_testing:
  web_application_factory:
    - In-memory test server
    - Custom configuration
    - Service replacement
  database:
    - In-memory provider
    - TestContainers
    - Respawn for cleanup

test_data:
  builders:
    - Fluent builders
    - Object mothers
    - AutoFixture
  fixtures:
    - Shared context
    - Disposable resources

coverage:
  tools:
    - Coverlet
    - ReportGenerator
  targets:
    - 80% line coverage
    - 70% branch coverage
```

### Caching Strategies
```yaml
in_memory:
  IMemoryCache:
    - Local cache
    - Fast access
    - Limited to single instance
  best_for:
    - Session data
    - Configuration
    - Frequently accessed data

distributed:
  IDistributedCache:
    providers:
      - Redis
      - SQL Server
      - NCache
    features:
      - Multi-instance support
      - Persistence
      - TTL policies

hybrid_cache:  # .NET 9
  features:
    - Two-tier caching
    - Automatic stampede protection
    - Tag-based invalidation
  configuration:
    - L1 (memory)
    - L2 (distributed)

output_caching:
  attributes:
    - [OutputCache(Duration = 60)]
    - [OutputCache(VaryByQuery = "id")]
  policies:
    - Custom cache policies
    - Cache profiles

response_caching:
  headers:
    - Cache-Control
    - ETag
    - Vary
```

### Performance Optimization
```yaml
async_patterns:
  best_practices:
    - Async all the way
    - Avoid .Result and .Wait()
    - Use CancellationToken
    - ConfigureAwait(false) in libraries
  common_issues:
    - Deadlocks
    - Thread starvation
    - Excessive allocations

memory_optimization:
  techniques:
    - Object pooling
    - Span<T> and Memory<T>
    - ArrayPool<T>
    - String interning
  tools:
    - dotnet-counters
    - dotnet-trace
    - Memory profilers

database_optimization:
  query_level:
    - Projection (select only needed)
    - Pagination
    - Indexing
    - Query splitting
  connection_level:
    - Connection pooling
    - Connection resiliency
    - Read replicas
```

## Code Examples

### Entity Framework Core Configuration
```csharp
// DbContext with production configuration
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(
            typeof(ApplicationDbContext).Assembly);

        // Global query filters
        modelBuilder.Entity<Product>()
            .HasQueryFilter(p => !p.IsDeleted);

        // JSON column mapping (.NET 8+)
        modelBuilder.Entity<Order>()
            .OwnsOne(o => o.ShippingAddress, builder =>
            {
                builder.ToJson();
            });
    }

    protected override void ConfigureConventions(
        ModelConfigurationBuilder configurationBuilder)
    {
        // Global conventions
        configurationBuilder.Properties<decimal>()
            .HavePrecision(18, 2);

        configurationBuilder.Properties<string>()
            .HaveMaxLength(500);
    }
}

// Entity configuration
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");

        builder.HasKey(p => p.Id);

        builder.Property(p => p.Name)
            .HasMaxLength(200)
            .IsRequired();

        builder.Property(p => p.Price)
            .HasPrecision(18, 2);

        builder.HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);

        builder.HasIndex(p => p.Sku)
            .IsUnique();

        builder.HasIndex(p => new { p.CategoryId, p.Name });
    }
}

// Registration with production settings
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Default"),
        sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: null);

            sqlOptions.CommandTimeout(30);
            sqlOptions.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery);
        });

    if (!builder.Environment.IsDevelopment())
    {
        options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTrackingWithIdentityResolution);
    }
});
```

### JWT Authentication Setup
```csharp
// Program.cs - JWT configuration
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidAudience = builder.Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
        ClockSkew = TimeSpan.FromMinutes(1)
    };

    options.Events = new JwtBearerEvents
    {
        OnAuthenticationFailed = context =>
        {
            if (context.Exception is SecurityTokenExpiredException)
            {
                context.Response.Headers.Append(
                    "Token-Expired", "true");
            }
            return Task.CompletedTask;
        }
    };
});

// Token generation service
public class TokenService : ITokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(User user, IEnumerable<string> roles)
    {
        var claims = new List<Claim>
        {
            new(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new(ClaimTypes.Email, user.Email),
            new(ClaimTypes.Name, user.UserName),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        claims.AddRange(roles.Select(role => new Claim(ClaimTypes.Role, role)));

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]!));

        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Repository Pattern with Unit of Work
```csharp
// Generic repository interface
public interface IRepository<T> where T : Entity
{
    Task<T?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IReadOnlyList<T>> GetAllAsync(CancellationToken ct = default);
    Task<IReadOnlyList<T>> GetAsync(
        Expression<Func<T, bool>> predicate,
        CancellationToken ct = default);
    Task<T> AddAsync(T entity, CancellationToken ct = default);
    void Update(T entity);
    void Delete(T entity);
}

// Generic repository implementation
public class Repository<T> : IRepository<T> where T : Entity
{
    protected readonly ApplicationDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(int id, CancellationToken ct = default)
    {
        return await _dbSet.FindAsync(new object[] { id }, ct);
    }

    public virtual async Task<IReadOnlyList<T>> GetAllAsync(CancellationToken ct = default)
    {
        return await _dbSet.ToListAsync(ct);
    }

    public virtual async Task<IReadOnlyList<T>> GetAsync(
        Expression<Func<T, bool>> predicate,
        CancellationToken ct = default)
    {
        return await _dbSet.Where(predicate).ToListAsync(ct);
    }

    public virtual async Task<T> AddAsync(T entity, CancellationToken ct = default)
    {
        await _dbSet.AddAsync(entity, ct);
        return entity;
    }

    public virtual void Update(T entity)
    {
        _dbSet.Attach(entity);
        _context.Entry(entity).State = EntityState.Modified;
    }

    public virtual void Delete(T entity)
    {
        if (_context.Entry(entity).State == EntityState.Detached)
        {
            _dbSet.Attach(entity);
        }
        _dbSet.Remove(entity);
    }
}

// Unit of Work
public interface IUnitOfWork : IDisposable
{
    IRepository<Product> Products { get; }
    IRepository<Category> Categories { get; }
    IRepository<Order> Orders { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IRepository<Product>? _products;
    private IRepository<Category>? _categories;
    private IRepository<Order>? _orders;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }

    public IRepository<Product> Products =>
        _products ??= new Repository<Product>(_context);

    public IRepository<Category> Categories =>
        _categories ??= new Repository<Category>(_context);

    public IRepository<Order> Orders =>
        _orders ??= new Repository<Order>(_context);

    public async Task<int> SaveChangesAsync(CancellationToken ct = default)
    {
        return await _context.SaveChangesAsync(ct);
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

### HybridCache Implementation (.NET 9)
```csharp
// Registration
builder.Services.AddHybridCache(options =>
{
    options.MaximumPayloadBytes = 1024 * 1024; // 1MB
    options.MaximumKeyLength = 512;
    options.DefaultEntryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromMinutes(5),
        LocalCacheExpiration = TimeSpan.FromMinutes(1)
    };
});

// Add Redis as L2 cache
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});

// Usage in service
public class ProductService : IProductService
{
    private readonly HybridCache _cache;
    private readonly ApplicationDbContext _context;
    private readonly ILogger<ProductService> _logger;

    public ProductService(
        HybridCache cache,
        ApplicationDbContext context,
        ILogger<ProductService> logger)
    {
        _cache = cache;
        _context = context;
        _logger = logger;
    }

    public async Task<ProductDto?> GetByIdAsync(int id, CancellationToken ct)
    {
        var cacheKey = $"product:{id}";

        var product = await _cache.GetOrCreateAsync(
            cacheKey,
            async token =>
            {
                _logger.LogDebug("Cache miss for product {ProductId}", id);

                return await _context.Products
                    .AsNoTracking()
                    .Where(p => p.Id == id)
                    .Select(p => new ProductDto
                    {
                        Id = p.Id,
                        Name = p.Name,
                        Price = p.Price,
                        CategoryName = p.Category.Name
                    })
                    .FirstOrDefaultAsync(token);
            },
            new HybridCacheEntryOptions
            {
                Expiration = TimeSpan.FromMinutes(10),
                LocalCacheExpiration = TimeSpan.FromMinutes(2)
            },
            cancellationToken: ct);

        return product;
    }

    public async Task InvalidateCacheAsync(int productId, CancellationToken ct)
    {
        await _cache.RemoveAsync($"product:{productId}", ct);
    }
}
```

### Integration Testing with WebApplicationFactory
```csharp
public class ProductsApiTests : IClassFixture<WebApplicationFactory<Program>>, IAsyncLifetime
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    private AsyncServiceScope _scope;
    private ApplicationDbContext _dbContext = null!;

    public ProductsApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Remove existing DbContext
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
                if (descriptor != null)
                    services.Remove(descriptor);

                // Add in-memory database
                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestDb");
                });

                // Add test authentication
                services.AddAuthentication("Test")
                    .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>(
                        "Test", options => { });
            });
        });

        _client = _factory.CreateClient();
        _client.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Test");
    }

    public async Task InitializeAsync()
    {
        _scope = _factory.Services.CreateAsyncScope();
        _dbContext = _scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        await _dbContext.Database.EnsureCreatedAsync();

        // Seed test data
        _dbContext.Products.AddRange(
            new Product { Id = 1, Name = "Test Product 1", Price = 10.00m },
            new Product { Id = 2, Name = "Test Product 2", Price = 20.00m }
        );
        await _dbContext.SaveChangesAsync();
    }

    public async Task DisposeAsync()
    {
        await _dbContext.Database.EnsureDeletedAsync();
        await _scope.DisposeAsync();
    }

    [Fact]
    public async Task GetProducts_ReturnsAllProducts()
    {
        // Act
        var response = await _client.GetAsync("/api/products");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);

        var products = await response.Content.ReadFromJsonAsync<List<ProductDto>>();
        products.Should().HaveCount(2);
    }

    [Fact]
    public async Task GetProduct_WhenExists_ReturnsProduct()
    {
        // Act
        var response = await _client.GetAsync("/api/products/1");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);

        var product = await response.Content.ReadFromJsonAsync<ProductDto>();
        product!.Name.Should().Be("Test Product 1");
    }

    [Fact]
    public async Task CreateProduct_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new CreateProductRequest
        {
            Name = "New Product",
            Price = 99.99m
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/products", request);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        response.Headers.Location.Should().NotBeNull();
    }
}

// Test authentication handler
public class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    public TestAuthHandler(
        IOptionsMonitor<AuthenticationSchemeOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder)
        : base(options, logger, encoder) { }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, "test-user-id"),
            new Claim(ClaimTypes.Email, "test@example.com"),
            new Claim(ClaimTypes.Role, "Admin")
        };

        var identity = new ClaimsIdentity(claims, "Test");
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, "Test");

        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

## Troubleshooting Guide

### Common Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| N+1 Query Problem | Excessive DB calls | Use `.Include()` or projection |
| Connection Pool Exhaustion | Timeouts | Use `using`, check connection lifetime |
| Deadlocks | Request hangs | Use async, check transaction isolation |
| Token Validation Fails | 401 Unauthorized | Check clock skew, issuer, audience |
| Cache Stampede | All requests hit DB | Use HybridCache or distributed locking |
| Memory Leaks | Growing memory | Check IDisposable, DI lifetimes |

### Debug Checklist

```yaml
step_1_ef_core:
  - Enable query logging
  - Check execution plans
  - Verify indexes
  - Monitor connection count

step_2_authentication:
  - Validate token format
  - Check signing key
  - Verify claims
  - Inspect authentication events

step_3_performance:
  - Profile with Application Insights
  - Check async patterns
  - Review caching strategy
  - Analyze memory allocations

step_4_testing:
  - Verify test isolation
  - Check service mocking
  - Review database cleanup
  - Inspect test authentication
```

## Assessment Criteria

- [ ] Design and implement database schemas with EF Core
- [ ] Implement authentication and authorization
- [ ] Write comprehensive unit tests (80%+ coverage)
- [ ] Write integration tests with WebApplicationFactory
- [ ] Optimize database queries
- [ ] Implement caching strategies
- [ ] Use design patterns appropriately
- [ ] Handle errors gracefully
- [ ] Implement monitoring and logging
- [ ] Apply performance optimizations

## References

- [Entity Framework Core](https://learn.microsoft.com/ef/core)
- [ASP.NET Core Security](https://learn.microsoft.com/aspnet/core/security)
- [Testing ASP.NET Core](https://learn.microsoft.com/aspnet/core/test)
- [Performance Best Practices](https://learn.microsoft.com/aspnet/core/performance)
