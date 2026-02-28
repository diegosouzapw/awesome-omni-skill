---
name: swift-standards
description: "MANDATORY for ALL Swift output - files AND conversational snippets. Covers SPM, strict concurrency, Result types, enums with associated values, let over var, async/await, SwiftUI. Trigger: any Swift code, iOS, macOS, Package.swift, Xcode projects. No exceptions."
---

# Swift Best Practices

## When to Use This Skill

This skill should be triggered when:

- Writing or reviewing Swift code
- Setting up Swift packages or Xcode projects
- Working with iOS, macOS, watchOS, tvOS, or visionOS
- Discussing Swift patterns, concurrency, or architecture
- Configuring Package.swift or build settings

## Core Capabilities

1. **Package Management**: Swift Package Manager (SPM) exclusively
2. **Type Safety**: Strict concurrency, avoid Any, leverage generics
3. **Error Handling**: Typed throws (Swift 6), Result types
4. **State Modeling**: Enums with associated values for impossible states
5. **Concurrency**: async/await, actors, structured concurrency

## Package Management with SPM

### Why SPM

- Built into Swift toolchain and Xcode
- No external dependencies (unlike CocoaPods, Carthage)
- First-class support for Swift concurrency
- Better security (no arbitrary scripts)

### Package.swift Structure

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [
        .iOS(.v17),
        .macOS(.v14)
    ],
    products: [
        .library(name: "Core", targets: ["Core"]),
        .executable(name: "cli", targets: ["CLI"])
    ],
    dependencies: [
        .package(url: "https://github.com/apple/swift-argument-parser", from: "1.3.0"),
        .package(url: "https://github.com/pointfreeco/swift-dependencies", from: "1.0.0")
    ],
    targets: [
        .target(
            name: "Core",
            dependencies: [
                .product(name: "Dependencies", package: "swift-dependencies")
            ]
        ),
        .target(
            name: "AppUI",
            dependencies: ["Core"]
        ),
        .executableTarget(
            name: "CLI",
            dependencies: [
                "Core",
                .product(name: "ArgumentParser", package: "swift-argument-parser")
            ]
        ),
        .testTarget(
            name: "CoreTests",
            dependencies: ["Core"]
        )
    ]
)
```

### Common Commands

```bash
# Create new package
swift package init --type library
swift package init --type executable

# Build
swift build

# Run tests
swift test

# Run executable
swift run cli

# Update dependencies
swift package update

# Generate Xcode project (if needed)
swift package generate-xcodeproj
```

## Compiler Settings

### Strict Concurrency (Swift 6)

Enable strict concurrency checking in Package.swift:

```swift
.target(
    name: "Core",
    dependencies: [],
    swiftSettings: [
        .enableExperimentalFeature("StrictConcurrency")
    ]
)
```

Or in Xcode: Build Settings → Swift Compiler → Strict Concurrency Checking = Complete

### Treat Warnings as Errors

```swift
swiftSettings: [
    .unsafeFlags(["-warnings-as-errors"])
]
```

## Immutability by Default

### let Over var

```swift
// BAD
var name = "Kevin"
var count = 0

// GOOD - use let unless mutation is required
let name = "Kevin"
var count = 0  // Only if actually mutated later
```

### Structs Over Classes

```swift
// BAD - using class for simple data
class User {
    var id: UUID
    var name: String

    init(id: UUID, name: String) {
        self.id = id
        self.name = name
    }
}

// GOOD - struct for value types
struct User {
    let id: UUID
    let name: String
}
```

**Use classes only when:**
- You need reference semantics (shared mutable state)
- You need inheritance
- You need deinit
- You're interfacing with Objective-C

## Optionals

### Never Force Unwrap

```swift
// BAD - crashes if nil
let name = user.name!
let first = array.first!

// GOOD - guard let for early exit
guard let name = user.name else {
    return
}

// GOOD - if let for conditional
if let first = array.first {
    process(first)
}

// GOOD - nil coalescing
let name = user.name ?? "Unknown"

// GOOD - optional chaining
let count = user.orders?.count ?? 0
```

### Avoid Implicitly Unwrapped Optionals

```swift
// BAD
var delegate: MyDelegate!

