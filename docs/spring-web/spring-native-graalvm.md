# Spring Native Images (GraalVM)

A normal Java app compiles to bytecode; the JVM interprets it at runtime and JIT-compiles hot paths. **GraalVM Native Image** uses **ahead-of-time (AOT)** compilation to turn the app into a standalone, platform-specific executable that needs no JVM. Spring Boot 3 supports it natively.

---

## How Native Image Works (Closed-World Assumption)

1. **Points-to analysis**: starting from `main`, the compiler traces every reachable class and method.
2. **Dead-code elimination**: anything unreachable is dropped.
3. **Heap snapshotting**: static initializers can run at build time and the resulting heap is written into the binary.
4. **Substrate VM**: a small embedded runtime handles memory management (GC) and threads.

## JIT JVM vs Native Executable

| Metric | JVM (JIT) | Native image |
|--------|-----------|--------------|
| Startup | Seconds for a Spring app (JVM boot, class loading, warm-up) | Typically tens of milliseconds |
| Memory | Higher (JIT and class metadata) | Much lower footprint |
| Peak throughput | Often better after warm-up (profile-guided JIT) | Good, usually not above a warmed JIT |
| Build time | Seconds | Minutes, CPU/RAM heavy |
| Container | Needs a JRE base image | Small distroless or scratch image |
| Portability | Same jar on any OS | One binary per OS/architecture |

Best for serverless (cold starts), scale-to-zero and dense Kubernetes workloads. A long-running, steady-state service often does fine on the JVM.

## Framework Comparison

| Framework | Startup | Memory | Ecosystem | Best for |
|-----------|---------|--------|-----------|----------|
| Spring Boot (JVM) | Moderate | Higher | Largest | Enterprise apps |
| Spring Boot (native) | Fast | Low | Largest, with native caveats | Serverless, scale-to-zero |
| Quarkus | Fast | Low | Very good | Kubernetes, serverless |
| Micronaut | Fast | Low | Very good | Lean microservices, functions |
| Javalin | Fast | Very low | Minimal | Small utilities, prototypes |

Relative, indicative values; measure your own workload. Quarkus and Micronaut do work at build time by design; Spring Boot 3 adds an AOT processing step to get a similar effect.

## Spring Boot 3 Setup

Spring AOT generates reflection/proxy/resource hints at build time, then `native-image` compiles the result.

```bash
# Maven: buildpacks produce a container with a native executable (no local GraalVM needed)
./mvnw -Pnative spring-boot:build-image

# Maven: local native executable (needs a GraalVM JDK and a C toolchain)
./mvnw -Pnative native:compile

# Gradle
./gradlew nativeCompile
./gradlew bootBuildImage
```

Minimal non-Spring example:

```bash
javac HelloWorld.java
native-image HelloWorld     # produces ./helloworld (helloworld.exe on Windows)
./helloworld                # starts in a few milliseconds, no 'java' command
```

## Gotchas

| Issue | Why | What to do |
|-------|-----|-----------|
| Reflection, dynamic proxies, resources, JNI | Not visible to analysis at build time (`Class.forName` fails at runtime) | Spring AOT handles its own; for your code use `@RegisterReflectionForBinding`, `RuntimeHintsRegistrar`, or reachability metadata JSON |
| Runtime conditions | `@ConditionalOnProperty` and `@Profile` are evaluated at build time in AOT mode | Fix the profile and properties that affect bean definitions when building |
| Slow builds | Whole-program analysis | Build native in CI/release, develop on the JVM |
| Dynamic class loading, agents, some libraries | Closed world | Check library native support; run `nativeTest` |
| Debugging/profiling | No JIT-style tooling | Use native-aware observability |

!!! tip "Alternatives to native"
    **Virtual threads** (`spring.threads.virtual.enabled=true`, Boot 3.2+, Java 21+) improve I/O concurrency, not startup. **CRaC** (checkpoint/restore) and **AppCDS** speed up JVM startup without closed-world limits.

## Interview Questions

- **Why is native startup fast?** Class loading, initialization and much of Spring's bean/proxy processing happen at build time; no JIT warm-up.
- **What is the closed-world assumption?** Everything that can run must be known at compile time; unreachable code is removed.
- **Main drawbacks?** Long builds, reflection/dynamic-feature configuration, lower peak throughput than a warmed JIT, per-platform binaries.
- **How does Spring Boot 3 help?** AOT processing generates hints and bean definitions so most apps need little manual config.

---

**See also:** [Spring Boot](spring-boot.md), [Spring Boot Advanced](spring-boot-advanced.md), [Docker](../devops/docker.md)
