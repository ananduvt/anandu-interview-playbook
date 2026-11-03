# Spring & Web

Master Spring Framework for building enterprise web applications: core concepts, dependency injection, REST APIs, security, database integration, and transaction management.

---

## Core

### [Spring Boot](spring-boot.md)
**Spring Boot - Auto-Configuration - Request Handling - Configuration Files - Starter Dependencies**  
Spring Boot fundamentals: @SpringBootApplication, @Configuration, @Bean, request handling annotations, basic property configuration, starter dependencies.

### [Spring Annotations & Components](spring-annotations-components.md)
**@Component - @Service - @Repository - @Controller - @RestController - Bean Lifecycle - Component Scanning - Qualifiers**  
Stereotype annotations and bean management: @Component, @Service, @Repository, @Controller vs @RestController, bean scopes (singleton/prototype), component scanning process, @Qualifier/@Primary disambiguation, real-world layered architecture.

### [Spring Boot Advanced](spring-boot-advanced.md)
**Dependency Injection - Bean Lifecycle - Application Lifecycle - Profiles - Environment Configuration - Conditional Beans**  
Advanced Spring concepts: DI patterns (constructor/setter/field), bean lifecycle hooks (@PostConstruct, @PreDestroy), application startup/shutdown flow, environment-specific profiles with examples, conditional bean creation.

### [Exception Handling](spring-exceptions.md)
**@ControllerAdvice - @ExceptionHandler - Global Error Handling - Validation Errors - HTTP Status Codes - Error Response DTO**  
Centralized exception handling: @ControllerAdvice for global handlers, @ExceptionHandler mapping exceptions to HTTP responses, validation error handling, custom exception hierarchies, async exception handling.

## Web & APIs

### [API Styles & Protocols](api-styles.md)
**REST - GraphQL - gRPC - WebSocket - Webhooks - HTTP Methods - Versioning - HATEOAS - OpenAPI/Swagger**  
API design: a quick chooser across API styles, HTTP methods deep-dive (GET/POST/PUT/PATCH/DELETE), status codes, REST best practices (versioning, pagination, filtering, HATEOAS), and API documentation with Swagger/OpenAPI.

### [Web Services](web-services.md)
**SOAP vs REST - GraphQL - Authentication Methods - SSO - RBAC/ABAC - OAuth2 Scopes**  
Web service protocols and access control: SOAP vs. REST trade-offs, authentication vs. authorization, GraphQL (queries, mutations, subscriptions, Spring Boot integration), API authentication mechanisms (Basic, API keys, JWT, OAuth 2.0, OIDC, mTLS, sessions), SSO, and authorization strategies.

### [WebFlux & Reactive](spring-webflux.md)
**WebFlux - Reactor (Mono/Flux) - WebClient - Reactive Data Access - WebFlux vs MVC - Testing**  
Non-blocking web stack: reactive programming fundamentals, WebFlux architecture and controllers, reactive data access, WebClient, error handling, WebFlux vs. Spring MVC, testing reactive code, performance and troubleshooting.

## Security

### [Security](security.md)
**Authentication - Authorization - Spring Security - OAuth2 - SSL/TLS - CORS - OWASP Top 10**  
API security fundamentals: user authentication, role-based authorization, Spring Security configuration, HTTPS/TLS, cross-origin handling, OWASP defenses.

### [JWT & OAuth2](jwt-oauth.md)
**JWT Tokens - OAuth2 Flows - Token Refresh - Role-Based Access - Authentication Filters - Security Best Practices**  
Token-based authentication: JWT structure and validation, OAuth2 grant types, implementation with Spring Security, refresh token patterns, token blacklisting, multi-tenancy, security best practices.

## Data & Production

### [JPA & ORM](jpa-orm.md)
**JPA - Hibernate - Entity Mapping - Relationships - Query Language - Transactions - Lazy/Eager Loading**  
Object-relational mapping: entity annotations (@Entity, @Column, @OneToMany), JPQL/HQL queries, transaction propagation (@Transactional), N+1 problem, eager vs. lazy loading.

### [Spring Boot Circuit Breaker](spring-boot-circuit-breaker.md)
**Circuit Breaker - Resilience4j - Hystrix - Failsafe - Sentinel - Vavr - Fault Tolerance - Retry/Timeout/Bulkhead**  
Fault tolerance patterns: circuit breaker states (CLOSED/OPEN/HALF_OPEN), Resilience4j implementation with examples, framework comparison (Hystrix, Vavr, Failsafe, Sentinel), combining patterns, migration guide.

### [Spring Threading & Async](spring-threading.md)
**@Async - CompletableFuture - TaskExecutor - Virtual Threads - Non-Blocking REST - Async Patterns**  
Asynchronous processing: @Async annotation with TaskExecutor configuration, CompletableFuture composition and chaining, REST endpoints returning futures, Java 21 virtual threads integration, comparison with WebFlux, production patterns.

### [Actuator & Monitoring](spring-boot-actuator.md)
**Actuator - Health Checks - Metrics - Liveness/Readiness Probes - Prometheus/Grafana - Endpoint Security**  
Production monitoring: built-in endpoints, configuration, Kubernetes liveness and readiness probes, Prometheus and Grafana integration, securing actuator endpoints, best practices.

---

## Quick Interview Prep

**Spring Boot essentials:**
- **Dependency Injection**: Constructor injection (preferred) → Setter → Field
- **Bean Scopes**: Singleton (default, shared), Prototype (new instance per request), Request/Session (web-scoped)
- **Annotations**: @Component, @Service, @Repository stereotypes; @Autowired for injection; @Value for properties
- **Profiles**: application.yml for defaults, application-dev.yml, application-prod.yml for environment-specific config

**REST API best practices:**
- **HTTP Methods**: GET (safe, idempotent), POST (create), PUT (replace, idempotent), PATCH (partial), DELETE (remove, idempotent)
- **Status Codes**: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Server Error
- **Versioning**: URL path (/v1/users), header (Accept: application/vnd.company.v1+json), or query param

**Security patterns:**
- **Authentication**: Session-based (server state) vs. JWT (stateless tokens)
- **Authorization**: Role-based (ROLE_USER, ROLE_ADMIN), @PreAuthorize("hasRole('ADMIN')")
- **OAuth2**: Authorization Code flow for third-party apps, Bearer token validation
