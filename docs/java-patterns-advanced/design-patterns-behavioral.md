# Behavioral Design Patterns

Behavioral patterns describe how objects **communicate and share responsibility**. This page covers the 11 behavioral GoF patterns; see the [Design Patterns catalog](design-patterns.md) for creational and structural patterns and the big picture.

## Comparison table

| Pattern | Intent | One-line distinction |
| :-- | :-- | :-- |
| Chain of Responsibility | Pass a request along handlers until one handles it | Sender does not know which handler acts |
| Command | Wrap a request as an object | Enables queue, log, undo, retry |
| Interpreter | Evaluate sentences of a small grammar | One class per grammar rule |
| Iterator | Traverse a collection without exposing internals | Separates traversal from storage |
| Mediator | Centralize how peers talk to each other | Many-to-many becomes many-to-one |
| Memento | Capture and restore an object's state | Undo without breaking encapsulation |
| Observer | Notify dependents when a subject changes | One-to-many, publish/subscribe |
| State | Change behavior when internal state changes | The object swaps its own behavior at runtime |
| Strategy | Make algorithms interchangeable | The client picks the algorithm |
| Template Method | Fix the skeleton, let subclasses fill steps | Inheritance-based, algorithm order is fixed |
| Visitor | Add operations to a type hierarchy without changing it | Double dispatch over a stable set of types |

!!! tip "Strategy vs State vs Template Method"
    - **Strategy**: the *client* chooses an algorithm; strategies usually don't know each other.
    - **State**: the *object itself* switches behavior as it transitions; states often know the next state.
    - **Template Method**: uses *inheritance* (fixed skeleton, overridden steps); Strategy uses *composition* (swap the whole algorithm).

---

## Chain of Responsibility

### Intent
Give more than one object a chance to handle a request by linking handlers into a chain. The sender only knows the first handler.

### When to use
- More than one object may handle a request and the handler is not known upfront.
- You want to add or reorder processing steps (validation, auth, logging) without touching the sender.
- Handlers should be decoupled and individually testable.

### Example
Approval limits (Java 17, records):

```java
record Request(String description, int amount) {}

abstract class Approver {
    private Approver next;

    Approver linkWith(Approver next) {
        this.next = next;
        return next;                       // allows fluent chaining
    }

    final void handle(Request r) {
        if (canApprove(r)) {
            System.out.println(name() + " approved " + r.description());
        } else if (next != null) {
            next.handle(r);                // pass along
        } else {
            System.out.println("Nobody can approve " + r.description());
        }
    }

    protected abstract boolean canApprove(Request r);
    protected abstract String name();
}

class Manager extends Approver {
    protected boolean canApprove(Request r) { return r.amount() <= 1_000; }
    protected String name() { return "Manager"; }
}

class Director extends Approver {
    protected boolean canApprove(Request r) { return r.amount() <= 10_000; }
    protected String name() { return "Director"; }
}

public class ChainDemo {
    public static void main(String[] args) {
        Approver chain = new Manager();
        chain.linkWith(new Director());

        chain.handle(new Request("Laptop", 900));       // Manager approved
        chain.handle(new Request("Server", 8_000));     // Director approved
        chain.handle(new Request("Datacenter", 99_000)); // Nobody can approve
    }
}
```

### Real-world Java/Spring usage
- **Servlet filters** (`javax/jakarta.servlet.Filter`, `FilterChain.doFilter`).
- **Spring Security filter chain** (`SecurityFilterChain`: authentication, CSRF, authorization filters in order).
- **Spring MVC `HandlerInterceptor`** chain and `java.util.logging` handler/logger parent propagation.
- Exception handling itself: `catch` blocks are tried in order.

### Trade-offs
- Pros: loose coupling, flexible order, single responsibility per handler.
- Cons: no guarantee a request is handled; long chains are hard to debug; order bugs are subtle.

### Interview tip
Mention the two variants: **"first handler wins"** (approval example) and **"every handler participates"** (servlet filters, which call `chain.doFilter` to continue). Name the Spring Security filter chain as your real example.

---

## Command

### Intent
Encapsulate a request as an object so you can parameterize, queue, log, schedule, or undo operations.

