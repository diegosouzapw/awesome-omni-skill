---
name: frontend-razor
description: Apply when working with Razor views, MVC layouts, partial views, and tag helpers
globs:
  - "**/*.cshtml"
  - "**/Views/**/*"
  - "**/Pages/**/*.cshtml"
  - "**/Shared/**/*.cshtml"
---

# Razor View Development Patterns

## View Structure

### Project Layout

```
Views/
├── Shared/
│   ├── _Layout.cshtml          # Main layout template
│   ├── _LayoutEmpty.cshtml     # Minimal layout (no header/footer)
│   ├── _Header.cshtml          # Header partial
│   ├── _Footer.cshtml          # Footer partial
│   ├── _Navigation.cshtml      # Navigation partial
│   ├── _Pagination.cshtml      # Reusable pagination
│   └── Components/             # View components
│       └── SearchBox/
│           └── Default.cshtml
├── Home/
│   ├── Index.cshtml
│   └── About.cshtml
├── Products/
│   ├── Index.cshtml
│   ├── Details.cshtml
│   └── _ProductCard.cshtml     # Page-specific partial
├── _ViewImports.cshtml         # Shared imports and tag helpers
└── _ViewStart.cshtml           # Default layout assignment
```

### _ViewImports.cshtml

```cshtml
@using MyApp.Web
@using MyApp.Web.Models
@using MyApp.Core.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, MyApp.Web
```

### _ViewStart.cshtml

```cshtml
@{
    Layout = "_Layout";
}
```

## Layout Templates

### Main Layout

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - My Application</title>
    
    @* Head section for page-specific styles *@
    @await RenderSectionAsync("Styles", required: false)
    
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body class="@ViewData["BodyClass"]">
    <header>
        @await Html.PartialAsync("_Header")
        @await Html.PartialAsync("_Navigation")
    </header>
    
    <main class="container">
        @RenderBody()
    </main>
    
    <footer>
        @await Html.PartialAsync("_Footer")
    </footer>
    
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    
    @* Scripts section for page-specific JavaScript *@
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### Page Using Layout

```cshtml
@model ProductListViewModel
@{
    ViewData["Title"] = "Products";
    ViewData["BodyClass"] = "products-page";
}

@section Styles {
    <link rel="stylesheet" href="~/css/products.css" asp-append-version="true" />
}

<h1>@ViewData["Title"]</h1>

<div class="product-grid">
    @foreach (var product in Model.Products)
    {
        @await Html.PartialAsync("_ProductCard", product)
    }
</div>

@section Scripts {
    <script src="~/js/products.js" asp-append-version="true"></script>
}
```

### Nested Layouts

```cshtml
@* Views/Shared/_LayoutAdmin.cshtml *@
@{
    Layout = "_Layout";
}

<div class="admin-container">
    <aside class="admin-sidebar">
        @await Html.PartialAsync("_AdminNav")
    </aside>
    <div class="admin-content">
        @RenderBody()
    </div>
</div>

@section Scripts {
    <script src="~/js/admin.js" asp-append-version="true"></script>
    @await RenderSectionAsync("AdminScripts", required: false)
}
```

## Partial Views

### Rendering Partials

```cshtml
@* Async (recommended) *@
@await Html.PartialAsync("_ProductCard", product)

@* With explicit view path *@
@await Html.PartialAsync("~/Views/Shared/_Header.cshtml")

@* Synchronous (avoid for I/O operations) *@
@Html.Partial("_Sidebar")

@* As tag helper *@
<partial name="_ProductCard" model="product" />

@* With view-data *@
<partial name="_Pagination" model="Model.Pagination" view-data="ViewData" />
```

### Partial View Example

```cshtml
@* Views/Products/_ProductCard.cshtml *@
@model Product

<article class="product-card">
    <a asp-action="Details" asp-route-id="@Model.Id" class="product-card__link">
        <img src="@Model.ImageUrl" 
             alt="@Model.Name" 
             class="product-card__image" />
        <div class="product-card__content">
            <h3 class="product-card__title">@Model.Name</h3>
            <p class="product-card__price">@Model.Price.ToString("C")</p>
            @if (Model.IsOnSale)
            {
                <span class="product-card__badge">Sale</span>
            }
        </div>
    </a>
</article>
```

## View Components

### Component Class

```csharp
// ViewComponents/RecentArticlesViewComponent.cs
public class RecentArticlesViewComponent : ViewComponent
{
    private readonly IArticleService _articleService;

    public RecentArticlesViewComponent(IArticleService articleService)
    {
        _articleService = articleService;
    }

    public async Task<IViewComponentResult> InvokeAsync(int count = 5)
    {
        var articles = await _articleService.GetRecentAsync(count);
        return View(articles);
    }
}
```

### Component View

