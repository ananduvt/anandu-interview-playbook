# Spring Core, DI & AOP Interview Questions

Spring container internals, dependency injection, bean registration, and aspect-oriented programming.

---

## 1. Explain the internal working of the Spring container

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The container (<code>ApplicationContext</code>, built on <code>BeanFactory</code>) reads configuration metadata, turns it into <code>BeanDefinition</code>s, then creates, wires, and manages the beans according to those definitions.</p>

<h3>Steps</h3>
<ol>
<li><strong>Load definitions:</strong> from <code>@Configuration</code> classes, component scanning, XML, or auto-configuration imports.</li>
<li><strong>BeanFactoryPostProcessors</strong> modify definitions (e.g. resolve <code>${...}</code> placeholders, process <code>@Configuration</code> classes).</li>
<li><strong>Instantiate</strong> non-lazy singletons (constructor/factory method), resolving dependencies recursively.</li>
<li><strong>Populate</strong> dependencies (setter/field injection).</li>
<li><strong>BeanPostProcessors</strong> run before and after initialization; this is where <code>@Autowired</code>, <code>@PostConstruct</code>, and AOP proxies are applied.</li>
<li><strong>Ready:</strong> singletons are cached in a registry (<code>singletonObjects</code>). On shutdown, <code>@PreDestroy</code> callbacks run.</li>
</ol>

<h3>Key Points</h3>
<ul>
<li><code>BeanFactory</code> is the basic lazy container; <code>ApplicationContext</code> adds events, i18n, resource loading, environment, and eager singleton creation.</li>
<li>Circular constructor dependencies fail; Spring Boot 2.6+ also forbids circular references between beans by default.</li>
<li>Proxies (AOP, <code>@Transactional</code>) are created in a <code>BeanPostProcessor</code>, so other beans receive the proxy, not the raw object.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 2. What is the difference between IoC and Dependency Injection? How does DI help modularity and testing?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Inversion of Control</strong> is the principle: the framework, not your code, controls object creation and flow. <strong>Dependency Injection</strong> is one technique to implement it: dependencies are supplied from outside (constructor, setter, field). Others are service locator and template-method/callbacks.</p>

<h3>Why DI Helps in Large Systems</h3>
<ul>
<li><strong>Loose coupling:</strong> classes depend on interfaces; implementations are chosen by configuration.</li>
<li><strong>Testability:</strong> pass mocks or fakes through the constructor, with no Spring context needed.</li>
<li><strong>Modularity:</strong> swap implementations per environment (profiles) or feature without touching callers.</li>
<li><strong>Cross-cutting concerns</strong> (transactions, security) attach via proxies without changing business classes.</li>
</ul>

```java
class OrderService {
    private final PaymentGateway gateway;
    OrderService(PaymentGateway gateway) { this.gateway = gateway; }
}

// Test: no container, no network
var service = new OrderService(Mockito.mock(PaymentGateway.class));
```

<h3>Gotchas</h3>
<ul>
<li>Overusing field injection hides dependencies and forces reflection in tests.</li>
<li>Too many constructor parameters signal a class doing too much.</li>
<li>DI is not the same as the Dependency Inversion Principle (the D in SOLID), though it supports it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 3. What is @Qualifier and how does it differ from @Primary?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>When several beans match an injection point's type, Spring throws <code>NoUniqueBeanDefinitionException</code>. <code>@Primary</code> marks a default bean; <code>@Qualifier</code> names the specific bean to inject at the injection point.</p>

<table>
<tr><th></th><th>@Primary</th><th>@Qualifier</th></tr>
<tr><td>Placed on</td><td>Bean definition</td><td>Injection point (and optionally the bean)</td></tr>
<tr><td>Meaning</td><td>"Use this one unless told otherwise"</td><td>"Use exactly this one"</td></tr>
<tr><td>Precedence</td><td>Lower</td><td>Higher, overrides <code>@Primary</code></td></tr>
</table>

```java
@Bean @Primary
PaymentGateway stripeGateway() { return new StripeGateway(); }

@Bean("paypal")
PaymentGateway paypalGateway() { return new PaypalGateway(); }

@Service
class Checkout {
    Checkout(@Qualifier("paypal") PaymentGateway gateway) { ... }  // overrides primary
}
```

<h3>Gotchas</h3>
<ul>
<li>Without <code>@Qualifier</code>, a constructor parameter name matching a bean name can also resolve the ambiguity, but this is fragile.</li>
<li>Inject all candidates with <code>List&lt;PaymentGateway&gt;</code> or <code>Map&lt;String, PaymentGateway&gt;</code>.</li>
<li><code>@Resource</code> (JSR-250) resolves by name first; <code>@Autowired</code> by type first.</li>
<li>Use custom qualifier annotations to avoid magic strings.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations & Components</a></p>

</details>

---

## 4. What does component scanning do?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Component scanning (<code>@ComponentScan</code>) searches packages on the classpath for classes annotated with <code>@Component</code> or a stereotype meta-annotated with it (<code>@Service</code>, <code>@Repository</code>, <code>@Controller</code>, <code>@Configuration</code>) and registers them as bean definitions.</p>

<h3>Details</h3>
<ul>
<li><code>@SpringBootApplication</code> includes <code>@ComponentScan</code> scanning the main class's package and sub-packages. A bean in a sibling or parent package is <strong>not</strong> found.</li>
<li>Customise with <code>@ComponentScan(basePackages = "com.example.api")</code> or <code>basePackageClasses</code>, and include/exclude filters.</li>
<li>Default bean name is the decapitalised class name (<code>UserService</code> becomes <code>userService</code>).</li>
<li>Spring reads class metadata via ASM at startup (without loading classes), so a huge scan range slows startup.</li>
</ul>