### When to use
- Undo/redo, macro recording, transaction-like operations.
- Queueing or scheduling work (thread pools, job queues).
- Decoupling the invoker (button, scheduler) from the receiver (the actual work).

### Example
Undoable text editor commands (Java 17):

```java
import java.util.ArrayDeque;
import java.util.Deque;

interface Command {
    void execute();
    void undo();
}

record AppendCommand(StringBuilder doc, String text) implements Command {
    public void execute() { doc.append(text); }
    public void undo()    { doc.setLength(doc.length() - text.length()); }
}

class History {
    private final Deque<Command> done = new ArrayDeque<>();

    void run(Command c) {
        c.execute();
        done.push(c);
    }

    void undoLast() {
        if (!done.isEmpty()) done.pop().undo();
    }
}

public class CommandDemo {
    public static void main(String[] args) {
        StringBuilder doc = new StringBuilder();
        History history = new History();

        history.run(new AppendCommand(doc, "Hello"));
        history.run(new AppendCommand(doc, " World"));
        System.out.println(doc);   // Hello World
        history.undoLast();
        System.out.println(doc);   // Hello
    }
}
```

When no undo is needed, a command is just a lambda:

```java
// Runnable is the Command interface; the lambda is the concrete command
Runnable saveCommand = () -> System.out.println("Saving...");
ExecutorService pool = Executors.newFixedThreadPool(2);
pool.submit(saveCommand);          // invoker queues and runs it later
pool.shutdown();
```

### Real-world Java/Spring usage
- `Runnable` / `Callable` submitted to an `ExecutorService`.
- `javax.swing.Action` / `ActionListener`.
- Spring Batch `Tasklet`/`Step`, Spring's `TransactionCallback`, CQRS "command" messages.

### Trade-offs
- Pros: decouples invoker from receiver, supports undo, queueing, logging, retries.
- Cons: many small classes (lambdas reduce this); undo needs care with side effects.

### Interview tip
Say "Command turns a method call into an object". Point out that **`Runnable` + `ExecutorService`** is Command with the thread pool as invoker, and that Java 8 lambdas removed most of the boilerplate.

---

## Interpreter

### Intent
Define a representation for a simple grammar and an interpreter that evaluates sentences in it, usually as a tree of expression objects.

### When to use
- A small, stable domain language: rules, filters, arithmetic, search queries.
- Grammar is simple; for complex grammars use a parser generator (ANTLR).

### Example
Arithmetic expression tree (Java 21: sealed interface, records, pattern-matching `switch`):

```java
sealed interface Expr permits Num, Add, Mul {}
record Num(int value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}

public class InterpreterDemo {
    static int eval(Expr e) {
        return switch (e) {
            case Num n -> n.value();
            case Add a -> eval(a.left()) + eval(a.right());
            case Mul m -> eval(m.left()) * eval(m.right());
        };   // exhaustive: no default needed because Expr is sealed
    }

    public static void main(String[] args) {
        // (2 + 3) * 4
        Expr expr = new Mul(new Add(new Num(2), new Num(3)), new Num(4));
        System.out.println(eval(expr));   // 20
    }
}
```

The classic form puts an `interpret()` method on each node class; the `switch` version keeps the logic in one place.

### Real-world Java/Spring usage
- **Spring Expression Language (SpEL)**: `ExpressionParser` builds an AST that is evaluated against a context.
- `java.util.regex.Pattern`, `java.text.Format`, SQL/JPQL/Criteria parsing in ORMs.

### Trade-offs
- Pros: easy to extend the grammar with new node types.
- Cons: class explosion and poor performance for large grammars.

### Interview tip
It is the rarest GoF pattern in interviews. Be ready to say "SpEL is an interpreter" and that it is essentially **Composite (the tree) plus evaluation**.

---

## Iterator

### Intent
Provide a way to access elements of an aggregate sequentially without exposing its underlying representation.

### When to use
- Multiple traversal styles over the same structure.
- You want a uniform way to loop over different collections (and support `for-each`).

### Example
A custom `Iterable` (Java 8+):

