# Design Principles and Applied Patterns Interview Questions

SOLID, Liskov substitution, interface refactoring, and how design patterns apply in high-stakes financial systems.

---

## 1. Explain the SOLID principles

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>SOLID is five object-oriented design guidelines that keep code easy to change and test.</p>

<table>
<tr><th>Principle</th><th>Meaning</th><th>Typical violation</th></tr>
<tr><td><strong>S</strong>ingle Responsibility</td><td>A class has one reason to change</td><td>A service that validates, persists, and sends emails</td></tr>
<tr><td><strong>O</strong>pen/Closed</td><td>Open for extension, closed for modification</td><td>Growing <code>if/switch</code> on type for each new case</td></tr>
<tr><td><strong>L</strong>iskov Substitution</td><td>Subtypes must be usable wherever the base type is expected</td><td>Subclass throws <code>UnsupportedOperationException</code> for an inherited method</td></tr>
<tr><td><strong>I</strong>nterface Segregation</td><td>Clients should not depend on methods they do not use</td><td>A 20-method interface where each client uses 3</td></tr>
<tr><td><strong>D</strong>ependency Inversion</td><td>Depend on abstractions, not concretions</td><td><code>new MySqlRepo()</code> inside a service</td></tr>
</table>

<h3>Example: Open/Closed and Dependency Inversion</h3>

```java
// Closed for modification: new discounts are new classes, no edits here
interface DiscountPolicy { double apply(double price); }

class PricingService {
    private final DiscountPolicy policy;           // depends on an abstraction
    PricingService(DiscountPolicy policy) { this.policy = policy; }   // injected
    double finalPrice(double price) { return policy.apply(price); }
}
```

<h3>Gotchas and Follow-ups</h3>
<ul>
<li><strong>Dependency Inversion is not Dependency Injection.</strong> DIP is the design principle; DI (for example Spring) is a technique that supplies the dependency.</li>
<li>These are guidelines, not laws: over-applying SRP/OCP leads to a swarm of tiny classes and needless abstractions.</li>
<li>Be ready to give a violation and fix for each letter from your own code.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/design-principles/">Learn more: Software Design Principles</a></p>

</details>

---

## 2. Explain the Liskov Substitution Principle. If a subclass can replace its parent, why is inheritance needed?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>LSP:</strong> if <code>S</code> is a subtype of <code>T</code>, objects of <code>T</code> can be replaced with objects of <code>S</code> without breaking the correctness of the program. Substitutability concerns <em>behavior and contracts</em>, not only method signatures.</p>

<h3>Clarifying the Question</h3>
<ul>
<li>LSP does not say inheritance is unnecessary. It is a <em>constraint</em> on when inheritance is valid: use inheritance for real "is-a" behavioral relationships, otherwise prefer composition.</li>
<li>Inheritance provides <strong>code reuse</strong> and <strong>polymorphism</strong>; LSP ensures that polymorphism is safe for callers.</li>
</ul>

<h3>Contract Rules</h3>
<ul>
<li>Preconditions cannot be strengthened in the subtype; postconditions cannot be weakened.</li>
<li>Invariants must be preserved; do not throw new unexpected exceptions.</li>
</ul>

<h3>Classic Violation</h3>

```java
class Rectangle {
    protected int w, h;
    void setWidth(int w)  { this.w = w; }
    void setHeight(int h) { this.h = h; }
    int area() { return w * h; }
}

class Square extends Rectangle {          // violates LSP
    @Override void setWidth(int w)  { this.w = w; this.h = w; }
    @Override void setHeight(int h) { this.w = h; this.h = h; }
}

void resize(Rectangle r) {
    r.setWidth(5); r.setHeight(4);
    assert r.area() == 20;                // fails for Square (16)
}
```

<h3>Fixes</h3>
<ul>
<li>Do not model Square as a subtype of a mutable Rectangle; use immutable shapes or a common <code>Shape</code> interface with <code>area()</code>.</li>
<li>Smell: a subclass that overrides a method to throw or do nothing (for example <code>ReadOnlyList.add</code>), or <code>instanceof</code> checks in client code.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/design-principles/">Learn more: Software Design Principles</a></p>

</details>

---

## 3. How do you break down a large "God interface" without breaking existing code? How do you decide which implementation each consumer uses?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Apply Interface Segregation <strong>incrementally</strong>: extract small role interfaces, make the old interface extend them, migrate consumers one by one, then deprecate the old interface. Nothing breaks because the old type still exists the whole time.</p>

<h3>Steps</h3>
<ol>
<li><strong>Analyze usage:</strong> group methods by which consumers call them (IDE usage search, call graph).</li>
<li><strong>Extract role interfaces:</strong> <code>OrderReader</code>, <code>OrderWriter</code>, <code>OrderReporting</code>.</li>
<li><strong>Keep compatibility:</strong> the old interface extends the new ones, so existing implementations and callers compile unchanged.</li>
<li><strong>Migrate consumers</strong> to the narrowest interface they need; migrate implementations one at a time.</li>
<li><strong>Deprecate</strong> (<code>@Deprecated(since=..., forRemoval=true)</code>) and remove once usages reach zero.</li>
</ol>

