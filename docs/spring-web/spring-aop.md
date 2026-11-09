# Spring AOP & Proxy Pitfalls

**Aspect-Oriented Programming** isolates cross-cutting concerns (logging, transactions, auditing, security) into **aspects** instead of repeating them in every service method. Spring AOP implements this with **runtime proxies**, which explains both how `@Transactional` works and why it sometimes silently does not.

---

## Core Terminology

| Term | Meaning |
|------|---------|
| Aspect | A class annotated `@Aspect` holding a cross-cutting concern |
| Join point | A point where advice can run; in Spring AOP always a **method execution** |
| Advice | The code to run (before, after, around...) |
| Pointcut | Expression selecting which join points get the advice |
| Target | The original bean being advised |
| Proxy | Object Spring generates around the target to apply advice |

## Types of Advice

| Annotation | Runs | Notes |
|-----------|------|-------|
| `@Before` | Before the method | Cannot stop execution unless it throws |
| `@AfterReturning` | After normal return | Can read the return value (`returning = "result"`) |
| `@AfterThrowing` | After an exception | Can read the exception (`throwing = "ex"`) |
| `@After` | Always (finally) | Success or failure |
| `@Around` | Wraps the call | Decides whether/when to call `proceed()`; can change args and result |

## Example

Add `spring-boot-starter-aop` (Boot auto-enables proxying; no `@EnableAspectJAutoProxy` needed).

```java
@Service
public class PaymentService {
    public String processPayment(String accountId, double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be > 0");
        return "TXN-9921";
    }
}

@Aspect
@Component
public class TimingAspect {

    @Pointcut("execution(* com.example.service.PaymentService.*(..))")
    void paymentMethods() {}

    @Before("paymentMethods()")
    public void logEntry(JoinPoint jp) {
        System.out.println("Entering " + jp.getSignature().getName());
    }

    @Around("paymentMethods()")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try {
            return pjp.proceed();                    // forgetting this skips the target
        } finally {
            System.out.printf("%s took %d ms%n", pjp.getSignature().getName(),
                    (System.nanoTime() - start) / 1_000_000);
        }
    }
}
```

!!! warning "Do not log sensitive arguments"
    Logging `getArgs()` blindly leaks card numbers, tokens and passwords into logs. Log IDs or masked values.

### Pointcut Expressions

| Expression | Matches |
|-----------|---------|
| `execution(* com.example.service.*.*(..))` | Any method of any class in the package |
| `execution(public * *(..))` | Any public method |
| `within(com.example.service..*)` | All methods in the package and subpackages |
| `@annotation(com.example.LogExecution)` | Methods carrying a custom annotation (good for opt-in aspects) |

Format: `execution(modifiers? return-type declaring-type? name(params) throws?)`.

---

## How It Works: Proxies

At startup, a `BeanPostProcessor` checks each bean against all pointcuts. On a match, Spring registers a **proxy** instead of the raw bean; callers hold the proxy.

| Proxy type | Used when | Limits |
|-----------|-----------|--------|
| JDK dynamic proxy | Bean implements an interface and `proxyTargetClass=false` | Only interface methods advised |
| CGLIB (subclass) | **Default in Spring Boot** (`spring.aop.proxy-target-class=true`) | Cannot advise `final` classes/methods or `private` methods |

Consequences:

- Only calls that **go through the proxy** (from another bean) are advised.
- Only Spring beans are advised; objects created with `new` are not.
- Spring AOP is proxy-based and method-level only. For field access, constructors or self-calls use **AspectJ** weaving (compile-time or load-time).

---

## @Transactional Self-Invocation

`@Transactional` is just an `@Around` advice applied by a proxy. When a method calls another method of the **same class** via `this`, the proxy is bypassed, so the annotation on the called method is ignored: no transaction starts, and no rollback happens.

```text
[Caller] -> [Proxy: opens tx] -> [Bean.method1()]
                                      |  this.method2()  (direct call)
                                      v
                                 [Bean.method2()]   proxy bypassed: no tx, no aspect
```

```java
@Service
public class UserService {
    public void registerUser(User user) {
        saveUserData(user);                  // ❌ same as this.saveUserData(): no transaction
    }

    @Transactional
    public void saveUserData(User user) {
        userRepository.save(user);
    }
}
```

The same applies to `@Async`, `@Cacheable`, `@Retryable` and any custom aspect.

### Fixes

| Fix | When | Trade-off |
|-----|------|-----------|
| 1. Move the method to another bean (recommended) | Almost always | Cleanest; call crosses the proxy |
| 2. Annotate the entry method `@Transactional` | The whole flow should be one transaction | Changes the transaction boundary |
| 3. Self-injection with `@Lazy` | Refactor not practical | Works, but smells |
| 4. `TransactionTemplate` | Only a block needs a transaction | Programmatic, explicit boundaries |
| 5. AspectJ mode (`@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)`) | Need internal/private-method advice | Extra build plugin; rarely used |

```java
// Fix 1: separate bean
@Service
public class UserService {
    private final UserDataService data;
    public UserService(UserDataService data) { this.data = data; }

    public void registerUser(User user) {
        data.saveUserData(user);             // ✅ goes through the proxy
    }
}

@Service
public class UserDataService {
    private final UserRepository repo;
    public UserDataService(UserRepository repo) { this.repo = repo; }

    @Transactional
    public void saveUserData(User user) { repo.save(user); }
}

// Fix 4: TransactionTemplate (details: spring-transactions.md)
@Service
public class UserTxService {
    private final TransactionTemplate tx;
    private final UserRepository repo;
    public UserTxService(TransactionTemplate tx, UserRepository repo) { this.tx = tx; this.repo = repo; }

    public void registerUser(User user) {
        tx.executeWithoutResult(status -> repo.save(user));
    }
}
```

!!! note "Other @Transactional gotchas"
    - Default rollback is for unchecked exceptions and `Error` only; use `rollbackFor = Exception.class` for checked ones.
    - Swallowing the exception inside the method prevents rollback.
    - Since Spring 6, class-based (CGLIB) proxies honour `@Transactional` on protected and package-private methods; `private` and `final` still do not work.
    - A `readOnly` or `REQUIRES_NEW` setting on the inner method is also ignored on self-invocation.

---

## Interview Questions

- **Join point vs pointcut vs advice?** Join point is where, pointcut selects which, advice is what runs.
- **JDK proxy vs CGLIB?** Interface-based vs subclass-based; Boot defaults to CGLIB.
- **Why does `@Transactional` not work on an internal call?** The call uses `this`, skipping the proxy that holds the transaction interceptor.
- **How do you fix it?** Extract to another bean (preferred), `TransactionTemplate`, or AspectJ mode.
- **Spring AOP vs AspectJ?** Runtime proxies, method execution only, Spring beans only vs bytecode weaving, any join point, no proxy limits.
- **Order of multiple aspects?** Use `@Order`; lower value runs first on the way in.

---

**See also:** [Spring Boot Advanced](spring-boot-advanced.md), [Spring Annotations & Components](spring-annotations-components.md), [JPA & ORM](jpa-orm.md)

**Practice questions:** [Spring Core, DI & AOP](../interview-questions/verbal/verbal-spring-core-di-questions.md), [Spring Data & JPA](../interview-questions/verbal/verbal-spring-data-jpa-questions.md)
