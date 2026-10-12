# Cloud & Microservices Principles

## Cloud Computing

## Service models

![](../assets/cloud-service-models.png)

* **IaaS, or infrastructure as a service**, is on-demand access to cloud-hosted physical and virtual servers, storage and networking—the backend IT infrastructure for running applications and workloads in the cloud.
* **PaaS, or platform as a service**, is on-demand access to a complete, ready-to-use, cloud-hosted platform for developing, running, maintaining and managing applications.
* **SaaS, or software as a service**, is on-demand access to ready-to-use, cloud-hosted application software.

**BPaaS**, business process as a service as the delivery of business process outsourcing (BPO) services that are sourced from the cloud and constructed for multitenancy. Services are often automated, and where human process actors are required, there is no overtly dedicated labor pool per client.

**BPO,** Business process outsourcing is a method of subcontracting various business-related operations to third-party vendors.

rds vs s3 ??

## Principles

Cap theorem
Consistency
Availability
Partition tolerance
Fault tolerance
Load balancing

Event driven systems

## MicroService

12 factor ??
Monolith vs microservice ??
Blue Green Deployment
Deployment strategies
Patterns
Micro service service discovery

The **12-Factor App** is a methodology for building **scalable, maintainable, and portable** cloud-native applications. It was created by developers at **Heroku** to define best practices for modern software-as-a-service (SaaS) applications.

### **The 12 Factors**

#### **1. Codebase – *One codebase, multiple deploys***

* A single **source code repository** should be used for all deployments (e.g., dev, staging, production).
* Different environments should not have separate codebases.

#### **2. Dependencies – *Explicitly declare and isolate dependencies***

* Dependencies should be **declared explicitly** in `pom.xml` (Maven) or `build.gradle` (Gradle) in Java projects.
* Use dependency managers (e.g., Maven, NPM, Pip) and avoid system-wide dependencies.

#### **3. Config – *Store config in the environment***

* **Configuration should be externalized** and stored in **environment variables**, not in the codebase.
* Avoid committing credentials (e.g., AWS keys, database passwords) in repositories.

#### **4. Backing Services – *Treat backing services as attached resources***

* External services (databases, message queues, caches) should be treated as **replaceable** and not hardcoded into the app.
* Example: Using AWS RDS instead of an in-memory database.

#### **5. Build, Release, Run – *Strictly separate build, release, and run stages***

* Build: Compile code and create artifacts.
* Release: Combine the build with the environment configuration.
* Run: Start the application instance.

#### **6. Processes – *Execute the app as one or more stateless processes***

* Apps should be **stateless** – session data should be stored in external services like Redis or databases.
* Avoid writing to local disk; instead, use object storage (e.g., AWS S3).

#### **7. Port Binding – *Export services via port binding***

* Apps should be self-contained and expose services via **ports** (e.g., Spring Boot runs on **port 8080**).
* Example: Instead of depending on Apache/Nginx, a Java microservice should expose APIs directly via Tomcat or Jetty.

#### **8. Concurrency – *Scale out via process model***

* Applications should **scale horizontally** by running multiple instances instead of relying on threads within a single instance.
* Example: Instead of increasing thread count, deploy multiple containers using Kubernetes.

#### **9. Disposability – *Fast startup and graceful shutdown***

* Apps should **start quickly** and **shutdown gracefully** to handle crashes and updates smoothly.
* Example: Use **graceful shutdown hooks** in Spring Boot to close DB connections properly.

#### **10. Dev/Prod Parity – *Keep development, staging, and production as similar as possible***

* Minimize differences between environments (e.g., run **Docker containers locally** that mimic production).
* Example: Use **Docker Compose** for local development if Kubernetes is used in production.

#### **11. Logs – *Treat logs as event streams***

* Logs should be **written to stdout/stderr**, not local files.
* Use **log aggregators** (e.g., ELK stack, CloudWatch) to manage logs.

#### **12. Admin Processes – *Run admin tasks as one-off processes***

* Tasks like **database migrations, cron jobs, and batch processing** should be separate from the main application.
* Example: Run **Flyway or Liquibase** migrations separately from the main app startup.

---

### **Why It’s Important?**

* Makes apps **scalable, portable, and cloud-ready**.
* Supports **DevOps, CI/CD, and microservices architectures**.
* Ensures **consistency between development and production environments**.

[https://www.geeksforgeeks.org/microservices-interview-questions/](https://www.geeksforgeeks.org/microservices-interview-questions/)

[https://www.baeldung.com/cs/service-discovery-microservices](https://www.baeldung.com/cs/service-discovery-microservices)

[https://www.geeksforgeeks.org/challenges-and-solutions-of-microservices-architecture/](https://www.geeksforgeeks.org/challenges-and-solutions-of-microservices-architecture/)
[https://rathod-ajay.medium.com/top-20-java-spring-boot-microservice-developer-interview-questions-3-7-experience-afb67b2e0725](https://rathod-ajay.medium.com/top-20-java-spring-boot-microservice-developer-interview-questions-3-7-experience-afb67b2e0725)
