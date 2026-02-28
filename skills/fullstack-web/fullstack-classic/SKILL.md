---
name: fullstack-classic
description: Apply when working with classic fullstack patterns including jQuery AJAX, form handling, and C# MVC integration
globs:
  - "**/Controllers/**/*.cs"
  - "**/Views/**/*.cshtml"
  - "**/scripts/**/*.js"
  - "**/js/**/*.js"
---

# Classic Fullstack Integration Patterns

## Form Handling

### Server-Side Form Processing

```csharp
// Controller
public class ContactController : Controller
{
    private readonly IContactService _contactService;
    private readonly ILogger<ContactController> _logger;

    public ContactController(
        IContactService contactService,
        ILogger<ContactController> logger)
    {
        _contactService = contactService;
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Index()
    {
        return View(new ContactFormModel());
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Index(ContactFormModel model, CancellationToken ct)
    {
        if (!ModelState.IsValid)
        {
            return View(model);
        }

        try
        {
            await _contactService.ProcessContactAsync(model, ct);
            TempData["SuccessMessage"] = "Thank you for your message!";
            return RedirectToAction(nameof(Index));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process contact form");
            ModelState.AddModelError("", "An error occurred. Please try again.");
            return View(model);
        }
    }
}
```

### Razor Form

```cshtml
@model ContactFormModel

@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success">
        @TempData["SuccessMessage"]
    </div>
}

<form asp-action="Index" method="post">
    @Html.AntiForgeryToken()
    
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>
    
    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" type="email" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Message"></label>
        <textarea asp-for="Message" class="form-control" rows="5"></textarea>
        <span asp-validation-for="Message" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Send Message</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

## jQuery AJAX Integration

### AJAX Form Submission

```javascript
// JavaScript
$(document).ready(function() {
    $('#contact-form').on('submit', function(e) {
        e.preventDefault();
        
        var $form = $(this);
        var $submitBtn = $form.find('button[type="submit"]');
        var $result = $('#form-result');
        
        // Disable button and show loading state
        $submitBtn.prop('disabled', true).text('Sending...');
        $result.empty();
        
        $.ajax({
            url: $form.attr('action'),
            type: 'POST',
            data: $form.serialize(),
            success: function(response) {
                if (response.success) {
                    $result.html('<div class="alert alert-success">' + response.message + '</div>');
                    $form[0].reset();
                } else {
                    showValidationErrors(response.errors);
                }
            },
            error: function(xhr, status, error) {
                $result.html('<div class="alert alert-danger">An error occurred. Please try again.</div>');
                console.error('Form submission failed:', error);
            },
            complete: function() {
                $submitBtn.prop('disabled', false).text('Send Message');
            }
        });
    });
    
    function showValidationErrors(errors) {
        // Clear previous errors
        $('.field-validation-error').text('');
        $('.input-validation-error').removeClass('input-validation-error');
        
        // Show new errors
        $.each(errors, function(field, messages) {
            var $field = $('[name="' + field + '"]');
            $field.addClass('input-validation-error');
            $field.siblings('.field-validation-error').text(messages.join(', '));
        });
    }
});
```

### AJAX Controller Action

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> SubmitAjax(ContactFormModel model, CancellationToken ct)
{
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value.Errors.Count > 0)
            .ToDictionary(
                kvp => kvp.Key,
                kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
            );
            
        return Json(new { success = false, errors });
    }

    try
    {
        await _contactService.ProcessContactAsync(model, ct);
        return Json(new { success = true, message = "Thank you for your message!" });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to process contact form");
        return Json(new { 
            success = false, 
            errors = new { General = new[] { "An error occurred. Please try again." } }
        });
    }
}
```

## Anti-Forgery Token Handling

### Include Token in AJAX Requests

```javascript
// Setup for all AJAX requests
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (settings.type === 'POST' || settings.type === 'PUT' || settings.type === 'DELETE') {
            var token = $('input[name="__RequestVerificationToken"]').val();
            if (token) {
                xhr.setRequestHeader('RequestVerificationToken', token);
            }
        }
    }
});

// Or include in data for form-encoded requests
$.ajax({
    url: '/api/items',
    type: 'POST',
    data: {
        __RequestVerificationToken: $('input[name="__RequestVerificationToken"]').val(),
        name: 'New Item'
    }
});
```

### Token in Layout

```cshtml
@* Add to _Layout.cshtml for global availability *@
<form id="__AjaxAntiForgeryForm" action="#" method="post">
    @Html.AntiForgeryToken()
</form>
```

