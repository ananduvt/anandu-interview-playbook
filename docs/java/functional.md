# Functional Java (Lambdas & Streams)

## Lambdas & functional interfaces
- **Lambda** — anonymous function: `(a, b) -> a + b`. Targets a **functional interface** (one abstract method, `@FunctionalInterface`).
- Built-ins: `Function<T,R>`, `Supplier<T>`, `Consumer<T>`, `Predicate<T>`, `BiFunction<T,U,R>`, `UnaryOperator<T>`.
- **Method references** (`::`): `String::toUpperCase`, `System.out::println`, `Integer::parseInt`, `ArrayList::new`.

## Streams
A pipeline: **source → intermediate (lazy) → terminal (eager)**. Streams are single-use, don't mutate the source.
```java
List<String> names = people.stream()
    .filter(p -> p.getAge() > 18)
    .map(Person::getName)
    .sorted()
    .collect(Collectors.toList());
```

### Creation
`collection.stream()`, `Stream.of(...)`, `Arrays.stream(arr)`, `IntStream.range(a,b)`, `Stream.iterate/generate`.

### Intermediate (lazy)
`filter`, `map`, `flatMap`, `distinct`, `sorted`, `limit`, `skip`, `peek`.

### Terminal (eager)
`forEach`, `collect`, `reduce`, `count`, `anyMatch/allMatch/noneMatch`, `findFirst/findAny`, `min/max`, `toList()`.

### Collectors
`toList/toSet/toMap`, `groupingBy`, `partitioningBy`, `joining`, `counting`, `summingInt`, `averagingDouble`.

## Comparable vs Comparator
- `Comparable.compareTo` — natural order (in the class).
- `Comparator` — external/multi-key: `comparing(Person::getAge).thenComparing(Person::getName).reversed()`.

## Optional
Avoid nulls: `Optional.ofNullable(x).map(...).orElseGet(...)`. Don't call `.get()` blindly; don't use as fields/params.

## Parallel streams
`.parallelStream()` uses the common ForkJoinPool — only for CPU-bound, large, stateless, associative ops.
