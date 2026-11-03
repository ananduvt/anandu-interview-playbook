# Security Questions

Verbal interview questions on application security, secrets, rate limiting, compliance and Spring Security.

---

## 1. How do you implement security in your Java projects?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Layered defense: authenticate and authorize every request, protect data in transit and at rest, validate input, manage secrets and dependencies, and log security events. Use the OWASP Top 10 as the checklist.</p>

<h3>Checklist</h3>
<ul>
<li><strong>Authentication:</strong> Spring Security with OAuth2/OIDC or JWT; passwords hashed with bcrypt/Argon2 (never reversible encryption); MFA for sensitive access.</li>
<li><strong>Authorization:</strong> deny by default, role/permission checks on the server (<code>@PreAuthorize</code>), object-level checks to stop access to other users' data.</li>
<li><strong>Input and injection:</strong> prepared statements/JPA parameters, bean validation, output encoding; never build SQL by string concatenation.</li>
<li><strong>Encryption:</strong> TLS everywhere; encrypt sensitive columns or disks; keys in a KMS.</li>
<li><strong>Secrets:</strong> no secrets in code or git; use a secret manager.</li>
<li><strong>Dependencies:</strong> scan for CVEs (OWASP Dependency-Check, Snyk), keep libraries patched.</li>
<li><strong>Web protections:</strong> CSRF for cookie sessions, CORS allow-list, security headers (CSP, HSTS), rate limiting.</li>
<li><strong>Logging and monitoring:</strong> audit logins and access failures; never log passwords, tokens or PII.</li>
<li><strong>Safe errors:</strong> no stack traces or internals to clients.</li>
</ul>

<h3>Example</h3>

```java
@Bean
SecurityFilterChain chain(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(a -> a
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated())
        .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
        .build();
}
```

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 2. How do you maintain logs in your Java projects?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use an SLF4J facade with Logback (or Log4j2), structured output, correct levels, correlation IDs, and ship logs to a central platform.</p>

<h3>Practices</h3>
<ul>
<li><strong>Framework:</strong> code against SLF4J; Spring Boot defaults to Logback.</li>
<li><strong>Levels:</strong> ERROR (needs action), WARN (unexpected but handled), INFO (business events), DEBUG/TRACE (off in production).</li>
<li><strong>Parameterized messages:</strong> <code>log.info("Order {} created", id)</code>, not string concatenation.</li>
<li><strong>Structured (JSON) logs</strong> with a correlation/trace ID in MDC, so one request can be followed across services.</li>
<li><strong>Management:</strong> rolling file appenders with size/time limits, or log to stdout in containers; centralize (ELK/EFK, Splunk, Loki, CloudWatch); set retention.</li>
<li><strong>Performance:</strong> async appenders, avoid logging in tight loops, guard expensive debug calls.</li>
<li><strong>Security:</strong> mask PII and secrets; protect log access; guard against log injection (sanitize user input with newlines).</li>
<li><strong>Dependencies:</strong> keep the logging library patched (Log4Shell lesson).</li>
</ul>

<h3>Example</h3>

```java
private static final Logger log = LoggerFactory.getLogger(OrderService.class);

MDC.put("traceId", traceId);
try {
    log.info("Processing order {}", orderId);
} catch (Exception e) {
    log.error("Order {} failed", orderId, e);  // exception last, so the stack trace is kept
} finally {
    MDC.clear();
}
```

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging & Performance</a></p>

</details>

---

## 3. What is a secret manager and how to read data from a secret manager?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A secret manager is a dedicated service for storing, encrypting, rotating and auditing sensitive values (passwords, API keys, certificates). Examples: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager.</p>

<h3>Why Use One</h3>
<ul>
<li>Central, encrypted storage with fine-grained access control and audit logs.</li>
<li>Rotation and short-lived or dynamic credentials.</li>
<li>Keeps secrets out of source control, images and config files.</li>
</ul>

<h3>How an App Reads Secrets</h3>
<ul>
<li>The app authenticates using a workload identity (IAM role, managed identity, Kubernetes service account), not a hard-coded password.</li>
<li>It fetches the secret at startup (or on refresh) via SDK/REST, or via Spring Cloud integration so it appears as a normal property.</li>
<li>Alternative: a sidecar/CSI driver mounts secrets as files or env vars.</li>
</ul>

<h3>Example (AWS SDK v2)</h3>

```java
try (SecretsManagerClient client = SecretsManagerClient.create()) {
    String json = client.getSecretValue(
        GetSecretValueRequest.builder().secretId("prod/db-credentials").build())
        .secretString();
    // parse JSON for username/password, build the DataSource
}
```

