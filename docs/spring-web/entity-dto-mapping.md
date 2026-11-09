# Entity vs DTO, Projections and MapStruct

An **entity** models persistence (table mapping, lazy associations, managed by the persistence context). A **DTO** (data transfer object) models a contract with a client or another layer. Keeping them separate lets the database and the API evolve independently. Records as DTOs and `select new` basics are in [OOP: mapping query results to records](../java-core/oop.md#mapping-query-results-to-records); entity lifecycle is in [JPA and ORM](jpa-orm.md).

## Why Not Return Entities from Controllers

| Problem | Effect |
| :-- | :-- |
| Lazy associations | `LazyInitializationException` or accidental N+1 during JSON serialization |
| Bidirectional links | Infinite recursion in Jackson |
| Over-exposure | Internal fields (password hash, audit columns) leak |
| Coupling | A column rename becomes a breaking API change (see [API Evolution](api-evolution.md)) |
| Mass assignment | Binding request JSON straight to an entity lets clients set `id`, `role`, `version` |
| Different shapes | List view, detail view and write request need different fields |

```java
public record CustomerResponse(Long id, String name, String email) {}

public record CreateCustomerRequest(
        @NotBlank String name,
        @Email String email) {}      // no id, no role: cannot be mass-assigned
```

Use separate request and response DTOs; they usually differ (validation annotations on input, computed fields on output).

## Projections (Read Only Shapes)

Fetch only the columns you need instead of full entities.

| Kind | Example | Notes |
| :-- | :-- | :-- |
| Interface (closed) | `interface NameOnly { String getName(); }` | Selects only the declared properties; proxy based |
| Interface (open) | `@Value("#{target.first + ' ' + target.last}")` | Loads the whole entity; no query optimization |
| Class / record DTO | `record CustomerRow(Long id, String name) {}` | Constructor-based, derived query or `select new`; fastest and clearest |
| Dynamic | `<T> List<T> findByActive(boolean a, Class<T> type)` | Caller picks the projection |
| Native / JdbcTemplate | `RowMapper`, `DataClassRowMapper` | Reports, complex SQL |

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {

    List<CustomerRow> findByActiveTrue();                       // record projection

    <T> Optional<T> findById(Long id, Class<T> type);           // dynamic projection

    @Query("""
           select new com.example.CustomerWithOrders(c.id, c.name, count(o))
           from Customer c left join c.orders o
           group by c.id, c.name
           """)
    List<CustomerWithOrders> summaries();                        // aggregate, no entity loaded
}
```

Projection results are **not managed**: no dirty checking, no lazy loading, less memory. Do not mutate and expect a save. Projections are the right answer to "avoid fetching the whole entity graph".

## Mapping Entity to DTO

| Option | Pros | Cons |
| :-- | :-- | :-- |
| Manual (static `from(entity)` / constructor) | Explicit, no dependency | Boilerplate, easy to forget a field |
| **MapStruct** | Compile-time generated code, type safe, fast, no reflection | Annotation processor setup |
| ModelMapper / Dozer | Little config | Reflection at runtime, errors found late, slower |

### MapStruct

MapStruct is an annotation processor that **generates** a plain Java mapper implementation at compile time.

```xml
<properties><mapstruct.version>1.6.3</mapstruct.version></properties>
<dependency><groupId>org.mapstruct</groupId><artifactId>mapstruct</artifactId><version>${mapstruct.version}</version></dependency>
<!-- maven-compiler-plugin annotationProcessorPaths: mapstruct-processor
     (and lombok + lombok-mapstruct-binding if Lombok is used, Lombok first) -->
```

```java
@Mapper(componentModel = "spring",
        unmappedTargetPolicy = ReportingPolicy.ERROR)   // fail build on forgotten fields
public interface CustomerMapper {

    @Mapping(source = "address.city", target = "city")
    @Mapping(target = "displayName", expression = "java(c.getFirst() + \" \" + c.getLast())")
    CustomerResponse toResponse(Customer c);

    List<CustomerResponse> toResponses(List<Customer> list);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "version", ignore = true)
    Customer toEntity(CreateCustomerRequest req);

    @Mapping(target = "id", ignore = true)
    void update(UpdateCustomerRequest req, @MappingTarget Customer entity);   // PATCH-style update
}
```

```java
@Service
class CustomerService {
    private final CustomerRepository repo;
    private final CustomerMapper mapper;     // injected: componentModel = "spring"

    @Transactional(readOnly = true)
    public CustomerResponse get(Long id) {
        return repo.findById(id).map(mapper::toResponse)
                   .orElseThrow(() -> new NotFoundException(id));
    }
}
```

Tips and gotchas:

- Map **inside** the transaction when the mapper touches lazy associations, or fetch them with `JOIN FETCH` / entity graph first; otherwise `LazyInitializationException`.
- Set `unmappedTargetPolicy = ERROR` so adding a field to the DTO breaks the build instead of returning null.
- Use `@MappingTarget` for updates so JPA keeps the same managed entity; map null handling with `nullValuePropertyMappingStrategy = IGNORE` for partial updates.
- Mapping a bidirectional entity graph can recurse; map one side only or use `@Context` with a cycle handler.
- Records and Lombok builders are supported; with Lombok the processor order and binding artifact matter.
- Do not put business logic in mappers; keep them declarative.

## Layering Guidance

| Layer | Type used |
| :-- | :-- |
| Controller | Request and response DTOs (validated) |
| Service | Maps at the boundary; works on entities or domain objects inside |
| Repository | Entities for writes, projections for reads |

Over-engineering warning: for a tiny CRUD service, a response record plus a manual mapping method is often enough. Add MapStruct when mappers multiply.

## Common Interview Questions

- **Why not expose entities in the API?** Lazy loading and serialization issues, leaks, tight coupling to schema, mass assignment.
- **Interface vs class projection?** Closed interface projections select only needed columns; DTO records are explicit and not proxies; open projections load the entity.
- **MapStruct vs ModelMapper?** MapStruct generates code at compile time (fast, safe, debuggable); ModelMapper uses reflection at runtime.
- **How do you do a partial update?** `@MappingTarget` with a request DTO, ignoring nulls, inside a transaction.
- **How do you avoid N+1 while mapping?** Fetch join, entity graph or a DTO projection query, not lazy access in the mapper.

Reference: [Spring Data projections](https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html), [MapStruct reference](https://mapstruct.org/documentation/stable/reference/html/).