```java
import java.util.Iterator;
import java.util.NoSuchElementException;

public class Range implements Iterable<Integer> {
    private final int start, end;   // end is exclusive

    public Range(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            private int current = start;

            @Override public boolean hasNext() { return current < end; }

            @Override public Integer next() {
                if (!hasNext()) throw new NoSuchElementException();
                return current++;
            }
        };
    }

    public static void main(String[] args) {
        for (int i : new Range(1, 4)) {
            System.out.print(i + " ");   // 1 2 3
        }
    }
}
```

### Real-world Java/Spring usage
- `java.util.Iterator`, `Iterable` (enables enhanced `for`), `ListIterator`, `Spliterator` (for streams).
- `ResultSet` iteration, `Stream.iterator()`.
- Fail-fast iterators (`ArrayList`) throw `ConcurrentModificationException`; concurrent collections use weakly consistent iterators.

### Trade-offs
- Pros: decouples traversal from storage; multiple independent cursors; uniform API.
- Cons: modifying the collection during iteration is risky; extra object per traversal.

### Interview tip
Know the gotchas: **fail-fast vs fail-safe** iterators and removing safely via `Iterator.remove()` or `removeIf`. Mention that streams are the modern internal-iteration alternative.

---

## Mediator

### Intent
Define an object that encapsulates how a set of objects interact, so peers do not refer to each other directly.

### When to use
- Many components talk to each other in tangled many-to-many ways (chat room, UI dialog, air traffic control).
- You want to change interaction rules in one place.

### Example

```java
import java.util.ArrayList;
import java.util.List;

interface ChatMediator {
    void register(User user);
    void send(String message, User from);
}

class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    public void register(User user) { users.add(user); }

    public void send(String message, User from) {
        for (User u : users) {
            if (u != from) u.receive(from.name(), message);   // peers never talk directly
        }
    }
}

class User {
    private final String name;
    private final ChatMediator room;

    User(String name, ChatMediator room) {
        this.name = name;
        this.room = room;
        room.register(this);
    }

    String name() { return name; }
    void say(String msg) { room.send(msg, this); }
    void receive(String from, String msg) {
        System.out.println(name + " got '" + msg + "' from " + from);
    }
}

public class MediatorDemo {
    public static void main(String[] args) {
        ChatMediator room = new ChatRoom();
        User ann = new User("Ann", room);
        new User("Bob", room);
        ann.say("Hi all");   // Bob got 'Hi all' from Ann
    }
}
```

### Real-world Java/Spring usage
- Spring's `ApplicationContext` mediating between beans (via events and DI).
- `java.util.Timer` (mediates tasks and the scheduling thread), MVC controllers, message brokers/event buses.

### Trade-offs
- Pros: fewer dependencies between peers; interaction logic in one place.
- Cons: the mediator can become a "god object".

### Interview tip
Contrast with **Observer**: Observer is one-way broadcast from a subject; Mediator is a central coordinator that **knows and controls** all participants.

---

## Memento

### Intent
Capture an object's internal state in a snapshot so it can be restored later, without violating encapsulation.

### When to use
- Undo/rollback, checkpoints, save games, draft versions.
- The state is private and should not be exposed via getters.

### Example
Immutable record as the snapshot (Java 16+):

```java
import java.util.ArrayDeque;
import java.util.Deque;

class Editor {
    private String text = "";

    void type(String s) { text += s; }
    String text() { return text; }

    Snapshot save() { return new Snapshot(text); }     // originator creates memento
    void restore(Snapshot s) { this.text = s.text(); }

    record Snapshot(String text) {}                    // immutable memento
}

public class MementoDemo {
    public static void main(String[] args) {
        Editor editor = new Editor();
        Deque<Editor.Snapshot> history = new ArrayDeque<>();   // caretaker

        editor.type("Hello");
        history.push(editor.save());
        editor.type(" World");
        System.out.println(editor.text());   // Hello World

        editor.restore(history.pop());
        System.out.println(editor.text());   // Hello
    }
}
```

### Real-world Java/Spring usage
- `java.io.Serializable` (state snapshot, e.g., HTTP session persistence).
- Database transactions/savepoints (`Connection.setSavepoint`), JPA entity snapshots for dirty checking.

### Trade-offs
- Pros: undo without exposing internals.
- Cons: memory cost if snapshots are large or frequent; combine with Command (store deltas) to reduce it.

