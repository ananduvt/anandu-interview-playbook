# Spring Boot Versions and What Changed

Interviewers ask "what changed between Boot 2 and 3?" to check you can maintain and migrate real systems. Know the big breaks (Java baseline, `javax` to `jakarta`) and the headline features per release. Details below are general; confirm exact numbers on the [Spring Boot release notes](https://github.com/spring-projects/spring-boot/wiki) before quoting them.

## Version Timeline

| Line | Baseline | Spring Framework | Headline changes |
| :-- | :-- | :-- | :-- |
| Boot 2.x (2018-2023) | Java 8+, `javax.*` | 5.x | Actuator rewrite on Micrometer, WebFlux, layered jars, graceful shutdown (2.3) |
| Boot 3.0 (Nov 2022) | **Java 17+**, **`jakarta.*`** | 6.0 | Jakarta EE 9+/10, Hibernate 6, GraalVM native image support (AOT), Micrometer Observation, `ProblemDetail` (RFC 7807) |
| Boot 3.1-3.3 | Java 17+ (21 recommended) | 6.0 (3.1), 6.1 (3.2-3.3) | Docker Compose and Testcontainers `@ServiceConnection` (3.1), virtual threads, `RestClient`, `JdbcClient` (3.2) |
| Boot 3.4-3.5 | Java 17+ | 6.2 | Structured logging, `@MockitoBean` replaces `@MockBean` |
| Boot 4.0 (Nov 2025) | Java 17+ (Java 25 LTS supported) | 7.0 | Jakarta EE 11, modularised auto-configuration jars, JSpecify null-safety, Jackson 3 support, API versioning in MVC |

!!! note "Support windows"
    Open-source support for each minor line is short (roughly 12 to 18 months); the final minor of a major (2.7, 3.5) gets the longest tail. Upgrade to the latest minor of your current line first, then hop to the next major.

## Boot 2 to Boot 3 Migration Checklist

| Area | What breaks | Fix |
| :-- | :-- | :-- |
| Java | Boot 3 will not run on Java 8/11 | Upgrade the JDK to 17+ first |
| Namespace | `javax.servlet`, `javax.persistence`, `javax.validation` no longer used | Rename to `jakarta.*` (OpenRewrite has Boot 3 upgrade recipes) |
| Auto-config registration | `spring.factories` entry for `EnableAutoConfiguration` ignored | Use `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` |
| Spring Security | `WebSecurityConfigurerAdapter` removed (Security 6) | Declare a `SecurityFilterChain` bean; `authorizeRequests` becomes `authorizeHttpRequests` |
| Hibernate 6 | Stricter HQL, changed ID generation defaults | Run integration tests against a real DB (see [Spring Testing](spring-testing.md)) |
| Trailing slash | `/users/` no longer matches `/users` (Framework 6) | Be explicit in mappings or clients |
| Properties | Several renamed (e.g. `spring.redis.*` to `spring.data.redis.*`) | Add `spring-boot-properties-migrator` temporarily |
| Third-party libs | Anything still on `javax` breaks | Upgrade to Jakarta-compatible versions |

```java
// Boot 2.x
import javax.persistence.Entity;
import javax.servlet.http.HttpServletRequest;

// Boot 3.x
import jakarta.persistence.Entity;
import jakarta.servlet.http.HttpServletRequest;
```

## Features Worth Naming in an Interview

- **Virtual threads (3.2, needs Java 21+):** `spring.threads.virtual.enabled=true` runs Tomcat request handling and `@Async` on virtual threads. See [MVC vs WebFlux with virtual threads](spring-webflux.md#virtual-threads-vs-webflux).
- **GraalVM native images (3.0):** AOT processing at build time, faster startup, lower memory. See [Native and GraalVM](spring-native-graalvm.md).
- **Testcontainers `@ServiceConnection` (3.1):** removes `@DynamicPropertySource` boilerplate. See [Spring Testing](spring-testing.md).
- **`RestClient` / `JdbcClient` (3.2):** fluent synchronous HTTP client and JDBC helper.
- **Observability (3.0):** Micrometer Observation gives metrics and traces from one instrumentation. See [Actuator](spring-boot-actuator.md).
- **`ProblemDetail` (3.0):** standard error body, see [Exception handling](spring-exceptions.md).

## Common Interview Questions

- **Minimum Java version for Boot 3?** 17. Boot 2.7 and earlier supported Java 8.
- **Why did everything rename to `jakarta`?** Java EE moved to the Eclipse Foundation as Jakarta EE, which could not reuse the `javax` namespace; Jakarta EE 9 renamed the packages and Boot 3 adopted it.
- **How would you migrate a large Boot 2 service?** Move to 2.7 first, fix deprecations, upgrade the JDK to 17, run the OpenRewrite recipe, replace removed security APIs, run full tests, then canary deploy.
- **What replaced `spring.factories` for auto-configuration?** The `AutoConfiguration.imports` file ([details](spring-boot.md)).

**See also:** [Spring Boot](spring-boot.md), [Spring Boot Advanced](spring-boot-advanced.md)
