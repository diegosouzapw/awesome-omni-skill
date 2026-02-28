---
name: umbraco-development
description: Apply when working with Umbraco CMS, Composers, services, or content APIs
globs:
  - "**/Composers/**/*.cs"
  - "**/Controllers/**/*.cs"
  - "**/App_Plugins/**/*"
  - "**/Views/**/*.cshtml"
  - "**/appsettings.json"
  - "**/*.scss"
  - "**/*.css"
  - "**/scripts/**/*.js"
  - "**/*.tsx"
  - "**/*.jsx"
---

## Bundled Skills

This plugin includes the following additional skills for comprehensive development context:

| Skill | Purpose |
|-------|---------|
| `frontend-razor` | Razor view syntax, layouts, partials, tag helpers |
| `frontend-classic` | CSS/SASS organization, JavaScript/jQuery patterns (traditional sites) |
| `frontend-modern` | React, Vue, TypeScript patterns (headless/decoupled sites) |
| `backend-csharp` | C#/.NET DI patterns, service architecture, async/await |
| `fullstack-classic` | jQuery AJAX integration, form handling (traditional) |
| `fullstack-modern` | REST/GraphQL APIs, Content Delivery API integration (headless) |

These skills are automatically included when you install the Umbraco Analyzer plugin.

# Umbraco Development Patterns

## Project Structure

```
src/
├── Web/                          # Main Umbraco web project
│   ├── App_Plugins/              # Backoffice extensions
│   ├── Composers/                # DI and configuration
│   ├── Controllers/              # Surface and API controllers
│   ├── Views/                    # Razor templates
│   └── Program.cs
├── Core/                         # Business logic (optional)
│   ├── Services/
│   ├── Models/
│   └── Interfaces/
└── Infrastructure/               # Data access (optional)
    ├── Repositories/
    └── ExternalServices/
```

## Composer Pattern

Composers are the entry point for dependency injection and configuration.

### Basic Composer

```csharp
using Umbraco.Cms.Core.Composing;

public class ServicesComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Register services
        builder.Services.AddScoped<IProductService, ProductService>();
        builder.Services.AddScoped<ISearchService, SearchService>();

        // Register notification handlers
        builder.AddNotificationHandler<ContentPublishedNotification, ContentPublishedHandler>();
        builder.AddNotificationAsyncHandler<ContentSavingNotification, ContentSavingHandler>();
    }
}
```

### Composer with Dependencies

```csharp
public class ConfiguredServicesComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Register with configuration
        builder.Services.Configure<EmailOptions>(
            builder.Config.GetSection("Email"));

        builder.Services.AddScoped<IEmailService, EmailService>();
    }
}
```

### Composer Ordering

```csharp
// Run after another Composer
[ComposeAfter(typeof(ServicesComposer))]
public class DependentComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder) { }
}

// Run before another Composer
[ComposeBefore(typeof(OtherComposer))]
public class EarlyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder) { }
}
```

## Notification Handlers

### Synchronous Handler

```csharp
public class ContentPublishedHandler : INotificationHandler<ContentPublishedNotification>
{
    private readonly ILogger<ContentPublishedHandler> _logger;

    public ContentPublishedHandler(ILogger<ContentPublishedHandler> logger)
    {
        _logger = logger;
    }

    public void Handle(ContentPublishedNotification notification)
    {
        foreach (var content in notification.PublishedEntities)
        {
            _logger.LogInformation("Content published: {Name}", content.Name);
        }
    }
}
```

### Asynchronous Handler

```csharp
public class ContentSavingHandler : INotificationAsyncHandler<ContentSavingNotification>
{
    private readonly IExternalService _externalService;

    public ContentSavingHandler(IExternalService externalService)
    {
        _externalService = externalService;
    }

    public async Task HandleAsync(ContentSavingNotification notification, CancellationToken ct)
    {
        foreach (var content in notification.SavedEntities)
        {
            await _externalService.SyncContentAsync(content.Key, ct);
        }
    }
}
```

## Content Access

### Using IPublishedContentQuery

```csharp
public class ContentService
{
    private readonly IPublishedContentQuery _contentQuery;

    public ContentService(IPublishedContentQuery contentQuery)
    {
        _contentQuery = contentQuery;
    }

    public IPublishedContent? GetContentByKey(Guid key)
    {
        return _contentQuery.Content(key);
    }

    public IPublishedContent? GetContentByRoute(string route)
    {
        return _contentQuery.ContentSingleAtXPath($"//{route}");
    }

    public IEnumerable<IPublishedContent> GetChildren(IPublishedContent parent)
    {
        return parent.Children.Where(x => x.IsVisible());
    }
}
```

### Using IUmbracoContextFactory (Singleton-Safe)

```csharp
public class SingletonService
{
    private readonly IUmbracoContextFactory _contextFactory;

    public SingletonService(IUmbracoContextFactory contextFactory)
    {
        _contextFactory = contextFactory;
    }

    public IPublishedContent? GetContent(Guid key)
    {
        using var cref = _contextFactory.EnsureUmbracoContext();
        return cref.UmbracoContext?.Content?.GetById(key);
    }
}
```

## Surface Controllers

### Form Handling

