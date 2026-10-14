# Testing / QA

## Test pyramid
- **Unit** (most) — fast, isolated, mock dependencies.
- **Integration** — components together (DB, external services / virtualization).
- **End-to-end** (fewest) — full system through the UI/API; slow, brittle.

## Black box vs white box

1. **Black Box Testing**
   The technique of testing in which the tester doesn’t have access to the source code of the software and is conducted at the software interface without any concern with the internal logical structure of the software is known as black-box testing.

2. **White-Box Testin**g
   The technique of testing in which the tester is aware of the internal workings of the product, has access to its source code, and is conducted by making sure that all internal operations are performed according to the specifications is known as white box testing.

**Unit Testing**
A level of the software testing process where individual units/components of a software/system are tested. The purpose is to validate that each unit of the software performs as designed.

**Integration Testing**
A level of the software testing process where individual units are combined and tested as a group. The purpose of this level of testing is to expose faults in the interaction between integrated units

**System Testing**
A level of the software testing process where a complete, integrated system/software is tested. The purpose of this test is to evaluate the system’s compliance with the specified requirements.

**Acceptance Testing**
A level of the software testing process where a system is tested for acceptability. The purpose of this test is to evaluate the system’s compliance with the business requirements and assess whether it is acceptable for delivery.

**Regression testing**
Checking whether new features break or degrade functionality

**Smoke tests**
check the basic functionality of an application, check ready for other tests, tests simple and basic functionalities, such as if the user is able to log in or log out.

**Performance tests**
Test the run-time performance of software - load testing

**Stress tests**
check how the system works under unfavorable conditions

