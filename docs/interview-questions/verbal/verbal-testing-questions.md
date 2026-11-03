# Testing Questions

Verbal interview questions on test levels, JUnit and Mockito practices, and keeping test suites reliable.

---

## 1. What are the different levels of testing?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Tests are layered by scope: unit, integration, system, and acceptance. The test pyramid says to have many fast unit tests, fewer integration tests, and very few end-to-end tests.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Level</th><th>Scope</th><th>Speed</th><th>Typical tools</th></tr>
<tr><td>Unit</td><td>One class/method, dependencies mocked</td><td>Milliseconds</td><td>JUnit 5, Mockito, AssertJ</td></tr>
<tr><td>Integration</td><td>Several components or a real DB/broker/HTTP boundary</td><td>Seconds</td><td>Spring Boot Test, Testcontainers, WireMock</td></tr>
<tr><td>System / end-to-end</td><td>Whole application through its public interface (UI/API)</td><td>Slow</td><td>REST Assured, Selenium, Playwright</td></tr>
<tr><td>Acceptance (UAT)</td><td>Business requirements validated by stakeholders</td><td>Slowest</td><td>Cucumber (BDD), manual UAT</td></tr>
</table>

<h3>Other Dimensions</h3>
<ul>
<li><strong>Contract tests:</strong> verify service-to-service API agreements (Pact, Spring Cloud Contract).</li>
<li><strong>Non-functional:</strong> performance/load (JMeter, Gatling), security, resilience.</li>
<li><strong>Regression and smoke:</strong> re-run suites after change; quick sanity check after deploy.</li>
</ul>

<h3>Gotcha</h3>
<p>The "ice cream cone" anti-pattern (mostly slow UI tests, few unit tests) gives slow, flaky feedback.</p>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 2. What is the difference between unit testing and functional or end-to-end testing? What are the developer responsibilities?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A unit test checks one small piece of logic in isolation, with collaborators mocked. A functional or end-to-end test checks a complete user-visible behavior through the real, integrated stack.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Unit</th><th>Functional / End-to-end</th></tr>
<tr><td>Question answered</td><td>Does this code do what I wrote?</td><td>Does the system do what the user needs?</td></tr>
<tr><td>Dependencies</td><td>Mocked or stubbed</td><td>Real (DB, services, UI), or a production-like environment</td></tr>
<tr><td>Speed and stability</td><td>Fast, deterministic</td><td>Slower, more prone to flakiness</td></tr>
<tr><td>Failure diagnosis</td><td>Pinpoints the method</td><td>Needs investigation across components</td></tr>
<tr><td>Quantity</td><td>Many</td><td>Few, covering critical journeys</td></tr>
</table>

<h3>Developer Responsibilities</h3>
<ul>
<li>Write and maintain unit tests for new logic, including edge cases and error paths; fix a bug by first writing a failing test.</li>
<li>Write integration tests for persistence, messaging and API boundaries.</li>
<li>Keep the build green in CI; do not merge failing or skipped tests; fix flaky tests.</li>
<li>Contribute to or support end-to-end tests for critical flows (often shared with QA).</li>
<li>Review test quality in code reviews; keep test data and environments reproducible.</li>
</ul>

<h3>Note</h3>
<p>"Functional" testing checks behavior against requirements and can be done at the API level, without a UI, which is cheaper and more stable than browser-driven E2E.</p>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 3. What are commonly used JUnit test cases and best practices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Write small, independent, readable tests that verify one behavior, using JUnit 5 annotations and clear assertions.</p>

<h3>Common Building Blocks (JUnit 5)</h3>
<table>
<tr><th>Annotation</th><th>Purpose</th><th>JUnit 4 equivalent</th></tr>
<tr><td><code>@Test</code></td><td>Marks a test method</td><td><code>@Test</code></td></tr>
<tr><td><code>@BeforeEach</code> / <code>@AfterEach</code></td><td>Run around every test</td><td><code>@Before</code> / <code>@After</code></td></tr>
<tr><td><code>@BeforeAll</code> / <code>@AfterAll</code></td><td>Once per class (static)</td><td><code>@BeforeClass</code> / <code>@AfterClass</code></td></tr>
<tr><td><code>@ParameterizedTest</code></td><td>Same test, many inputs</td><td>Parameterized runner</td></tr>
<tr><td><code>@Disabled</code>, <code>@Tag</code>, <code>@Nested</code></td><td>Skip, group, organize</td><td><code>@Ignore</code>, categories</td></tr>
</table>