// GOOD - make it explicit optional or non-optional
var delegate: MyDelegate?
// or
let delegate: MyDelegate  // Set in init
```

**Exception**: `@IBOutlet` in UIKit (required by Interface Builder)

## Enums with Associated Values

Swift enums are powerful discriminated unions. Use them to make invalid states unrepresentable:

### State Modeling

```swift
// BAD - bag of optionals
struct LoadingState {
    var isLoading: Bool
    var data: Data?
    var error: Error?
}

// GOOD - impossible states are impossible
enum LoadingState<T> {
    case idle
    case loading
    case success(T)
    case failure(Error)
}

// Usage
func render(state: LoadingState<User>) {
    switch state {
    case .idle:
        showPlaceholder()
    case .loading:
        showSpinner()
    case .success(let user):
        showUser(user)
    case .failure(let error):
        showError(error)
    }
}
```

### Events

```swift
enum UserEvent {
    case created(User)
    case updated(User, changes: [String: Any])
    case deleted(id: UUID)
}

func handle(event: UserEvent) {
    switch event {
    case .created(let user):
        notifyCreation(user)
    case .updated(let user, let changes):
        notifyUpdate(user, changes: changes)
    case .deleted(let id):
        notifyDeletion(id)
    }
}
```

### Exhaustive Switch

Always handle all cases - compiler enforces this:

```swift
// Compiler error if you miss a case
switch state {
case .idle: break
case .loading: break
case .success: break
// Missing .failure - won't compile
}
```

## Error Handling

### Typed Throws (Swift 6)

```swift
// Define specific error types
enum ValidationError: Error {
    case emptyName
    case invalidEmail(String)
    case ageTooLow(minimum: Int)
}

// Typed throws - callers know exactly what can fail
func validate(user: UserInput) throws(ValidationError) -> User {
    guard !user.name.isEmpty else {
        throw .emptyName
    }
    guard user.email.contains("@") else {
        throw .invalidEmail(user.email)
    }
    guard user.age >= 18 else {
        throw .ageTooLow(minimum: 18)
    }
    return User(name: user.name, email: user.email, age: user.age)
}

// Caller gets typed error
do {
    let user = try validate(user: input)
} catch {
    // error is ValidationError, not any Error
    switch error {
    case .emptyName:
        showNameError()
    case .invalidEmail(let email):
        showEmailError(email)
    case .ageTooLow(let minimum):
        showAgeError(minimum)
    }
}
```

### Result Type

For async operations or when you want to pass errors as values:

```swift
func fetchUser(id: UUID) async -> Result<User, NetworkError> {
    do {
        let data = try await network.get("/users/\(id)")
        let user = try decoder.decode(User.self, from: data)
        return .success(user)
    } catch let error as NetworkError {
        return .failure(error)
    } catch {
        return .failure(.unknown(error))
    }
}

// Usage
let result = await fetchUser(id: userId)
switch result {
case .success(let user):
    display(user)
case .failure(let error):
    handleError(error)
}
```

## Concurrency

### async/await Over Callbacks

```swift
// BAD - callback hell
func fetchUser(id: UUID, completion: @escaping (Result<User, Error>) -> Void) {
    network.get("/users/\(id)") { result in
        switch result {
        case .success(let data):
            do {
                let user = try decoder.decode(User.self, from: data)
                completion(.success(user))
            } catch {
                completion(.failure(error))
            }
        case .failure(let error):
            completion(.failure(error))
        }
    }
}

// GOOD - async/await
func fetchUser(id: UUID) async throws -> User {
    let data = try await network.get("/users/\(id)")
    return try decoder.decode(User.self, from: data)
}
```

### Actors for Shared Mutable State

```swift
// BAD - manual locking
class Counter {
    private var value = 0
    private let lock = NSLock()

    func increment() {
        lock.lock()
        value += 1
        lock.unlock()
    }
}

// GOOD - actor handles synchronization
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        value
    }
}

// Usage
let counter = Counter()
await counter.increment()
let value = await counter.getValue()
```

### Task Groups for Parallel Work

```swift
func fetchAllUsers(ids: [UUID]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await fetchUser(id: id)
            }
        }

        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        return users
    }
}
```

## Type Safety

### Avoid Any

```swift
// BAD
func process(data: Any) {
    if let string = data as? String {
        // ...
    }
}

// GOOD - use generics
func process<T: Processable>(data: T) {
    data.process()
}