### Interview tip
Name the three roles: **Originator, Memento, Caretaker**. Keep the memento immutable and opaque to the caretaker.

---

## Observer

### Intent
Define a one-to-many dependency so that when one object (the subject) changes state, all its dependents (observers) are notified automatically.

### When to use
- A change in one object must trigger updates in others, and you don't know how many or who.
- You want loose coupling between event producer and consumers.

### Example
A tiny type-safe event publisher using lambdas (Java 8+):

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.function.Consumer;

class StockTicker {
    // CopyOnWriteArrayList: safe if a listener (un)subscribes during notification
    private final List<Consumer<Double>> observers = new CopyOnWriteArrayList<>();
    private double price;

    void subscribe(Consumer<Double> observer)   { observers.add(observer); }
    void unsubscribe(Consumer<Double> observer) { observers.remove(observer); }

    void setPrice(double newPrice) {
        this.price = newPrice;
        observers.forEach(o -> o.accept(newPrice));   // notify all
    }
}

public class ObserverDemo {
    public static void main(String[] args) {
        StockTicker ticker = new StockTicker();
        ticker.subscribe(p -> System.out.println("Dashboard: " + p));
        ticker.subscribe(p -> { if (p > 100) System.out.println("Alert: high price " + p); });

        ticker.setPrice(95.5);    // Dashboard: 95.5
        ticker.setPrice(101.0);   // Dashboard: 101.0, Alert: high price 101.0
    }
}
```

### Real-world Java/Spring usage
- `java.beans.PropertyChangeListener` / `PropertyChangeSupport`; Swing listeners.
- **Spring events**: `ApplicationEventPublisher.publishEvent(...)` with `@EventListener` / `ApplicationListener`.
- Reactive streams (`Flow.Publisher/Subscriber`, Project Reactor); JMS/Kafka pub-sub at system scale.
- `java.util.Observable`/`Observer` are **deprecated since Java 9**: do not use them.

### Trade-offs
- Pros: loose coupling; observers can be added or removed at runtime.
- Cons: unexpected update cascades, memory leaks if observers are never unsubscribed ("lapsed listener"), notification order is unspecified, and Spring events run **synchronously on the publisher's thread** by default (use `@Async` for asynchronous delivery).

### Interview tip
Distinguish **Observer** (subject holds the observer list, direct call) from **pub/sub** (a broker/topic decouples publisher and subscriber). Mention the lapsed-listener leak.

---

## State

### Intent
Let an object alter its behavior when its internal state changes, so it appears to change its class.

### When to use
- An object has distinct modes and behavior depends on the current mode.
- You have large `if/else` or `switch` blocks on a state field repeated in many methods.

### Example
Order lifecycle where each state decides what is legal (enum with per-constant behavior):

```java
enum OrderState {
    NEW {
        @Override OrderState pay()  { return PAID; }
        @Override OrderState ship() { throw new IllegalStateException("Pay first"); }
    },
    PAID {
        @Override OrderState pay()  { throw new IllegalStateException("Already paid"); }
        @Override OrderState ship() { return SHIPPED; }
    },
    SHIPPED {
        @Override OrderState pay()  { throw new IllegalStateException("Already paid"); }
        @Override OrderState ship() { throw new IllegalStateException("Already shipped"); }
    };

    abstract OrderState pay();
    abstract OrderState ship();
}

class Order {
    private OrderState state = OrderState.NEW;

    void pay()  { state = state.pay(); }     // behavior delegated to the state
    void ship() { state = state.ship(); }
    OrderState state() { return state; }
}

public class StateDemo {
    public static void main(String[] args) {
        Order order = new Order();
        order.pay();
        order.ship();
        System.out.println(order.state());   // SHIPPED
        order.ship();                        // IllegalStateException: Already shipped
    }
}
```

For states that carry data, use a **sealed interface with records** (Java 17+) and a pattern-matching `switch` (Java 21) that returns the next state.

### Real-world Java/Spring usage
- **Spring State Machine** (`StateMachine`, `StateMachineFactory`).
- `Thread.State` and thread lifecycle; TCP connection states; JDBC `Connection` open/closed behavior.
- Workflow engines (Camunda, Spring Statemachine, order/payment lifecycles).

### Trade-offs
- Pros: removes conditional sprawl; new states are localized; illegal transitions are explicit.
- Cons: more classes; overkill for two or three simple states.

### Interview tip
Be ready to explain **State vs Strategy**: same class diagram, different intent. In State the context's current state **changes itself over time** and states trigger transitions; in Strategy the client **chooses** one algorithm and it stays put.

---

## Strategy

### Intent
Define a family of algorithms, encapsulate each one, and make them interchangeable at runtime.

### When to use
- Several ways to do the same thing (pricing, sorting, payment, compression).
- You want to eliminate `if/else` chains choosing an algorithm.
- The algorithm should be swappable or injected (and easy to unit test).

### Example
Discount strategies as lambdas (Java 8+):

```java
import java.util.Map;

