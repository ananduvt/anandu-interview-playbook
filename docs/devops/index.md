# DevOps

Operational practices and tooling for deploying, monitoring, and maintaining systems: version control, build automation, containerization, orchestration, and observability.

---

## Topics

### [Git](git.md)
**Version Control - Branching - Merging - Rebasing - Conflicts - Remote - Push/Pull**  
Git workflows: branching strategies (feature branches, trunk-based development), merge vs. rebase trade-offs, conflict resolution, remote collaboration, commit hygiene.

### [Build Tools](build-tools.md)
**Maven - Gradle - Ant - Build Lifecycle - Dependency Management - Plugins - Profiles**  
Build automation: Maven (POM, lifecycle, repositories), Gradle (build scripts, tasks), dependency resolution, multi-module projects, release automation.

### [CI/CD](ci-cd.md)
**Jenkins - GitLab CI - GitHub Actions - Pipelines - Automated Testing - Deployment Automation**  
Continuous integration/deployment: automated builds, testing gates, artifact repositories, deployment pipelines, rollback strategies, environment promotion.

### [Docker & Containers](docker.md)
**Docker - Images - Containers - Dockerfile - Registry - Networking - Volumes - Best Practices**  
Containerization: Docker fundamentals, image layers, container lifecycle, multi-stage builds, registry management (Docker Hub, ECR), networking, persistent volumes.

### [Kubernetes](kubernetes.md)
**K8s - Pods - Deployments - Services - ConfigMaps - Secrets - Ingress - Helm - Operators**  
Kubernetes orchestration: pod management, deployments with rolling updates, service discovery and load balancing, configuration management, storage, networking, package management with Helm.

### [Observability](observability.md)
**Logging - Monitoring - Metrics - Tracing - Alerting - ELK Stack - Prometheus - Grafana - Jaeger**  
System visibility: centralized logging (ELK, Splunk), metrics collection (Prometheus), distributed tracing (Jaeger), alerting rules, dashboards, SLOs/SLIs.

---

## Quick Interview Prep

**Git essentials:**
- **Branching**: feature branches off main; code review via PR; merge or rebase to main
- **Conflict resolution**: merge conflicts require manual resolution; rebase replays commits (linear history)
- **Remote workflow**: fetch (download), merge (integrate), push (upload)

**CI/CD pipeline stages:**
1. **Source**: Code commit triggers pipeline
2. **Build**: Compile, run unit tests, build artifacts
3. **Test**: Integration tests, security scans, code coverage
4. **Stage**: Deploy to staging/UAT environment
5. **Prod**: Deploy to production; canary or blue-green strategy

**Docker best practices:**
- **Multi-stage builds**: Reduce final image size; separate build and runtime
- **Layer caching**: Order Dockerfile commands by change frequency (stable → dynamic)
- **.dockerignore**: Exclude unnecessary files (like .git, node_modules)
- **Minimal base images**: alpine, distroless (security, size)

**Kubernetes architecture:**
- **Pods**: Smallest deployable unit; usually 1 container per pod; ephemeral
- **Deployments**: Declarative pod management; rolling updates; replica scaling
- **Services**: Stable endpoint for pods; load balancing (ClusterIP, NodePort, LoadBalancer)
- **ConfigMaps/Secrets**: External configuration and sensitive data
- **Ingress**: HTTP(S) routing into cluster; hostname/path-based routing

**Observability best practices:**
- **Logs**: Structured JSON logging (searchable); correlation IDs across services
- **Metrics**: Counter, gauge, histogram; dashboard visualization; alert on thresholds
- **Traces**: Distributed tracing for request flow across services; identify bottlenecks
- **SLO/SLI**: Service Level Objective (target availability); Service Level Indicator (measurement)