<h3>Gotchas</h3>
<ul>
<li>Cache values and handle rotation (refresh the connection pool when credentials change).</li>
<li>Do not log secrets; grant least privilege per application.</li>
<li>You still need a bootstrap identity; use platform identity rather than a stored token.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 4. How do you manage all the secrets in a Java application?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Never commit secrets. Keep them outside the code, inject at runtime from the safest source available, and rotate them. The ladder from weakest to strongest is: property files in git, environment variables, mounted secrets, then a secret manager.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Approach</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Properties in repo</td><td>Simple</td><td>Leaks via git history; avoid for secrets</td></tr>
<tr><td>Environment variables</td><td>12-factor, easy in containers</td><td>Visible to process inspection, dumps and crash reports; no rotation or audit</td></tr>
<tr><td>Kubernetes Secrets / mounted files</td><td>Per-pod access, file permissions</td><td>Base64 only unless encryption at rest is enabled</td></tr>
<tr><td>Secret manager (Vault, cloud)</td><td>Audit, rotation, dynamic creds, access policies</td><td>Extra infrastructure and a bootstrap identity</td></tr>
</table>

<h3>Practices</h3>
<ul>
<li>Spring: reference placeholders (<code>${DB_PASSWORD}</code>); use profiles per environment; Spring Cloud Vault or cloud config integrations.</li>
<li>Separate secrets per environment; least privilege; rotate and revoke on exposure.</li>
<li>Prevent leaks: secret scanning in CI and pre-commit, <code>.gitignore</code>, mask in logs and Actuator (<code>/env</code>, <code>/configprops</code> sanitization).</li>
<li>Encrypt config values (Jasypt or similar) only as a stopgap; the key must still be managed.</li>
</ul>

<h3>Gotcha</h3>
<p>If a secret was ever committed, rotate it; deleting the commit is not enough.</p>

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 5. What is rate limiting and how to restrict API access?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Rate limiting caps how many requests a client can make in a time window, protecting the service from abuse, brute force, and overload, and ensuring fair use. Exceeding the limit returns <strong>429 Too Many Requests</strong> (with a <code>Retry-After</code> header).</p>

<h3>Algorithms</h3>
<table>
<tr><th>Algorithm</th><th>Idea</th><th>Trade-off</th></tr>
<tr><td>Fixed window</td><td>Counter per window (e.g., 100/min)</td><td>Simple; bursts at window edges</td></tr>
<tr><td>Sliding window (log or counter)</td><td>Counts over a moving window</td><td>Smoother; more memory/compute</td></tr>
<tr><td>Token bucket</td><td>Tokens refill at a rate; each request spends one</td><td>Allows controlled bursts; most common</td></tr>
<tr><td>Leaky bucket</td><td>Requests drain at a constant rate</td><td>Smooths output; queues or drops bursts</td></tr>
</table>

<h3>Where to Enforce</h3>
<ul>
<li><strong>API gateway / edge:</strong> Spring Cloud Gateway <code>RequestRateLimiter</code> (Redis-backed), NGINX, Kong, cloud API gateways.</li>
<li><strong>In the app:</strong> libraries such as Bucket4j or Resilience4j <code>RateLimiter</code>.</li>
<li><strong>Distributed state:</strong> Redis (atomic increments or Lua scripts) so limits hold across instances.</li>
</ul>

<h3>Design Choices</h3>
<ul>
<li>Key by API key, user, or IP; different tiers per plan; stricter on login endpoints.</li>
<li>Combine with authentication, quotas, payload size limits, and timeouts.</li>
</ul>

<h3>Gotcha</h3>
<p>Per-instance in-memory limits multiply with instance count; use a shared store for a true global limit.</p>

<p>📚 <a href="/anandu-interview-playbook/system-design/building-blocks/">Learn more: System Design Building Blocks</a></p>

</details>

---

## 6. How do you ensure secure data handling and compliance with financial regulations?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Combine technical controls (encryption, access control, auditing, secure coding) with process controls (policies, reviews, evidence) mapped to the applicable standards. The standards depend on the domain, for example PCI DSS for card data, SOX for financial reporting, GDPR for personal data.</p>