@FunctionalInterface
interface DiscountStrategy {
    double apply(double price);
}

class Checkout {
    private final DiscountStrategy discount;

    Checkout(DiscountStrategy discount) { this.discount = discount; }   // injected

    double total(double price) { return discount.apply(price); }
}

public class StrategyDemo {
    // Registry replaces an if/else chain
    private static final Map<String, DiscountStrategy> STRATEGIES = Map.of(
        "NONE",     price -> price,
        "SEASONAL", price -> price * 0.90,
        "VIP",      price -> price * 0.80
    );

    public static void main(String[] args) {
        System.out.println(new Checkout(STRATEGIES.get("VIP")).total(200));       // 160.0
        System.out.println(new Checkout(STRATEGIES.get("SEASONAL")).total(200));  // 180.0
    }
}
```

The JDK's own example is `Comparator`:

```java
List<String> names = new ArrayList<>(List.of("Charlie", "al", "Bob"));
names.sort(Comparator.comparing(String::length));          // strategy 1
names.sort(String.CASE_INSENSITIVE_ORDER);                 // strategy 2
```

### Real-world Java/Spring usage
- **`Comparator`** passed to `sort`, `TreeMap`, streams.
- Spring: injecting an interface with multiple beans (`Map<String, PaymentService>`, `@Qualifier`), `PasswordEncoder`, `HandlerMapping`/`ViewResolver`, `ResourceLoader`.
- `java.util.concurrent.RejectedExecutionHandler` policies; `ThreadPoolExecutor` rejection strategies.

### Trade-offs
- Pros: open/closed; easy testing; removes conditionals; combines well with DI.
- Cons: clients must know the strategies; more types (lambdas reduce this).

### Interview tip
Say "Strategy is composition over inheritance for algorithms". Give the `Comparator` example and mention Spring DI as a natural Strategy selector (e.g., a map of beans keyed by name).

---

## Template Method

### Intent
Define the skeleton of an algorithm in a base class and let subclasses override specific steps without changing the structure.

### When to use
- Several classes share the same algorithm structure but differ in some steps.
- You want to enforce the order of steps and avoid duplicated code.

### Example

```java
abstract class DataExporter {

    // Template method: final so subclasses cannot change the order
    public final void export() {
        open();
        String data = fetch();
        write(transform(data));
        close();
    }

    protected void open()  { System.out.println("Opening"); }       // default step
    protected void close() { System.out.println("Closing"); }
    protected abstract String fetch();                               // mandatory step
    protected String transform(String data) { return data; }         // optional hook
    protected abstract void write(String data);
}

class CsvExporter extends DataExporter {
    @Override protected String fetch() { return "id,name"; }
    @Override protected String transform(String data) { return data.toUpperCase(); }
    @Override protected void write(String data) { System.out.println("CSV: " + data); }
}

