# Silvermine Coding Standards Specific to Swift

## Formatting

### Whitespace

   * No trailing whitespace
   * One blank line between method definitions
   * No blank line after opening brace
   * No blank line before closing brace
   * One space after colon in type declarations
   * No space before colon in type declarations
   * One newline at the end of the file

```swift
// Good
struct Person {
   let name: String
   let age: Int

   var greeting: String {
      "Hello, \(name)!"
   }
}

// Bad
struct Person{
  let name:String
  let age:Int

  var greeting: String {
    "Hello, \(name)!"
  }
}
```

### Line Length

   * Break long lines at logical points
   * Indent continuation lines by our standard indentation

```swift
// Good
let longString = "This is a very long string that exceeds the line limit " +
   "and needs to be broken into multiple lines"

// Bad
let longString = "This is a very long string that exceeds the line limit and needs to be broken into multiple lines"
```

## Naming Conventions

### General

   * Use camelCase for variable and function names
   * Use PascalCase for type names (classes, structs, enums)
   * Use descriptive names that convey meaning

```swift
// Good
let userName = "John"
func calculateTotalPrice() { }
class UserProfile { }

// Bad
let u = "John"
func calc() { }
class up { }
```

### Functions and Methods

   * Use verbs for function names that perform actions
   * Use noun phrases for functions that return values without side effects
   * Omit the 'get' prefix for accessor methods (Swift convention)
   * Use clear parameter labels
   * Omit redundant words

```swift
// Good
func remove(at index: Int)
func convert(from string: String) -> Int
func user(id: String) -> User?
func publicProfile() -> Profile

// With side effects
func fetchUser(id: String) async throws -> User
func loadPublicProfile() async -> Profile

// Bad
func removeItem(atIndex index: Int)
func convertStringToInt(string: String) -> Int
func getUser(id: String) -> User?
func getPublicProfile() -> Profile
```

### Types and Properties

   * Use nouns for types
   * Use descriptive property names
   * Boolean properties should read as assertions
   * Prefer computed properties over parameterless functions
   * Use computed properties for values that are derived from other properties

```swift
// Good
struct User { }
let isEnabled = true
let hasPermission = false

// Computed property (preferred)
var fullName: String {
   "\(firstName) \(lastName)"
}

// Bad
struct UserClass { }
let enabled = true
let permission = false

// Avoid parameterless functions when a computed property would be clearer
func getFullName() -> String {
   "\(firstName) \(lastName)"
}
```

## Code Organization

### Access Control

   * Use the most restrictive access control possible
   * Explicitly declare access control for all properties and methods
   * Default to `private` unless wider access is needed

```swift
// Good
public class ProfileManager {
   private let userData: UserData

   internal func updateProfile() { }

   public func publicProfile() -> Profile { }
}
```

### Type Definitions

   * One type per file unless types are closely related
   * Group related properties and methods together
   * Order: properties, initializers, public methods, private methods

```swift
struct User {
   // Properties
   let id: String
   let name: String

   // Initializers
   init(id: String, name: String) {
      self.id = id
      self.name = name
   }

   // Computed properties
   var displayName: String {
      name
   }

   // Private methods
   private func formatName() -> String {
      name.trimmingCharacters(in: .whitespaces)
   }
}
```

## Swift Features

### Optionals

   * Avoid force unwrapping (`!`) except in tests or when truly impossible to be nil
   * Use optional binding (`if let`, `guard let`) for safe unwrapping
   * Use nil coalescing operator (`??`) for default values

```swift
// Good
if let name = optionalName {
   print("Hello, \(name)")
}

let displayName = userName ?? "Guest"

// Bad
print("Hello, \(optionalName!)")
```

### Error Handling

   * Use `throw`/`catch` for recoverable errors
   * Use `guard` for early returns
   * Provide meaningful error types

```swift
// Good
enum DataError: Error {
   case invalidFormat
   case missingValue
}

func processData() throws {
   guard isValidFormat else {
      throw DataError.invalidFormat
   }
   // Process data
}

// Bad
func processData() -> Bool {
   if !isValidFormat {
      return false
   }
   // Process data
   return true
}
```

### Concurrency

   * Use Swift Concurrency (async/await, actors)
   * Leverage Swift 6 data race safety when available

