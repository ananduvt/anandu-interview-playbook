# JWT & OAuth2 Authentication

Token-based authentication, JWT structure, implementation patterns, and security considerations.

---

## JWT Fundamentals

**JWT (JSON Web Token)** — stateless, cryptographically signed token containing user claims.

### Structure: Header.Payload.Signature

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

1. **Header** (Base64): Algorithm and token type
   ```json
   { "alg": "HS256", "typ": "JWT" }
   ```

2. **Payload** (Base64): User claims (subject, name, roles, expiry, etc.)
   ```json
   {
     "sub": "1234567890",
     "name": "John Doe",
     "roles": ["ADMIN", "USER"],
     "iat": 1516239022,
     "exp": 1516242622
   }
   ```

3. **Signature** (HMAC/RSA): Cryptographic signature of header + payload
   - Verifies token integrity
   - Proves issuer authenticity

---

## JWT vs Session-Cookie

| Aspect | JWT | Session-Cookie |
|--------|-----|-----------------|
| **Storage** | Client stores token | Server stores session state |
| **Stateless** | ✅ Yes | ❌ No (requires session store) |
| **Scalability** | ✅ Easy (no server state) | ❌ Hard (sticky sessions or shared session store) |
| **Security** | ✅ HMAC/RSA signed | ✅ Opaque, server-controlled |
| **Revocation** | ❌ Hard (can't revoke until expiry) | ✅ Immediate |
| **Mobile/SPA** | ✅ Natural fit | ❌ Awkward (CORS, credentials) |
| **Size** | ⚠️ Larger (in every request) | ✅ Small (session ID) |

**Use JWT for:** REST APIs, SPAs, mobile apps, microservices
**Use Sessions for:** Traditional server-rendered apps

---

## JWT Implementation in Spring Boot

### 1. Dependencies

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

---

### 2. JWT Utility Service

```java
@Component
public class JwtUtil {
    @Value("${app.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.jwt.expiration}")
    private long jwtExpiration;  // e.g., 3600000 (1 hour)
    
    // Generate token
    public String generateToken(UserDetails userDetails) {
        return createToken(userDetails.getUsername(), userDetails.getAuthorities());
    }
    
    private String createToken(String username, Collection<? extends GrantedAuthority> authorities) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", authorities.stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(SignatureAlgorithm.HS256, jwtSecret)
            .compact();
    }
    
    // Validate token
    public boolean validateToken(String token) {
        try {
            Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token);
            return true;
        } catch (ExpiredJwtException e) {
            logger.error("Expired JWT token");
            return false;
        } catch (UnsupportedJwtException | MalformedJwtException | SignatureException | IllegalArgumentException e) {
            logger.error("Invalid JWT token");
            return false;
        }
    }
    
    // Extract username from token
    public String getUsernameFromToken(String token) {
        return Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
    
    // Extract expiration
    public Date getExpirationFromToken(String token) {
        return Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody()
            .getExpiration();
    }
}
```

---

### 3. Authentication Controller

```java
@RestController
@RequestMapping("/auth")
public class AuthController {
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @PostMapping("/login")
    public ResponseEntity<Map<String, String>> login(@RequestBody LoginRequest req) {
        try {
            // Authenticate user with username/password
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    req.getUsername(),
                    req.getPassword()
                )
            );
            
            // Get authenticated user
            UserDetails userDetails = (UserDetails) authentication.getPrincipal();
            
            // Generate JWT token
            String token = jwtUtil.generateToken(userDetails);
            
            return ResponseEntity.ok(Map.of(
                "token", token,
                "username", userDetails.getUsername(),
                "expiresIn", "3600"  // seconds
            ));
        } catch (BadCredentialsException e) {
            throw new RuntimeException("Invalid credentials", e);
        }
    }
}
```

---

### 4. JWT Authentication Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Autowired
    private JwtUtil jwtUtil;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain chain) throws ServletException, IOException {
        try {
            // Extract token from Authorization header
            String token = extractToken(request);
            
            if (token != null && jwtUtil.validateToken(token)) {
                // Extract username from token
                String username = jwtUtil.getUsernameFromToken(token);
                
                // Load user details
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                // Create authentication token
                UsernamePasswordAuthenticationToken auth = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );
                
                // Set in security context (user now authenticated for this request)
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        } catch (Exception e) {
            logger.error("Cannot authenticate user", e);
        }
        
        chain.doFilter(request, response);
    }
    
    private String extractToken(HttpServletRequest request) {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            return header.substring(7);  // Remove "Bearer " prefix
        }
        return null;
    }
}
```

---

### 5. Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()  // Stateless → no CSRF needed
            .authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .antMatchers("/api/**").authenticated()
                .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  // No sessions
                .and()
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

---

## JWT Security Best Practices

### ✅ Do's
- **Use HTTPS** — Tokens in transit must be encrypted
- **Short expiration** — 15-60 minutes for access tokens
- **Store securely** — Browser: httpOnly cookie (immune to XSS); app: secure storage
- **Refresh tokens** — Long-lived (days) to get new access tokens without re-login
- **HMAC over RSA** — HMAC (HS256) simpler; RSA for distributed services
- **Include minimal claims** — Only necessary data (avoid PII in payload)
- **Verify signature** — Always validate on every request

### ❌ Don'ts
- **Don't store in localStorage** — Vulnerable to XSS
- **Don't use weak secret** — Use cryptographically strong keys (>256 bits)
- **Don't trust unverified tokens** — Always validate signature
- **Don't embed passwords** — Only user ID, roles, claims
- **Don't rely on expiration alone** — Implement token blacklist for logout
- **Don't forget CORS** — Allow token in Authorization header

---

## Token Refresh Flow

**Access Token**: Short-lived (15-60 min), for API access
**Refresh Token**: Long-lived (7-30 days), to get new access tokens

```java
@PostMapping("/auth/refresh")
public ResponseEntity<Map<String, String>> refreshToken(@RequestBody RefreshTokenRequest req) {
    String refreshToken = req.getRefreshToken();
    
    // Validate refresh token
    if (refreshTokenService.isValid(refreshToken)) {
        String username = refreshTokenService.getUsername(refreshToken);
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        String newAccessToken = jwtUtil.generateToken(userDetails);
        
        return ResponseEntity.ok(Map.of("token", newAccessToken));
    }
    
    throw new RuntimeException("Invalid refresh token");
}
```

**Flow:**
1. POST /auth/login → Get access token + refresh token
2. API calls with access token until expiry
3. POST /auth/refresh with refresh token → Get new access token
4. POST /auth/logout → Invalidate refresh token

---

## OAuth2 Overview

**OAuth2** — Delegated authorization (not authentication).
- User never gives password to app
- User authorizes app to access resources on their behalf

### OAuth2 Grant Types
1. **Authorization Code** (most common for web apps)
2. **Implicit** (deprecated, for SPAs)
3. **Client Credentials** (service-to-service)
4. **Resource Owner Password** (legacy, not recommended)
5. **Refresh Token** (get new access token)

### OAuth2 + OpenID Connect (OIDC)
- **OAuth2** — Handles authorization
- **OIDC** — Adds authentication (ID token with user info)

```yaml
# Spring Boot OAuth2 Resource Server Config
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth-server.com
          jwk-set-uri: https://auth-server.com/.well-known/jwks.json