```java
@SpringBootApplication(scanBasePackages = {"com.example.app", "com.example.shared"})
public class App { }
```

<h3>Gotchas</h3>
<ul>
<li>Typical "No qualifying bean" cause: class outside the scanned package.</li>
<li>Scanning handles your own classes; third-party classes need <code>@Bean</code> or <code>@Import</code>.</li>
<li>Scanning is separate from JPA entity/repository scanning (<code>@EntityScan</code>, <code>@EnableJpaRepositories</code>).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations & Components</a></p>

</details>

---

## 5. What is the difference between @Component, @Service, and @Repository?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>All three register a bean via component scanning (the latter two are meta-annotated with <code>@Component</code>). They differ in intent and, for <code>@Repository</code>, in behaviour.</p>

<table>
<tr><th>Annotation</th><th>Layer</th><th>Extra behaviour</th></tr>
<tr><td><code>@Component</code></td><td>Generic</td><td>None</td></tr>
<tr><td><code>@Service</code></td><td>Business logic</td><td>None today; semantic marker and handy pointcut target</td></tr>
<tr><td><code>@Repository</code></td><td>Persistence</td><td>Persistence exception translation: vendor exceptions (e.g. <code>SQLException</code>, Hibernate) become Spring's <code>DataAccessException</code> hierarchy</td></tr>
</table>

<h3>Key Points</h3>
<ul>
<li>Exception translation is applied by a <code>BeanPostProcessor</code> (<code>PersistenceExceptionTranslationPostProcessor</code>), registered automatically by Boot.</li>
<li><code>@Controller</code>/<code>@RestController</code> are the web-layer stereotypes picked up by MVC.</li>
<li>Using the right stereotype documents architecture and lets AOP target layers (e.g. <code>@within(org.springframework.stereotype.Service)</code>).</li>
<li>Spring Data repository interfaces need no <code>@Repository</code>; they are proxied automatically.</li>
</ul>

<h3>Follow-up</h3>
<p>See also "@Component vs @Bean" in the Spring Boot questions: stereotypes are class-level and scanned; <code>@Bean</code> is a method that returns an object, useful for third-party classes.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations & Components</a></p>

</details>

---

## 6. What is AOP and how does Spring implement it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Aspect-Oriented Programming separates cross-cutting concerns (logging, security, transactions, metrics) from business logic. Spring AOP implements it with <strong>runtime proxies</strong>: JDK dynamic proxies for interface-based beans, CGLIB subclass proxies otherwise (Boot defaults to CGLIB).</p>

<h3>Terms</h3>
<ul>
<li><strong>Aspect:</strong> class holding cross-cutting logic (<code>@Aspect</code>).</li>
<li><strong>Join point:</strong> a point in execution; in Spring AOP always a method execution.</li>
<li><strong>Pointcut:</strong> expression selecting join points.</li>
<li><strong>Advice:</strong> the code run: <code>@Before</code>, <code>@After</code>, <code>@AfterReturning</code>, <code>@AfterThrowing</code>, <code>@Around</code>.</li>
<li><strong>Weaving:</strong> linking aspects to targets (Spring: at runtime via proxies).</li>
</ul>

```java
@Aspect
@Component
public class TimingAspect {
    @Around("execution(* com.example.service..*(..))")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try {
            return pjp.proceed();
        } finally {
            long ms = (System.nanoTime() - start) / 1_000_000;
            System.out.println(pjp.getSignature() + " took " + ms + " ms");
        }
    }
}
```

<h3>Gotchas</h3>
<ul>
<li><strong>Self-invocation</strong> (<code>this.other()</code>) bypasses the proxy, so advice does not run.</li>
<li>Only public/protected-overridable methods on Spring-managed beans are advised; <code>final</code> classes/methods cannot be CGLIB-proxied.</li>
<li>Spring AOP is limited to method execution on beans; AspectJ (compile/load-time weaving) can advise fields, constructors, and non-beans.</li>
<li>Built-in uses: <code>@Transactional</code>, <code>@Cacheable</code>, <code>@Async</code>, method security.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 7. What are the performance implications of using AOP in Spring?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Overhead is usually small (a proxy call plus advice logic, typically negligible compared with I/O), but it grows with hot-path calls, many aspects, expensive advice, and broad pointcuts.</p>

<h3>Where Cost Comes From</h3>
<ul>
<li><strong>Startup:</strong> evaluating pointcuts against every bean and creating proxies; broad expressions like <code>execution(* *(..))</code> are costly.</li>
<li><strong>Per call:</strong> proxy dispatch, interceptor chain, argument boxing, and whatever the advice does (logging, reflection, serialising arguments).</li>
<li><strong>Memory:</strong> extra proxy classes and objects.</li>
</ul>

<h3>Best Practices</h3>
<ul>
<li>Use <strong>narrow pointcuts</strong> (specific packages or annotations) instead of matching everything.</li>
<li>Keep advice light: avoid heavy string formatting or <code>toString()</code> of large arguments; guard logging with log levels.</li>
<li>Do not advise tight inner loops or trivial getters; apply AOP at service boundaries.</li>
<li>Prefer <code>@Around</code> only when you need it; <code>@Before</code>/<code>@AfterReturning</code> are simpler.</li>
<li>Measure with JFR or a profiler before optimising; use Micrometer <code>@Timed</code>/Observation instead of hand-rolled timing.</li>
<li>If you need advice on non-bean code or very hot paths, consider AspectJ weaving, which has no proxy indirection.</li>
</ul>

<h3>Follow-up</h3>
<p>Correctness pitfalls (self-invocation, ordering with <code>@Order</code>, final methods) usually matter more than raw speed.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---
