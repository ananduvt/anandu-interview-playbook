# Spring MVC Internals: Request Flow and Server-Side Templating

How a request travels through Tomcat and Spring MVC, where filters and interceptors sit, and how server-side templates (Thymeleaf, FreeMarker) fit the same flow. The short overview diagram is in [Spring Boot Advanced](spring-boot-advanced.md); this page goes one level deeper.

## Request Flow

```
Client -> Tomcat (Servlet container)
       -> Servlet Filter chain (Security, CORS, encoding, logging)
       -> DispatcherServlet
            1. HandlerMapping            -> finds handler + interceptor chain
            2. Interceptor.preHandle()
            3. HandlerAdapter            -> invokes handler (RequestMappingHandlerAdapter)
            4. Argument resolvers        -> @PathVariable, @RequestBody (HttpMessageConverter)
            5. Controller method
            6. Return value handlers     -> @ResponseBody via HttpMessageConverter, or ModelAndView
            7. Interceptor.postHandle()
            8. ViewResolver + View       -> only for template views
            9. Interceptor.afterCompletion()
            (exceptions -> HandlerExceptionResolver -> @ControllerAdvice)
```

| Component | Role |
| :-- | :-- |
| `DispatcherServlet` | Front controller; one servlet mapped to `/`, orchestrates every step |
| `HandlerMapping` | Maps request to handler; `RequestMappingHandlerMapping` reads `@RequestMapping` and friends |
| `HandlerAdapter` | Knows how to call that handler type and resolve its arguments |
| `HttpMessageConverter` | Converts body to/from objects (Jackson for JSON) using `Accept` / `Content-Type` |
| `HandlerExceptionResolver` | Turns exceptions into responses; backs `@ExceptionHandler` / [`@ControllerAdvice`](spring-exceptions.md) |
| `ViewResolver` | Turns a view name into a `View` (template) when the handler returns a name or `ModelAndView` |

## Filters vs Interceptors

| Aspect | Servlet `Filter` | `HandlerInterceptor` |
| :-- | :-- | :-- |
| Belongs to | Servlet spec (container) | Spring MVC |
| Runs | Before `DispatcherServlet` | Inside it, after the handler is chosen |
| Knows the handler | No | Yes (`Object handler`, usually a `HandlerMethod`, so it can read annotations) |
| Can wrap request/response | Yes (`HttpServletRequestWrapper`) | Not really |
| Typical use | Security, CORS, compression, request logging, MDC correlation id | Per-controller auth checks, timing, locale, audit |
| Sees non-Spring resources | Yes (other servlets, static files) | Only requests routed through `DispatcherServlet` |

```java
@Component
public class TimingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        req.setAttribute("start", System.nanoTime());
        return true;                      // false stops the chain
    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res,
                                Object handler, Exception ex) {
        long ms = (System.nanoTime() - (long) req.getAttribute("start")) / 1_000_000;
        // runs even when the handler threw
    }
}

@Configuration
class WebConfig implements WebMvcConfigurer {
    private final TimingInterceptor timing;
    WebConfig(TimingInterceptor timing) { this.timing = timing; }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(timing).addPathPatterns("/api/**");
    }
}
```

!!! tip "Gotchas"
    - Spring Security is a servlet filter chain (`FilterChainProxy`), so it runs before any interceptor.
    - `postHandle` is skipped if the handler throws; `afterCompletion` is the reliable cleanup hook.
    - `@ControllerAdvice` cannot catch exceptions thrown inside a filter; handle those in the filter or an error controller.
    - Register a servlet filter as a `@Component`, or use `FilterRegistrationBean` for ordering and URL patterns.

## Server-Side Templating: Thymeleaf and FreeMarker

REST APIs return JSON, but server-rendered pages (admin UIs, emails, reports) return HTML built on the server. The controller puts data in a `Model` and returns a **view name**; a `ViewResolver` finds the template.

| Engine | Style | Notes |
| :-- | :-- | :-- |
| Thymeleaf | "Natural templates": `th:*` attributes in valid HTML, viewable in a browser without a server | `spring-boot-starter-thymeleaf`, templates in `src/main/resources/templates/*.html` |
| FreeMarker (FTL) | Text template with `${...}` and `<#list>`, `<#if>` directives | `spring-boot-starter-freemarker`, `*.ftlh` for HTML auto-escaping; good for emails and non-HTML text |
| JSP | Servlet-container based | Poor fit for executable jars; avoid for new work |

```java
@Controller                                   // not @RestController: returns view names
class UserPageController {
    private final UserService userService;
    UserPageController(UserService userService) { this.userService = userService; }

    @GetMapping("/users")
    String list(Model model) {
        model.addAttribute("users", userService.findAll());
        return "users";                       // resolves to templates/users.html
    }
}
```

```html
<!-- Thymeleaf: templates/users.html -->
<ul>
  <li th:each="u : ${users}">
    <a th:href="@{/users/{id}(id=${u.id})}" th:text="${u.name}">placeholder name</a>
  </li>
</ul>
```

```
<#-- FreeMarker: templates/users.ftlh -->
<ul>
<#list users as u>
  <li>${u.name}</li>
</#list>
</ul>
```

- **Escaping:** `th:text` and `.ftlh` escape HTML by default (XSS protection). `th:utext` and `?no_esc` do not; use only on trusted content.
- **CSRF:** Thymeleaf's `th:action` adds the CSRF token automatically with Spring Security.
- **Caching:** disable in dev (`spring.thymeleaf.cache=false`), keep it on in production.
- **Email/report rendering:** both engines can render to a `String` without a web request (Thymeleaf `TemplateEngine.process`, FreeMarker `Template.process`).
- **When not to use:** SPA or mobile clients (use JSON APIs, see [API Styles](api-styles.md)) or heavy client interactivity.

## Common Interview Questions

- **Walk through `GET /users/1`.** Filters, `DispatcherServlet`, `HandlerMapping` picks the method, argument resolvers bind `1`, the method runs, Jackson serialises the result, interceptor `afterCompletion`.
- **Filter or interceptor for a correlation id?** Filter (earlier, covers everything, can set MDC for the whole request); use an interceptor when you need handler metadata.
- **`@Controller` vs `@RestController`?** `@RestController` = `@Controller` + `@ResponseBody`, so return values go through message converters instead of the `ViewResolver`.
- **How does Spring pick JSON vs XML?** Content negotiation on `Accept`, then the converters on the classpath.
- **Thymeleaf vs FreeMarker?** Thymeleaf keeps templates valid HTML with tight Spring integration; FreeMarker is more general text templating.

**See also:** [Spring Boot Advanced](spring-boot-advanced.md), [Exception Handling](spring-exceptions.md), [Spring Annotations](spring-annotations-components.md), [Spring WebFlux](spring-webflux.md)
