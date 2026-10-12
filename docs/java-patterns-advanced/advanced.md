# Advanced Java

## Modern language features
- **Records** (16) — immutable data carriers: `record Point(int x, int y) {}` (auto ctor/getters/equals/hashCode/toString).
- **Sealed classes** (17) — restrict which types can extend/implement (`sealed ... permits ...`).
- **Pattern matching** — `instanceof` patterns, `switch` patterns, **record patterns** (destructuring).
- **Text blocks** (15) — multi-line string literals `"""..."""`.
- **`var`** (10) — local type inference (still statically typed).

## Generics
- Type safety at compile time; **type erasure** at runtime (no reified generics).
- Bounded wildcards — **PECS**: `<? extends T>` (producer/read), `<? super T>` (consumer/write).

## Annotations
- Metadata; **marker** (no members, e.g. `@Override`), single-value, full.
- Retention: SOURCE / CLASS / **RUNTIME** (readable via reflection).
- Custom: `@Retention @Target @interface Audit {}`; processed by frameworks (Spring) or annotation processors (Lombok).

## Reflection
Inspect/invoke types at runtime (`Class`, `Method`, `Field`). Powerful but slow + breaks encapsulation — used by frameworks.

## Garbage Collection (GC)

### GC Types and Algorithms

| GC | Throughput | Latency | Heap Size | Use Case |
|----|-----------|---------|-----------|----|
| **Serial GC** | Low (single-threaded) | High pauses | Small (~100MB) | Small apps, desktops |
| **Parallel GC** | High (multi-threaded) | Variable pauses | Medium/large | Batch jobs, non-latency-sensitive |
| **CMS (Concurrent Mark Sweep)** | Medium | Low-medium pauses | Large | Latency-sensitive (deprecated Java 14+) |
| **G1GC** | High | Low-medium pauses | Large (>4GB) | Production servers, modern default |
| **ZGC (Java 11+)** | High | Ultra-low (<10ms pauses) | Very large (TB) | Ultra-low-latency apps |
| **Shenandoah (Java 12+)** | High | Ultra-low pauses | Very large | Low-pause alternative to ZGC |

### G1 Garbage Collector (Default in Java 9+)

**Key characteristics:**
- **Generational** — Separate young and old generations
- **Region-based** — Heap divided into ~2000 regions (1-32MB each)
- **Low-pause** — Target pause time (default 200ms) regardless of heap size
- **Concurrent** — Marks live objects while app runs (reduces stop-the-world time)
- **Incremental** — Collects a few regions per GC cycle, not entire heap

**Phases:**
1. **Young GC** — Collects young regions (via parallel threads)
2. **Concurrent Mark** — Marks live objects from roots (runs during app execution)
3. **Remark** — Finalizes marking (brief pause)
4. **Cleanup** — Compacts regions, frees empty ones
5. **Mixed GC** — Collects young + some old regions (partial old-gen collection)

**Configuration:**
```bash
# Set initial and max heap
java -Xms4G -Xmx4G \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:InitiatingHeapOccupancyPercent=35 \
  -XX:G1HeapRegionSize=16M \
  -XX:+PrintGCDetails \
  -XX:+PrintGCDateStamps \
  MyApp
```

**Tuning tips:**
- **Max pause time**: Lowering `MaxGCPauseMillis` increases frequency of GC cycles
- **Occupancy threshold**: `InitiatingHeapOccupancyPercent` triggers concurrent marking earlier
- **Region size**: Larger regions for big heaps; smaller for low-latency needs
- Monitor via JVM metrics: `jstat -gc` or `jconsole`

### Common GC Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| Full GC thrashing | Frequent "Full GC", high CPU | Increase heap size; tune young gen ratio |
| Long pause times | Application freezes | Switch to low-pause GC (G1, ZGC); tune pause target |
| High GC overhead | >30% CPU on GC | Heap too small; increase `-Xmx` |
| Memory leak | Heap grows indefinitely | Find object chains (heap dump); fix references |

**Heap dump analysis:**
```bash
# Generate heap dump
jmap -dump:live,format=b,file=heap.bin <pid>

# Analyze with Eclipse MAT or JProfiler
# Look for: unreachable objects, retained sets, memory footprint by class
```

### GC Monitoring

```bash
# Real-time GC stats
jstat -gc -h10 <pid> 1s

# Generate GC log (Java 9+)
-Xlog:gc*:file=gc.log:time,level,tags

# Parse logs with GCeasy.io (web tool) or local parser
```

## Other
- **`Optional`** — null-safety (don't overuse).
- **equals/hashCode** contract; **immutability** as a design + concurrency tool.
- **Serialization** — `Serializable`, `serialVersionUID`; prefer JSON for interop.
- **Atomic reference**, `ThreadLocal`, **Cleaner** (replaces finalize).