```javascript
// Get token from hidden form
function getAntiForgeryToken() {
    return $('#__AjaxAntiForgeryForm input[name="__RequestVerificationToken"]').val();
}
```

## Loading Content Dynamically

### Partial View Loading

```csharp
// Controller
[HttpGet]
public async Task<IActionResult> LoadProducts(
    int page = 1, 
    string category = null,
    CancellationToken ct = default)
{
    var products = await _productService.GetPagedAsync(page, 12, category, ct);
    return PartialView("_ProductGrid", products);
}

[HttpGet]
public async Task<IActionResult> ProductDetails(Guid id, CancellationToken ct)
{
    var product = await _productService.GetByIdAsync(id, ct);
    if (product == null)
    {
        return NotFound();
    }
    return PartialView("_ProductDetails", product);
}
```

```javascript
// Load more products
$('#load-more').on('click', function() {
    var $btn = $(this);
    var page = parseInt($btn.data('page')) + 1;
    var category = $btn.data('category');
    
    $btn.prop('disabled', true).text('Loading...');
    
    $.get('/Products/LoadProducts', { page: page, category: category })
        .done(function(html) {
            $('#product-grid').append(html);
            $btn.data('page', page);
        })
        .fail(function() {
            alert('Failed to load products');
        })
        .always(function() {
            $btn.prop('disabled', false).text('Load More');
        });
});

// Load product details in modal
$(document).on('click', '[data-product-details]', function(e) {
    e.preventDefault();
    var productId = $(this).data('product-details');
    
    $.get('/Products/ProductDetails/' + productId)
        .done(function(html) {
            $('#modal-content').html(html);
            $('#product-modal').modal('show');
        })
        .fail(function() {
            alert('Failed to load product details');
        });
});
```

## Search with Debounce

### JavaScript

```javascript
var MYAPP = MYAPP || {};

MYAPP.search = (function($) {
    var debounceTimer;
    var $input;
    var $results;
    var minChars = 3;
    var debounceDelay = 300;
    
    function init() {
        $input = $('#search-input');
        $results = $('#search-results');
        
        $input.on('keyup', function() {
            var query = $(this).val().trim();
            
            clearTimeout(debounceTimer);
            
            if (query.length < minChars) {
                $results.empty().hide();
                return;
            }
            
            debounceTimer = setTimeout(function() {
                performSearch(query);
            }, debounceDelay);
        });
        
        // Close results when clicking outside
        $(document).on('click', function(e) {
            if (!$(e.target).closest('.search-container').length) {
                $results.hide();
            }
        });
    }
    
    function performSearch(query) {
        $results.html('<div class="search-loading">Searching...</div>').show();
        
        $.get('/Search/Results', { q: query })
            .done(function(html) {
                $results.html(html).show();
            })
            .fail(function() {
                $results.html('<div class="search-error">Search failed</div>');
            });
    }
    
    return { init: init };
})(jQuery);

$(document).ready(function() {
    MYAPP.search.init();
});
```

### Controller

```csharp
[HttpGet]
public async Task<IActionResult> Results(string q, CancellationToken ct)
{
    if (string.IsNullOrWhiteSpace(q) || q.Length < 3)
    {
        return PartialView("_NoResults");
    }

    var results = await _searchService.SearchAsync(q, maxResults: 10, ct);
    return PartialView("_SearchResults", results);
}
```

## Pagination

### Controller

```csharp
[HttpGet]
public async Task<IActionResult> Index(int page = 1, CancellationToken ct = default)
{
    const int pageSize = 12;
    var result = await _productService.GetPagedAsync(page, pageSize, ct);
    
    ViewBag.CurrentPage = page;
    ViewBag.TotalPages = result.TotalPages;
    ViewBag.HasPrevious = page > 1;
    ViewBag.HasNext = page < result.TotalPages;
    
    return View(result.Items);
}
```

### Razor Partial