// GOOD - use protocols
func process(data: some Processable) {
    data.process()
}
```

### Use Generics

```swift
// BAD - separate implementations
func firstString(in array: [String]) -> String? { array.first }
func firstInt(in array: [Int]) -> Int? { array.first }

// GOOD - generic
func first<T>(in array: [T]) -> T? {
    array.first
}
```

### Phantom Types for Type Safety

```swift
// Prevent mixing IDs of different entity types
struct ID<Entity>: Hashable {
    let rawValue: UUID
}

struct User {
    let id: ID<User>
    let name: String
}

struct Order {
    let id: ID<Order>
    let userId: ID<User>
}

// Compiler prevents: order.id == user.id (different types)
```

## Project Structure

### Shared Core for Multi-Target

```text
MyApp/
├── Package.swift
├── Sources/
│   ├── Core/               # Shared business logic
│   │   ├── Models/
│   │   │   └── User.swift
│   │   ├── Services/
│   │   │   └── UserService.swift
│   │   └── Database/
│   │       └── Repository.swift
│   ├── AppUI/              # SwiftUI views import Core
│   │   ├── App.swift
│   │   └── Views/
│   │       └── UserView.swift
│   └── CLI/                # ArgumentParser commands import Core
│       └── Main.swift
└── Tests/
    └── CoreTests/
        └── UserServiceTests.swift
```

### Example Core Module

```swift
// Sources/Core/Services/UserService.swift
public struct UserService {
    private let repository: UserRepository

    public init(repository: UserRepository) {
        self.repository = repository
    }

    public func createUser(name: String, email: String) async throws -> User {
        let user = User(id: ID(rawValue: UUID()), name: name, email: email)
        try await repository.save(user)
        return user
    }
}
```

### Example SwiftUI Using Core

```swift
// Sources/AppUI/Views/UserView.swift
import SwiftUI
import Core

struct UserView: View {
    let userService: UserService
    @State private var state: LoadingState<User> = .idle

    var body: some View {
        switch state {
        case .idle:
            Button("Load") { Task { await load() } }
        case .loading:
            ProgressView()
        case .success(let user):
            Text(user.name)
        case .failure(let error):
            Text(error.localizedDescription)
        }
    }

    private func load() async {
        state = .loading
        do {
            let user = try await userService.fetchUser()
            state = .success(user)
        } catch {
            state = .failure(error)
        }
    }
}
```

### Example CLI Using Core

```swift
// Sources/CLI/Main.swift
import ArgumentParser
import Core

@main
struct CLI: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "myapp",
        subcommands: [CreateUser.self]
    )
}

struct CreateUser: AsyncParsableCommand {
    @Argument var name: String
    @Argument var email: String

    func run() async throws {
        let service = UserService(repository: .live)
        let user = try await service.createUser(name: name, email: email)
        print("Created user: \(user.id.rawValue)")
    }
}
```

## SwiftUI Patterns

### View as Function of State

```swift
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}
```

### Extract Subviews

```swift
// BAD - massive view
struct UserProfileView: View {
    var body: some View {
        VStack {
            // 200 lines of UI code
        }
    }
}

// GOOD - composed from smaller views
struct UserProfileView: View {
    let user: User

    var body: some View {
        VStack {
            AvatarView(url: user.avatarURL)
            UserInfoSection(user: user)
            UserStatsSection(stats: user.stats)
        }
    }
}
```

### Dependency Injection with Environment

```swift
// Define dependency
struct UserServiceKey: EnvironmentKey {
    static let defaultValue: UserService = .live
}

extension EnvironmentValues {
    var userService: UserService {
        get { self[UserServiceKey.self] }
        set { self[UserServiceKey.self] = newValue }
    }
}

// Use in view
struct UserView: View {
    @Environment(\.userService) var userService

    var body: some View {
        // ...
    }
}

// Inject for testing
UserView()
    .environment(\.userService, .mock)
```

## macOS Scripting

### When to Use Swift for Scripts

**Use Swift when:**
- You need macOS APIs (Keychain, Accessibility, FSEvents, XPC, IOKit)
- Performance matters (image processing, large file ops)
- You want to distribute a binary without dependencies

**Use Python/shell when:**
- Quick automation, text processing
- Cross-platform needed
- Rapid iteration more important than type safety

### Script Execution Modes

```bash
# Interpreted - slow startup (~500ms cold)
#!/usr/bin/swift
import Foundation
print(FileManager.default.currentDirectoryPath)