<h3>Best Practices</h3>
<ul>
<li><strong>Arrange-Act-Assert</strong> structure; one logical assertion focus per test.</li>
<li><strong>Descriptive names</strong> (<code>shouldRejectNegativeAmount</code>) or <code>@DisplayName</code>.</li>
<li><strong>Independent and repeatable:</strong> no shared mutable state, no order dependence, no reliance on time or network.</li>
<li><strong>Test behavior, not implementation</strong> so refactoring does not break tests.</li>
<li>Use <code>assertThrows</code> for exceptions, <code>assertAll</code> for grouped checks, parameterized tests for tables of cases.</li>
<li>Keep them fast; avoid <code>Thread.sleep</code>.</li>
</ul>

<h3>Example</h3>

```java
class FeeCalculatorTest {
    private final FeeCalculator calc = new FeeCalculator();

    @ParameterizedTest
    @CsvSource({"0,0", "100,1", "1000,10"})
    void feeIsOnePercent(int amount, int expected) {
        assertEquals(expected, calc.fee(amount));
    }

    @Test
    void rejectsNegativeAmount() {
        assertThrows(IllegalArgumentException.class, () -> calc.fee(-1));
    }
}
```

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 4. How do you approach writing unit tests and integration tests for Java applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Unit-test business logic in isolation with JUnit and Mockito; integration-test the boundaries (database, HTTP, messaging) against real or containerized dependencies; run all of it automatically in CI. Optionally drive design with TDD.</p>

<h3>Unit Tests</h3>
<ul>
<li>Plain JUnit 5 for pure logic; Mockito for collaborators (<code>@Mock</code>, <code>@InjectMocks</code>).</li>
<li>Prefer constructor injection so classes are easy to test without Spring.</li>
<li>TDD loop: red (failing test), green (minimal code), refactor.</li>
</ul>

<h3>Integration Tests</h3>
<ul>
<li><code>@SpringBootTest</code> or slice tests (<code>@WebMvcTest</code>, <code>@DataJpaTest</code>) for faster feedback.</li>
<li><strong>Testcontainers</strong> for a real PostgreSQL/Kafka instead of an in-memory substitute that behaves differently.</li>
<li>WireMock or MockWebServer for external HTTP APIs; contract tests between services.</li>
<li>Keep data isolated per test (transactions rolled back, or clean setup).</li>
</ul>

<h3>Continuous Integration</h3>
<ul>
<li>Unit tests on every commit (Maven Surefire); integration tests in a separate phase (Failsafe, <code>*IT</code>).</li>
<li>Fail the build on test failures; track coverage (JaCoCo) as a guide, not a goal.</li>
</ul>

