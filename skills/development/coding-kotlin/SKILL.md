---
name: coding-kotlin
cluster: coding
description: "Kotlin: coroutines, data classes, sealed classes, extension functions, Gradle, KMM multiplatform"
tags: ["kotlin","android","coding"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "kotlin android coroutine sealed data class gradle kmm"
---

# coding-kotlin

## Purpose
This skill equips the AI to generate, debug, and optimize Kotlin code, focusing on coroutines for async operations, data classes for immutable data handling, sealed classes for type-safe hierarchies, extension functions for reusable code, Gradle for project builds, and KMM for multiplatform development.

## When to Use
Use this skill for Android app development requiring async tasks (e.g., network calls), data modeling in APIs, or cross-platform projects with shared logic. Apply it when performance matters, like using coroutines to avoid blocking threads, or when setting up Gradle for dependency management in KMM setups.

## Key Capabilities
- Coroutines: Handle asynchronous code with structured concurrency; use `kotlinx.coroutines` for non-blocking I/O.
- Data classes: Automatically generate equals, hashCode, and toString; define with `data class User(val name: String, val id: Int)`.
- Sealed classes: Restrict class inheritance for exhaustive when expressions; e.g., `sealed class Result { data class Success(val data: String) : Result() }`.
- Extension functions: Add methods to existing classes without inheritance; e.g., `fun String.reverse() = this.reversed()`.
- Gradle: Manage builds with scripts like build.gradle.kts for dependencies and tasks.
- KMM: Share code between Android/iOS; configure with shared modules in Gradle.

## Usage Patterns
To accomplish tasks, always import necessary packages first (e.g., `import kotlinx.coroutines.*`). For async operations, wrap code in a coroutine scope like `runBlocking { }` and use `launch` for fire-and-forget or `async` for awaited results. When modeling data, use data classes for POJOs and sealed classes for state machines. For Gradle tasks, run commands from the project root and pass flags like `--info` for debugging. In KMM projects, structure code into shared and platform-specific modules, then link via Gradle configurations. Always check for nullability with `?` to prevent crashes.

## Common Commands/API
- Gradle commands: Run `gradle build --stacktrace` to compile and debug; use `gradle assembleDebug` for Android builds; in KMM, add dependencies in build.gradle.kts like `implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4")`.
- Coroutines API: Use `launch { delay(1000L); println("Task completed") }` for delayed execution; or `val deferred = async { someFunction() }; deferred.await()` for concurrent tasks.
- Data classes: Define and use as `data class Person(val name: String); val p = Person("John"); println(p.copy(name = "Jane"))` to create copies with modifications.
- Sealed classes: Handle with `when (result) { is Result.Success -> process(data); is Result.Error -> handleError() }` for exhaustive checking.
- Extension functions: Implement as `fun Int.isEven() = this % 2 == 0; val num = 4; println(num.isEven())` to extend built-ins.
- KMM setup: In build.gradle.kts, add `kotlin { android { compilations.all { kotlinOptions.jvmTarget = "1.8" } } ios { binaries { framework() } } }` for multiplatform configuration.
- API endpoints: For external services in Kotlin, use libraries like Ktor; e.g., `HttpClient().get("https://api.example.com/data")` with auth via env var like `$KOTLIN_API_KEY` in headers.

## Integration Notes
Integrate this skill by ensuring the environment has Kotlin SDK installed (e.g., via SDKMAN: `sdk install kotlin`). For Android, add Kotlin plugins in build.gradle: `plugins { id("org.jetbrains.kotlin.android") }` and set minSdk to 21 for coroutines. In KMM projects, link shared modules with `expect fun platformFunction()` in common code and `actual fun platformFunction() = ...` in platform-specific files. Use environment variables for secrets, like setting `$GRADLE_API_KEY` for repository access in gradle.properties: `apiKey=$GRADLE_API_KEY`. Test integrations with `./gradlew test` and handle platform differences by checking OS via `System.getProperty("os.name")`.

## Error Handling
Always use structured error handling in coroutines: wrap code in `try { launch { riskyOperation() } } catch (e: CancellationException) { log("Coroutine cancelled") } finally { cleanup() }`. For data classes, validate inputs at construction: e.g., `data class User(val name: String) { init { require(name.isNotBlank()) { "Name cannot be blank" } } }`. In sealed classes, ensure all subclasses are handled in when expressions to avoid `UnreachableCode` errors. For Gradle, parse output with `--stacktrace` and check for common issues like dependency conflicts by running `gradle dependencies`. In KMM, handle platform-specific errors with `expect class PlatformException() { actual constructor() }` and catch in common code.

## Concrete Usage Examples
1. **Example: Asynchronous network call with coroutines**  
   Import coroutines and launch a scope: `import kotlinx.coroutines.*; suspend fun fetchData() = "Data"; fun main() = runBlocking { val result = async { fetchData() }.await(); println(result) }`. This fetches data without blocking the main thread.
   
2. **Example: Modeling API response with sealed classes and data classes**  
   Define structures: `sealed class ApiResult { data class Success(val data: String) : ApiResult(); data class Error(val message: String) : ApiResult() }; fun process(result: ApiResult) = when (result) { is Success -> println(result.data); is Error -> println(result.message) }`. Use this to handle responses safely in KMM shared code.

## Graph Relationships
- Related to: coding-android (shares tags like "android" and focuses on Kotlin integration).
- Connected via: coding cluster (links to other coding skills like coding-java for JVM interoperability).
- Overlaps with: tools-kmm (via KMM multiplatform features) and build-gradle (for shared Gradle usage).