public class TemplateDemo {
    public static void main(String[] args) {
        new CsvExporter().export();
        // Opening, CSV: ID,NAME, Closing
    }
}
```

### Real-world Java/Spring usage
- `HttpServlet.service()` calls `doGet`/`doPost` (the hooks).
- `JdbcTemplate`, `RestTemplate`, `TransactionTemplate`, `JmsTemplate`: the template handles connections/resources/exceptions; **you supply a callback** (`RowMapper`, `PreparedStatementCreator`), which is Template Method implemented with Strategy/callbacks.
- `AbstractList`, `InputStream.read(byte[])`, Spring's `AbstractApplicationContext.refresh()`.

### Trade-offs
- Pros: removes duplication; central control of the algorithm order.
- Cons: inheritance-based and rigid (single inheritance); subclasses can break the Liskov principle; hard to follow flow through hooks.

### Interview tip
Mention the **Hollywood principle** ("don't call us, we'll call you") and that Spring's `*Template` classes use the **callback** flavor to avoid subclassing. Contrast with Strategy (composition vs inheritance).

---

## Visitor

### Intent
Represent an operation to perform on the elements of an object structure, letting you add new operations without changing the element classes.

### When to use
- A **stable** set of element types with **frequently changing** operations (export, pricing, validation, printing).
- Operations need type-specific logic across a hierarchy (AST, file system, documents).

### Example
Classic double dispatch:

```java
interface ShapeVisitor<R> {
    R visit(Circle c);
    R visit(Square s);
}

interface Shape {
    <R> R accept(ShapeVisitor<R> v);    // first dispatch; visit overload is the second
}

record Circle(double radius) implements Shape {
    public <R> R accept(ShapeVisitor<R> v) { return v.visit(this); }
}

record Square(double side) implements Shape {
    public <R> R accept(ShapeVisitor<R> v) { return v.visit(this); }
}

class AreaVisitor implements ShapeVisitor<Double> {
    public Double visit(Circle c) { return Math.PI * c.radius() * c.radius(); }
    public Double visit(Square s) { return s.side() * s.side(); }
}

// new Square(3).accept(new AreaVisitor())  -> 9.0
```

Modern alternative (Java 21: sealed types plus pattern-matching `switch`): no `accept` methods are needed, and the compiler checks exhaustiveness.

```java
sealed interface Figure permits Disc, Box {}
record Disc(double radius) implements Figure {}
record Box(double side) implements Figure {}

static double area(Figure f) {
    return switch (f) {
        case Disc d -> Math.PI * d.radius() * d.radius();
        case Box b  -> b.side() * b.side();
    };   // adding a new Figure causes a compile error here until handled
}
```

### Real-world Java/Spring usage
- `java.nio.file.FileVisitor` with `Files.walkFileTree`.
- `javax.lang.model.element.ElementVisitor` (annotation processing), ASM `ClassVisitor`, ANTLR parse-tree visitors.
- Spring's `BeanDefinitionVisitor`.

### Trade-offs
- Pros: new operations are easy; related logic is grouped in one place.
- Cons: adding a new element type forces changes in every visitor; elements must expose enough state; the double-dispatch boilerplate is confusing.

### Interview tip
Explain **double dispatch** (Java dispatches only on the receiver at runtime, so `accept` + `visit` simulates dispatch on two types). Add that sealed types and pattern-matching `switch` often replace Visitor in modern Java.

---

## Common interview questions

- **Strategy vs State?** Same structure; Strategy is chosen by the client and is usually stateless, State transitions itself as the context changes.
- **Strategy vs Template Method?** Composition (swap the entire algorithm at runtime) vs inheritance (fixed skeleton, overridden steps at compile time).
- **Where is Observer used in Spring?** `ApplicationEventPublisher` and `@EventListener`; events are synchronous by default.
- **Why is `java.util.Observable` deprecated?** Not type-safe, a class (not interface), not thread-safe, and poorly designed; use `PropertyChangeListener`, `Flow`, or Spring events.
- **How does the Spring Security filter chain apply Chain of Responsibility?** Ordered filters each process or reject the request, then call `chain.doFilter` to pass it on.
- **How do you implement undo?** Command (with `undo()`) and/or Memento (state snapshots).
- **Mediator vs Observer?** Mediator is a central coordinator of two-way interaction; Observer is one-way broadcast.
- **When would you avoid Visitor?** When the element hierarchy changes often; with sealed types and pattern matching, a `switch` is simpler.
- **Is `JdbcTemplate` Template Method?** Yes, in its callback form: it owns the fixed flow (connection, statement, exception translation, cleanup) and you provide the varying step.

For full answers, see:

- [Design patterns interview questions](../interview-questions/verbal/verbal-design-patterns-questions.md)
- [Design principles and patterns (SOLID and more)](../interview-questions/verbal/verbal-design-patterns-principles-questions.md)
- [Design Patterns catalog](design-patterns.md)
