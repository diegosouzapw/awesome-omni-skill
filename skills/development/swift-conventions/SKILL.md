---
name: swift-conventions
description: Swift coding conventions and best practices for modern Swift development. Use when writing, reviewing, or refactoring Swift code to ensure consistency with naming conventions, access control, async/await patterns, and SwiftUI/framework best practices.
compatibility: Swift 5.9+
---

# Swift Coding Conventions

## Naming Conventions

### Types (Classes, Structs, Enums, Protocols)

Use **PascalCase** for all type names:

```swift
public class ProcessManager { }
public struct Location { }
public enum ProcessQuality { }
public protocol ProcessController { }
```

### Properties & Variables

Use **camelCase** for properties, variables, and instance names:

```swift
let locationManager = LocationManager()
var subscriptions: [ProcessSubscription] = []
private let updateInterval: TimeInterval = 60
```

### Functions & Methods

Use **camelCase** with descriptive action verbs:

```swift
func refreshData(for location: Location) async throws -> ProcessSensor?
func updateLocation(location: Location)
private func significantLocationChange(previous: Location?, current: Location) -> Bool
```

### Constants

Use **camelCase** for constant properties, **UPPER_SNAKE_CASE** for macro-like constants:

```swift
public static let houseOfWorldCultures = Location(latitude: 52.51889, longitude: 13.36528)
private let updateInterval: TimeInterval = 60
```

### Enums

Use **PascalCase** for enum name, **camelCase** for cases:

```swift
public enum ProcessQuality {
    case good
    case uncertain
    case bad
}

public enum ProcessSelector: Hashable {
    case weather(Weather)
    case forecast(Forecast)
}
```

## Code Structure

### Braces

Opening brace on same line, closing brace on new line:

```swift
public class ProcessManager {
    func updateSubscriptions() {
        for subscription in subscriptions {
            subscription.update(timeout: updateInterval)
        }
    }
}
```

### Indentation

- Use **4 spaces** (no tabs)
- Align continuation lines with opening delimiter

### Line Length

Target **120 characters** maximum per line. Break at logical points:

```swift
public func dataWithRetry(
    from url: URL, retryCount: Int = 3, retryInterval: TimeInterval = 1.0
) async throws -> (Data, URLResponse) {
    // Implementation
}
```

## Access Control

Always explicit: Mark APIs as `public` intentionally; avoid relying on default `internal`.

```swift
public class ProcessManager: Identifiable {
    public let id = UUID()
    public static let shared = ProcessManager()
    private let locationManager = LocationManager()
    private var location: Location?
}
```

Order of access levels (most to least restrictive):

1. `private` - Only within current declaration
2. `fileprivate` - Only within same source file
3. `internal` - Within module (default)
4. `public` - Visible to consumers
5. `open` - Visible and subclassable (use rarely)

## Type Declarations

### Classes

```swift
public class ProcessManager: Identifiable, LocationManagerDelegate {
    public let id = UUID()
    public static let shared = ProcessManager()
    private init() { }
}
```

### Structs

```swift
public struct Location: Equatable, Hashable {
    public let latitude: Double
    public let longitude: Double

    public init(latitude: Double, longitude: Double) {
        self.latitude = latitude
        self.longitude = longitude
    }
}
```

### Enums

```swift
public enum ProcessQuality {
    case good
    case uncertain
    case bad
    case unknown
}

public enum Weather: Int, CaseIterable {
    case temperature = 0
    case apparentTemperature = 1
}

public enum ProcessSelector: Hashable {
    case weather(Weather)
    case forecast(Forecast)
    case covid(Covid)
}
```

### Protocols

```swift
public protocol ProcessController {
    func refreshData(for location: Location) async throws -> ProcessSensor?
}

public protocol LocationManagerDelegate: Identifiable where ID == UUID {
    func locationManager(didUpdateLocation location: Location)
}
```

## Properties

### Stored Properties

```swift
public let id = UUID()
private let locationManager = LocationManager()
private var location: Location?
private let updateInterval: TimeInterval = 60
```

### Computed Properties

```swift
public var coordinate: CLLocationCoordinate2D {
    return CLLocationCoordinate2D(latitude: self.latitude, longitude: self.longitude)
}

var isReady: Bool {
    return location != nil && subscriptions.isEmpty == false
}
```

### Property Observers

```swift
var location: Location? {
    willSet {
        print("About to set location")
    }
    didSet {
        if location != oldValue {
            refreshSubscriptions()
        }
    }
}
```

### Lazy Properties

```swift
lazy var dateFormatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
    return formatter
}()
```

## Functions & Methods

### Basic Structure

```swift
public func refreshData(for location: Location) async throws -> ProcessSensor? {
    let weather = try await WeatherService.shared.weather(for: clLocation)
    return ProcessSensor(name: "", location: location, measurements: [:], timestamp: Date.now)
}
```