<h3>Example</h3>

```java
interface OrderReader  { Order find(long id); }
interface OrderWriter  { void save(Order o); }

/** @deprecated use OrderReader / OrderWriter */
@Deprecated
interface OrderService extends OrderReader, OrderWriter {
    void exportAll();            // legacy leftovers, move out later
}

class OrderServiceImpl implements OrderService { /* unchanged */ }

class ReportJob {
    private final OrderReader reader;           // narrowest dependency
    ReportJob(OrderReader reader) { this.reader = reader; }
}
```

<h3>Choosing the Implementation per Consumer</h3>
<ul>
<li>With Spring, inject by role interface; one bean can implement several role interfaces and be injected under each.</li>
<li>For multiple implementations of the same interface use <code>@Qualifier</code>, <code>@Primary</code>, or inject a <code>Map&lt;String, Interface&gt;</code>/<code>List</code> and select by key (Strategy).</li>
<li>Default methods can add new methods to an interface without breaking implementers, but use them sparingly; they do not fix a bloated interface.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Cover current behavior with tests first. Binary compatibility matters for published libraries (keep old signatures until a major version).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/design-principles/">Learn more: Software Design Principles</a></p>

</details>

---

## 4. Discuss commonly used design patterns in Java and how they apply to financial systems like derivatives clearing

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>In high-stakes financial systems, patterns are used to keep rules swappable, flows auditable, and failures contained. The domain here is generic (trade processing, margin, settlement); the same ideas apply to any rules-heavy back-office system.</p>

<table>
<tr><th>Pattern</th><th>Use in a clearing/trade-processing context</th></tr>
<tr><td>Strategy</td><td>Interchangeable margin, pricing, or fee calculation models per product or rule set</td></tr>
<tr><td>Factory</td><td>Create the right trade/instrument handler from a product type</td></tr>
<tr><td>Builder</td><td>Construct immutable trade objects with many optional fields</td></tr>
<tr><td>Observer / events</td><td>Notify risk, reporting, and ledger components when a trade changes state</td></tr>
<tr><td>State</td><td>Trade lifecycle: new, matched, novated, settled, cancelled</td></tr>
<tr><td>Chain of Responsibility</td><td>Validation and risk-check pipeline</td></tr>
<tr><td>Command</td><td>Capture requests as objects for audit, replay, and retry</td></tr>
<tr><td>Template Method</td><td>Common end-of-day batch skeleton with product-specific steps</td></tr>
<tr><td>Decorator / Proxy</td><td>Add audit logging, caching, or access control without touching business logic</td></tr>
</table>

<h3>Example: Strategy for Calculation Models</h3>

```java
interface MarginModel {
    BigDecimal requiredMargin(Position position);
}

class PortfolioMarginModel implements MarginModel { /* ... */ }
class FlatRateMarginModel  implements MarginModel { /* ... */ }

class MarginService {
    private final Map<ProductType, MarginModel> models;
    MarginService(Map<ProductType, MarginModel> models) { this.models = models; }

    BigDecimal calculate(Position p) {
        return models.get(p.productType()).requiredMargin(p);
    }
}
```

<h3>Gotchas</h3>
<ul>
<li>Money values should use <code>BigDecimal</code> (with an explicit scale and rounding mode), never <code>double</code>.</li>
<li>Prefer immutable domain objects and idempotent handlers so replays are safe.</li>
<li>Do not force patterns: justify each by a concrete change or risk it addresses.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 5. Which design pattern do you find most relevant in clearing systems and why?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A defensible answer is <strong>Strategy</strong>, paired with <strong>State</strong> for lifecycle. Clearing/trade-processing systems are dominated by rules that differ per product, market, or regulation and change often, and by entities that move through strict lifecycles.</p>

<h3>Why Strategy</h3>
<ul>
<li>Margin, fee, and valuation rules vary by product; new ones arrive regularly. Strategy lets you add a rule without modifying tested code (Open/Closed).</li>
<li>Each model is unit-testable in isolation and can be selected by configuration or product type.</li>
<li>Versioned strategies support reproducing a past calculation for audit.</li>
</ul>

<h3>Why State</h3>
<ul>
<li>Illegal transitions (settling a cancelled trade) must be impossible; encoding the lifecycle in states or an explicit transition table makes it enforceable and auditable.</li>
</ul>

<h3>How to Present It in an Interview</h3>
<ul>
<li>Name the pattern, the concrete problem it solved, the alternative you rejected (a big switch on product type), and the trade-off (more classes, indirection).</li>
<li>If you are asked for a different answer, Chain of Responsibility (validation/risk pipeline) and Observer (event-driven downstream updates) are also strong picks.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---