```cshtml
@* Views/Shared/Components/RecentArticles/Default.cshtml *@
@model IEnumerable<Article>

<section class="recent-articles">
    <h3>Recent Articles</h3>
    <ul>
        @foreach (var article in Model)
        {
            <li>
                <a asp-controller="Articles" 
                   asp-action="Details" 
                   asp-route-slug="@article.Slug">
                    @article.Title
                </a>
                <time datetime="@article.PublishedDate.ToString("yyyy-MM-dd")">
                    @article.PublishedDate.ToString("MMM dd, yyyy")
                </time>
            </li>
        }
    </ul>
</section>
```

### Invoking Components

```cshtml
@* Tag helper syntax (preferred) *@
<vc:recent-articles count="5" />

@* Async syntax *@
@await Component.InvokeAsync("RecentArticles", new { count = 5 })

@* With cache *@
<cache expires-after="@TimeSpan.FromMinutes(10)">
    <vc:recent-articles count="5" />
</cache>
```

## Tag Helpers

### Built-in Tag Helpers

```cshtml
@* Anchor tag helper *@
<a asp-controller="Products" 
   asp-action="Details" 
   asp-route-id="@product.Id"
   asp-route-category="@product.Category">
    View Details
</a>

@* Form tag helpers *@
<form asp-controller="Contact" asp-action="Submit" method="post">
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
        <label asp-for="Category"></label>
        <select asp-for="Category" asp-items="Model.Categories" class="form-control">
            <option value="">Select a category</option>
        </select>
        <span asp-validation-for="Category" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Submit</button>
</form>

@* Image with cache busting *@
<img src="~/images/logo.png" asp-append-version="true" alt="Logo" />

@* Environment-specific rendering *@
<environment include="Development">
    <link rel="stylesheet" href="~/css/site.css" />
</environment>
<environment exclude="Development">
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
</environment>
```

### Custom Tag Helper

```csharp
// TagHelpers/EmailTagHelper.cs
[HtmlTargetElement("email")]
public class EmailTagHelper : TagHelper
{
    public string Address { get; set; }
    public string Subject { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        
        var href = $"mailto:{Address}";
        if (!string.IsNullOrEmpty(Subject))
        {
            href += $"?subject={Uri.EscapeDataString(Subject)}";
        }
        
        output.Attributes.SetAttribute("href", href);
        output.Content.SetContent(Address);
    }
}
```

```cshtml
@* Usage *@
<email address="support@example.com" subject="Help Request" />
```

## Model Binding

### Strongly-Typed Views

```cshtml
@model ContactFormViewModel

<form asp-action="Submit" method="post">
    @Html.AntiForgeryToken()
    
    <div class="form-group">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" placeholder="Your name" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Email" class="form-label"></label>
        <input asp-for="Email" class="form-control" type="email" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Message" class="form-label"></label>
        <textarea asp-for="Message" class="form-control" rows="5"></textarea>
        <span asp-validation-for="Message" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Send Message</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### ViewModel with Validation

```csharp
public class ContactFormViewModel
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, ErrorMessage = "Name cannot exceed 100 characters")]
    [Display(Name = "Full Name")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Please enter a valid email address")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Message is required")]
    [StringLength(2000, MinimumLength = 10, 
        ErrorMessage = "Message must be between 10 and 2000 characters")]
    public string Message { get; set; }
}
```

## Conditional Rendering

### If/Else Statements

```cshtml
@if (Model.Products.Any())
{
    <div class="product-grid">
        @foreach (var product in Model.Products)
        {
            <partial name="_ProductCard" model="product" />
        }
    </div>
}
else
{
    <div class="empty-state">
        <p>No products found.</p>
        <a asp-action="Index" asp-controller="Home" class="btn btn-link">
            Return to Home
        </a>
    </div>
}
```

### Switch Statements

```cshtml
@switch (Model.Status)
{
    case OrderStatus.Pending:
        <span class="badge badge-warning">Pending</span>
        break;
    case OrderStatus.Processing:
        <span class="badge badge-info">Processing</span>
        break;
    case OrderStatus.Shipped:
        <span class="badge badge-primary">Shipped</span>
        break;
    case OrderStatus.Delivered:
        <span class="badge badge-success">Delivered</span>
        break;
    default:
        <span class="badge badge-secondary">Unknown</span>
        break;
}
```

### Null Checking

```cshtml
@* Null conditional *@
<p>Author: @Model.Author?.Name ?? "Unknown"</p>

@* With fallback *@
@if (Model.ImageUrl != null)
{
    <img src="@Model.ImageUrl" alt="@Model.Title" />
}
else
{
    <img src="~/images/placeholder.jpg" alt="No image available" />
}

@* Ternary operator *@
<div class="@(Model.IsActive ? "active" : "inactive")">
    @Model.Title
</div>
```

## Loops and Collections

### For Loop

```cshtml
@for (int i = 0; i < Model.Items.Count; i++)
{
    <div class="item @(i % 2 == 0 ? "even" : "odd")">
        <span class="item-number">@(i + 1)</span>
        <span class="item-name">@Model.Items[i].Name</span>
    </div>
}
```

### Foreach with Index

```cshtml
@{ var index = 0; }
@foreach (var item in Model.Items)
{
    <div class="item" data-index="@index">
        @item.Name
    </div>
    index++;
}
```

### Rendering Lists

```cshtml
<ul class="breadcrumb">
    @foreach (var crumb in Model.Breadcrumbs)
    {
        var isLast = crumb == Model.Breadcrumbs.Last();
        <li class="breadcrumb-item @(isLast ? "active" : "")">
            @if (isLast)
            {
                @crumb.Title
            }
            else
            {
                <a href="@crumb.Url">@crumb.Title</a>
            }
        </li>
    }