```csharp
public class ContactFormController : SurfaceController
{
    private readonly IContactService _contactService;
    private readonly ILogger<ContactFormController> _logger;

    public ContactFormController(
        IUmbracoContextAccessor umbracoContextAccessor,
        IUmbracoDatabaseFactory databaseFactory,
        ServiceContext services,
        AppCaches appCaches,
        IProfilingLogger profilingLogger,
        IPublishedUrlProvider publishedUrlProvider,
        IContactService contactService,
        ILogger<ContactFormController> logger)
        : base(umbracoContextAccessor, databaseFactory, services, appCaches, profilingLogger, publishedUrlProvider)
    {
        _contactService = contactService;
        _logger = logger;
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Submit(ContactFormModel model, CancellationToken ct)
    {
        if (!ModelState.IsValid)
        {
            return CurrentUmbracoPage();
        }

        try
        {
            await _contactService.ProcessContactAsync(model, ct);
            TempData["ContactSuccess"] = true;
            return RedirectToCurrentUmbracoPage();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process contact form");
            ModelState.AddModelError("", "An error occurred. Please try again.");
            return CurrentUmbracoPage();
        }
    }
}
```

### AJAX Response

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> SubmitAjax(ContactFormModel model, CancellationToken ct)
{
    if (!ModelState.IsValid)
    {
        return Json(new
        {
            success = false,
            errors = ModelState.SelectMany(x => x.Value.Errors.Select(e => e.ErrorMessage))
        });
    }

    await _contactService.ProcessContactAsync(model, ct);
    return Json(new { success = true, message = "Thank you for your message!" });
}
```

## API Controllers

### Umbraco API Controller

```csharp
[Route("api/products")]
public class ProductsApiController : UmbracoApiController
{
    private readonly IProductService _productService;

    public ProductsApiController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<IActionResult> GetAll(CancellationToken ct)
    {
        var products = await _productService.GetAllAsync(ct);
        return Ok(products);
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetById(Guid id, CancellationToken ct)
    {
        var product = await _productService.GetByIdAsync(id, ct);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }
}
```

## Examine (Search)

### Basic Search

```csharp
public class SearchService
{
    private readonly IExamineManager _examineManager;

    public SearchService(IExamineManager examineManager)
    {
        _examineManager = examineManager;
    }

    public IEnumerable<ISearchResult> Search(string term)
    {
        if (!_examineManager.TryGetIndex("ExternalIndex", out var index))
        {
            return Enumerable.Empty<ISearchResult>();
        }

        var searcher = index.Searcher;
        var query = searcher.CreateQuery("content")
            .NativeQuery($"+__NodeTypeAlias:blogPost +bodyText:{term}*");

        return query.Execute();
    }
}
```

### Advanced Search with Filters

```csharp
public ISearchResults SearchProducts(string term, string category, int page, int pageSize)
{
    if (!_examineManager.TryGetIndex("ExternalIndex", out var index))
    {
        return EmptySearchResults.Instance;
    }

    var query = index.Searcher.CreateQuery("content")
        .NodeTypeAlias("product");

    if (!string.IsNullOrEmpty(term))
    {
        query = query.And().ManagedQuery(term);
    }

    if (!string.IsNullOrEmpty(category))
    {
        query = query.And().Field("category", category);
    }

    var results = query.Execute(new QueryOptions(
        skip: (page - 1) * pageSize,
        take: pageSize
    ));

    return results;
}
```

## Caching

### Runtime Cache

```csharp
public class CachedContentService
{
    private readonly AppCaches _appCaches;
    private readonly IContentService _contentService;

    public CachedContentService(AppCaches appCaches, IContentService contentService)
    {
        _appCaches = appCaches;
        _contentService = contentService;
    }

    public object GetCachedData(string key)
    {
        return _appCaches.RuntimeCache.GetCacheItem(
            key,
            () => _contentService.GetData(),
            TimeSpan.FromMinutes(5)
        );
    }

    public void ClearCache(string key)
    {
        _appCaches.RuntimeCache.ClearByKey(key);
    }
}
```

## Configuration

### appsettings.json

```json
{
  "Umbraco": {
    "CMS": {
      "Content": {
        "AllowEditInvariantFromNonDefault": true
      },
      "Global": {
        "UseHttps": true
      },
      "ModelsBuilder": {
        "ModelsMode": "SourceCodeAuto"
      }
    }
  },
  "MyApp": {
    "Email": {
      "SmtpHost": "smtp.example.com",
      "SmtpPort": 587,
      "FromAddress": "noreply@example.com"
    }
  }
}
```

### Options Pattern

```csharp
public class EmailOptions
{
    public string SmtpHost { get; set; } = string.Empty;
    public int SmtpPort { get; set; } = 587;
    public string FromAddress { get; set; } = string.Empty;
}

// In Composer
builder.Services.Configure<EmailOptions>(
    builder.Config.GetSection("MyApp:Email"));

// In Service
public class EmailService
{
    private readonly EmailOptions _options;

    public EmailService(IOptions<EmailOptions> options)
    {
        _options = options.Value;
    }
}
```

## Razor Views

### Template with Model

```cshtml
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<ContentModels.BlogPost>
@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels

@{
    Layout = "Master.cshtml";
}

<article>
    <h1>@Model.Title</h1>
    <p class="meta">
        Published: @Model.PublishDate.ToString("MMMM dd, yyyy")
        by @Model.Author?.Name
    </p>
    <div class="content">
        @Html.Raw(Model.BodyText)
    </div>
</article>
```

### Partial View with Model

```cshtml
@* Views/Partials/_Navigation.cshtml *@
@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage
@{
    var root = Model.Root();
    var items = root.Children.Where(x => x.IsVisible());
}

<nav>
    <ul>
        @foreach (var item in items)
        {
            <li class="@(item.IsAncestorOrSelf(Model) ? "active" : "")">
                <a href="@item.Url()">@item.Name</a>
            </li>
        }
    </ul>
</nav>
```

### Form in View

```cshtml
@using (Html.BeginUmbracoForm<ContactFormController>(nameof(ContactFormController.Submit)))
{
    @Html.AntiForgeryToken()

    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Message"></label>
        <textarea asp-for="Message" class="form-control"></textarea>
        <span asp-validation-for="Message" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Send</button>
}
```
