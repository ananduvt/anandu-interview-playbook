# Exceptions

## Hierarchy
`Throwable` → `Error` (JVM, don't catch) and `Exception`.
- **Checked** (compile-time) — `IOException`, `SQLException`: must catch or declare.
- **Unchecked** (`RuntimeException`) — `NullPointerException`, `IllegalArgumentException`: programming errors.

## Handling
- `try / catch / finally`; **try-with-resources** auto-closes `AutoCloseable` (preferred over finally).
- **Multi-catch**: `catch (IOException | SQLException e)`.
- Catch specific before general; never swallow silently.
- Wrap and rethrow with context; preserve the cause (`throw new X("msg", e)`).

## Best practices
- Fail fast; validate at boundaries.
- Don't use exceptions for control flow.
- Custom exceptions for domain errors; extend `RuntimeException` unless the caller can recover.
- Clean up in `finally` / try-with-resources.

## final vs finally vs finalize
- **final** — keyword: unchangeable variable / non-overridable method / non-extendable class.
- **finally** — block that always runs after try/catch (cleanup).
- **finalize** — deprecated `Object` method called before GC; avoid — use try-with-resources / `Cleaner`.

## Spring
`@ControllerAdvice` + `@ExceptionHandler` centralize REST error mapping to status codes + error bodies.