### Parameter Labels

Descriptive external labels; omit with underscore when appropriate:

```swift
func updateLocation(location: Location)
func add(subscriber: any ProcessSubscriber, timeout: TimeInterval)
func process(_ data: Data) -> Result
```

### Default Parameters

Place at end of parameter list:

```swift
public func dataWithRetry(
    from url: URL,
    retryCount: Int = 3,
    retryInterval: TimeInterval = 1.0
) async throws -> (Data, URLResponse) {
    // Implementation
}
```

### Multiple Initializers

Designated initializer with convenience initializers:

```swift
public init(value: Measurement<T>, customData: [String: Any]?, quality: ProcessQuality, timestamp: Date) {
    self.value = value
    self.customData = customData
    self.quality = quality
    self.timestamp = timestamp
}

public convenience init(value: Measurement<T>, quality: ProcessQuality) {
    self.init(value: value, customData: nil, quality: quality, timestamp: Date.now)
}
```

## Control Flow

### If Statements

```swift
if location != nil {
    refreshSubscriptions()
}

if let location = self.location {
    delegate.locationManager(didUpdateLocation: location)
}
```

### Guard Statements

```swift
guard let location = self.location else {
    return
}

guard let data = data,
      let response = response as? HTTPURLResponse,
      (200...299).contains(response.statusCode) else {
    throw NetworkError.invalidResponse
}
```

### For Loops

```swift
for subscription in subscriptions {
    subscription.update(timeout: updateInterval)
}

for (index, item) in items.enumerated() {
    print("\(index): \(item)")
}

for subscription in subscriptions where subscription.isPending() {
    subscription.reset()
}
```

### Switch Statements

```swift
switch quality {
    case .good:
        return "✓"
    case .uncertain:
        return "~"
    case .bad:
        return "✗"
    case .unknown:
        return "?"
}

switch connectionType {
    case .wifi, .ethernet:
        return true
    case .cellular:
        return false
}
```

## Error Handling

### Error Definitions

```swift
enum NetworkError: Error {
    case invalidResponse
    case serverError(statusCode: Int)
    case noData
}
```

### Throwing Functions

```swift
public func refreshData(for location: Location) async throws -> ProcessSensor? {
    let weather = try await WeatherService.shared.weather(for: clLocation)
    return sensor
}
```

### Try-Catch Blocks

```swift
do {
    let (data, response) = try await self.data(from: url)
    return (data, response)
} catch NetworkError.invalidResponse {
    return nil
} catch {
    print("Error: \(error)")
    return nil
}
```

### Optional Try

```swift
if let placemark = try? await geocoder.reverseGeocodeLocation(location).first {
    // Use placemark
}

let config = try! Configuration.load()  // Only if guaranteed to succeed
```

## Async/Await

### Async Functions

```swift
public func refreshData(for location: Location) async throws -> ProcessSensor? {
    let weather = try await WeatherService.shared.weather(for: clLocation)
    let placemark = await LocationManager.reverseGeocodeLocation(location: location)
    return ProcessSensor(/* ... */)
}
```

### Task Creation

```swift
Task {
    await delegate.refreshData(location: location)
}

Task {
    do {
        let result = try await fetchData()
        process(result)
    } catch {
        print("Error: \(error)")
    }
}
```

### Actor Usage

```swift
actor NetworkManager {
    private var isConnected = true

    func updateConnectionStatus(_ status: Bool) {
        self.isConnected = status
    }
}
```

### Sendable Conformance

```swift
public class UnitRadiation: Dimension, @unchecked Sendable {
    public static let sieverts = UnitRadiation(
        symbol: "Sv/h",
        converter: UnitConverterLinear(coefficient: 1.0)
    )
}
```

## Protocols & Extensions

### Protocol Conformance

```swift
public class ProcessManager: Identifiable, LocationManagerDelegate {
    // Implementation
}

extension ProcessManager: CustomStringConvertible {
    public var description: String {
        return "ProcessManager with \(subscriptions.count) subscriptions"
    }
}
```

### Extension Organization

Use MARK comments to organize extensions by purpose:

```swift
// MARK: - LocationManagerDelegate
extension ProcessManager: LocationManagerDelegate {
    public func locationManager(didUpdateLocation location: Location) {
        // Implementation
    }
}

// MARK: - Subscription Management
extension ProcessManager {
    public func add(subscriber: any ProcessSubscriber, timeout: TimeInterval) {
        // Implementation
    }
}
```

## Generics

### Generic Types

```swift
public struct ProcessValue<T: Dimension>: Identifiable {
    public let id = UUID()
    public let value: Measurement<T>
    public let quality: ProcessQuality
}
```

### Generic Functions

```swift
func measure<T: Dimension>(_ value: Double, unit: T) -> Measurement<T> {
    return Measurement(value: value, unit: unit)
}
```