```cshtml
@* _Pagination.cshtml *@
@{
    var currentPage = (int)ViewBag.CurrentPage;
    var totalPages = (int)ViewBag.TotalPages;
    var hasPrevious = (bool)ViewBag.HasPrevious;
    var hasNext = (bool)ViewBag.HasNext;
}

@if (totalPages > 1)
{
    <nav aria-label="Page navigation">
        <ul class="pagination">
            <li class="page-item @(!hasPrevious ? "disabled" : "")">
                <a class="page-link" 
                   asp-action="Index" 
                   asp-route-page="@(currentPage - 1)"
                   aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            </li>
            
            @for (int i = 1; i <= totalPages; i++)
            {
                <li class="page-item @(i == currentPage ? "active" : "")">
                    <a class="page-link" asp-action="Index" asp-route-page="@i">@i</a>
                </li>
            }
            
            <li class="page-item @(!hasNext ? "disabled" : "")">
                <a class="page-link" 
                   asp-action="Index" 
                   asp-route-page="@(currentPage + 1)"
                   aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
        </ul>
    </nav>
}
```

### AJAX Pagination

```javascript
$(document).on('click', '.pagination a', function(e) {
    e.preventDefault();
    
    var url = $(this).attr('href');
    
    $.get(url)
        .done(function(html) {
            $('#content-container').html(html);
            // Update browser URL without reload
            history.pushState(null, '', url);
        })
        .fail(function() {
            alert('Failed to load page');
        });
});

// Handle browser back/forward
$(window).on('popstate', function() {
    $.get(location.href)
        .done(function(html) {
            $('#content-container').html(html);
        });
});
```

## File Upload

### Razor Form

```cshtml
<form asp-action="Upload" method="post" enctype="multipart/form-data">
    @Html.AntiForgeryToken()
    
    <div class="form-group">
        <label for="file">Select file</label>
        <input type="file" name="file" id="file" class="form-control-file" accept=".jpg,.png,.pdf" />
        <small class="form-text text-muted">Max size: 5MB. Allowed: JPG, PNG, PDF</small>
    </div>
    
    <button type="submit" class="btn btn-primary">Upload</button>
</form>
```