<h3>Technical Controls</h3>
<ul>
<li><strong>Encryption:</strong> TLS in transit; AES-based encryption at rest; keys in a KMS/HSM with rotation.</li>
<li><strong>Data minimization:</strong> do not store what you do not need; mask or tokenize sensitive fields; never log them.</li>
<li><strong>RBAC and least privilege</strong>, segregation of duties, MFA for privileged access.</li>
<li><strong>Audit trails:</strong> immutable record of who did what and when for sensitive actions.</li>
<li><strong>Secure SDLC:</strong> code review, SAST/DAST, dependency scanning, secrets scanning, tested backups.</li>
<li><strong>Money handling:</strong> <code>BigDecimal</code>, idempotent operations, reconciliation.</li>
</ul>

<h3>Process Controls</h3>
<ul>
<li>Change management and approvals, documented evidence for audits.</li>
<li>Incident response plan, data retention and deletion rules.</li>
<li>Regular penetration tests and access reviews.</li>
</ul>

<h3>Gotcha</h3>
<p>Compliance is not the same as security: a passed audit proves controls existed at a point in time. Automate evidence collection and keep monitoring continuous.</p>

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 7. Can you give examples of security measures you implemented in a past project?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Pick two or three concrete, truthful measures and describe the risk, what you did, and the result. Below is a generic pattern; replace it with your own real experience.</p>

<h3>Example Measures</h3>
<ul>
<li><strong>Stateless authentication:</strong> moved from session cookies to OAuth2/JWT bearer tokens validated at the gateway; short token lifetimes with refresh tokens.</li>
<li><strong>Fine-grained authorization:</strong> method-level <code>@PreAuthorize</code> and ownership checks, which fixed an insecure direct object reference found in a review.</li>
<li><strong>Injection prevention:</strong> replaced concatenated SQL with parameterized queries; added input validation.</li>
<li><strong>Secrets:</strong> moved credentials from property files to a secret manager and enabled secret scanning in CI.</li>
<li><strong>Dependency hygiene:</strong> added automated CVE scanning and patched a vulnerable library.</li>
<li><strong>Hardening:</strong> TLS, security headers, rate limiting on login, audit logging for sensitive operations, masking of PII in logs.</li>
</ul>

<h3>How to Structure the Answer</h3>
<ol>
<li>Situation and risk (what could go wrong).</li>
<li>Action (specific technology and decision).</li>
<li>Result (finding closed, scan clean, incident avoided) and what you learned.</li>
</ol>

<h3>Tip</h3>
<p>Be ready for follow-ups (why JWT over sessions, how revocation works, how you tested it). Do not name clients or reveal internal details.</p>

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 8. How do you stay updated with evolving financial regulations?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>As a developer, you rely on a mix of organizational channels and personal habits: compliance partnership, regulatory change tracking, training, and building systems that are easy to adapt.</p>

<h3>Practices</h3>
<ul>
<li><strong>Work with compliance and legal teams:</strong> they interpret regulations; engineers translate them into requirements and acceptance criteria.</li>
<li><strong>Follow primary sources:</strong> regulator and standards-body publications, release notes (for example PCI DSS versions), industry newsletters and vendor advisories.</li>
<li><strong>Mandatory training:</strong> internal compliance and security courses; certifications where relevant.</li>
<li><strong>Change tracking:</strong> regulatory changes tracked as backlog items with owners and deadlines; impact analysis on affected systems.</li>
<li><strong>Design for change:</strong> externalize rules and thresholds into configuration, keep audit logs and data lineage, version rules with effective dates.</li>
<li><strong>Verification:</strong> automated compliance tests, periodic audits and reviews, post-incident learning.</li>
</ul>

<h3>Tip</h3>
<p>Show that you treat regulation as a requirement stream, not an afterthought: traceability from regulation to requirement to test.</p>

<p>📚 <a href="/anandu-interview-playbook/security/">Learn more: Vulnerabilities & Secure Practices</a></p>

</details>

---

## 9. Explain the fundamentals of Spring Security.

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Spring Security is a framework that secures Spring applications through a chain of servlet filters that handle authentication (who are you) and authorization (what may you do), plus protections like CSRF and security headers.</p>

<h3>Core Concepts</h3>
<ul>
<li><strong>SecurityFilterChain:</strong> an ordered list of filters; every request passes through it before reaching controllers (entry point is <code>DelegatingFilterProxy</code>/<code>FilterChainProxy</code>).</li>
<li><strong>Authentication:</strong> object representing the principal and its credentials/authorities.</li>
<li><strong>AuthenticationManager / AuthenticationProvider:</strong> verify credentials (for example <code>DaoAuthenticationProvider</code>).</li>
<li><strong>UserDetailsService:</strong> loads the user; <strong>PasswordEncoder</strong> (bcrypt) verifies passwords.</li>
<li><strong>SecurityContext / SecurityContextHolder:</strong> holds the current authenticated user (thread-local by default).</li>
<li><strong>Authorization:</strong> URL rules (<code>authorizeHttpRequests</code>) and method security (<code>@PreAuthorize</code> with <code>@EnableMethodSecurity</code>).</li>
<li><strong>Authorities vs roles:</strong> roles are authorities prefixed with <code>ROLE_</code>.</li>
</ul>

