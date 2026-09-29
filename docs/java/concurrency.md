# Concurrency & Threads

## Basics
- **Process vs thread** — threads share heap within a process; each has its own stack.
- **Concurrency vs parallelism** — interleaving vs simultaneous execution (multi-core).
- **Thread lifecycle** — NEW → RUNNABLE → RUNNING → BLOCKED/WAITING/TIMED_WAITING → TERMINATED.

## Creating threads
- Implement **`Runnable`** (preferred) or **`Callable<V>`** (returns a value / throws); pass to an `ExecutorService`.
- Extend `Thread` (rarely). Start via `start()` (never `run()` directly).

## Synchronization
- **Race condition** — outcome depends on timing over shared mutable state.
- `synchronized` methods/blocks — mutual exclusion via an intrinsic lock (monitor).
- **`volatile`** — visibility only (not atomicity) — good for flags.
- **Atomic** classes (`AtomicInteger`) — lock-free CAS for counters.
- **`ReentrantLock`** — explicit lock with `tryLock`, fairness, interruptibility.
- **Java Memory Model** — happens-before guarantees.

## Runnable vs Callable
| | Runnable | Callable |
|--|----------|----------|
| Returns | void | `V` |
| Throws checked | no | yes |
| Submit | `execute` / `submit` | `submit` → `Future<V>` |

## Executors & Futures
- **Thread pool** (`ExecutorService`, `ThreadPoolTaskExecutor`) — reuse threads, bound concurrency; tune core/max/queue.
- **`Future.get()`** blocks; **`CompletableFuture`** composes async pipelines
  (`thenApply/thenCompose/thenCombine/allOf/exceptionally`). Async exceptions wrap in `CompletionException` — unwrap.

## Problems & fixes
- **Deadlock** (4 conditions) → lock ordering, timeouts, `tryLock`.
- **Livelock / starvation** → fairness.
- Thread-safety strategies: immutability, confinement (`ThreadLocal`), synchronization, concurrent collections.

## Java 21
**Virtual threads (Loom)** — cheap threads for blocking I/O; thread-per-request without pool exhaustion.

## IPC (inter-process communication)
Pipes, message queues, shared memory, sockets — across processes (vs threads sharing memory in one process).