## TDD & BDD
See [SDLC → BDD & TDD](sdlc.md#bdd--tdd) for the driven-development approaches.

## Principles & tooling
- **AAA** — Arrange, Act, Assert. Test behavior, not implementation. Deterministic, independent, fast.
- Java: **JUnit 5** + **Mockito**; `@SpringBootTest` / `@WebMvcTest` (MockMvc) for Spring; Testcontainers for real deps.
- Test doubles: dummy · stub · spy · mock · fake. Coverage (JaCoCo) is a guide, not a goal.

---

## JUnit 5 Basics

**JUnit 5** is the testing framework for Java. Structure: `@Test` → Arrange → Act → Assert.

### Maven Dependency
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>
```

### Basic Test Structure

```java
public class CalculatorTest {
    
    private Calculator calculator;
    
    // Run before each test
    @BeforeEach
    public void setup() {
        calculator = new Calculator();
    }
    
    // Run after each test (cleanup)
    @AfterEach
    public void tearDown() {
        calculator = null;
    }
    
    // Actual test method
    @Test
    public void testAddition() {
        // Arrange: set up test data
        int a = 5;
        int b = 3;
        
        // Act: perform the action
        int result = calculator.add(a, b);
        
        // Assert: verify the result
        assertEquals(8, result);
    }
    
    // Skip this test
    @Disabled("Skipping for now")
    @Test
    public void testDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
}
```

### Lifecycle Annotations

| Annotation | Runs | Usage |
|-----------|------|-------|
| `@BeforeEach` | Before each test | Initialize test fixtures |
| `@AfterEach` | After each test | Cleanup resources |
| `@BeforeAll` | Once before all tests (static) | Expensive setup |
| `@AfterAll` | Once after all tests (static) | Resource cleanup |
| `@Disabled` | Skips test | Temporarily disable |

```java
public class DatabaseTest {
    private static Database db;
    
    // Run once at the beginning (static method)
    @BeforeAll
    static void setupDatabase() {
        db = new Database("jdbc:mysql://localhost:3306/testdb");
        db.connect();
    }
    
    // Run once at the end (static method)
    @AfterAll
    static void tearDownDatabase() {
        db.disconnect();
    }
}
```

### Display Names & Tags

```java
@DisplayName("Calculator Tests")
public class CalculatorTest {
    
    @DisplayName("Should add two numbers correctly")
    @Test
    @Tag("math")
    public void testAdd() {
        assertEquals(8, calculator.add(5, 3));
    }
    
    @DisplayName("Should throw exception on division by zero")
    @Test
    @Tag("exception")
    public void testDivisionError() {
        assertThrows(ArithmeticException.class, () -> calculator.divide(10, 0));
    }
}
```

Run specific tag: `mvn test -Dgroups=math`

---

## JUnit Assertions

Common assertions for verifying test outcomes.

### Basic Assertions

```java
// Equality
assertEquals(expected, actual);
assertEquals(8, calculator.add(5, 3));

// Inequality
assertNotEquals(5, calculator.add(2, 2));

// Null checks
assertNull(nullValue);
assertNotNull(nonNullValue);

// Boolean
assertTrue(condition);
assertFalse(!condition);
```

### Exception Assertions

```java
// Verify exception is thrown
@Test
public void testException() {
    ArithmeticException ex = assertThrows(
        ArithmeticException.class,
        () -> calculator.divide(10, 0)
    );
    assertEquals("Cannot divide by zero", ex.getMessage());
}

// Verify no exception
@Test
public void testNoException() {
    assertDoesNotThrow(() -> {
        calculator.divide(10, 2);
    });
}
```

### Collection Assertions

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

// Contains
assertTrue(list.contains(3));
assertAll(
    () -> assertTrue(list.contains(1)),
    () -> assertTrue(list.contains(5))
);

// Size
assertEquals(5, list.size());

// Is empty
assertTrue(emptyList.isEmpty());
```

### Grouped Assertions

```java
@Test
public void testUser() {
    User user = userService.createUser("John", "Doe");
    
    // All assertions must pass; failures reported together
    assertAll("User validation",
        () -> assertEquals("John", user.getFirstName()),
        () -> assertEquals("Doe", user.getLastName()),
        () -> assertNotNull(user.getId())
    );
}
```

### Timeout Assertions

```java
@Test
public void testPerformance() {
    // Code must complete within 100ms
    assertTimeout(Duration.ofMillis(100), () -> {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            list.add(i);
        }
    });
    
    // Aborts if exceeds timeout
    assertTimeoutPreemptively(Duration.ofSeconds(1), () -> {
        Thread.sleep(2000);  // Aborted
    });
}
```

### Custom Messages

```java
@Test
public void testWithMessage() {
    assertEquals(
        8, 
        calculator.add(5, 3),
        "Addition should equal 8"  // Message on failure
    );
    
    assertEquals(
        8, 
        calculator.add(5, 3),
        () -> "Addition of 5 + 3 should equal 8"  // Lazy evaluation (only if fails)
    );
}
```

---

## Mockito - Mocking Framework

**Mockito** creates mock objects to isolate units under test. Mock external dependencies instead of real ones.

### Maven Dependency

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.3.1</version>
    <scope>test</scope>
</dependency>

<!-- For Spring Boot tests -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Creating Mocks

```java
public class OrderServiceTest {
    
    private OrderService orderService;
    private PaymentGateway paymentGateway;  // Mock
    private UserRepository userRepository;  // Mock
    
    @BeforeEach
    public void setup() {
        // Method 1: Create mock explicitly
        paymentGateway = mock(PaymentGateway.class);
        userRepository = mock(UserRepository.class);
        
        // Inject mocks into service under test
        orderService = new OrderService(paymentGateway, userRepository);
    }
    
    // OR use @Mock annotation with MockitoAnnotations.openMocks()
    @Mock
    private PaymentGateway paymentGateway;
    
    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);  // Initializes @Mock fields
    }
}
```

### Stubbing (Configure Mock Behavior)

```java
@Test
public void testOrderProcessing() {
    // Arrange: configure mock to return specific value
    when(userRepository.findById(1L))
        .thenReturn(Optional.of(new User(1L, "John")));
    
    when(paymentGateway.charge(100.0))
        .thenReturn(true);
    
    // Act
    Order order = orderService.createOrder(1L, 100.0);
    
    // Assert
    assertTrue(order.isPaid());
}
```

### Stubbing Different Scenarios

```java
// Return value on each call
when(mock.method())
    .thenReturn(1)
    .thenReturn(2)
    .thenReturn(3);

// Throw exception
when(userRepository.findById(999L))
    .thenThrow(new UserNotFoundException("Not found"));

// Return based on argument
when(userRepository.findById(1L))
    .thenReturn(Optional.of(user1));
when(userRepository.findById(2L))
    .thenReturn(Optional.of(user2));

// Matcher-based stubbing (any, eq, argThat)
when(userRepository.findById(any(Long.class)))
    .thenReturn(Optional.of(anyUser));

when(paymentGateway.charge(anyDouble()))
    .thenReturn(true);

// Custom matcher
when(userRepository.findByEmail(argThat(email -> email.endsWith("@example.com"))))
    .thenReturn(Optional.of(user));

// Do-nothing for void methods
doNothing().when(notificationService).sendEmail(any());

// Do-something for void methods
doAnswer(invocation -> {
    String email = invocation.getArgument(0);
    System.out.println("Sending email to: " + email);
    return null;
}).when(notificationService).sendEmail(any());
```

### Verifying Mock Interactions

```java
@Test
public void testOrderPayment() {
    orderService.createOrder(1L, 100.0);
    
    // Verify method was called at least once
    verify(paymentGateway).charge(100.0);
    
    // Verify called exactly N times
    verify(paymentGateway, times(1)).charge(100.0);
    
    // Verify called with specific arguments
    verify(userRepository).findById(1L);
    
    // Verify never called
    verify(notificationService, never()).sendEmail(any());
    
    // Verify call order
    InOrder inOrder = inOrder(userRepository, paymentGateway);
    inOrder.verify(userRepository).findById(1L);
    inOrder.verify(paymentGateway).charge(100.0);
}
```

### Spies (Partial Mocks)

Wrap real object; mock specific methods, call real methods for others:

```java
@Test
public void testWithSpy() {
    List<String> list = new ArrayList<>();
    List<String> spyList = spy(list);  // Spy on real ArrayList
    
    // Stub one method
    when(spyList.size()).thenReturn(100);
    
    spyList.add("item");
    
    // size() returns stubbed value
    assertEquals(100, spyList.size());
    
    // add() executes real method
    assertEquals(1, list.size());  // Real list has 1 item
    
    verify(spyList).add("item");
}
```

### Spring + Mockito Integration

```java
@SpringBootTest
public class UserServiceIntegrationTest {
    
    @MockBean  // Mock bean in Spring context
    private UserRepository userRepository;
    
    @Autowired
    private UserService userService;  // Real service with mocked repo
    
    @Test
    public void testGetUser() {
        // Arrange
        User user = new User(1L, "John");
        when(userRepository.findById(1L))
            .thenReturn(Optional.of(user));
        
        // Act
        User result = userService.getUser(1L);
        
        // Assert
        assertEquals("John", result.getName());
        verify(userRepository).findById(1L);
    }
}
```

### Best Practices

✅ **Do's**
- Mock external dependencies (DB, HTTP, APIs)
- Use `verify()` to confirm interactions
- Keep tests focused and independent
- Use argument matchers for flexibility
- Test behavior, not implementation

❌ **Don'ts**
- Don't mock the class under test
- Don't over-mock (keep tests readable)
- Don't verify details that shouldn't matter
- Don't use real objects when mocks make sense
- Don't forget to verify void method calls
