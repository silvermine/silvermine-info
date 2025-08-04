# Silvermine Coding Standards Specific to Kotlin

## Formatting

### Whitespace

   * No trailing whitespace
   * One blank line between method definitions
   * No blank line after opening brace
   * No blank line before closing brace
   * One space after colon in type declarations
   * No space before colon in type declarations
   * One newline at the end of the file

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

### Line Length

   * Break long lines at logical points
   * Indent continuation lines by our standard indentation

   ```kotlin
   // Good:
   val result = someFunctionWithAVeryLongName(
       parameter1,
       parameter2,
       parameter3
   )

   // Bad:
   val result = someFunctionWithAVeryLongName(parameter1, parameter2, parameter3)
   ```

## Naming Conventions

### General

   * Use camelCase for variable and function names
   * Use PascalCase for type names (classes, structs, enums)
   * Use `UPPER_SNAKE_CASE` for constants
   * Use backticks with descriptive sentences for test functions
   * Use descriptive names that convey meaning

   ```kotlin
   // Good:
   val userAge = 30
   fun sendEmail(to: String) { /* ... */ }
   class UserProfile { /* ... */ }
   const val MAX_CONNECTIONS = 100
   @Test fun `test that user can be created`() { /* ... */ }

   // Bad:
   val user_age = 30
   fun SendEmail(to: String) { /* ... */ }
   class userProfile { /* ... */ }
   const val max_connections = 100
   @Test fun testUserCreation() { /* ... */ }
   ```

## Language Features

### Null Safety

   * Prefer non-nullable types over nullable types.
   * Use the safe call operator (`?.`) or the Elvis operator (`?:`) to handle
     nullable types.
   * Avoid the not-null assertion operator (`!!`) unless you are absolutely sure that
     the value is not null.

   ```kotlin
   // Good:
   val name: String? = "John"
   val length = name?.length ?: 0

   // Bad:
   val name: String? = "John"
   val length = name!!.length
   ```

### Data Classes

   * Use data classes to represent simple data-holding classes.
   * Keep data classes immutable by using `val` instead of `var`.

   ```kotlin
   // Good:
   data class User(val name: String, val age: Int)

   // Bad:
   class User(var name: String, var age: Int)
   ```

### Coroutines

   * Use coroutines for asynchronous programming.
   * Use `suspend` functions for long-running operations.
   * Use `viewModelScope` or `lifecycleScope` to launch coroutines in Android.

   ```kotlin
   // Good:
   viewModelScope.launch {
       val user = repository.getUser()
       // ...
   }
   ```

## Comments

   * Write comments to explain *why* code is written a certain way, not *what* the
     code does.
   * Use KDoc for documenting public APIs.

   ```kotlin
   // Good:
   /**
    * Returns the user with the given ID.
    * @param userId The ID of the user to return.
    * @return The user with the given ID, or null if not found.
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
