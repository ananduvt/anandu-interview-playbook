# Programmatic Transactions (TransactionTemplate)

`@Transactional` works through a proxy (see [Spring AOP](spring-aop.md)), so it is skipped on self-invocation, private methods and non-Spring-managed objects. **Programmatic transactions** put the boundary in code, so it is explicit, narrow and proxy-free. Spring Boot auto-configures a `PlatformTransactionManager` and a `TransactionTemplate` bean when a data source or JPA is present (Spring 5/6, Java 17+ for Spring 6).

## When to Choose Programmatic

| Situation | Why programmatic helps |
| :-- | :-- |
| Only part of a method needs a transaction | Keep slow I/O (HTTP, file) outside the transaction |
| Self-invocation inside one class | No proxy involved |
| Dynamic settings (isolation, timeout chosen at runtime) | Configure per call |
| Retry the transaction itself | Wrap `execute` in a retry loop, each attempt is a fresh transaction |
| Reactive or non-proxied code | `TransactionalOperator` (reactive) / template works anywhere |

Use `@Transactional` for the common case; it is shorter and keeps business code free of infrastructure.

## TransactionTemplate

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.support.TransactionTemplate;

@Service
public class OrderService {
    private final TransactionTemplate tx;
    private final OrderRepository orders;
    private final PaymentClient payments;

    public OrderService(TransactionTemplate tx, OrderRepository orders, PaymentClient payments) {
        this.tx = tx;
        this.orders = orders;
        this.payments = payments;
    }

    public Order place(OrderRequest req) {
        PaymentToken token = payments.authorize(req);          // slow call, NO transaction held

        return tx.execute(status -> {                          // returns a value
            Order o = orders.save(Order.from(req, token));
            if (o.total().signum() <= 0) {
                status.setRollbackOnly();                      // manual rollback, no exception
            }
            return o;
        });                                                    // RuntimeException => rollback
    }

    public void audit(String msg) {
        tx.executeWithoutResult(status -> orders.log(msg));    // Spring 5.2+, no return value
    }
}
```

- `execute(TransactionCallback<T>)` commits on normal return, rolls back on an unchecked exception or `setRollbackOnly()`.
- A **checked** exception inside the callback must be wrapped; the callback cannot throw it.
- The callback runs on the calling thread, bound to a connection held for its duration.

## Customising Behaviour

`TransactionTemplate` is a `TransactionDefinition`; set properties once and share the bean, or create a derived template per use.

```java
@Configuration
class TxConfig {
    @Bean
    TransactionTemplate requiresNewTx(PlatformTransactionManager tm) {
        TransactionTemplate t = new TransactionTemplate(tm);
        t.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        t.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        t.setTimeout(5);                    // seconds
        t.setReadOnly(false);
        return t;
    }
}
```

Several templates of the same type need `@Qualifier`, or inject the manager and build one: `new TransactionTemplate(tm, definition)`.

## Low-Level: PlatformTransactionManager

```java
@Service
class ManualTx {
    private final PlatformTransactionManager tm;
    ManualTx(PlatformTransactionManager tm) { this.tm = tm; }

    void run(Runnable work) {
        TransactionStatus st = tm.getTransaction(new DefaultTransactionDefinition());
        try {
            work.run();
            tm.commit(st);
        } catch (RuntimeException e) {
            tm.rollback(st);
            throw e;
        }
    }
}
```

Prefer the template; it removes this boilerplate and the risk of forgetting `rollback`.

## Gotchas

- **Do the slow work outside** the template; a held transaction pins a pooled connection.
- **Propagation still applies**: calling `execute` while a transaction is active joins it (`REQUIRED`) unless you use `REQUIRES_NEW`.
- **Events**: use `TransactionSynchronizationManager.registerSynchronization(...)` or `@TransactionalEventListener(AFTER_COMMIT)` to act only after commit.
- **Threads**: transactions are thread-bound; work moved to another thread (`@Async`, parallel streams, `CompletableFuture`) is outside the transaction.
- **JPA lazy loading** must finish inside the callback, or the entity detaches.
- Mixing `@Transactional` and the template on the same path makes boundaries hard to reason about; pick one per use case.

## Common Interview Questions

- *How do you run a transaction without `@Transactional`?* Inject `TransactionTemplate` and call `execute`/`executeWithoutResult`, or use `PlatformTransactionManager` directly.
- *Why might you prefer it?* Narrow scope, no proxy limitations, per-call settings, explicit rollback.
- *How do you roll back without throwing?* `status.setRollbackOnly()`.
- *Reactive equivalent?* `TransactionalOperator` with `transactional(Mono/Flux)`.

**See also:** [Spring AOP and self-invocation](spring-aop.md), [JPA and ORM](jpa-orm.md), [Spring threading](spring-threading.md)

**Practice questions:** [Spring Data JPA](../interview-questions/verbal/verbal-spring-data-jpa-questions.md)
