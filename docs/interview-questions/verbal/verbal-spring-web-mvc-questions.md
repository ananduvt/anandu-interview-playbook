# Spring Web & MVC Interview Questions

Spring MVC request processing, request binding, and REST exception handling.

---

## 1. What steps happen inside Spring MVC when a client calls a controller endpoint?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The request goes through the servlet filter chain to the <code>DispatcherServlet</code> (front controller), which finds a handler, invokes it, and renders the response.</p>

<h3>Flow</h3>
<ol>
<li><strong>Servlet filters</strong> (security, CORS, encoding) run first.</li>
<li><strong>DispatcherServlet</strong> receives the request.</li>
<li><strong>HandlerMapping</strong> picks the controller method (<code>@RequestMapping</code> match) plus any <code>HandlerInterceptor</code>s.</li>
<li><strong>Interceptors</strong> <code>preHandle</code>.</li>
<li><strong>HandlerAdapter</strong> invokes the method: argument resolvers bind <code>@PathVariable</code>, <code>@RequestParam</code>, <code>@RequestBody</code> (via <code>HttpMessageConverter</code>, e.g. Jackson), and validation (<code>@Valid</code>) runs.</li>
<li><strong>Controller</strong> executes and returns a value.</li>
<li><strong>Return handling:</strong> for <code>@RestController</code>, a message converter writes the object as JSON; for views, a <code>ViewResolver</code> resolves a template.</li>
<li><strong>Interceptors</strong> <code>postHandle</code>/<code>afterCompletion</code>; exceptions go to <code>HandlerExceptionResolver</code>s (<code>@ExceptionHandler</code>, <code>@ControllerAdvice</code>).</li>
</ol>

<h3>Follow-ups</h3>
<ul>
<li>Filter vs interceptor: filters are servlet-level (see raw request, before Spring); interceptors are Spring MVC-level (know the handler).</li>
<li>Content negotiation uses the <code>Accept</code> header and available converters.</li>
<li>Thread-per-request model on servlet stack; WebFlux uses a different dispatcher (<code>DispatcherHandler</code>).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 2. What is the purpose of @ModelAttribute?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@ModelAttribute</code> binds request data (query parameters and form fields) to an object, and/or adds attributes to the model that views can use.</p>

<h3>Two Uses</h3>
<ul>
<li><strong>On a method parameter:</strong> Spring creates the object, binds request parameters to its properties by name, and (with <code>@Valid</code>) validates it. This is the default for non-simple types without <code>@RequestBody</code>.</li>
<li><strong>On a method:</strong> the return value is added to the model before every handler in that controller runs (e.g. dropdown data).</li>
</ul>

```java
@PostMapping("/users")
String create(@Valid @ModelAttribute UserForm form, BindingResult errors) {
    if (errors.hasErrors()) return "user-form";
    userService.create(form);
    return "redirect:/users";
}

@ModelAttribute("countries")
List<String> countries() { return List.of("IN", "US", "DE"); }
```

<h3>@ModelAttribute vs @RequestBody</h3>
<table>
<tr><th></th><th>@ModelAttribute</th><th>@RequestBody</th></tr>
<tr><td>Source</td><td>Query params, form data (<code>application/x-www-form-urlencoded</code>, multipart)</td><td>Request body (JSON/XML) via message converter</td></tr>
<tr><td>Typical use</td><td>Server-rendered forms</td><td>REST APIs</td></tr>
</table>

<h3>Gotchas</h3>
<ul>
<li>Place <code>BindingResult</code> right after the validated parameter, or validation errors throw an exception.</li>
<li>Over-binding: use a dedicated form class or <code>setAllowedFields</code>, never bind directly to an entity.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations & Components</a></p>

</details>

---

## 3. What is the difference between path variables and request parameters?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@PathVariable</code> reads a value from the URL path (identifies a resource); <code>@RequestParam</code> reads a query-string (or form) parameter (filters, paging, options).</p>

<table>
<tr><th></th><th>@PathVariable</th><th>@RequestParam</th></tr>
<tr><td>URL</td><td><code>/users/42</code></td><td><code>/users?role=admin&amp;page=2</code></td></tr>
<tr><td>Required</td><td>Yes (part of the path), unless <code>required=false</code> with multiple mappings</td><td>Required by default; supports <code>required=false</code> and <code>defaultValue</code></td></tr>
<tr><td>Purpose</td><td>Identify a specific resource</td><td>Filter, sort, paginate, optional modifiers</td></tr>
<tr><td>Caching/bookmarking</td><td>Distinct URL per resource</td><td>Query string part of the cache key</td></tr>
</table>

```java
@GetMapping("/users/{id}")
User get(@PathVariable Long id) { ... }

@GetMapping("/users")
Page<User> search(@RequestParam(required = false) String role,
                  @RequestParam(defaultValue = "0") int page) { ... }
```

