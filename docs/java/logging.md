# Logging

## Landscape
- **SLF4J** — logging facade (API). Code against it, bind an implementation at runtime.
- **Implementations**: **Logback** (default in Spring Boot), **Log4j2**, `java.util.logging`.
- `log4j-over-slf4j` / bridges route legacy logging through SLF4J.

## Levels
`TRACE < DEBUG < INFO < WARN < ERROR`. Set per-package; raise to DEBUG only when diagnosing.

## Best practices
- Use parameterized logging (avoids string building when disabled):
  ```java
  log.debug("user {} did {}", userId, action);
  ```
- **Correlation id** via **MDC** (`MDC.put("traceId", id)`) to trace a request across components/threads.
- **Never log secrets/PII** — mask sensitive fields (card numbers, tokens, emails).
- Structured/JSON logging for aggregation (ELK, Splunk, OpenSearch).
- Async appenders for throughput; don't log in tight loops.

## Spring Boot
- `application.yml` sets levels; Logback config in `logback-spring.xml`.
- Runtime level changes via Actuator `/loggers` endpoint.
