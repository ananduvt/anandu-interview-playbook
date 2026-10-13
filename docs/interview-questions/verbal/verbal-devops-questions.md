# DevOps & Cloud Interview Questions

Infrastructure, deployment, containerization, CI/CD, cloud platforms, and operational concerns.

---

## Question Categories

### Infrastructure & Cloud
- Cloud platforms (AWS, GCP, Azure)
- Compute, storage, networking services
- Infrastructure as Code (IaC)
- Cost optimization

### Containerization
- Docker basics, images, containers
- Container registries, image layers
- Docker networking, volumes

### Orchestration
- Kubernetes fundamentals
- Pods, deployments, services
- StatefulSets, DaemonSets
- Scaling (HPA, VPA)
- GitOps with Argo CD

### CI/CD
- Continuous integration practices
- Continuous deployment strategies
- Build pipelines, testing, automation
- Git workflows

### Deployment Strategies
- Rolling updates, blue-green, canary
- Shadow deployments
- Rollback mechanisms

### Monitoring & Observability
- Logging (ELK, Splunk)
- Metrics (Prometheus, Grafana)
- Tracing (Jaeger, Zipkin)
- Alerting

### Security in DevOps
- Secrets management
- Network policies, firewalls
- Container security, image scanning
- Compliance, audit logs

### Databases & Data
- Database backups, disaster recovery
- Replication, failover
- Scaling databases (read replicas, sharding)

---

## Sample Topics (Questions to be added)

- Explain Docker and why containers are useful
- How do Docker images differ from containers?
- What are Docker volumes and when to use them?
- Explain Kubernetes architecture (control plane, nodes)
- What's the difference between a Pod and a Deployment?
- How does Kubernetes handle service discovery?
- Explain rolling updates and why they matter
- What's blue-green deployment and when to use it?
- Explain canary deployments
- How do you scale applications in Kubernetes?
- What's GitOps and how does Argo CD implement it?
- How would you set up a CI/CD pipeline?
- What's the difference between CI and CD?
- Explain infrastructure as code (Terraform, Ansible)
- How do you monitor Kubernetes applications?
- How would you handle secrets in production?
- Explain AWS ECS vs EKS
- How do you plan for disaster recovery?
- What's a health check and why is it important?

---

## Interview Questions (With Answers)

<details>
<summary>What is Kubernetes deployment?</summary>

**Answer:** Kubernetes Deployment manages ReplicaSets to ensure desired pod replicas run. Provides rolling updates, rollback, and scaling. Declaratively define desired state (image, replicas, resources); Deployment controller reconciles actual state. Used for stateless applications with zero-downtime deployments.

📚 [Learn more: Kubernetes](../../devops/kubernetes.md)
</details>

<details>
<summary>Why is Maven pom.xml used? Explain the key tags.</summary>

**Answer:** pom.xml defines project metadata, dependencies, and build configuration. Key tags: <groupId> (organization), <artifactId> (project name), <version>, <dependencies> (external libs), <plugins> (build tasks), <build> (output config). Maven manages transitive dependencies and standardizes builds.

📚 [Learn more: Build Tools](../../devops/build-tools.md)
</details>

---

[Content to be added progressively]

**See also:** [Kubernetes](../../devops/kubernetes.md), [Deployment Strategies](../../devops/kubernetes-deployment-strategies.md), [CI/CD](../../devops/ci-cd.md), [Docker](../../devops/docker.md), [Interview Questions Overview](../index.md)
