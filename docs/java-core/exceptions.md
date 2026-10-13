# Exception Handling

Comprehensive guide to Java exceptions, exception hierarchy, handling patterns, and best practices.

---

## Exception Hierarchy

```
Throwable
├── Error (JVM-level, don't catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   ├── VirtualMachineError
│   └── LinkageError
└── Exception
    ├── Checked (must catch or declare)
    │   ├── IOException
    │   ├── SQLException
    │   ├── ClassNotFoundException
    │   └── InterruptedException
    └── RuntimeException (unchecked, programming errors)
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IndexOutOfBoundsException
        └── ClassCastException
```

### Checked vs Unchecked

| Type | When | Examples | Handling |
|------|------|----------|----------|
| **Checked** | Compile-time enforced | IOException, SQLException, ClassNotFoundException | Must `catch` or declare `throws` |
| **Unchecked** | Runtime, programming errors | NullPointerException, IllegalArgumentException, IndexOutOfBoundsException | Optional to catch |
| **Error** | JVM-level, unrecoverable | OutOfMemoryError, StackOverflowError | Never catch |

---

## Exception Handling Patterns

### try / catch / finally

```java
try {
    // Code that may throw exception
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // Handle specific exception
    System.out.println("Error: " + e.getMessage());
} catch (Exception e) {
    // Catch more general exceptions (always last)
    System.out.println("Unexpected error: " + e);
} finally {
    // Always runs (cleanup resources)
    System.out.println("Cleanup");
}
```

### try-with-resources (Preferred)

Auto-closes `AutoCloseable` resources. Preferred over finally.

```java
try (FileReader fr = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(fr)) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    System.out.println("Error reading file: " + e.getMessage());
}
// Resources automatically closed
```

### Multi-catch

Handle multiple exception types in one catch block.

```java
try {
    // Risky operation
    Database.query();
} catch (IOException | SQLException | TimeoutException e) {
    // Handle all three
    logger.error("Operation failed", e);
    throw new ServiceException("Service unavailable", e);
}
```

### Catch Ordering

Always catch specific exceptions before general ones.

```java
try {
    parse();
} catch (NumberFormatException e) {
    // Specific: catches int parsing errors
    logger.error("Invalid number format", e);
} catch (IllegalArgumentException e) {
    // More general: parent class
    logger.error("Invalid argument", e);
} catch (Exception e) {
    // Most general: catches everything else
    logger.error("Unexpected error", e);
}
```

---

## Exception Propagation & Re-throwing

### Preserving the Cause

```java
try {
    databaseOperation();
} catch (SQLException e) {
    // ✅ Good: wrap with context and preserve cause
    throw new DataAccessException("Failed to fetch user", e);
}

// ❌ Bad: swallows original exception
catch (SQLException e) {
    throw new DataAccessException("Failed to fetch user");
}
```

### Wrapping and Re-throwing

```java
public User getUser(Long id) throws ServiceException {
    try {
        return userRepository.findById(id);
    } catch (SQLException e) {
        // Wrap checked exception in unchecked
        throw new ServiceException("Database error: " + e.getMessage(), e);
    }
}
```

---

## Custom Exceptions

### When to Create Custom Exceptions

- Domain-specific errors (e.g., `UserNotFoundException`, `InvalidOrderException`)
- Distinguish between different failure modes in your application
- Allow callers to handle specific errors differently

### Best Practice: Extend RuntimeException

```java
// ✅ Unchecked exception (preferred for most cases)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
    
    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Usage
public User getUserOrThrow(Long id) {
    if (id == null || id <= 0) {
        throw new UserNotFoundException("Invalid user ID: " + id);
    }
    // ...
}
```

### Custom Exception Hierarchy

```java
// Base exception
public class ApplicationException extends RuntimeException {
    public ApplicationException(String msg, Throwable cause) {
        super(msg, cause);
    }
}

// Domain-specific
public class UserException extends ApplicationException { }
public class UserNotFoundException extends UserException { }
public class UserAlreadyExistsException extends UserException { }

public class OrderException extends ApplicationException { }
public class InvalidOrderException extends OrderException { }
```

---

## Best Practices

### Do's

- ✅ **Fail fast**: Validate inputs early, throw early
- ✅ **Be specific**: Throw and catch specific exceptions
- ✅ **Preserve context**: Always include cause in re-throws
- ✅ **Use try-with-resources**: For auto-closeable resources
- ✅ **Log at the boundary**: Log once where you handle the exception
- ✅ **Provide meaningful messages**: Include context (IDs, values, state)

### Don'ts

- ❌ **Never silently swallow exceptions**: `catch (Exception e) { }` is dangerous
- ❌ **Don't use exceptions for control flow**: Don't throw to return early
- ❌ **Don't catch `Exception` or `Throwable`**: Too broad; catches programming errors
- ❌ **Don't ignore InterruptedException**: Re-interrupt the thread: `Thread.currentThread().interrupt()`
- ❌ **Don't throw generic `Exception`**: Use specific custom exceptions

### Example: Error Handling at Boundaries

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        try {
            User user = userService.getUser(id);
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            // Handle specific exception at boundary
            return ResponseEntity.status(404).body(null);
        } catch (ServiceException e) {
            logger.error("Service error fetching user " + id, e);
            return ResponseEntity.status(500).body(null);
        }
    }
}
```

---

## Java Keywords: final vs finally vs finalize

| Keyword | Usage | Example |
|---------|-------|---------|
| **final** | Modifier: unchangeable variable, non-overridable method, non-extendable class | `final int x = 10;` `final void method() { }` |
| **finally** | Block: always executes after try/catch (cleanup) | `try { } finally { close(); }` |
| **finalize** | Method: deprecated, called before GC (avoid) | Use try-with-resources instead |

---

## Spring Exception Handling

For Spring-specific exception handling with `@ControllerAdvice` and global error handling, see [Spring Exception Handling](../spring-web/spring-exceptions.md).

---

## Interview Talking Points

- **Checked vs Unchecked?** Checked: compile-time enforced (IOException); Unchecked: runtime (NullPointerException)
- **Why custom exceptions?** Domain-specific errors; easier testing and error handling
- **Preserve the cause?** Always pass original exception to new exception to preserve stack trace
- **try-with-resources vs finally?** try-with-resources auto-closes; simpler, no null checks needed
- **Multi-catch benefit?** Cleaner code; less repetition for similar error handling
- **Never silently catch?** Swallowing exceptions hides bugs; always log or re-throw with context
- **When throw vs when handle?** Throw: caller can recover; Handle: you can fix it locally
- **InterruptedException?** Always re-interrupt the thread: `Thread.currentThread().interrupt()`

---

**See also:** [Spring Exception Handling](../spring-web/spring-exceptions.md), [Java Core Basics](basics.md)