</ul>
```

## HTML Helpers vs Tag Helpers

### Prefer Tag Helpers

```cshtml
@* HTML Helper (older approach) *@
@Html.ActionLink("Details", "Details", "Products", new { id = product.Id }, new { @class = "btn btn-link" })

@* Tag Helper (modern, preferred) *@
<a asp-controller="Products" asp-action="Details" asp-route-id="@product.Id" class="btn btn-link">
    Details
</a>

@* HTML Helper for form *@
@Html.TextBoxFor(m => m.Name, new { @class = "form-control", placeholder = "Enter name" })

@* Tag Helper (cleaner) *@
<input asp-for="Name" class="form-control" placeholder="Enter name" />
```

### When to Use HTML Helpers

```cshtml
@* Complex dynamic attributes *@
@Html.TextBoxFor(m => m.Name, Model.GetInputAttributes())

@* Raw HTML content *@
@Html.Raw(Model.HtmlContent)

@* Display templates *@
@Html.DisplayFor(m => m.CreatedDate)

@* Editor templates *@
@Html.EditorFor(m => m.Address)
```

## ViewData, ViewBag, and TempData

### ViewData (Dictionary)

```cshtml
@* In Controller *@
ViewData["Title"] = "Product Details";
ViewData["ShowSidebar"] = true;

@* In View *@
<h1>@ViewData["Title"]</h1>
@if ((bool?)ViewData["ShowSidebar"] == true)
{
    <partial name="_Sidebar" />
}
```

### ViewBag (Dynamic)

```cshtml
@* In Controller *@
ViewBag.Categories = await _categoryService.GetAllAsync();

@* In View *@
<select asp-items="@(new SelectList(ViewBag.Categories, "Id", "Name"))">
    <option value="">All Categories</option>
</select>
```

### TempData (Survives Redirect)

```csharp
// In Controller
TempData["SuccessMessage"] = "Product saved successfully!";
return RedirectToAction("Index");
```

```cshtml
@* In View *@
@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success alert-dismissible">
        @TempData["SuccessMessage"]
        <button type="button" class="close" data-dismiss="alert">&times;</button>
    </div>
}
```

## AJAX and Partial Rendering

### AJAX Form Submission

```cshtml
<form id="contact-form" asp-action="SubmitAjax" method="post">
    @Html.AntiForgeryToken()
    
    <div class="form-group">
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Submit</button>
</form>

<div id="result"></div>

@section Scripts {
    <script>
    $('#contact-form').on('submit', function(e) {
        e.preventDefault();
        
        var $form = $(this);
        var $button = $form.find('button[type="submit"]');
        
        $button.prop('disabled', true).text('Sending...');
        
        $.ajax({
            url: $form.attr('action'),
            type: 'POST',
            data: $form.serialize(),
            success: function(response) {
                $('#result').html(response);
                $form[0].reset();
            },
            error: function(xhr) {
                $('#result').html('<div class="alert alert-danger">An error occurred.</div>');
            },
            complete: function() {
                $button.prop('disabled', false).text('Submit');
            }
        });
    });
    </script>
}
```

### Returning Partial Views from Controller

```csharp
[HttpGet]
public async Task<IActionResult> LoadMore(int page)
{
    var products = await _productService.GetPagedAsync(page, 10);
    return PartialView("_ProductGrid", products);
}

[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> SubmitAjax(ContactFormViewModel model)
{
    if (!ModelState.IsValid)
    {
        return PartialView("_ContactFormErrors", model);
    }
    
    await _contactService.ProcessAsync(model);
    return PartialView("_ContactFormSuccess");
}
```

## Accessibility Considerations

```cshtml
@* Semantic HTML with ARIA *@
<nav aria-label="Main navigation">
    <ul class="nav" role="menubar">
        @foreach (var item in Model.NavItems)
        {
            <li role="none">
                <a href="@item.Url" 
                   role="menuitem"
                   aria-current="@(item.IsActive ? "page" : null)">
                    @item.Title
                </a>
            </li>
        }
    </ul>
</nav>

@* Form accessibility *@
<div class="form-group">
    <label asp-for="Email" id="email-label"></label>
    <input asp-for="Email" 
           aria-labelledby="email-label" 
           aria-describedby="email-help email-error" />
    <small id="email-help" class="form-text text-muted">
        We'll never share your email.
    </small>
    <span asp-validation-for="Email" id="email-error" class="text-danger" role="alert"></span>
</div>

@* Skip link for keyboard navigation *@
<a href="#main-content" class="skip-link">Skip to main content</a>
```