<h3>Gotchas</h3>
<ul>
<li>Parameter name binding to the variable name needs <code>-parameters</code> (Spring Boot's Maven/Gradle plugins set it) or an explicit name: <code>@PathVariable("id")</code>.</li>
<li>Type mismatch (e.g. "abc" for <code>Long</code>) yields 400.</li>
<li>REST convention: path for identity, query for filtering; do not put sensitive data in either, since URLs are logged.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 4. What are best practices for exception handling in Spring Boot REST APIs? What is @ControllerAdvice?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Handle exceptions centrally and return a consistent error body with the right HTTP status. <code>@ExceptionHandler</code> handles exceptions in one controller; <code>@ControllerAdvice</code> (or <code>@RestControllerAdvice</code>, which adds <code>@ResponseBody</code>) applies handlers across all controllers.</p>

```java
@RestControllerAdvice
class ApiExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    ResponseEntity<ApiError> notFound(NotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ApiError("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<ApiError> invalid(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest()
                .body(new ApiError("VALIDATION", "Invalid request"));
    }

    @ExceptionHandler(Exception.class)
    ResponseEntity<ApiError> fallback(Exception ex) {
        log.error("Unhandled", ex);   // log full detail server-side only
        return ResponseEntity.status(500).body(new ApiError("INTERNAL", "Unexpected error"));
    }
}
record ApiError(String code, String message) {}
```

<h3>Best Practices</h3>
<ul>
<li>One advice class; map domain exceptions to statuses (400, 404, 409, 422) and keep 5xx for real server faults.</li>
<li>Consistent error format; consider RFC 9457 <code>ProblemDetail</code> (Spring 6+, <code>spring.mvc.problemdetails.enabled</code>) or <code>ResponseEntityExceptionHandler</code> for standard MVC exceptions.</li>
<li>Never leak stack traces, SQL, or internals; include a correlation id for support.</li>
<li>Validate input with <code>@Valid</code> and Bean Validation; handle <code>ConstraintViolationException</code> too.</li>
<li>Use unchecked custom exceptions; put <code>@ResponseStatus</code> on them only for simple cases.</li>
<li>Most specific handler wins; use <code>@Order</code> or <code>basePackages</code> to scope advices.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-exceptions/">Learn more: Spring Exception Handling</a></p>

</details>

---

## 5. REST recall: verb idempotency, status codes, versioning, pagination, statelessness, sensitive data

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A REST API should use verbs by their semantics, return precise status codes, stay stateless, paginate collections, version its contract, and never leak sensitive data.</p>

<h3>Verbs and Idempotency</h3>
<table>
<tr><th>Verb</th><th>Safe</th><th>Idempotent</th><th>Note</th></tr>
<tr><td>GET</td><td>Yes</td><td>Yes</td><td>No side effects</td></tr>
<tr><td>PUT</td><td>No</td><td>Yes</td><td>Full replace</td></tr>
<tr><td>DELETE</td><td>No</td><td>Yes</td><td>Second call may return 404 but the state is the same</td></tr>
<tr><td>POST</td><td>No</td><td>No</td><td>Make it retry-safe with an idempotency key</td></tr>
<tr><td>PATCH</td><td>No</td><td>Not guaranteed</td><td>Depends on the patch semantics</td></tr>
</table>

<h3>Status Codes</h3>
<table>
<tr><th>Code</th><th>Use</th></tr>
<tr><td>400</td><td>Malformed or invalid request</td></tr>
<tr><td>401</td><td>Not authenticated</td></tr>
<tr><td>403</td><td>Authenticated but not allowed</td></tr>
<tr><td>404</td><td>Resource not found</td></tr>
<tr><td>409</td><td>Conflict (duplicate, version mismatch)</td></tr>
<tr><td>429</td><td>Rate limit exceeded (send <code>Retry-After</code>)</td></tr>
<tr><td>5xx</td><td>Server fault (500 unexpected, 502/503/504 upstream or availability)</td></tr>
</table>

<h3>Other Essentials</h3>
<ul>
<li><strong>Versioning:</strong> URI (<code>/v1/orders</code>) is the most common and easiest to route and cache; header or media-type versioning is cleaner but harder to use. Prefer additive, backward-compatible changes.</li>
<li><strong>Pagination:</strong> offset (<code>page</code>, <code>size</code>) is simple but slow and unstable on large or changing data; cursor (keyset) pagination is stable and fast.</li>
<li><strong>Statelessness:</strong> each request carries what the server needs (token), which lets any instance serve it and makes scaling easy.</li>
<li><strong>Sensitive data:</strong> mask or omit secrets, tokens and personal data in responses and logs; use HTTPS and DTOs rather than exposing entities.</li>
</ul>

<h3>Example / Real-World Use</h3>

```
GET /v1/orders?limit=50&cursor=eyJpZCI6MTIzfQ    -> 200 { "items": [...], "nextCursor": "..." }
POST /v1/payments  Idempotency-Key: 7c1f...      -> 201 (a retry returns the same result)
```

<h3>When to Use</h3>
<ul>
<li>Cursor pagination for feeds and big tables; offset for small admin lists.</li>
<li>Gotcha: 401 vs 403 and 400 vs 409 are the usual mix-ups; PUT idempotency means same final state, not same response code.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles</a></p>

</details>

---