# Compiled - fast, but requires build step
swift build -c release
.build/release/my-script

# For scripts with dependencies, use swift-sh
brew install swift-sh
```

### swift-sh for Dependencies

```swift
#!/usr/bin/swift sh
import ArgumentParser  // @apple/swift-argument-parser ~> 1.3
import Rainbow         // @onevcat/Rainbow

@main
struct MyScript: ParsableCommand {
    @Argument var name: String

    func run() {
        print("Hello, \(name)".green)
    }
}
```

Run directly: `./my-script.swift Kevin`

### Common macOS APIs

#### FileManager - File Operations

```swift
import Foundation

let fm = FileManager.default
let home = fm.homeDirectoryForCurrentUser

// List directory
let contents = try fm.contentsOfDirectory(at: home, includingPropertiesForKeys: nil)

// Check existence
if fm.fileExists(atPath: "/tmp/file.txt") { }

// Create directory
try fm.createDirectory(at: home.appendingPathComponent("Scripts"),
                       withIntermediateDirectories: true)

// Copy/move/delete
try fm.copyItem(at: source, to: destination)
try fm.moveItem(at: source, to: destination)
try fm.removeItem(at: path)

// Attributes
let attrs = try fm.attributesOfItem(atPath: path)
let size = attrs[.size] as? UInt64
```

#### Process - Run Shell Commands

```swift
import Foundation

func shell(_ command: String) throws -> String {
    let process = Process()
    let pipe = Pipe()

    process.standardOutput = pipe
    process.standardError = pipe
    process.executableURL = URL(fileURLWithPath: "/bin/zsh")
    process.arguments = ["-c", command]

    try process.run()
    process.waitUntilExit()

    let data = pipe.fileHandleForReading.readDataToEndOfFile()
    return String(data: data, encoding: .utf8) ?? ""
}

// Usage
let output = try shell("ls -la")
let gitStatus = try shell("git status --porcelain")
```

#### Async Process Execution

```swift
func shellAsync(_ command: String) async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        let process = Process()
        let pipe = Pipe()

        process.standardOutput = pipe
        process.standardError = pipe
        process.executableURL = URL(fileURLWithPath: "/bin/zsh")
        process.arguments = ["-c", command]

        process.terminationHandler = { _ in
            let data = pipe.fileHandleForReading.readDataToEndOfFile()
            let output = String(data: data, encoding: .utf8) ?? ""
            continuation.resume(returning: output)
        }

        do {
            try process.run()
        } catch {
            continuation.resume(throwing: error)
        }
    }
}
```

#### Keychain Access

```swift
import Security

func getKeychainPassword(service: String, account: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecReturnData as String: true
    ]

    var result: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &result)

    guard status == errSecSuccess,
          let data = result as? Data,
          let password = String(data: data, encoding: .utf8) else {
        return nil
    }
    return password
}

func setKeychainPassword(service: String, account: String, password: String) throws {
    let data = password.data(using: .utf8)!

    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecValueData as String: data
    ]

    // Delete existing
    SecItemDelete(query as CFDictionary)

    // Add new
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
        throw KeychainError.unableToStore
    }
}
```

#### NSWorkspace - App Control

```swift
import AppKit

let workspace = NSWorkspace.shared

// Open file with default app
workspace.open(URL(fileURLWithPath: "/path/to/file.pdf"))

// Open with specific app
workspace.open([URL(fileURLWithPath: "/path/to/file.txt")],
               withApplicationAt: URL(fileURLWithPath: "/Applications/Sublime Text.app"),
               configuration: .init())

// Launch app
workspace.launchApplication("Safari")

// Get running apps
let runningApps = workspace.runningApplications
for app in runningApps where app.isActive {
    print(app.localizedName ?? "Unknown")
}

// Activate app
if let app = runningApps.first(where: { $0.bundleIdentifier == "com.apple.Safari" }) {
    app.activate()
}

// Get frontmost app
if let frontmost = workspace.frontmostApplication {
    print(frontmost.localizedName ?? "")
}
```

#### FSEvents - File Watching

```swift
import Foundation

class FileWatcher {
    private var stream: FSEventStreamRef?

