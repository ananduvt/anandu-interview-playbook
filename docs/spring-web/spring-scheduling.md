# Spring Scheduling: `@Scheduled`, ShedLock and Quartz

Run code periodically (cleanup, reports, polling, retries). `@Scheduled` is trivial on one instance but **runs on every instance** once you scale out, so clusters need a lock or a clustered scheduler.

## `@Scheduled` Basics

```java
@Configuration
@EnableScheduling                       // required (Boot does not enable it automatically)
class SchedulingConfig { }

@Component
class Jobs {
    @Scheduled(fixedRate = 5000)                      // every 5s from start to start
    void heartbeat() { }

    @Scheduled(fixedDelay = 5000, initialDelay = 10000)   // 5s after the previous run ENDS
    void poll() { }

    @Scheduled(cron = "0 0 2 * * *", zone = "UTC")    // 02:00 daily; seconds minutes hours dom month dow
    void nightlyCleanup() { }

    @Scheduled(cron = "${jobs.report.cron}")          // externalised
    void report() { }
}
```

| Attribute | Meaning |
| :-- | :-- |
| `fixedRate` | Fixed interval between **starts**; a late run starts late, it does not run concurrently with itself |
| `fixedDelay` | Fixed gap between **end** of one run and start of next |
| `initialDelay` | Wait before the first run |
| `cron` | Six fields: second, minute, hour, day-of-month, month, day-of-week (Spring has the seconds field, unlike Unix cron) |
| `zone` | Time zone for cron; default is the server zone |

!!! warning "Single thread by default"
    The default scheduler has **one thread**, so a slow job delays every other job. Set `spring.task.scheduling.pool.size=5` (Boot) or define a `TaskScheduler` bean. Exceptions are logged and the next run still happens (for repeating tasks), and methods must be `void` with no arguments. Use `@Scheduled` methods in a Spring bean, and keep `@Transactional` boundaries inside a separate service because of proxy self-invocation ([AOP](spring-aop.md)).

## The Cluster Problem

With 3 pods, `@Scheduled` fires 3 times: duplicate emails, double billing. Options:

| Option | How | Fits |
| :-- | :-- | :-- |
| **ShedLock** | Lock row/key in a shared store (JDBC, Redis, Mongo); only the instance that gets the lock runs | Existing `@Scheduled` jobs, simple "run once per tick" |
| **Quartz (JDBC job store)** | Full scheduler with persisted jobs/triggers, cluster mode coordinates through DB | Dynamic schedules, misfire handling, job history, many jobs |
| External trigger | Kubernetes `CronJob`, cloud scheduler hitting an endpoint | Infrequent batch work, isolated resources |
| Leader election | One elected leader runs jobs (e.g. Spring Integration, ZooKeeper, K8s lease) | When all jobs should live on one node |
| Idempotent job | Make the work safe to run twice (unique constraints, `SELECT ... FOR UPDATE SKIP LOCKED`) | Always desirable as a second line of defence |

### ShedLock

```java
@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "10m")
class LockConfig {
    @Bean
    LockProvider lockProvider(DataSource ds) {
        return new JdbcTemplateLockProvider(ds);      // needs a "shedlock" table
    }
}

@Component
class ReportJob {
    @Scheduled(cron = "0 0 * * * *")
    @SchedulerLock(name = "hourlyReport", lockAtMostFor = "50m", lockAtLeastFor = "5m")
    void run() { /* runs on one node per tick */ }
}
```

- `lockAtMostFor`: safety net if the node dies holding the lock; set it **longer than the longest normal run**.
- `lockAtLeastFor`: minimum hold time, which stops a fast job from running again on another node whose clock is slightly behind.
- ShedLock is a **lock, not a scheduler**: it does not retry missed runs, and it relies on reasonably synced clocks.

### Quartz

```yaml
spring:
  quartz:
    job-store-type: jdbc
    properties:
      org.quartz.jobStore.isClustered: true
      org.quartz.scheduler.instanceId: AUTO
```

```java
@Component
class CleanupJob extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext ctx) { /* work */ }
}
// JobDetail + Trigger beans (or a Scheduler) register it; the cluster fires each trigger on exactly one node.
```

Quartz adds misfire policies (what to do when a trigger was missed), persistent state, pause/resume, and runtime-created schedules, at the cost of more tables and configuration.

## Practical Guidance

- Prefer `fixedDelay` for polling so runs never pile up.
- Make jobs idempotent and log start/end plus duration; expose metrics ([Actuator](spring-boot-actuator.md)).
- Shut down gracefully: `spring.task.scheduling.shutdown.await-termination=true` lets running jobs finish.
- Long-running or heavy jobs: hand off to a worker/queue instead of occupying the scheduler thread; see [Threading and Async](spring-threading.md).
- Test the job's logic as a plain method; do not wait for the cron in tests.

## Common Interview Questions

- **`fixedRate` vs `fixedDelay`?** Start-to-start vs end-to-start spacing.
- **What happens to `@Scheduled` when you run 3 instances?** It runs on all three; use ShedLock, Quartz clustering, or an external scheduler.
- **ShedLock vs Quartz?** ShedLock only prevents concurrent execution of `@Scheduled` jobs; Quartz is a full persistent clustered scheduler.
- **Why did my second job not run on time?** Single-threaded default scheduler is blocked by the first job; increase the pool size.
- **What is `lockAtMostFor` for?** Releasing the lock if the node crashes mid-job.

**See also:** [Spring Boot](spring-boot.md), [Threading and Async](spring-threading.md), [Spring Annotations](spring-annotations-components.md)
