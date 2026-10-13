# Spring & Web

Master Spring Framework for building enterprise web applications: core concepts, dependency injection, REST APIs, security, database integration, and transaction management.

---

## Topics

### [Spring Boot](spring-boot.md)
**Spring Boot - Auto-Configuration - Request Handling - Configuration Files - Starter Dependencies**  
Spring Boot fundamentals: @SpringBootApplication, @Configuration, @Bean, request handling annotations, basic property configuration, starter dependencies.

### [Spring Annotations & Components](spring-annotations-components.md)
**@Component - @Service - @Repository - @Controller - @RestController - Bean Lifecycle - Component Scanning - Qualifiers**  
Stereotype annotations and bean management: @Component, @Service, @Repository, @Controller vs @RestController, bean scopes (singleton/prototype), component scanning process, @Qualifier/@Primary disambiguation, real-world layered architecture.

### [Spring Boot Advanced](spring-boot-advanced.md)
**Dependency Injection - Bean Lifecycle - Application Lifecycle - Profiles - Environment Configuration - Conditional Beans**  
Advanced Spring concepts: DI patterns (constructor/setter/field), bean lifecycle hooks (@PostConstruct, @PreDestroy), application startup/shutdown flow, environment-specific profiles with examples, conditional bean creation.

### [Spring Boot Circuit Breaker](spring-boot-circuit-breaker.md)
**Circuit Breaker - Resilience4j - Hystrix - Failsafe - Sentinel - Vavr - Fault Tolerance - Retry/Timeout/Bulkhead**  
Fault tolerance patterns: circuit breaker states (CLOSED/OPEN/HALF_OPEN), Resilience4j implementation with examples, framework comparison (Hystrix, Vavr, Failsafe, Sentinel), combining patterns, migration guide.

### [Spring Threading & Async](spring-threading.md)
**@Async - CompletableFuture - TaskExecutor - Virtual Threads - Non-Blocking REST - Async Patterns**  
Asynchronous processing: @Async annotation with TaskExecutor configuration, CompletableFuture composition and chaining, REST endpoints returning futures, Java 21 virtual threads integration, comparison with WebFlux, production patterns.

### [Web Services](web-services.md)
**SOAP - WSDL - REST Principles - Web Services Standards - XML/JSON Serialization**  
Web service architectures: SOAP vs. REST trade-offs, WSDL contracts, RESTful resource design, content negotiation, versioning strategies.

### [API Styles & Protocols](api-styles.md)
**REST - GraphQL - gRPC - OpenAPI/Swagger - HATEOAS - Versioning - Status Codes**  
API design paradigms: REST conventions (GET/POST/PUT/DELETE/PATCH), GraphQL query language, gRPC binary protocol, API documentation with Swagger/OpenAPI, deprecation strategies.

### [Exception Handling](spring-exceptions.md)
**@ControllerAdvice - @ExceptionHandler - Global Error Handling - Validation Errors - HTTP Status Codes - Error Response DTO**  
Centralized exception handling: @ControllerAdvice for global handlers, @ExceptionHandler mapping exceptions to HTTP responses, validation error handling, custom exception hierarchies, async exception handling.

### [Security](security.md)
**Authentication - Authorization - Spring Security - OAuth2 - SSL/TLS - CORS - OWASP Top 10**  
API security fundamentals: user authentication, role-based authorization, Spring Security configuration, HTTPS/TLS, cross-origin handling, OWASP defenses.

### [JWT & OAuth2](jwt-oauth.md)
**JWT Tokens - OAuth2 Flows - Token Refresh - Role-Based Access - Authentication Filters - Security Best Practices**  
Token-based authentication: JWT structure and validation, OAuth2 grant types, implementation with Spring Security, refresh token patterns, token blacklisting, multi-tenancy, security best practices.

### [JPA & ORM](jpa-orm.md)
**JPA - Hibernate - Entity Mapping - Relationships - Query Language - Transactions - Lazy/Eager Loading**  
Object-relational mapping: entity annotations (@Entity, @Column, @OneToMany), JPQL/HQL queries, transaction propagation (@Transactional), N+1 problem, eager vs. lazy loading.

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