```swift
// Good - Simple async/await usage
func fetchUserProfile(id: String) async throws -> UserProfile {
   // Asynchronously fetch user data
   let userData = try await networkService.fetchData(from: "/users/\(id)")
   
   // Process the data on the current task
   let profile = try JSONDecoder().decode(UserProfile.self, from: userData)
   return profile
}

// Usage
func updateUI() async {
   do {
      // Suspend execution until the profile is fetched
      let profile = try await fetchUserProfile(id: "123")
      
      // Continue execution after profile is available
      nameLabel.text = profile.name
      emailLabel.text = profile.email
   } catch {
      errorLabel.text = "Failed to load profile: \(error.localizedDescription)"
   }
}

// Bad - Callback hell
func fetchUserProfile(id: String, completion: @escaping (Result<UserProfile, Error>) -> Void) {
   networkService.fetchData(from: "/users/\(id)") { result in
      switch result {
      case .success(let data):
         do {
            let profile = try JSONDecoder().decode(UserProfile.self, from: data)
            completion(.success(profile))
         } catch {
            completion(.failure(error))
         }
      case .failure(let error):
         completion(.failure(error))
      }
   }
}
```

```swift
// Good - Using actors for shared state
actor UserManager {
   private var users: [String: User] = [:]
   
   func addUser(_ user: User) {
      users[user.id] = user
   }
   
   func user(id: String) -> User? {
      return users[id]
   }
}

// Usage with async/await
func updateUserProfile() async throws {
   let userManager = UserManager()
   
   // Concurrent tasks
   async let newUserData = fetchUserData()
   async let preferences = fetchUserPreferences()
   
   // Wait for both tasks to complete
   let user = try await User(data: newUserData, preferences: preferences)
   
   // Thread-safe access to shared state through actor
   await userManager.addUser(user)
}

// Bad
class UnsafeUserManager {
   private var users: [String: User] = [:] // Shared mutable state without protection
   
   func addUser(_ user: User) {
      users[user.id] = user // Potential data race
   }
}
```

### Extensions

   * Use extensions to organize code by functionality
   * Use extensions to conform to protocols
   * Keep extensions focused on a single responsibility

```swift
// Good
extension User: Equatable {
   static func == (lhs: User, rhs: User) -> Bool {
      return lhs.id == rhs.id
   }
}

extension User: Codable {
   // Codable implementation
}

// Bad
extension User {
   // Mixed functionality
   static func == (lhs: User, rhs: User) -> Bool {
      return lhs.id == rhs.id
   }

   func encode(to encoder: Encoder) throws {
      // Encoding implementation
   }
}
```

## Memory Management

   * Avoid strong reference cycles with `weak` and `unowned`
   * Use value types (structs) when appropriate
   * Be explicit about capture lists in closures

```swift
// Good
class ProfileViewController {
   private weak var delegate: ProfileDelegate?

   func updateProfile() {
      networkService.fetch { [weak self] result in
         guard let self = self else { return }
         self.updateUI(with: result)
      }
   }
}
```

## Documentation

   * Document public interfaces with comments
   * Use `///` for documentation comments
   * Include parameter descriptions and return values

```swift
/// Calculates the total price including tax
///
/// - Parameters:
///   - items: The items to calculate the price for.
///   - taxRate: The tax rate to apply.
/// - Returns: The total price with tax applied
func calculateTotal(for items: [Item], taxRate: Double) -> Double {
   // Implementation
}
```

## Testing

   * Test all public interfaces
   * Name tests clearly
   * Follow the Arrange-Act-Assert pattern

### Swift Testing Framework (available in Swift 6+)

```swift
// Using Swift Testing framework
@Test func userCreation() {
   // Arrange
   let id = "123"
   let name = "John"

   // Act
   let user = User(id: id, name: name)

   // Assert
   #expect(user.id == id)
   #expect(user.name == name)
}
```

## Dependency Management

### Swift Package Manager

   * Always specify exact versions for dependencies
   * Avoid using version ranges or branch dependencies in production code

```swift
// Good - Exact version specified
dependencies: [
    .package(url: "https://github.com/apple/swift-log.git", exact: "1.4.2"),
    .package(url: "https://github.com/apple/swift-nio.git", exact: "2.31.2")
]

// Bad - Using version ranges
dependencies: [
    .package(url: "https://github.com/apple/swift-log.git", from: "1.0.0"),
    .package(url: "https://github.com/apple/swift-nio.git", .upToNextMajor(from: "2.0.0"))
]

// Bad - Using branch dependencies
dependencies: [
    .package(url: "https://github.com/apple/swift-log.git", .branch("main"))
]
```

### XCTest Framework (legacy framework)

```swift
// Using XCTest framework
func testUserCreation() {
   // Arrange
   let id = "123"
   let name = "John"

   // Act
   let user = User(id: id, name: name)

   // Assert
   XCTAssertEqual(user.id, id)
   XCTAssertEqual(user.name, name)
}
```
