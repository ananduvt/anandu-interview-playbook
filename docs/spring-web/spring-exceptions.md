# Spring Exception Handling

Centralized error handling with `@ControllerAdvice`, global exception mappers, and REST error responses.

---

## Global Exception Handler with @ControllerAdvice

Centralize exception handling across all controllers.

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    // Handle specific domain exception
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            404,
            e.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(404).body(error);
    }
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException e) {
        String message = e.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(err -> err.getField() + ": " + err.getDefaultMessage())
            .collect(Collectors.joining(", "));
        
        ErrorResponse error = new ErrorResponse(400, message, System.currentTimeMillis());
        return ResponseEntity.status(400).body(error);
    }
    
    // Handle generic runtime exceptions
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseEntity<ErrorResponse> handleRuntimeException(RuntimeException e) {
        logger.error("Unexpected error", e);
        ErrorResponse error = new ErrorResponse(
            500,
            "Internal server error",
            System.currentTimeMillis()
        );
        return ResponseEntity.status(500).body(error);
    }
    
    // Handle all other exceptions (catch-all)
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception e) {
        logger.error("Unhandled exception", e);
        ErrorResponse error = new ErrorResponse(
            500,
            "An unexpected error occurred",
            System.currentTimeMillis()
        );
        return ResponseEntity.status(500).body(error);
    }
}
```

---

## Error Response DTO

Standard error response format for all endpoints.

```java
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;
    
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String path;
    
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private Map<String, String> errors;
}
```

Usage with path:

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(
            UserNotFoundException e,
            HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse(
            404,
            e.getMessage(),
            System.currentTimeMillis(),
            request.getRequestURI(),
            null
        );
        return ResponseEntity.status(404).body(error);
    }
}
```

---

## @ExceptionHandler on Individual Controllers

Handle exceptions specific to one controller.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUser(id));
    }
    
    // Exception handler only for this controller
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException e) {
        ErrorResponse error = new ErrorResponse(404, e.getMessage(), System.currentTimeMillis());
        return ResponseEntity.status(404).body(error);
    }
}
```

---

## RestControllerAdvice vs ControllerAdvice

```java
// ✅ For REST APIs (auto-converts to JSON)
@RestControllerAdvice
public class RestExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException e) {
        return new ErrorResponse(404, e.getMessage(), System.currentTimeMillis());
        // Automatically serialized to JSON
    }
}

// For MVC views (returns view names)
@ControllerAdvice
public class MvcExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public String handleUserNotFound(UserNotFoundException e, Model model) {
        model.addAttribute("error", e.getMessage());
        return "error-page";  // Returns view name
    }
}
```

---

## Validation Exceptions

Handle `@Valid` and `@Validated` annotation errors.

```java
@RestControllerAdvice
public class ValidationExceptionHandler {
    
    // Request body validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(MethodArgumentNotValidException e) {
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach(err ->
            errors.put(err.getField(), err.getDefaultMessage())
        );
        
        ErrorResponse error = new ErrorResponse(400, "Validation failed", System.currentTimeMillis());
        error.setErrors(errors);
        return error;
    }
    
    // Path variable / query param validation errors
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(ConstraintViolationException e) {
        Map<String, String> errors = new HashMap<>();
        e.getConstraintViolations().forEach(cv ->
            errors.put(cv.getPropertyPath().toString(), cv.getMessage())
        );
        
        ErrorResponse error = new ErrorResponse(400, "Validation failed", System.currentTimeMillis());
        error.setErrors(errors);
        return error;
    }
}
```

---

## Custom Exception with HTTP Status

```java
// Custom exception with status code
public abstract class AppException extends RuntimeException {
    public abstract HttpStatus getStatus();
    
    public AppException(String message) {
        super(message);
    }
    
    public AppException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class UserNotFoundException extends AppException {
    @Override
    public HttpStatus getStatus() {
        return HttpStatus.NOT_FOUND;
    }
    
    public UserNotFoundException(Long userId) {
        super("User not found: " + userId);
    }
}

public class InvalidOrderException extends AppException {
    @Override
    public HttpStatus getStatus() {
        return HttpStatus.BAD_REQUEST;
    }
    
    public InvalidOrderException(String reason) {
        super("Invalid order: " + reason);
    }
}

// Handler using status from exception
@RestControllerAdvice
public class AppExceptionHandler {
    
    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleAppException(AppException e) {
        ErrorResponse error = new ErrorResponse(
            e.getStatus().value(),
            e.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(e.getStatus()).body(error);
    }
}
```

---

## Async Exception Handling

Handle exceptions from `@Async` methods.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.initialize();
        return executor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) -> {
            System.err.println("Async method " + method.getName() + " threw exception: " + ex.getMessage());
            ex.printStackTrace();
        };
    }
}

// Using Future to catch async exceptions
@Service
public class AsyncService {
    
    @Async
    public CompletableFuture<String> asyncOperation() {
        try {
            // Async work
            if (something_fails) {
                throw new ServiceException("Operation failed");
            }
            return CompletableFuture.completedFuture("Success");
        } catch (Exception e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}

// Handle in controller
@RestController
public class AsyncController {
    
    @Autowired
    private AsyncService asyncService;
    
    @GetMapping("/async-operation")
    public CompletableFuture<ResponseEntity<String>> startAsync() {
        return asyncService.asyncOperation()
            .thenApply(result -> ResponseEntity.ok(result))
            .exceptionally(ex -> ResponseEntity.status(500).build());
    }
}
```

---

## Exception Handling Best Practices

### Do's

- ✅ **Use @ControllerAdvice**: Centralize exception handling
- ✅ **Return structured error responses**: Consistent format (status, message, timestamp)
- ✅ **Log exceptions**: Log with full context at handler level
- ✅ **Catch specific exceptions first**: Order handlers from specific to general
- ✅ **Use appropriate HTTP status codes**: 400 Bad Request, 404 Not Found, 500 Server Error
- ✅ **Preserve exception cause**: Pass `Throwable cause` to show full stack

### Don't's

- ❌ **Don't silently catch and ignore**: Always log or handle
- ❌ **Don't leak implementation details**: Hide internal exception messages in responses
- ❌ **Don't return sensitive data**: Don't expose system paths, SQL queries, etc.
- ❌ **Don't use exceptions for control flow**: E.g., don't throw to signal "not found"

---

## Interview Talking Points

- **@ControllerAdvice vs @RestControllerAdvice?** @ControllerAdvice for views (returns view name); @RestControllerAdvice for JSON (auto-serializes)
- **Where to handle exceptions?** At the boundary (controllers); let exceptions bubble up from services
- **HTTP status codes?** 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Server Error
- **Preserve the cause?** Always include original exception in re-throws to maintain stack trace
- **Async exception handling?** Use CompletableFuture or AsyncUncaughtExceptionHandler
- **Validation errors?** @Valid triggers MethodArgumentNotValidException; catch and return field-level errors
- **Custom exceptions?** Extend RuntimeException; include HTTP status; throw with context

---

**See also:** [Java Exception Handling](../java-core/exceptions.md), [Spring Boot Fundamentals](spring-boot.md), [Security & Validation](security.md)
