# Silvermine Coding Standards Specific to Kotlin

## Formatting

### Whitespace

   * One blank line between method definitions
   * No blank line after opening brace
   * No blank line before closing brace
   * One space after colon in type declarations
   * No space before colon in type declarations

   ```kotlin
   // Good:
   class MyClass {
       fun myFunction1() {
           // ...
       }

       fun myFunction2() {
           // ...
       }
   }

   val name: String = "John"

   // Bad:
   class MyClass{

       fun myFunction1() {
           // ...
       }
       fun myFunction2() {
           // ...
       }

   }

   val name : String = "John"
   ```

## Naming Conventions

### General

   The following conventions extend the general Silvermine naming standards:

   * **All functions** (including companion object functions) use `camelCase` — not
     `snake_case` as the general standards specify for static functions
   * **Private** and **protected** members do **not** use a leading underscore prefix —
     Kotlin's `private` and `protected` visibility modifiers are sufficient
   * **Constants** (`const val` or top-level `val`) use `UPPER_SNAKE_CASE`
   * **Variables** and **properties** use `camelCase`
   * **Classes**, **interfaces**, and **enums** use `PascalCase`
   * **Enum values** use `UPPER_SNAKE_CASE`
   * Use backticks with descriptive sentences for test functions

   ```kotlin
   // Good:
   val userAge = 30
   fun sendEmail(to: String) { /* ... */ }
   class UserProfile { /* ... */ }
   const val MAX_CONNECTIONS = 100
   @Test fun `test that user can be created`() { /* ... */ }

   companion object {
       fun fromJson(json: String): User { /* ... */ }
   }

   private val cache = mutableMapOf<String, User>()

   // Bad:
   val user_age = 30
   fun SendEmail(to: String) { /* ... */ }
   class userProfile { /* ... */ }
   const val max_connections = 100
   @Test fun testUserCreation() { /* ... */ }

   companion object {
       fun from_json(json: String): User { /* ... */ } // Don't use snake_case
   }

   private val _cache = mutableMapOf<String, User>() // Don't use underscore prefix
   ```

## Language Features

### Null Safety

   * Prefer non-nullable types over nullable types
   * Use the safe call operator (`?.`) or the Elvis operator (`?:`) to handle
     nullable types
   * Avoid the not-null assertion operator (`!!`) unless you are absolutely sure that
     the value is not null

   ```kotlin
   // Good:
   val name: String? = "John"
   val length = name?.length ?: 0

   // Bad:
   val name: String? = "John"
   val length = name!!.length
   ```

### Data Classes

   * Use data classes to represent simple data-holding classes
   * Keep data classes immutable by using `val` instead of `var`

   ```kotlin
   // Good:
   data class User(val name: String, val age: Int)

   // Bad:
   class User(var name: String, var age: Int)
   ```

### Collections

   * Prefer Kotlin collection factory functions (`listOf`, `setOf`, `mapOf`) over Java
     collection constructors (`ArrayList()`, `HashSet()`, `HashMap()`)
   * Prefer read-only collection types by default; use mutable variants (`mutableListOf`,
     `mutableSetOf`, `mutableMapOf`) only when mutation is required
   * Prefer read-only collection interfaces (`List`, `Set`, `Map`) in function signatures
     and return types

   ```kotlin
   // Good:
   val names: List<String> = listOf("Alice", "Bob")
   val mutableNames: MutableList<String> = mutableListOf("Alice", "Bob")
   mutableNames.add("Charlie")

   fun getUsers(): List<User> {
       // ...
   }

   // Bad:
   val names: ArrayList<String> = ArrayList()
   names.add("Alice")
   names.add("Bob")

   fun getUsers(): MutableList<User> {
       // ...
   }
   ```

### Coroutines

   * Use coroutines for asynchronous programming
   * Use `suspend` functions for long-running operations
   * Use `viewModelScope` or `lifecycleScope` to launch coroutines in Android

   ```kotlin
   // Good:
   viewModelScope.launch {
       val user = repository.getUser()
       // ...
   }
   ```

## Comments

   * Use KDoc for documenting public APIs

   ```kotlin
   // Good:
   /**
    * Returns the user with the given ID
    * @param userId The ID of the user to return
    * @return The user with the given ID, or null if not found
    */
   fun getUser(userId: String): User? {
       // ...
   }

   // Bad:
   // Gets the user
   fun getUser(userId: String): User? {
       // ...
   }
   ```

## Testing

   * Follow the given-when-then pattern for structuring tests.

   ```kotlin
   @Test
   fun `test that user can be created`() {
       // Given
       val user = User("John", 30)

       // When
       val result = repository.saveUser(user)

       // Then
       assertTrue(result)
   }
   ```
