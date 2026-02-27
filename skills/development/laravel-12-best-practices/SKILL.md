---
name: laravel-12-best-practices
description: Software engineering best practices for Laravel 12.x, covering architecture, Eloquent, testing, security, and the new starter kits.
compatibility: Requires Laravel 12.x and PHP 8.2+
license: Apache-2.0
---

# Laravel 12.x Best Practices

This skill outlines the recommended software engineering practices for developing applications with Laravel 12.x. It incorporates modern PHP 8.2+ features, the latest Laravel 12 enhancements (such as the new starter kits and UUIDv7 defaults), and industry-standard architecture patterns.

## When to Use

- **New or existing Laravel 12.x apps:** Apply to greenfield or existing apps, with or without the official starter kits, including setups that still adopt Inertia 2.0 or UUIDv7 defaults.
- **Architecture decisions:** Apply when choosing between standard MVC and domain-driven structure for complex domains.
- **Quality gates:** Apply when establishing linting, static analysis, and refactoring automation (Pint, PHPStan, Rector).
- **Eloquent & data access:** Apply when designing models, resource transformations, and query strictness to avoid N+1 issues.
- **Testing strategy:** Apply when selecting Pest or PHPUnit and standardizing factories and JSON assertions.
- **Security posture:** Apply when enforcing FormRequest validation, policy authorization, and safe query practices.

## 1. Project Setup & Architecture

### Starter Kits
For new projects, prefer using the official Laravel 12 starter kits which now feature improved defaults:
- **Frontend:** Use the React, Vue, or Livewire starter kits which now integrate **Inertia 2.0**, **TypeScript**, **Shadcn/UI**, and **Tailwind CSS** out of the box.
- **Authentication:** Consider the **WorkOS AuthKit** variant included in starter kits for robust social auth, passkeys, and SSO support if building SaaS or enterprise apps.

### Directory Structure
- **Standard MVC:** Follow the default `app/` structure for small-to-medium applications.
- **Domain Driven Design (DDD):** For complex applications, organize code by domain (e.g., `src/Domains/Order`, `src/Domains/User`) rather than by technical layer.
- **Strict Typing:** Enable strict types in all PHP files (`declare(strict_types=1);`).

## 2. Code Quality & Style

- **Linting:** Use **Laravel Pint** (built on top of PHP-CS-Fixer) to enforce the PER Coding Style.
- **Static Analysis:** Maintain a high level of code safety using **PHPStan** (Level 5 minimum recommended).
- **Refactoring:** Utilize the `rector/rector` package to automatically upgrade older PHP syntax to modern PHP 8.2+ standards.

## 3. Eloquent & Database

### Model Practices
- **UUIDs:** Leverage Laravel 12's default support for **UUIDv7** in the `HasUuids` trait for time-sortable unique identifiers.
- **Resources:** Use the new fluent methods for transforming models into API resources:
    ```php
    // Preferred
    return User::find(1)->toResource();
    return User::query()->paginate()->toResourceCollection();
    ```
- **Mass Assignment:** Always use `$fillable` or `$guarded` properties to prevent mass assignment vulnerabilities.
- **Strictness:** Prevent "lazy loading" in development to avoid N+1 query performance issues.
    ```php
    // In AppServiceProvider::boot()
    Model::preventLazyLoading(! $this->app->isProduction());
    ```

### Data Handling
- **Collections:** Use strict collection methods like Arr::sole() when you expect exactly one result.
- **Context:** Use the `Context` facade for handling request-scoped data (logging, tracing) instead of global state.

## 4. Testing

### Frameworks
- **Pest PHP:** Preferred for its expressive syntax and minimal boilerplate.
- **PHPUnit 12:** If using PHPUnit, ensure you are utilizing v12 features supported by Laravel 12.

### Best Practices
- **Factories:** Always use Model Factories for test data generation. Avoid manually inserting rows.
- **Assertions:** Use fluent JSON assertions for API testing:
    ```php
    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('data')
             ->where('data.id', 1)
             ->etc()
    );
    ```
- **Queue Faking:** Utilize the enhanced Queue::fake() assertions to verify job dispatch logic without running the actual background processes.

## 5. Security

- **Validation:** Put all validation logic in FormRequest classes, not Controllers.
- **Authorization:** Use Policies and Gates strictly. Ensure every controller action authorizes the user action (e.g., $this->authorize('update', $post)).
- **Sanitization:** Rely on Blade's {{ }} auto-escaping and Eloquent's parameter binding. Never pass user input directly to raw SQL queries.

## 6. Performance

- **Caching:** Utilize Cache::remember for expensive queries.
- **HTTP Client:** Use the afterResponse() hook in the HTTP client to handle cross-cutting concerns (logging, error mapping) for outgoing API requests.
- **Queues:** Offload heavy tasks (emails, report generation) to the queue system. Monitor queues using Laravel Horizon.

## 7. Dependency Management

- **Updates:** Laravel 12 is a "maintenance release" focusing on stability. Keep dependencies updated regularly using composer update.
- **Breaking Changes:** Be aware that while Laravel 12 has zero breaking changes in the framework code, upstream dependencies (like Carbon 3) may have subtle differences.