<h3>Request Flow</h3>
<ol>
<li>Filter extracts credentials (form, Basic header, JWT).</li>
<li>AuthenticationManager authenticates and returns a populated Authentication.</li>
<li>Stored in the SecurityContext; authorization filter checks access rules.</li>
<li>Failures give 401 (unauthenticated) or 403 (forbidden) via entry point / access denied handler.</li>
</ol>

<h3>Gotchas</h3>
<ul>
<li>Since Spring Security 5.7/6, configure a <code>SecurityFilterChain</code> bean (the <code>WebSecurityConfigurerAdapter</code> is removed).</li>
<li>CSRF protection is on by default; it is usually disabled only for stateless token APIs.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/security/">Learn more: Security (Spring & Web)</a></p>

</details>

---

## 10. How do you implement Spring Security in a Spring Boot REST API?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Add the starter, define a stateless <code>SecurityFilterChain</code> that authenticates a token (JWT from an authorization server, or Basic for simple cases), authorize by URL and method, and return proper 401/403 JSON.</p>

<h3>Steps</h3>
<ol>
<li>Add <code>spring-boot-starter-security</code> (and <code>spring-boot-starter-oauth2-resource-server</code> for JWT).</li>
<li>Configure the filter chain: stateless sessions, CSRF disabled for token auth, route rules.</li>
<li>Provide a <code>PasswordEncoder</code> if you store users; otherwise configure the issuer URI for JWT validation.</li>
<li>Map claims to authorities; secure methods with <code>@PreAuthorize</code>.</li>
<li>Add CORS config, exception handling, and tests (<code>@WithMockUser</code>, <code>spring-security-test</code>).</li>
</ol>

<h3>Example</h3>

```java
@Configuration
@EnableMethodSecurity
class SecurityConfig {
    @Bean
    SecurityFilterChain api(HttpSecurity http) throws Exception {
        return http
            .csrf(c -> c.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(a -> a
                .requestMatchers("/auth/**", "/actuator/health").permitAll()
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
            .build();
    }

    @Bean PasswordEncoder encoder() { return new BCryptPasswordEncoder(); }
}
```

<h3>Gotchas</h3>
<ul>
<li>Order matters: put specific matchers before <code>anyRequest()</code>.</li>
<li>Disabling CSRF is only safe when you do not rely on cookies for auth.</li>
<li>Never expose Actuator endpoints unauthenticated beyond health.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/security/">Learn more: Security (Spring & Web)</a></p>

</details>

---

## 11. How do you implement Spring Security across distributed microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Centralize authentication in an identity provider (OAuth2/OIDC authorization server such as Keycloak, Okta, Spring Authorization Server). Services act as stateless resource servers that validate JWT access tokens and enforce authorization themselves.</p>

<h3>Architecture</h3>
<ul>
<li><strong>Identity provider:</strong> issues short-lived access tokens (and refresh tokens) after login (Authorization Code with PKCE for users, Client Credentials for service-to-service).</li>
<li><strong>API gateway:</strong> single entry point; validates tokens, applies rate limiting and coarse rules, and forwards the token.</li>
<li><strong>Each microservice:</strong> a resource server; validates signature, issuer, audience and expiry (using the issuer's JWKS public keys), then performs its own fine-grained authorization (zero trust: do not trust the network or the gateway alone).</li>
<li><strong>Token propagation:</strong> forward the user's token on downstream calls, or use token exchange or client credentials for service identity.</li>
<li><strong>Transport:</strong> TLS, and mutual TLS (service mesh) for service-to-service.</li>
<li><strong>Secrets and keys:</strong> in a secret manager; rotate signing keys through JWKS.</li>
</ul>

<h3>Example</h3>

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://idp.example.com/realms/main
```

<h3>Gotchas</h3>
<ul>
<li>JWTs cannot be revoked easily: keep lifetimes short, or use introspection for sensitive operations.</li>
<li>Keep claims small and non-sensitive; they are only signed, not encrypted.</li>
<li>Use shared libraries/config so every service applies the same validation rules.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jwt-oauth/">Learn more: JWT & OAuth</a></p>

</details>

---
