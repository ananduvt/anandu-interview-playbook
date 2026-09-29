# Java Testing

## JUnit 5
- Modules: Jupiter (API), Platform (runner), Vintage (JUnit 4).
- Annotations: `@Test`, `@BeforeEach/@AfterEach`, `@BeforeAll/@AfterAll`, `@DisplayName`, `@Nested`,
  `@ParameterizedTest`, `@Disabled`.
- Assertions: `assertEquals`, `assertThrows`, `assertAll`, `assertTimeout`. Or AssertJ `assertThat(x).isEqualTo(...)`.

## Mockito
- `mock()`, `@Mock`, `@InjectMocks`.
- Stub: `when(repo.find(1)).thenReturn(entity)`.
- Verify: `verify(repo).save(any())`; `verify(x, times(2))`.
- Spy for partial mocks; `ArgumentCaptor` to assert arguments.

## Spring testing
- `@SpringBootTest` — full context (integration).
- `@WebMvcTest` + **MockMvc** — controller slice.
- `@DataJpaTest` — repository slice.
- Testcontainers — real DB/Kafka in Docker for integration tests.

## Testing private methods
Usually a **smell** — test via the public API that uses them. If truly needed: refactor to a package-private
method/class, or use reflection (last resort).

## Principles
AAA (Arrange-Act-Assert), one logical assertion, deterministic, independent, fast. Test behavior not internals.
