# Spring Boot

## Core: IoC & DI
- The **IoC container** creates/wires/manages beans. **Dependency Injection** supplies collaborators.
- **Constructor injection** preferred (immutability, testability, no reflection on fields).
- Stereotypes: `@Component`, `@Service`, `@Repository`, `@Controller`/`@RestController`.

## Bean scopes & lifecycle
- **singleton** (default, must be **stateless**), prototype, request/session.
- Lifecycle: instantiate → inject → `@PostConstruct` → ready → `@PreDestroy`.

## Configuration
- `@Configuration` + `@Bean`; `@Value("${prop:default}")`; `@ConfigurationProperties` for grouped props.
- Profiles (`@Profile`, `application-<profile>.yml`); externalized config; `bootstrap.yml`.
- **Auto-configuration** + **starters** (opinionated dependency bundles).

## Web
- `@RestController`, `@GetMapping/@PostMapping`, `@PathVariable/@RequestBody/@RequestParam`, `ResponseEntity`.
- Global errors: `@ControllerAdvice` + `@ExceptionHandler`.

## AOP
Cross-cutting concerns (logging, tx, security) via proxies. JDK dynamic proxy (interface) vs CGLIB (class).
**Self-invocation bypasses the proxy.**

## Transactions
`@Transactional` — proxy commits on normal return, **rolls back on unchecked exceptions** by default
(`rollbackFor` for checked). Propagation (REQUIRED, REQUIRES_NEW), isolation levels.

## Boot 3
Java 17+ baseline, **Jakarta** namespace (`javax.*`→`jakarta.*`), Spring 6, `SecurityFilterChain` bean (no
`WebSecurityConfigurerAdapter`), observability via Micrometer.

## Resilience
No built-in circuit breaker — use **Resilience4j** (successor to the deprecated **Hystrix**): circuit breaker,
retry, rate limiter, bulkhead, time limiter. See [Scalability & Resilience](../system-design/scalability-resilience.md).

## Actuator
Health, metrics (`/actuator/health`, `/metrics`, `/loggers`), Prometheus endpoint for scraping.