    func watch(paths: [String], callback: @escaping ([String]) -> Void) {
        var context = FSEventStreamContext()
        context.info = Unmanaged.passUnretained(self).toOpaque()

        let flags = UInt32(kFSEventStreamCreateFlagFileEvents | kFSEventStreamCreateFlagUseCFTypes)

        stream = FSEventStreamCreate(
            nil,
            { _, _, numEvents, eventPaths, _, _ in
                guard let paths = unsafeBitCast(eventPaths, to: NSArray.self) as? [String] else { return }
                // Call back on main thread
                DispatchQueue.main.async {
                    callback(paths)
                }
            },
            &context,
            paths as CFArray,
            FSEventStreamEventId(kFSEventStreamEventIdSinceNow),
            1.0,  // Latency in seconds
            flags
        )

        FSEventStreamScheduleWithRunLoop(stream!, CFRunLoopGetMain(), CFRunLoopMode.defaultMode.rawValue)
        FSEventStreamStart(stream!)
    }

    func stop() {
        if let stream = stream {
            FSEventStreamStop(stream)
            FSEventStreamInvalidate(stream)
            FSEventStreamRelease(stream)
        }
    }
}

// Usage
let watcher = FileWatcher()
watcher.watch(paths: ["~/Downloads"]) { changedPaths in
    for path in changedPaths {
        print("Changed: \(path)")
    }
}
RunLoop.main.run()  // Keep script alive
```

#### Accessibility - UI Automation

```swift
import ApplicationServices

// Check accessibility permissions
func checkAccessibilityPermissions() -> Bool {
    let options = [kAXTrustedCheckOptionPrompt.takeUnretainedValue() as String: true]
    return AXIsProcessTrustedWithOptions(options as CFDictionary)
}

// Get focused element
func getFocusedElement() -> AXUIElement? {
    let systemWide = AXUIElementCreateSystemWide()
    var focusedElement: CFTypeRef?
    let result = AXUIElementCopyAttributeValue(systemWide, kAXFocusedUIElementAttribute as CFString, &focusedElement)
    guard result == .success else { return nil }
    return (focusedElement as! AXUIElement)
}

// Click menu item
func clickMenuItem(app: String, menu: String, item: String) {
    guard let runningApp = NSWorkspace.shared.runningApplications.first(where: {
        $0.localizedName == app
    }) else { return }

    let appElement = AXUIElementCreateApplication(runningApp.processIdentifier)
    // Navigate menu bar → menu → item and perform press action
    // ... (implementation depends on specific needs)
}
```

#### UserDefaults for Script Config

```swift
import Foundation

// Read/write to app defaults
let defaults = UserDefaults.standard
defaults.set("value", forKey: "myScriptSetting")
let setting = defaults.string(forKey: "myScriptSetting")

// Read other app's defaults (sandboxing permitting)
if let safariDefaults = UserDefaults(suiteName: "com.apple.Safari") {
    let homepage = safariDefaults.string(forKey: "HomePage")
}
```

### Script Project Structure

For non-trivial scripts, use a proper SPM package:

```
my-script/
├── Package.swift
├── Sources/
│   └── my-script/
│       ├── main.swift       # Entry point
│       ├── Commands/        # ArgumentParser commands
│       └── Utilities/       # Shared helpers
└── scripts/
    └── install.sh           # Copy binary to ~/bin
```

```swift
// Package.swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "my-script",
    platforms: [.macOS(.v14)],
    dependencies: [
        .package(url: "https://github.com/apple/swift-argument-parser", from: "1.3.0"),
        .package(url: "https://github.com/onevcat/Rainbow", from: "4.0.0")
    ],
    targets: [
        .executableTarget(
            name: "my-script",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
                "Rainbow"
            ]
        )
    ]
)
```

### Install Script

```bash
#!/bin/bash
swift build -c release
cp .build/release/my-script ~/bin/
```

### CLI Output Libraries

| Purpose | Package |
|---------|---------|
| Colors | Rainbow |
| Argument parsing | swift-argument-parser |
| Progress/spinners | No good Swift option - use print-based |

Note: Swift CLI ecosystem is thinner than Python's. For complex TUI, consider whether Python (Rich, tqdm) is more practical.

### LLM-Friendly Output

All CLIs must support both human and machine consumption:

```swift
import ArgumentParser
import Foundation
import Rainbow