### Controller

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Upload(IFormFile file, CancellationToken ct)
{
    if (file == null || file.Length == 0)
    {
        ModelState.AddModelError("file", "Please select a file");
        return View();
    }

    if (file.Length > 5 * 1024 * 1024) // 5MB
    {
        ModelState.AddModelError("file", "File size cannot exceed 5MB");
        return View();
    }

    var allowedExtensions = new[] { ".jpg", ".png", ".pdf" };
    var extension = Path.GetExtension(file.FileName).ToLowerInvariant();
    if (!allowedExtensions.Contains(extension))
    {
        ModelState.AddModelError("file", "Invalid file type");
        return View();
    }

    var fileName = $"{Guid.NewGuid()}{extension}";
    var filePath = Path.Combine(_uploadPath, fileName);

    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await file.CopyToAsync(stream, ct);
    }

    TempData["SuccessMessage"] = "File uploaded successfully";
    return RedirectToAction(nameof(Index));
}
```

### AJAX File Upload

```javascript
$('#upload-form').on('submit', function(e) {
    e.preventDefault();
    
    var formData = new FormData(this);
    var $progress = $('#upload-progress');
    var $progressBar = $progress.find('.progress-bar');
    
    $progress.show();
    
    $.ajax({
        url: $(this).attr('action'),
        type: 'POST',
        data: formData,
        processData: false,
        contentType: false,
        xhr: function() {
            var xhr = new window.XMLHttpRequest();
            xhr.upload.addEventListener('progress', function(e) {
                if (e.lengthComputable) {
                    var percent = Math.round((e.loaded / e.total) * 100);
                    $progressBar.css('width', percent + '%').text(percent + '%');
                }
            });
            return xhr;
        },
        success: function(response) {
            if (response.success) {
                showSuccess('File uploaded successfully');
                $('#file-input').val('');
            } else {
                showError(response.message);
            }
        },
        error: function() {
            showError('Upload failed');
        },
        complete: function() {
            setTimeout(function() {
                $progress.hide();
                $progressBar.css('width', '0%').text('');
            }, 1000);
        }
    });
});
```

## Error Handling

### Global AJAX Error Handler

```javascript
$(document).ajaxError(function(event, xhr, settings, error) {
    if (xhr.status === 401) {
        // Redirect to login
        window.location.href = '/Account/Login?returnUrl=' + encodeURIComponent(window.location.pathname);
        return;
    }
    
    if (xhr.status === 403) {
        showError('You do not have permission to perform this action');
        return;
    }
    
    if (xhr.status === 404) {
        showError('The requested resource was not found');
        return;
    }
    
    if (xhr.status >= 500) {
        showError('A server error occurred. Please try again later.');
        return;
    }
    
    console.error('AJAX error:', settings.url, error);
});
```

### Display Errors from Server

```csharp
// Controller returning JSON errors
[HttpPost]
public IActionResult Process(ProcessRequest request)
{
    try
    {
        // Process...
        return Json(new { success = true });
    }
    catch (ValidationException ex)
    {
        return BadRequest(new { 
            success = false, 
            errors = ex.Errors 
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Processing failed");
        return StatusCode(500, new { 
            success = false, 
            message = "An unexpected error occurred" 
        });
    }
}
```

```javascript
$.ajax({
    url: '/api/process',
    type: 'POST',
    data: formData,
    success: function(response) {
        if (response.success) {
            showSuccess('Operation completed');
        }
    },
    error: function(xhr) {
        if (xhr.responseJSON) {
            if (xhr.responseJSON.errors) {
                displayFieldErrors(xhr.responseJSON.errors);
            } else if (xhr.responseJSON.message) {
                showError(xhr.responseJSON.message);
            }
        } else {
            showError('An error occurred');
        }
    }
});
```

## Session and State Management

### Server-Side Session

```csharp
// Store in session
HttpContext.Session.SetString("UserPreference", "dark");
HttpContext.Session.SetInt32("CartCount", 5);

// Complex objects
HttpContext.Session.SetString("Cart", JsonSerializer.Serialize(cart));

// Retrieve from session
var preference = HttpContext.Session.GetString("UserPreference");
var cartCount = HttpContext.Session.GetInt32("CartCount");
var cart = JsonSerializer.Deserialize<Cart>(HttpContext.Session.GetString("Cart"));
```

### Client-Side with Cookies

```javascript
// Set cookie
function setCookie(name, value, days) {
    var expires = '';
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        expires = '; expires=' + date.toUTCString();
    }
    document.cookie = name + '=' + encodeURIComponent(value) + expires + '; path=/';
}

// Get cookie
function getCookie(name) {
    var nameEQ = name + '=';
    var cookies = document.cookie.split(';');
    for (var i = 0; i < cookies.length; i++) {
        var cookie = cookies[i].trim();
        if (cookie.indexOf(nameEQ) === 0) {
            return decodeURIComponent(cookie.substring(nameEQ.length));
        }
    }
    return null;
}

// Delete cookie
function deleteCookie(name) {
    setCookie(name, '', -1);
}
```

## Notification/Toast Messages

### JavaScript Toast System

```javascript
var MYAPP = MYAPP || {};

MYAPP.toast = (function($) {
    var $container;
    
    function init() {
        $container = $('<div id="toast-container"></div>').appendTo('body');
    }
    
    function show(message, type, duration) {
        type = type || 'info';
        duration = duration || 5000;
        
        var $toast = $('<div class="toast toast-' + type + '">' + 
            '<span class="toast-message">' + message + '</span>' +
            '<button class="toast-close">&times;</button>' +
            '</div>');
        
        $container.append($toast);
        
        setTimeout(function() {
            $toast.addClass('show');
        }, 10);
        
        var timer = setTimeout(function() {
            remove($toast);
        }, duration);
        
        $toast.find('.toast-close').on('click', function() {
            clearTimeout(timer);
            remove($toast);
        });
    }
    
    function remove($toast) {
        $toast.removeClass('show');
        setTimeout(function() {
            $toast.remove();
        }, 300);
    }
    
    function success(message) { show(message, 'success'); }
    function error(message) { show(message, 'error'); }
    function warning(message) { show(message, 'warning'); }
    function info(message) { show(message, 'info'); }
    
    return {
        init: init,
        show: show,
        success: success,
        error: error,
        warning: warning,
        info: info
    };
})(jQuery);

$(document).ready(function() {
    MYAPP.toast.init();
});
```

### CSS for Toasts

```scss
#toast-container {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 9999;
}

.toast {
    min-width: 300px;
    padding: 15px 40px 15px 15px;
    margin-bottom: 10px;
    border-radius: 4px;
    opacity: 0;
    transform: translateX(100%);
    transition: all 0.3s ease;
    
    &.show {
        opacity: 1;
        transform: translateX(0);
    }
    
    &-success { background: #28a745; color: white; }
    &-error { background: #dc3545; color: white; }
    &-warning { background: #ffc107; color: #333; }
    &-info { background: #17a2b8; color: white; }
    
    &-close {
        position: absolute;
        top: 10px;
        right: 10px;
        background: none;
        border: none;
        color: inherit;
        font-size: 20px;
        cursor: pointer;
        opacity: 0.7;
        
        &:hover { opacity: 1; }
    }
}
```