```

---

## Common Scenarios

### Scenario 1: Logout
**Problem:** JWT can't be "revoked" until expiration.

**Solution:** Token blacklist
```java
@Service
public class TokenBlacklistService {
    private final Set<String> blacklist = new HashSet<>();
    
    public void blacklist(String token) {
        long expiryTime = extractExpiration(token).getTime();
        blacklist.add(token);
        // Clean up after expiration
        scheduleBlacklistCleanup(token, expiryTime);
    }
    
    public boolean isBlacklisted(String token) {
        return blacklist.contains(token);
    }
}
```

### Scenario 2: Multi-Tenancy
**Problem:** Different orgs need isolation.

**Solution:** Include tenant ID in JWT
```java
claims.put("tenant_id", user.getTenantId());

// Filter extracts and sets tenant context
String tenantId = (String) claims.get("tenant_id");
TenantContext.setCurrentTenant(tenantId);
```

### Scenario 3: Role-Based Access Control (RBAC)
```java
@PostMapping("/admin/users")
@PreAuthorize("hasRole('ADMIN')")  // Only ADMIN role
public ResponseEntity<User> createUser(@RequestBody CreateUserRequest req) {
    return ResponseEntity.ok(userService.create(req));
}

@GetMapping("/users")
@PreAuthorize("hasAnyRole('ADMIN', 'USER')")  // ADMIN or USER
public ResponseEntity<List<User>> listUsers() {
    return ResponseEntity.ok(userService.list());
}
```

---

## Configuration (application.yml)

```yaml
app:
  jwt:
    secret: ${JWT_SECRET:your-super-secret-key-min-256-bits-long}
    expiration: 3600000  # 1 hour in milliseconds
    refresh-expiration: 604800000  # 7 days

server:
  servlet:
    context-path: /api
```

---

**See also:** [Spring Security](security.md), [Spring Annotations](spring-annotations-components.md), [Spring Boot Advanced](spring-boot-advanced.md)