struct User: Codable {
    let id: String
    let name: String
    let email: String
}

struct ListUsers: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        abstract: "List all users.",
        discussion: """
            Returns array of user objects with id, name, and email fields.
            Use --json for structured output suitable for piping to other tools or LLMs.
            """
    )

    @Flag(name: .long, help: "Output as JSON for programmatic consumption")
    var json = false

    @Option(name: .shortAndLong, help: "Maximum number of users to return")
    var limit: Int = 50

    func run() async throws {
        let users = try await getUsers(limit: limit)

        if json {
            // Machine-readable: structured, no formatting
            let encoder = JSONEncoder()
            encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
            let data = try encoder.encode(users)
            print(String(data: data, encoding: .utf8)!)
        } else {
            // Human-readable: colors, pleasant
            print("\nUsers (\(users.count)):".bold)
            for user in users {
                print("  \(user.name.cyan) <\(user.email)>")
            }
            print()
        }
    }
}
```

**Rules:**
1. `--json` flag on every command that outputs data
2. JSON output: Codable structs, `prettyPrinted`, no ANSI
3. Default output: human-readable with Rainbow colors
4. Use `discussion` in CommandConfiguration to explain what the command returns

## Cocoa Framework

Always use Cocoa (`import Cocoa`) for macOS applications. Cocoa provides the full macOS application stack (AppKit, Foundation, CoreData) in a single import. For iOS, use UIKit/SwiftUI as appropriate.

```swift
// BAD - piecemeal imports for macOS
import Foundation
import AppKit
import CoreGraphics

// GOOD - single Cocoa import covers all macOS frameworks
import Cocoa
```

**When to use Cocoa vs individual imports:**
- macOS apps and scripts → `import Cocoa`
- Cross-platform packages (iOS + macOS) → `import Foundation` + platform-specific imports
- Pure logic modules with no UI → `import Foundation`

## Self-Consuming Logging

Always create structured, self-consuming logging patterns for troubleshooting during development. Logs must be useful enough that you can diagnose issues by reading them alone — no debugger required.

### OSLog (Preferred)

```swift
import os
import Cocoa

/// Centralized log categories per subsystem.
/// Each module gets its own category for filtered tailing.
enum Log {
    /// Subsystem matches bundle identifier for os_log filtering
    private static let subsystem = Bundle.main.bundleIdentifier ?? "com.app.dev"

    static let network = Logger(subsystem: subsystem, category: "network")
    static let database = Logger(subsystem: subsystem, category: "database")
    static let ui = Logger(subsystem: subsystem, category: "ui")
    static let lifecycle = Logger(subsystem: subsystem, category: "lifecycle")
    static let auth = Logger(subsystem: subsystem, category: "auth")
}

// Usage - rich context at every call site
func fetchUser(id: UUID) async throws -> User {
    Log.network.info("⬆️ fetchUser started — id=\(id.uuidString, privacy: .public)")
    let start = ContinuousClock.now

    do {
        let user = try await api.get("/users/\(id)")
        let elapsed = ContinuousClock.now - start
        Log.network.info("⬇️ fetchUser success — id=\(id.uuidString, privacy: .public) elapsed=\(elapsed)")
        return user
    } catch {
        let elapsed = ContinuousClock.now - start
        Log.network.error("❌ fetchUser failed — id=\(id.uuidString, privacy: .public) elapsed=\(elapsed) error=\(error.localizedDescription, privacy: .public)")
        throw error
    }
}

// State transitions are always logged
func handleStateChange(from old: AppState, to new: AppState) {
    Log.lifecycle.notice("🔄 state transition — from=\(String(describing: old), privacy: .public) to=\(String(describing: new), privacy: .public)")
}
```

### Logging Rules

1. **Every public function logs entry and exit** — include parameters and elapsed time
2. **Errors always log full context** — what was attempted, with what inputs, what failed
3. **State transitions are always logged** — old state → new state
4. **Use emoji prefixes** for visual scanning: ⬆️ request, ⬇️ response, ❌ error, 🔄 transition, ✅ success, ⚠️ warning
5. **Include timing** — `ContinuousClock` for elapsed durations on any I/O or async operation
6. **Privacy-aware** — use `.public` only for non-sensitive data; defaults to redacted in release

### File Logging for CLI Tools

For command-line tools where OSLog isn't practical, write to a known log file:

```swift
import Foundation

