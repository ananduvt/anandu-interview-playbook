# Security

## Authentication mechanisms
- **Session-cookie** — server stores session; cookie holds session id.
- **JWT (JSON Web Token)** — stateless, signed token (header.payload.signature); self-contained claims.
- **OAuth2 / OIDC** — delegated authorization; access + refresh tokens; OIDC adds identity (ID token).
- **API keys**, **mTLS** (mutual TLS), **HMAC request signing** (shared-secret signature per request).

## Authentication vs Authorization
- **AuthN** — verify identity. **AuthZ** — verify permissions (RBAC roles, scopes, ABAC).

## Hashing vs encryption
- **Hashing** — one-way, irreversible (passwords: bcrypt/argon2 + salt; integrity: SHA-256).
- **Encryption** — two-way with a key (confidentiality). Symmetric (AES) vs asymmetric (RSA/ECC).
- **HMAC** — keyed hash for message authenticity + integrity (symmetric secret).

## OWASP Top 10 (know these)
Broken access control · cryptographic failures · **injection** (SQL/command) · insecure design ·
security misconfiguration · vulnerable/outdated components · auth failures · data integrity failures ·
logging/monitoring failures · **SSRF**.

## Defenses
- Parameterized queries (prevent SQLi); input validation/output encoding (XSS); CSRF tokens.
- Least privilege; secrets in a vault (never in code); rotate credentials.
- TLS everywhere; mask/redact PII in logs; dependency scanning (CVE) + patching.
- Constant-time comparison for secrets/signatures; rate limiting.

## Spring Security

### Core Components
**SecurityFilterChain** bean; method security (`@PreAuthorize`, `@Secured`); OAuth2 resource server; `BCryptPasswordEncoder` for passwords.

### Role-Based Login & Authorization

#### 1. User Service with Roles
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        
        // Convert user roles to Spring Security authorities
        List<GrantedAuthority> authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
        
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),  // BCrypt hashed
            authorities
        );
    }
}
```

#### 2. Login Controller
```java
@RestController
@RequestMapping("/auth")
public class LoginController {
    private AuthenticationManager authenticationManager;
    private JwtTokenProvider tokenProvider;
    
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest loginRequest) {
        try {
            // Authenticate with username + password
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    loginRequest.getUsername(),
                    loginRequest.getPassword()
                )
            );
            
            // Get authenticated user with roles
            UserDetails userDetails = (UserDetails) authentication.getPrincipal();
            String token = tokenProvider.generateToken(userDetails.getUsername(), 
                userDetails.getAuthorities());
            
            return ResponseEntity.ok(new LoginResponse(
                token,
                userDetails.getUsername(),
                userDetails.getAuthorities().stream()
                    .map(GrantedAuthority::getAuthority)
                    .collect(Collectors.toList())
            ));
        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body("Invalid username or password");
        }
    }
}
```

#### 3. Security Configuration (Role-Based Access)
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)  // Enable @PreAuthorize
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
                // Public endpoints
                .antMatchers("/auth/**", "/public/**").permitAll()
                
                // Admin only
                .antMatchers("/admin/**").hasRole("ADMIN")
                
                // User or higher
                .antMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                
                // Any authenticated user
                .anyRequest().authenticated()
                .and()
            .httpBasic()
            .and()
            .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

#### 4. Method-Level Security (Fine-Grained Access)
```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    // Only ADMIN role
    @GetMapping("/admin/users")
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    // ADMIN or MANAGER role
    @DeleteMapping("/users/{id}")
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
    
    // User can only modify their own profile
    @PutMapping("/users/{id}")
    @PreAuthorize("hasRole('USER') and #id == authentication.principal.id")
    public ResponseEntity<User> updateUser(
        @PathVariable Long id,
        @RequestBody UserUpdateRequest req) {
        return ResponseEntity.ok(userService.update(id, req));
    }
    
    // Custom permission check (using SpEL)
    @PostMapping("/posts")
    @PreAuthorize("hasRole('USER') and isPostingEnabled(authentication.principal.username)")
    public ResponseEntity<Post> createPost(@RequestBody CreatePostRequest req) {
        return ResponseEntity.created(URI.create("/posts/new"))
            .body(postService.create(req));
    }
}
```

#### 5. Custom Permission Evaluator
```java
@Component
public class CustomPermissionEvaluator extends DefaultPermissionEvaluator {
    private UserService userService;
    
    @Override
    public boolean hasPermission(
        Authentication authentication,
        Object targetDomainObject,
        Object permission) {
        
        if (targetDomainObject == null) return false;
        
        String username = authentication.getName();
        String permissionName = permission.toString().toUpperCase();
        
        if (targetDomainObject instanceof Post) {
            Post post = (Post) targetDomainObject;
            // Allow author to edit, admins to do anything
            if ("EDIT".equals(permissionName)) {
                return isAdmin(username) || isAuthor(username, post);
            }
        }
        return false;
    }
    
    private boolean isAdmin(String username) {
        User user = userService.findByUsername(username);
        return user.getRoles().stream()
            .anyMatch(role -> "ADMIN".equals(role.getName()));
    }
    
    private boolean isAuthor(String username, Post post) {
        return post.getAuthor().getUsername().equals(username);
    }
}

// Usage in controller
@PutMapping("/posts/{id}")
@PreAuthorize("hasPermission(#post, 'EDIT')")
public ResponseEntity<Post> editPost(
    @PathVariable Long id,
    Post post) {  // @EntityGraph injected by Spring Data
    return ResponseEntity.ok(postService.update(post));
}
```

#### 6. Role-Based Response Filtering
```java
@Component
public class ResponseFilter {
    
    @Transactional
    public List<UserDTO> filterUsersForRole(List<User> users, Collection<GrantedAuthority> authorities) {
        boolean isAdmin = authorities.stream()
            .anyMatch(a -> "ROLE_ADMIN".equals(a.getAuthority()));
        
        return users.stream()
            .map(user -> UserDTO.builder()
                .id(user.getId())
                .username(user.getUsername())
                .email(user.getEmail())
                .roles(isAdmin ? user.getRoles() : emptyList())  // Hide roles from non-admins
                .createdAt(isAdmin ? user.getCreatedAt() : null)  // Hide audit fields
                .build())
            .collect(Collectors.toList());
    }
}
```

#### 7. Access Denied Handling
```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
    
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException accessDeniedException) throws IOException {
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setContentType("application/json");
        
        ErrorResponse errorResponse = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.FORBIDDEN.value())
            .error("Access Denied")
            .message("You don't have permission to access this resource")
            .path(request.getRequestURI())
            .build();
        
        response.getWriter().write(objectMapper.writeValueAsString(errorResponse));
    }
}

// Register in Security Config
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .exceptionHandling()
            .accessDeniedHandler(new CustomAccessDeniedHandler())
            .and()
        // ... rest of config
    return http.build();
}
```

#### 8. Audit Logging
```java
@Component
@Aspect
public class SecurityAuditLog {
    private AuditEventRepository auditRepository;
    
    @Around("@annotation(com.example.security.Auditable)")
    public Object auditAccess(ProceedingJoinPoint joinPoint) throws Throwable {
        String username = SecurityContextHolder.getContext()
            .getAuthentication().getName();
        String operation = joinPoint.getSignature().getName();
        
        auditRepository.save(AuditEvent.builder()
            .username(username)
            .operation(operation)
            .timestamp(LocalDateTime.now())
            .build());
        
        return joinPoint.proceed();
    }
}

// Usage
@DeleteMapping("/users/{id}")
@PreAuthorize("hasRole('ADMIN')")
@Auditable
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    // ...
}
```
