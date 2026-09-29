# Scalability & Resilience

## Failover Mechanisms

[https://www.geeksforgeeks.org/failover-mechanisms-in-system-design/](https://www.geeksforgeeks.org/failover-mechanisms-in-system-design/)

A failover mechanism is a system that automatically switches to a backup system when the primary system fails. It's a key part of disaster recovery (DR) and is used to ensure business continuity.

**How does it work?**

* When the primary system fails, the failover mechanism automatically switches to the backup system.
* The backup system mimics the operating system environment of the primary system.
* Requests are redirected from the failed system to the backup system.

**Where is it used?**

* In data centers
* In network services
* In cloud computing environments
* In backend database support
* In servers.

**Benefits**

* Minimizes downtime and data loss
* Ensures business continuity
* Safeguards customer experiences and brand value.

**Key Terminologies**

1. **Circuit Breaker**
   A mechanism that monitors service health. When a predefined number of consecutive failures occurs, it interrupts further calls to the failing service, preventing cascading failures.

2. **Command**
   Represents a potentially risky action or service call wrapped with circuit breaker functionality. It corresponds to the method call making the remote call to another service.

3. **Fallback**
   A backup mechanism providing default or alternative behavior when the command fails or when the circuit breaker is open. It prevents application crashes due to failures.

4. **Timeout**
   The maximum time a command takes to execute before considering it a failure. If the command exceeds this time, it is aborted, and the fallback mechanism is invoked.

5. **Rolling Window**
   The time window within which Hystrix monitors service health. It calculates metrics such as error percentage and request volume within this window to determine the circuit state.

6. **Isolation**
   Hystrix provides isolation of commands using separate threads, thread pools, or semaphores. It ensures that the failure in one command does not affect others, providing fault tolerance.

7. **Metrics**
   Data collected by Hystrix, such as success rate, failure rate, latency, and concurrency. These metrics are used to make decisions about the circuit state and for monitoring and debugging purposes.

8. **Thread Pool**
    Used by Hystrix to execute commands asynchronously. Each command can have its thread pool, providing isolation and preventing resource contention between commands.

**Types of failover**

1. **Cold Standby**
   The backup system is inactive (offline or powered off) until needed, requiring time to boot up and become operational, resulting in longer downtime.

2. **Cozy Standby Failure Mode**
   A warm standby system is partially operational, ready to take over with minimal downtime, but not handling live traffic until the primary system fails.

3. **Warm Standby Failure-Over**
   A fully functional, synchronized backup system is always running in the background, ready to take over immediately, minimizing downtime.

4. **Active-Passive Switching**
   Only one system is active at a time, with the other(s) in standby mode, ready to take over if the active system fails.

5. **Dynamic-Active Switchover**
   Both primary and standby systems are active concurrently, processing traffic and fulfilling requests, ensuring high availability and no downtime.

**Types of Failover Mechanisms**

1. **Active-Passive**
   A primary system is active, with a secondary system in standby, ready to take over if the primary fails.

2. **Active-Active**
   Multiple systems are active simultaneously, distributing the workload, so if one fails, the others can continue serving requests.

3. **Manual Failover**
   Requires human intervention to switch to the backup system.

4. **Automatic Failover**
   The system automatically detects a failure and switches to the backup without human intervention.

5. **Failover Cluster**
   A group of servers that work together to provide high availability, where if one server fails, the others can take over its workload.

6. **Replication**
   Creating copies of data or applications on a secondary system, allowing for a swift switchover in case of failure.

7. **Redundancy**
   Having multiple systems or components that can perform the same function, ensuring that if one fails, others can take over.

8. **Failback**
   The process of switching back to the original system once it has been repaired or restored after a failure.

**Best practices**

* Determine critical systems
* Define recovery objectives
* Add redundancy to hardware, networking, storage, and power supply systems
* Employ load balancers
* Automate failover procedures
* Maintain system health

## Resilience patterns (quick reference)
- **Timeout · Retry** (idempotent + backoff+jitter) **· Circuit breaker · Bulkhead · Fallback/graceful degradation.**
- **Resilience4j** (successor to Hystrix) provides these as composable decorators/annotations in Spring.