/// File-based logger for CLI tools.
/// Writes structured log lines to ~/.local/log/<tool>.log
actor FileLog {
    static let shared = FileLog()

    private let logFile: URL
    private let dateFormatter: ISO8601DateFormatter = {
        let f = ISO8601DateFormatter()
        f.formatOptions = [.withInternetDateTime, .withFractionalSeconds]
        return f
    }()

    private init() {
        let logDir = FileManager.default.homeDirectoryForCurrentUser
            .appendingPathComponent(".local/log")
        try? FileManager.default.createDirectory(at: logDir, withIntermediateDirectories: true)

        let processName = ProcessInfo.processInfo.processName
        logFile = logDir.appendingPathComponent("\(processName).log")
    }

    /// Append a structured log line.
    /// - Parameters:
    ///   - level: Log severity (debug, info, warn, error)
    ///   - category: Subsystem category for filtering
    ///   - message: Human-readable log message
    func log(_ level: String, category: String, _ message: String) {
        let timestamp = dateFormatter.string(from: Date())
        let line = "\(timestamp) [\(level.uppercased())] [\(category)] \(message)\n"

        // Also print to stderr so it doesn't pollute stdout (pipe-friendly)
        FileHandle.standardError.write(Data(line.utf8))

        // Append to file for tailing
        if let handle = try? FileHandle(forWritingTo: logFile) {
            handle.seekToEndOfFile()
            handle.write(Data(line.utf8))
            handle.closeFile()
        } else {
            try? line.data(using: .utf8)?.write(to: logFile)
        }
    }
}

// Usage
await FileLog.shared.log("info", category: "network", "⬆️ fetchUser id=\(id)")
```

## Always Tail Logs During Development

When developing or debugging Swift code, **always run a log tail in a background terminal**. This is non-negotiable — logs are useless if nobody is watching them.

### Tailing OSLog (macOS apps)

```bash
# Tail all logs from your app's subsystem
log stream --predicate 'subsystem == "com.yourapp.dev"' --level debug

# Filter to specific category
log stream --predicate 'subsystem == "com.yourapp.dev" AND category == "network"' --level debug

# With timestamps and process info
log stream --predicate 'subsystem == "com.yourapp.dev"' --level debug --style compact
```

### Tailing File Logs (CLI tools)

```bash
# Tail the log file for a CLI tool
tail -f ~/.local/log/my-tool.log

# Tail with grep for specific category
tail -f ~/.local/log/my-tool.log | grep '\[NETWORK\]'

# Tail all tool logs
tail -f ~/.local/log/*.log
```

### Development Workflow

1. **Start log tail first** — before running the app or script
2. **Use `bg_bash`** to run the tail in the background when working in pi:
   ```
   bg_bash: log stream --predicate 'subsystem == "com.yourapp.dev"' --level debug
   ```
3. **Check `task_output`** periodically to review log output
4. **Filter aggressively** — use category predicates to reduce noise
5. **Never ship without reviewing logs** — if the log tail shows unexpected entries, investigate before committing

## Quick Reference

| Tool | Purpose |
|------|---------|
| SPM | Package management (not CocoaPods, Carthage) |
| swift build | Compile |
| swift test | Run tests |
| swift run | Execute |

| Pattern | Preference |
|---------|------------|
| Mutability | `let` over `var` |
| Value types | `struct` over `class` |
| Optionals | `guard let`, `if let`, `??` (never `!`) |
| State modeling | Enums with associated values |
| Error handling | Typed throws (Swift 6), Result |
| Concurrency | async/await, actors (not callbacks) |
| Type safety | Generics, protocols (avoid `Any`) |
| Architecture | Shared Core for multi-target |
| macOS imports | `import Cocoa` (not piecemeal) |
| Logging | OSLog with categories (apps), FileLog (CLI) |
| Log tailing | Always run `log stream` or `tail -f` during dev |

## Notes

- Swift 6 requires strict concurrency - design for it from the start
- Enums with associated values are Swift's killer feature for state modeling
- SPM is the only package manager worth using in 2024+
- Prefer value types (struct, enum) over reference types (class)
- Use actors instead of manual locking for shared state
- SwiftUI is declarative - views should be pure functions of state
