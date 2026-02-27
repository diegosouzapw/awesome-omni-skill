---
name: tech-specification
description: The master skill for generating technical specifications. Identifies project technology and delegates deep analysis to specialized sub-skills.
---

# Technical Specification Generator

## Instructions
You are the entry point for generating technical documentation. Your goal is to **Identify** the technology stack and **Delegate** to the specific sub-skill for deep analysis.

## Supported Stacks & Strategies

### Node.js
Focus on analyzing `package.json` for core dependencies and scripts. You must drill down into the entry point (e.g., `app.js`) to identify the architectural pattern (Monolithic vs. MVC vs. Clean Architecture). Pay special attention to global middleware chains and how the database connection is established.
*Please refer to [node_docbook.md](node_docbook.md) for better deep understanding.*

### Python
Inspect `pyproject.toml` or `requirements.txt` to identify the framework (Django, FastAPI, or Flask). Trace the import graph from the entry point to understand the project structure. Critically analyze the usage of Async IO vs. Synchronous execution, and identify the ORM strategy (SQLAlchemy, Django ORM).
*Please refer to [python_docbook.md](python_docbook.md) for better deep understanding.*

### PHP
Check `composer.json` to determine if the project uses Laravel, Symfony, or is a custom build. Analyze the `index.php` or `artisan` console runner to understand the request lifecycle. Look for specific architectural signatures like Controllers, Service Providers, and Hook systems in WordPress.
*Please refer to [php_docbook.md](php_docbook.md) for better deep understanding.*

### Go (Golang)
Examine `go.mod` for dependencies and version. Analyze the package layout—distinguish between "Standard Layout" (`cmd/`, `pkg/`, `internal/`) and Flat layouts. Pay close attention to interface definitions and concurrency patterns using Channels and Goroutines.
*Please refer to [go_docbook.md](go_docbook.md) for better deep understanding.*

### Java / Kotlin
Identify the build tool (`pom.xml` for Maven or `build.gradle` for Gradle) and framework (usually Spring Boot). Analyze Class Annotations to map the Dependency Injection graph and separate Controllers from Services and Repositories. Check for Domain-Driven Design patterns.
*Please refer to [java_docbook.md](java_docbook.md) for better deep understanding.*

### .NET
Read the `.sln` and `.csproj` files to understand the solution structure and Target Framework. Analyze `Program.cs` or `Startup.cs` to map the Dependency Injection settings. Determine if the project follows Clean Architecture, Vertical Slice, or CQS/CQRS patterns using libraries like MediatR.
*Please refer to [dotnet_docbook.md](dotnet_docbook.md) for better deep understanding.*

### Rust
Check `Cargo.toml` to identify if the project is a Binary or Library crate. Analyze the module system and `mod.rs` structure. Crucially, determine the Async Runtime (like Tokio) and web framework (Axum, Actix) usage, along with error handling patterns.
*Please refer to [rust_docbook.md](rust_docbook.md) for better deep understanding.*

### React (Web)
Look beyond the component folder structure; analyze the Component Composition pattern (Hooks vs Classes). functionality. key areas to document include Client-side vs Server-side data fetching approaches, Global State management (Redux, Context), and the Routing library implementation.
*Please refer to [react_docbook.md](react_docbook.md) for better deep understanding.*

### React Native (Mobile)
Focus on the Navigation structure (React Navigation stacks) and the bridge to Native Modules. Identify if the project uses Expo or the bare CLI. Document the State Management strategy and any specific native permissions required (Camera, Location, etc.).
*Please refer to [react_native_docbook.md](react_native_docbook.md) for better deep understanding.*

### Vue.js
Determine if the project uses Vue 2 (Options API) or Vue 3 (Composition API & `<script setup>`). If Nuxt is present, analyze the file-system routing and server routes. Check for State Management using Pinia (modern) or Vuex (legacy).
*Please refer to [vue_docbook.md](vue_docbook.md) for better deep understanding.*

### NestJS
Trace the Dependency Injection graph starting from the root `AppModule`. Identify Modules, Controllers, and Providers. Specifically look for decorators to understand the API surface and Transport layers (HTTP vs Microservices) and Guard/Interceptor usage.
*Please refer to [nest_docbook.md](nest_docbook.md) for better deep understanding.*

### Next.js
Distinguish between the App Router (`app/` directory) and Pages Router. Analyze the Data Fetching strategy (SSR, SSG, ISR) and the usage of Server Components ("use client" directives). details on Middleware and API Routes are critical.
*Please refer to [next_docbook.md](next_docbook.md) for better deep understanding.*

## References
For advanced usage and stack detection rules, see [reference.md](reference.md).
