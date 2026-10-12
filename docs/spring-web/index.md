# Spring & Web

Master Spring Framework for building enterprise web applications: core concepts, dependency injection, REST APIs, security, database integration, and transaction management.

---

## Topics

### [Spring Boot](spring-boot.md)
**Spring Boot - IoC/DI - Bean Scopes - Annotations - Configuration - Profiles - Actuator**  
Spring fundamentals: dependency injection patterns (constructor first), bean lifecycle, component scopes (singleton, prototype, request, session), @SpringBootApplication, @Configuration, profiles for dev/prod environments.

### [Web Services](web-services.md)
**SOAP - WSDL - REST Principles - Web Services Standards - XML/JSON Serialization**  
Web service architectures: SOAP vs. REST trade-offs, WSDL contracts, RESTful resource design, content negotiation, versioning strategies.

### [API Styles & Protocols](api-styles.md)
**REST - GraphQL - gRPC - OpenAPI/Swagger - HATEOAS - Versioning - Status Codes**  
API design paradigms: REST conventions (GET/POST/PUT/DELETE/PATCH), GraphQL query language, gRPC binary protocol, API documentation with Swagger/OpenAPI, deprecation strategies.

### [Security](security.md)
**Authentication - Authorization - Spring Security - OAuth2 - JWT - SSL/TLS - CORS**  
API security: user authentication (session vs. token-based), role-based authorization, Spring Security configuration, OAuth2 flows, JWT validation, HTTPS/TLS, cross-origin handling.

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