### Type Erasure

```swift
private var subscribers: [UUID: any ProcessSubscriber] = [:]

public func add(subscriber: any ProcessSubscriber, timeout: TimeInterval) {
    subscribers[subscriber.id] = subscriber
}
```

## Comments & Documentation

### Single-Line Comments

Comment the "why", not the "what":

```swift
// Check if device is connected before attempting network request
guard ReachabilityManager.shared.isConnected else {
    throw URLError(.notConnectedToInternet)
}
```

### Documentation Comments

Use DocC-style documentation for public APIs:

```swift
/// A simple and fast logging facility with support for different log levels.
public class Trace {
    /// Creates a new Logger instance
    /// - Parameters:
    ///   - minimumLevel: Minimum level of logs to display
    ///   - showColors: Whether to use ANSI colors
    ///   - dateFormat: Format string for timestamps
    public init(
        minimumLevel: Level = .debug,
        showColors: Bool = true,
        dateFormat: String = "yyyy-MM-dd HH:mm:ss.SSS"
    ) {
        // Implementation
    }
}
```

### MARK Comments

```swift
public class WeatherController {
    // MARK: - Properties
    private let service = WeatherService.shared

    // MARK: - Initialization
    public init() { }

    // MARK: - Public Methods
    public func refreshData(for location: Location) async throws -> ProcessSensor? {
        // Implementation
    }

    // MARK: - Private Helpers
    private func processWeatherData(_ data: WeatherData) -> ProcessSensor {
        // Implementation
    }
}
```

## Formatting & Whitespace

### Blank Lines

Separate logical sections with blank lines:

```swift
public class ProcessManager {
    public let id = UUID()
    public static let shared = ProcessManager()

    private let locationManager = LocationManager()
    private var location: Location?

    public func refreshSubscriptions() {
        // Implementation
    }
}
```

### Spacing

- Space after comma: `[1, 2, 3]`
- Space around operators: `a + b`
- No space before colon, space after: `var dict: [String: Int] = [:]`
- No space around range operators: `for i in 0..<count`, `0...10`

### Trailing Whitespace

Remove all trailing whitespace at end of lines.

## Swift-Specific Patterns

### Optionals

```swift
if let location = self.location {
    process(location)
}

guard let location = self.location else {
    return
}

let count = subscribers[id]?.subscriptions.count
let value = optionalValue ?? defaultValue
```

### Type Inference

```swift
let manager = ProcessManager.shared
let id = UUID()
let values = [1, 2, 3]

let timeout: TimeInterval = 60  // Explicit when needed
```

### Closures

```swift
Timer.scheduledTimer(withTimeInterval: updateInterval, repeats: true) { _ in
    self.updateSubscriptions()
}

items.map { $0.value * 2 }

UIView.animate(withDuration: 0.3) {
    view.alpha = 0
} completion: { _ in
    view.removeFromSuperview()
}
```

### Collections

```swift
var subscriptions: [ProcessSubscription] = []
var measurements: [ProcessSelector: [ProcessValue<Dimension>]] = [:]
let uniqueIds: Set<UUID> = []
```

### Property Wrappers

```swift
@Published var measurements: [ProcessValue<Dimension>] = []
@AppStorage("refreshInterval") var refreshInterval: TimeInterval = 60
```

## Code Review Checklist

- [ ] All public APIs have explicit `public` access control
- [ ] All types follow naming conventions (PascalCase)
- [ ] All functions/properties follow naming conventions (camelCase)
- [ ] Code is formatted with 4-space indentation
- [ ] Documentation comments for public APIs
- [ ] Error handling is comprehensive
- [ ] Async/await used consistently
- [ ] No platform-specific UI dependencies in libraries
- [ ] Custom Dimension types conform to `@unchecked Sendable`
- [ ] Protocol conformance is explicit
- [ ] MARK comments organize code sections
- [ ] No force unwrapping (!) unless absolutely safe
- [ ] **CRITICAL: Always verify documentation against actual implementation**

## Architectural Patterns

### Controller Pattern

```swift
public class WeatherController: ProcessController {
    public func refreshData(for location: Location) async throws -> ProcessSensor? {
        // Fetch data from service
        // Process into ProcessSensor
        // Return structured data
    }
}
```

### Service Pattern

```swift
public class CovidService {
    static func fetchDistricts(for location: Location, radius: Double) async throws -> Data? {
        // Perform HTTP request
        // Return raw data
    }
}
```

### Transformer Pattern

```swift
public class WeatherTransformer: ProcessTransformer {
    override public func renderCurrent(measurements: [ProcessSelector: [ProcessValue<Dimension>]])
        -> [ProcessSelector: ProcessValue<Dimension>] {
        // Transform measurements into current values
    }
}
```

### Data Flow

Service (HTTP) → Controller (Parse) → Transformer (Process) → Consumer (Display)
