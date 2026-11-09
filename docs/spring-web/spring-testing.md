# Spring Boot Testing: Mockito, Slices and Testcontainers

Pick the smallest test that proves the behaviour. Plain unit tests with Mockito are fastest; **test slices** load only part of the context; `@SpringBootTest` loads everything. General testing theory, the pyramid and Mockito basics live in [Testing](../foundations/testing.md).

## Test Types at a Glance

| Test | Annotation | Loads | Speed | Use for |
| :-- | :-- | :-- | :-- | :-- |
| Unit | `@ExtendWith(MockitoExtension.class)` | Nothing from Spring | Milliseconds | Service logic |
| Web slice | `@WebMvcTest` | Controllers, advice, filters, converters, MockMvc | Fast | Routing, validation, JSON, status codes, security rules |
| JPA slice | `@DataJpaTest` | Entities, repositories, `DataSource`, Hibernate (transactional) | Medium | Queries, mappings, constraints |
| JSON slice | `@JsonTest` | Jackson config | Fast | Serialisation shape |
| Full | `@SpringBootTest` | Entire context | Slow | Wiring, end-to-end with real or contained dependencies |

## Mockito: `@Mock` vs `@InjectMocks`

| Annotation | Meaning |
| :-- | :-- |
| `@Mock` | Creates a mock of a **collaborator** (all methods return defaults until stubbed) |
| `@InjectMocks` | Creates a **real instance of the class under test** and injects the available mocks/spies into it |
| `@Spy` | Wraps a real object; real methods run unless stubbed |
| `@Captor` | Creates an `ArgumentCaptor` for inspecting arguments |

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock PaymentClient payments;            // collaborator: fake
    @Mock OrderRepository repo;              // collaborator: fake
    @InjectMocks OrderService service;       // real OrderService(payments, repo)

    @Test
    void placesOrder() {
        when(payments.charge(any())).thenReturn(true);
        service.place(new Order("A1"));
        verify(repo).save(any(Order.class));
    }
}
```

**Injection order:** Mockito tries the biggest **constructor** first, then **setter/property** injection, then **field** injection, matching by type (then by name when several mocks share a type). If the constructor injection works, it does not fall through to fields.

!!! warning "Gotchas"
    - Constructor injection failing silently: if a constructor argument has no matching mock, Mockito passes `null`. Prefer constructor-injected services and check for NPEs.
    - Never put `@InjectMocks` on a field you also mock; the class under test is real.
    - `@Mock` needs `MockitoExtension` (or `MockitoAnnotations.openMocks(this)`); without it the fields stay `null`.
    - In Spring tests use `@MockitoBean` (Boot 3.4+, Framework 6.2) or `@MockBean` (older) to replace a **bean in the context**; that is different from plain `@Mock`.

## `@WebMvcTest`

Starts only the web layer for the controllers you name. Services and repositories are **not** created, so mock them.

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mvc;
    @MockitoBean UserService service;         // @MockBean before Boot 3.4

    @Test
    void returnsUser() throws Exception {
        when(service.get(1L)).thenReturn(new UserDto(1L, "Alice"));

        mvc.perform(get("/users/1").accept(MediaType.APPLICATION_JSON))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.name").value("Alice"));
    }

    @Test
    void rejectsInvalidBody() throws Exception {
        mvc.perform(post("/users").contentType(MediaType.APPLICATION_JSON).content("{\"name\":\"\"}"))
           .andExpect(status().isBadRequest());   // @Valid + @ControllerAdvice are exercised
    }
}
```

- Spring Security is auto-applied: add `@WithMockUser` or `.with(csrf())`, or import your `SecurityFilterChain`.
- `@Service`/`@Repository` beans are excluded; a missing bean error means you forgot a mock.
- `MockMvc` does not start a server; use `@SpringBootTest(webEnvironment = RANDOM_PORT)` with `TestRestTemplate`/`RestTestClient` for real HTTP.

## `@DataJpaTest`

Configures only JPA components, rolls back each test, and by default **replaces your DataSource with an embedded H2**.

```java
@DataJpaTest
class UserRepositoryTest {
    @Autowired UserRepository repo;
    @Autowired TestEntityManager em;

    @Test
    void findsByEmail() {
        em.persist(new User("a@x.com"));
        em.flush(); em.clear();               // force SQL and bypass first-level cache
        assertThat(repo.findByEmail("a@x.com")).isPresent();
    }
}
```

!!! warning "H2 is not your production database"
    SQL dialect differences (JSON columns, sequences, `ON CONFLICT`, locking) pass on H2 and fail on PostgreSQL. Run repository tests against the real engine with Testcontainers and add `@AutoConfigureTestDatabase(replace = Replace.NONE)` (not needed when using `@ServiceConnection`).

## Testcontainers

Runs real dependencies (PostgreSQL, Kafka, Redis) in disposable Docker containers. Needs a Docker runtime on the machine/CI.

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryPgTest {

    @Container
    @ServiceConnection                        // Boot 3.1+: wires spring.datasource.* automatically
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired UserRepository repo;
    // tests...
}
```

| Approach | Notes |
| :-- | :-- |
| `@ServiceConnection` (Boot 3.1+) | Preferred; no property wiring |
| `@DynamicPropertySource` | Manual: `registry.add("spring.datasource.url", pg::getJdbcUrl)` |
| `static` container | Shared across tests in the class (fast); non-static restarts per test (slow) |
| Singleton / reuse | Share one container across classes via a base class or `withReuse(true)` for local dev |
| Schema | Let Flyway/Liquibase migrations run so the test uses the real schema |

## Choosing and Speed Tips

- Prefer slices over `@SpringBootTest`; Spring caches contexts, so identical config across tests reuses the context. Avoid `@MockitoBean` sprinkled differently in every class, since each combination creates a new context.
- Keep controller tests about HTTP concerns, repository tests about SQL, service tests about logic.
- Run integration tests with real dependencies in CI, not only mocks.

## Common Interview Questions

- **`@Mock` vs `@InjectMocks`?** `@Mock` is a fake collaborator; `@InjectMocks` builds the real class under test and injects the mocks.
- **`@Mock` vs `@MockBean`/`@MockitoBean`?** The first is plain Mockito with no Spring; the others replace a bean inside the Spring context.
- **What does `@WebMvcTest` load?** Only the web layer; no services or repositories.
- **Why not rely on H2 for repository tests?** Dialect and behaviour drift; use Testcontainers for the production engine.
- **`@SpringBootTest` vs slices?** Full context is slow and tests wiring; slices are focused and fast.

**See also:** [Testing](../foundations/testing.md), [JPA and ORM](jpa-orm.md), [Spring Boot Versions](spring-boot-versions.md)