<h3>Example</h3>

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock PaymentClient payments;
    @InjectMocks OrderService service;

    @Test
    void marksOrderPaidWhenPaymentSucceeds() {
        when(payments.charge(any())).thenReturn(PaymentResult.OK);
        assertEquals(Status.PAID, service.checkout(new Order(100)).status());
    }
}
```

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 5. How do you ensure that tests cover edge cases in critical financial systems?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Systematically enumerate boundaries and failure modes, test money arithmetic precisely, and add property-based and concurrency tests where example-based tests are not enough.</p>

<h3>Techniques</h3>
<ul>
<li><strong>Boundary value analysis:</strong> zero, one cent, maximum, just below and above limits, negative amounts, empty and null inputs.</li>
<li><strong>Precise arithmetic:</strong> use <code>BigDecimal</code> (never <code>double</code>), with explicit scale and <code>RoundingMode</code>; test rounding (half-even versus half-up), currency scale differences, and allocation that must sum exactly (remainder cents).</li>
<li><strong>Equivalence classes</strong> and decision tables for rules (fees, limits, tiers).</li>
<li><strong>Parameterized tests</strong> with tables of known-good values, ideally from the specification or an independent reference calculation.</li>
<li><strong>Property-based tests</strong> (jqwik): invariants such as "total debits equal total credits" or "split parts sum to the original".</li>
<li><strong>Time:</strong> inject a <code>Clock</code>; test month-end, leap years, DST and time zones.</li>
<li><strong>Failure and concurrency:</strong> timeouts, duplicate requests (idempotency), concurrent updates, partial failure and rollback.</li>
<li><strong>Quality checks:</strong> branch coverage plus mutation testing (PIT) to show assertions actually catch bugs; review production incidents and add regression tests.</li>
</ul>

<h3>Example</h3>

```java
@Test
void splitSumsExactly() {
    List<BigDecimal> parts = Money.split(new BigDecimal("100.00"), 3);
    assertEquals(new BigDecimal("100.00"),
        parts.stream().reduce(BigDecimal.ZERO, BigDecimal::add));
}
```

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 6. How do you ensure that tests remain efficient and relevant as the system grows?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Treat test code as production code: keep the pyramid balanced, remove flakiness and duplication, run tests in parallel in CI, and prune tests that no longer protect real behavior.</p>

<h3>Practices</h3>
<ul>
<li><strong>Balance the pyramid:</strong> push checks down to fast unit tests; keep E2E to critical journeys.</li>
<li><strong>Test behavior, not implementation:</strong> avoid over-mocking, so refactors do not force mass test edits.</li>
<li><strong>Refactor tests:</strong> shared builders and fixtures, helper methods, no copy-paste.</li>
<li><strong>Fix or quarantine flaky tests</strong> immediately; flaky tests destroy trust.</li>
<li><strong>Speed:</strong> parallel execution, reuse containers across a test class or suite, slice tests instead of full context, cache Spring contexts; track slowest tests.</li>
<li><strong>CI strategy:</strong> fast checks on every PR, heavier suites nightly; run only affected modules where possible.</li>
<li><strong>Relevance:</strong> delete obsolete tests, review coverage and mutation score for gaps, add a regression test with each bug fix.</li>
<li><strong>Metrics:</strong> build duration, flake rate, failure causes.</li>
</ul>

<h3>Gotcha</h3>
<p>Chasing a coverage percentage encourages low-value tests; focus on risk and meaningful assertions.</p>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 7. What strategies do you use for mocking external dependencies?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Isolate the code under test by replacing external collaborators (databases, HTTP services, queues, clock) with test doubles, and use real dependencies where behavior at the boundary matters.</p>

<h3>Test Doubles</h3>
<table>
<tr><th>Type</th><th>Purpose</th></tr>
<tr><td>Dummy</td><td>Fills a parameter, never used</td></tr>
<tr><td>Stub</td><td>Returns canned answers</td></tr>
<tr><td>Mock</td><td>Stub plus verification of interactions</td></tr>
<tr><td>Spy</td><td>Real object with some methods overridden or recorded</td></tr>
<tr><td>Fake</td><td>Lightweight working implementation (in-memory repository)</td></tr>
</table>

<h3>Strategies</h3>
<ul>
<li><strong>Unit level:</strong> Mockito (<code>when(...).thenReturn(...)</code>, <code>verify(...)</code>); inject dependencies via constructor.</li>
<li><strong>HTTP services:</strong> WireMock or MockWebServer to simulate responses, errors and timeouts.</li>
<li><strong>Databases/brokers:</strong> Testcontainers rather than mocks when query or protocol behavior matters.</li>
<li><strong>Time and randomness:</strong> inject <code>Clock</code> and random sources.</li>
<li><strong>Spring:</strong> <code>@MockBean</code> (or <code>@MockitoBean</code> in Spring Framework 6.2+/Boot 3.4+) to replace beans in the context.</li>
<li><strong>Contract tests</strong> to keep mocks honest against the real provider.</li>
</ul>

<h3>Example</h3>

```java
when(rateClient.getRate("USD")).thenReturn(new BigDecimal("1.10"));
when(rateClient.getRate("XXX")).thenThrow(new RateUnavailableException());

service.convert(new BigDecimal("10"), "USD");
verify(rateClient).getRate("USD");
```

<h3>Gotchas</h3>
<ul>
<li>Do not mock types you do not own deeply or value objects; do not over-verify interactions (brittle tests).</li>
<li>Mocks can drift from reality; pair them with a few integration or contract tests.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---
