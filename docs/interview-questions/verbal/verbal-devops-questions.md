# DevOps & Cloud Interview Questions

Infrastructure, deployment, containerization, CI/CD, cloud platforms, and operational concerns.

---

## 1. Explain Docker and why containers are useful

<details>
<summary>View Answer</summary>

<h3>What is Docker?</h3>

<p><strong>Docker:</strong> Containerization platform that packages application + dependencies into isolated, portable units.</p>

<h3>Why Containers?</h3>

<table>
<tr>
<th>Property</th>
<th>Benefit</th>
</tr>
<tr>
<td><strong>Portability</strong></td>
<td>Runs same everywhere (dev, test, prod)</td>
</tr>
<tr>
<td><strong>Isolation</strong></td>
<td>One container doesn't affect others</td>
</tr>
<tr>
<td><strong>Lightweight</strong></td>
<td>MBs not GBs (vs VMs)</td>
</tr>
<tr>
<td><strong>Fast startup</strong></td>
<td>Seconds not minutes</td>
</tr>
<tr>
<td><strong>Consistency</strong></td>
<td>Identical environment across stages</td>
</tr>
<tr>
<td><strong>Scalability</strong></td>
<td>Easy to scale (orchestrators like Kubernetes)</td>
</tr>
</table>

<h3>Containers vs VMs</h3>

<table>
<tr>
<th>Aspect</th>
<th>Container</th>
<th>VM</th>
</tr>
<tr>
<td><strong>Size</strong></td>
<td>100-500MB</td>
<td>1-5GB</td>
</tr>
<tr>
<td><strong>Startup</strong></td>
<td>Milliseconds</td>
<td>Minutes</td>
</tr>
<tr>
<td><strong>Density</strong></td>
<td>Hundreds per host</td>
<td>Tens per host</td>
</tr>
<tr>
<td><strong>Isolation</strong></td>
<td>Process-level (namespaces)</td>
<td>Full OS (hypervisor)</td>
</tr>
</table>

<h3>Container Workflow</h3>

```
1. Write Dockerfile
2. docker build → Image
3. docker run → Container (running instance)
4. Push to registry
5. Deploy to Kubernetes
```

<p>📚 <a href="/anandu-interview-playbook/devops/docker/">Learn more: Docker & Containers</a></p>

</details>

---

## 2. What's the difference between Docker images and containers?

<details>
<summary>View Answer</summary>

<h3>Docker Image: Template</h3>

<p><strong>Immutable blueprint containing application + dependencies + configuration.</strong></p>

```
Base OS (Ubuntu 20.04)
  ↓
Java runtime
  ↓
Dependencies (libraries)
  ↓
Application code
  ↓
Configuration
= Image (immutable)
```

<h3>Docker Container: Running Instance</h3>

<p><strong>Running process with isolated filesystem, network, environment.</strong></p>

```
Image + Writable layer + Runtime = Container
```

<h3>Analogy</h3>

<p><strong>Image = Class, Container = Object</strong></p>

<ul>
<li>Image: Blueprint (one per application)</li>
<li>Container: Instance (many can run from same image)</li>
</ul>

<h3>Key Differences</h3>

<table>
<tr>
<th>Property</th>
<th>Image</th>
<th>Container</th>
</tr>
<tr>
<td><strong>Mutability</strong></td>
<td>Immutable</td>
<td>Mutable (writable layer)</td>
</tr>
<tr>
<td><strong>Persistence</strong></td>
<td>Stored (registry)</td>
<td>Temporary (unless saved)</td>
</tr>
<tr>
<td><strong>Size</strong></td>
<td>MB</td>
<td>MB + running state</td>
</tr>
<tr>
<td><strong>Quantity</strong></td>
<td>One per app</td>
<td>Many from one image</td>
</tr>
</table>

<h3>Layers</h3>

<p><strong>Images built from layers</strong></p>

```
Layer 1: Base OS (ubuntu:20.04)
Layer 2: Java 11
Layer 3: Dependencies
Layer 4: Application code
Layer 5: Configuration

Each layer = file system diff
```

<p>📚 <a href="/anandu-interview-playbook/devops/docker/">Learn more: Docker Images & Containers</a></p>

</details>

---

## 3. Explain Kubernetes architecture

<details>
<summary>View Answer</summary>

<h3>Kubernetes Cluster Components</h3>

```
Control Plane (Master)
├─ API Server (REST endpoint)
├─ etcd (key-value store, state)
├─ Scheduler (assigns pods to nodes)
├─ Controller Manager (ensures desired state)
└─ Cloud Controller (cloud integration)

Worker Nodes (run containers)
├─ Node 1
│  ├─ kubelet (agent, runs containers)
│  ├─ kube-proxy (networking)
│  └─ Pods (smallest deployable unit)
├─ Node 2
└─ Node 3
```

<h3>Control Plane: Brain of Cluster</h3>

<table>
<tr>
<th>Component</th>
<th>Responsibility</th>
</tr>
<tr>
<td><strong>API Server</strong></td>
<td>REST API for all operations</td>
</tr>
<tr>
<td><strong>etcd</strong></td>
<td>Distributed database (cluster state)</td>
</tr>
<tr>
<td><strong>Scheduler</strong></td>
<td>Places pods on nodes</td>
</tr>
<tr>
<td><strong>Controller Manager</strong></td>
<td>Ensures desired state (reconciliation)</td>
</tr>
</table>

<h3>Worker Node: Executes Workload</h3>

<table>
<tr>
<th>Component</th>
<th>Responsibility</th>
</tr>
<tr>
<td><strong>kubelet</strong></td>
<td>Agent that runs containers</td>
</tr>
<tr>
<td><strong>kube-proxy</strong></td>
<td>Network routing, load balancing</td>
</tr>
<tr>
<td><strong>Container Runtime</strong></td>
<td>Docker, containerd, rkt</td>
</tr>
</table>

<h3>Communication Flow</h3>

```
kubectl apply -f deployment.yaml
  ↓
API Server (validates, stores in etcd)
  ↓
Controller Manager (detects desired ≠ actual)
  ↓
Scheduler (finds node for new pod)
  ↓
kubelet on Node (pulls image, runs container)
```

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 4. What's the difference between a Pod and a Deployment?

<details>
<summary>View Answer</summary>

<h3>Pod: Smallest Unit</h3>

<p><strong>Smallest deployable unit in Kubernetes. Wrapper around container(s).</strong></p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

<p><strong>Characteristics:</strong></p>
<ul>
<li>One or more containers (usually one)</li>
<li>Shared network (localhost communication)</li>
<li>Short-lived (deleted when done)</li>
<li>❌ No automatic restart or scaling</li>
</ul>

<h3>Deployment: Manager of Pods</h3>

<p><strong>Higher-level abstraction. Manages ReplicaSets which manage Pods.</strong></p>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

<p><strong>Characteristics:</strong></p>
<ul>
<li>✅ Automatic pod recreation if crashes</li>
<li>✅ Scaling (replicas)</li>
<li>✅ Rolling updates (zero downtime)</li>
<li>✅ Rollback</li>
</ul>

<h3>Hierarchy</h3>

```
Deployment
  ↓
ReplicaSet (manages pod copies)
  ↓
Pod (actual container)
```

<h3>When to Use</h3>

<p><strong>Pod:</strong> Temporary debugging, one-off jobs</p>
<p><strong>Deployment:</strong> Production applications (99% of cases)</p>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes Pods & Deployments</a></p>

</details>

---

## 5. Explain rolling updates, blue-green, and canary deployments

<details>
<summary>View Answer</summary>

<h3>Rolling Update: Gradual Replacement</h3>

<p><strong>Gradually replace old pods with new ones.</strong></p>

```
Step 1: 5 pods v1
Step 2: 4 pods v1 + 1 pod v2
Step 3: 3 pods v1 + 2 pods v2
Step 4: 2 pods v1 + 3 pods v2
Step 5: 1 pod v1 + 4 pods v2
Step 6: 5 pods v2 (done)
```

<p><strong>Advantages:</strong> Zero downtime, automatic rollback on error</p>
<p><strong>Disadvantages:</strong> Gradual (slower), temporary mixed versions</p>

<p><strong>Default in Kubernetes</strong></p>

<h3>Blue-Green Deployment: Switch All At Once</h3>

<p><strong>Run two complete versions, switch traffic instantly.</strong></p>

```
Load Balancer
  ↓ (all traffic to Blue)
Blue Environment (v1, production)
  ↓
Green Environment (v2, staging)
  ↓ (test thoroughly)
  ↓ (if OK: switch traffic to Green)
Green Environment (v2, now production)
```

<p><strong>Advantages:</strong> Instant switch, easy rollback</p>
<p><strong>Disadvantages:</strong> Resource-intensive (2x environments), no gradual testing</p>

<h3>Canary Deployment: Small Rollout First</h3>

<p><strong>Deploy to small % of users, monitor, then increase.</strong></p>

```
Step 1: Route 5% to v2 (canary)
  ↓ Monitor (errors, latency)
Step 2: If OK, route 25% to v2
  ↓ Monitor
Step 3: If OK, route 50% to v2
  ↓ Monitor
Step 4: Route 100% to v2
```

<p><strong>Advantages:</strong> Safe, gradual validation, early error detection</p>
<p><strong>Disadvantages:</strong> Slower deployment, complex monitoring</p>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Strategy</th>
<th>Speed</th>
<th>Risk</th>
<th>Rollback</th>
<th>Resources</th>
</tr>
<tr>
<td><strong>Rolling</strong></td>
<td>Slow</td>
<td>Medium</td>
<td>Automatic</td>
<td>1x</td>
</tr>
<tr>
<td><strong>Blue-Green</strong></td>
<td>Instant</td>
<td>High (binary)</td>
<td>Instant</td>
<td>2x</td>
</tr>
<tr>
<td><strong>Canary</strong></td>
<td>Medium</td>
<td>Low (gradual)</td>
<td>Manual</td>
<td>1.5-2x</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-deployment-strategies/">Learn more: Deployment Strategies</a></p>

</details>

---

## 6. What's GitOps and how does Argo CD implement it?

<details>
<summary>View Answer</summary>

<h3>GitOps: Git as Source of Truth</h3>

<p><strong>Git repository is the single source of truth for infrastructure and applications.</strong></p>

```
Git Repo (desired state)
  ↓ (Argo CD pulls)
  ↓ (compares with actual)
  ↓
Kubernetes Cluster (applies changes)
```

<h3>GitOps Workflow</h3>

```
1. Developer commits to Git
2. Argo CD detects change
3. Argo CD syncs to Kubernetes
4. Application deployed automatically
5. Git history = deployment history
```

<h3>Argo CD: GitOps Tool for Kubernetes</h3>

<p><strong>Continuous delivery tool that syncs Git to Kubernetes.</strong></p>

```yaml
# Git repo structure
/k8s
├─ base/
│  ├─ deployment.yaml
│  ├─ service.yaml
│  └─ kustomization.yaml
├─ overlays/
│  ├─ dev/
│  ├─ staging/
│  └─ prod/
```

<h3>Argo CD Components</h3>

<ul>
<li><strong>Application:</strong> Defines Git repo + target cluster</li>
<li><strong>Sync:</strong> Compares Git vs cluster, applies changes</li>
<li><strong>Health:</strong> Monitors resource health</li>
<li><strong>Webhook:</strong> Triggered by Git push</li>
</ul>

<h3>Benefits</h3>

<ul>
<li><strong>✅ Audit trail:</strong> Git history = all changes</li>
<li><strong>✅ Rollback:</strong> Simple git revert</li>
<li><strong>✅ Declarative:</strong> Yaml files define desired state</li>
<li><strong>✅ Automation:</strong> No manual kubectl commands</li>
<li><strong>✅ Consistency:</strong> Same deploy process everywhere</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes & GitOps</a></p>

</details>

---

## 7. How do you set up a CI/CD pipeline?

<details>
<summary>View Answer</summary>

<h3>CI/CD Pipeline Stages</h3>

```
Source (Git push)
  ↓
Build (compile, test)
  ↓
Test (unit, integration, e2e)
  ↓
Package (Docker image)
  ↓
Registry (push to Docker Hub)
  ↓
Deploy (staging)
  ↓
Test (smoke tests)
  ↓
Deploy (production)
```

<h3>Continuous Integration (CI)</h3>

<p><strong>Automated testing & building on every commit.</strong></p>

```
Developer pushes code
  ↓
Webhook triggers pipeline
  ↓
Automated:
  - Compile
  - Unit tests
  - Code quality (SonarQube)
  - Security scan
  - Build artifact
```

<h3>Continuous Deployment (CD)</h3>

<p><strong>Automatically deploy to production (if tests pass).</strong></p>

```
Tests pass
  ↓
Create Docker image
  ↓
Push to registry
  ↓
Deploy to Kubernetes
  ↓
Health checks
```

<h3>Tools & Tech Stack</h3>

<table>
<tr>
<th>Stage</th>
<th>Tools</th>
</tr>
<tr>
<td><strong>Source Control</strong></td>
<td>GitHub, GitLab, Bitbucket</td>
</tr>
<tr>
<td><strong>CI/CD Platform</strong></td>
<td>Jenkins, GitLab CI, GitHub Actions, CircleCI</td>
</tr>
<tr>
<td><strong>Build Tool</strong></td>
<td>Maven, Gradle, npm</td>
</tr>
<tr>
<td><strong>Container Registry</strong></td>
<td>Docker Hub, ECR, GCR</td>
</tr>
<tr>
<td><strong>Deployment</strong></td>
<td>Kubernetes, Argo CD, Helm</td>
</tr>
<tr>
<td><strong>Monitoring</strong></td>
<td>Prometheus, Grafana, ELK</td>
</tr>
</table>

<h3>Example: GitHub Actions Pipeline</h3>

```yaml
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: mvn clean test
      - run: docker build -t app:latest .
      - run: docker push app:latest
      - run: kubectl apply -f k8s/
```

<p>📚 <a href="/anandu-interview-playbook/devops/ci-cd/">Learn more: CI/CD Pipelines</a></p>

</details>

---

## 8. How do you monitor Kubernetes applications?

<details>
<summary>View Answer</summary>

<h3>Three Pillars of Observability</h3>

<h4>1. Metrics (Prometheus)</h4>

<p><strong>Quantitative measurements: CPU, memory, latency, error rates</strong></p>

```
Prometheus (scrapes metrics)
  ↓
Alert Manager (triggers alerts)
  ↓
Grafana (visualize dashboards)
```

<h4>2. Logs (ELK Stack)</h4>

<p><strong>Event data: application logs, events</strong></p>

```
Pods (generate logs)
  ↓
Fluentd/Logstash (collect)
  ↓
Elasticsearch (store)
  ↓
Kibana (visualize)
```

<h4>3. Traces (Jaeger)</h4>

<p><strong>Request flow: how requests move through services</strong></p>

```
Service A → Service B → Service C
(Jaeger tracks timing, latency, errors)
```

<h3>Kubernetes-Specific Monitoring</h3>

<table>
<tr>
<th>Resource</th>
<th>Metrics to Monitor</th>
</tr>
<tr>
<td><strong>Pod</strong></td>
<td>CPU, memory, restart count, ready status</td>
</tr>
<tr>
<td><strong>Node</strong></td>
<td>CPU, memory, disk, network</td>
</tr>
<tr>
<td><strong>Deployment</strong></td>
<td>Replicas desired vs actual, rollout status</td>
</tr>
<tr>
<td><strong>Application</strong></td>
<td>Request latency, error rate, throughput</td>
</tr>
</table>

<h3>Health Checks</h3>

```yaml
livenessProbe:      # Is pod alive?
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:     # Ready to serve traffic?
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

<h3>Key Alerts to Set</h3>

<ul>
<li>Pod crashes/restarts frequently</li>
<li>Memory usage > 80%</li>
<li>CPU usage sustained high</li>
<li>Error rate > threshold</li>
<li>Response latency > SLA</li>
<li>Deployment replicas not matching desired</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/observability/">Learn more: Monitoring & Observability</a></p>

</details>

---

## 9. What is orchestration in software and DevOps?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Orchestration</strong> is the automated coordination of many components (containers, services, infrastructure, pipeline steps) so they are deployed, connected, scaled and healed as one system, driven by a declared desired state instead of manual steps.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Container orchestration:</strong> scheduling containers onto nodes, restarting failed ones, scaling, service discovery, rolling updates (Kubernetes, Docker Swarm, Nomad, ECS).</li>
<li><strong>Infrastructure orchestration:</strong> provisioning resources declaratively (Terraform, CloudFormation).</li>
<li><strong>Workflow orchestration:</strong> ordering dependent jobs and retries (CI/CD pipelines, Airflow, Step Functions).</li>
<li><strong>Service orchestration vs choreography:</strong> a central coordinator calls services in order (orchestration) versus services reacting to events on their own (choreography, e.g. saga patterns).</li>
</ul>

<h3>Comparison Table</h3>
<table>
<tr><th>Concept</th><th>Orchestration</th><th>Configuration management</th></tr>
<tr><td>Focus</td><td>Running and coordinating many units at runtime</td><td>Configuring a single machine (Ansible, Chef)</td></tr>
<tr><td>Model</td><td>Declarative desired state + reconciliation loop</td><td>Often imperative or idempotent scripts</td></tr>
</table>

<h3>Example / Real-World Use</h3>
<p>You declare "run 3 replicas of image X". The orchestrator places pods, replaces a crashed one, and rolls out version Y gradually. No human restarts anything.</p>

<h3>When to Use</h3>
<ul>
<li>Many services or replicas, frequent deploys, need for self-healing and scaling.</li>
<li>Skip it for a single small app on one host; plain Docker Compose or a VM is simpler.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 10. How are Kubernetes and Docker related?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Docker builds and runs individual containers; Kubernetes orchestrates many containers across a cluster of machines. They solve different layers, so they are complementary, not competitors.</p>

<h3>Why / Context</h3>
<ul>
<li>Docker produces an OCI image (Dockerfile, <code>docker build</code>). Kubernetes pulls that image and runs it in a pod.</li>
<li>Kubernetes talks to a container runtime through the <strong>CRI</strong> (Container Runtime Interface). Since v1.24 the built-in <code>dockershim</code> was removed, so clusters use <strong>containerd</strong> or <strong>CRI-O</strong> directly.</li>
<li>Images built with Docker still run unchanged, because they follow the OCI standard.</li>
<li>Docker Swarm and Docker Compose are Docker's own (simpler) orchestration options.</li>
</ul>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Docker</th><th>Kubernetes</th></tr>
<tr><td>Scope</td><td>Build/run containers on one host</td><td>Cluster-wide scheduling and management</td></tr>
<tr><td>Scaling</td><td>Manual (or Compose/Swarm)</td><td>Built-in, declarative, autoscaling</td></tr>
<tr><td>Self-healing</td><td>Restart policies only</td><td>Replaces failed pods, reschedules on node loss</td></tr>
<tr><td>Networking</td><td>Bridge networks, port mapping</td><td>Services, Ingress, DNS, network policies</td></tr>
</table>

<h3>Example / Real-World Use</h3>
<pre><code>docker build -t registry.example.com/app:1.0 .
docker push registry.example.com/app:1.0
kubectl create deployment app --image=registry.example.com/app:1.0 --replicas=3</code></pre>

<h3>When to Use</h3>
<ul>
<li>Docker alone: local development, single-host deployments, CI builds.</li>
<li>Kubernetes: production workloads needing HA, scaling and rolling releases.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 11. What is Role-Based Access Control (RBAC) in Kubernetes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>RBAC is the Kubernetes authorization mechanism that decides which <em>subjects</em> (users, groups, service accounts) can perform which <em>verbs</em> (get, list, create, delete...) on which <em>resources</em>, in which namespaces. It is additive only: there are no deny rules.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Role</strong>: permissions within one namespace. <strong>ClusterRole</strong>: cluster-wide resources (nodes, PVs) or reusable permission sets.</li>
<li><strong>RoleBinding</strong> / <strong>ClusterRoleBinding</strong>: attach a role to subjects. A RoleBinding can reference a ClusterRole but only grants it inside its namespace.</li>
<li>The API server first authenticates (certs, OIDC, tokens), then authorizes via RBAC, then runs admission control.</li>
<li><strong>ServiceAccounts</strong> give pods an identity; apply least privilege and avoid the default account for workloads that call the API.</li>
</ul>

<h3>Example / Real-World Use</h3>

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: { name: pod-reader, namespace: dev }
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: { name: read-pods, namespace: dev }
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

<p>Check with <code>kubectl auth can-i list pods --as=system:serviceaccount:dev:app-sa -n dev</code>.</p>

<h3>When to Use</h3>
<ul>
<li>Always: separate dev/ops access, restrict CI service accounts, limit access to Secrets.</li>
<li>Gotcha: <code>cluster-admin</code> bindings and wildcard verbs/resources are a common security hole.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-advanced/">Learn more: Kubernetes Advanced Topics</a></p>

</details>

---

## 12. How does Kubernetes autoscaling work?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Three layers: <strong>HPA</strong> changes the number of pod replicas, <strong>VPA</strong> changes pod CPU/memory requests, and the <strong>Cluster Autoscaler</strong> (or Karpenter on AWS) adds or removes nodes.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Autoscaler</th><th>Scales</th><th>Signal</th><th>Notes</th></tr>
<tr><td>HPA</td><td>Replicas (horizontal)</td><td>CPU, memory, custom/external metrics</td><td>Needs metrics-server and resource requests; <code>desired = ceil(current * currentMetric / targetMetric)</code></td></tr>
<tr><td>VPA</td><td>Requests/limits (vertical)</td><td>Historical usage</td><td>Applying changes typically restarts pods; do not combine with HPA on the same CPU/memory metric</td></tr>
<tr><td>Cluster Autoscaler</td><td>Nodes</td><td>Pending pods that cannot be scheduled; underused nodes</td><td>Reacts in minutes; relies on correct pod requests</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: api }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: api }
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target: { type: Utilization, averageUtilization: 70 }
```

<p>Utilization is measured against the pod's CPU <em>request</em>, so missing requests break HPA. For Java apps, allow warm-up time (readiness probes, stabilization window) or HPA may over-scale during JVM startup.</p>

<h3>When to Use</h3>
<ul>
<li>HPA for stateless services; add event-driven scaling (KEDA) for queue-based workloads.</li>
<li>VPA in recommendation mode to right-size requests.</li>
<li>Cluster Autoscaler so HPA scale-out does not stall on full nodes.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 13. What is the difference between StatefulSets and Deployments?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A <strong>Deployment</strong> manages interchangeable, stateless pods. A <strong>StatefulSet</strong> manages pods that need a stable identity, stable network name and their own persistent storage (databases, Kafka, ZooKeeper).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Deployment</th><th>StatefulSet</th></tr>
<tr><td>Pod names</td><td>Random suffix</td><td>Ordered and stable: <code>db-0, db-1, db-2</code></td></tr>
<tr><td>Storage</td><td>Shared or ephemeral volumes</td><td>One PVC per pod via <code>volumeClaimTemplates</code>, kept across restarts</td></tr>
<tr><td>Network identity</td><td>Regular Service (load balanced)</td><td>Headless Service gives per-pod DNS: <code>db-0.db.ns.svc</code></td></tr>
<tr><td>Start/stop order</td><td>Parallel</td><td>Ordered by default (<code>OrderedReady</code>); reverse on scale down</td></tr>
<tr><td>Updates</td><td>Rolling, any pod</td><td>Rolling in reverse ordinal order; <code>partition</code> for staged rollouts</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata: { name: db }
spec:
  serviceName: db            # headless Service
  replicas: 3
  selector: { matchLabels: { app: db } }
  template:
    metadata: { labels: { app: db } }
    spec:
      containers:
      - name: db
        image: postgres:16
        volumeMounts: [{ name: data, mountPath: /var/lib/postgresql/data }]
  volumeClaimTemplates:
  - metadata: { name: data }
    spec:
      accessModes: ["ReadWriteOnce"]
      resources: { requests: { storage: 10Gi } }
```

<h3>When to Use</h3>
<ul>
<li>Deployment: web/API services, workers (the default choice).</li>
<li>StatefulSet: clustered data stores where each member has a role. Gotcha: deleting a StatefulSet does not delete its PVCs, and running a database yourself still needs backup and failover handling (operators help). Often a managed database is simpler.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 14. How to manage ConfigMaps and Secrets in Kubernetes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>ConfigMaps</strong> hold non-sensitive configuration; <strong>Secrets</strong> hold sensitive values. Both are injected into pods as environment variables or mounted files, keeping config out of the image.</p>

<h3>Why / Context</h3>
<ul>
<li>Secrets are only <strong>base64-encoded</strong> by default, not encrypted. Enable encryption at rest for etcd and restrict access with RBAC.</li>
<li><strong>Env vars</strong> are read at pod start; changes need a restart. <strong>Mounted volumes</strong> update eventually (not for <code>subPath</code> mounts), but the app must re-read the file.</li>
<li>Rollout trick: put a hash of the config into a pod annotation (Helm <code>checksum/config</code>) so a change triggers a rolling restart.</li>
<li><code>immutable: true</code> prevents accidental edits and reduces API server load.</li>
<li>Do not commit plain Secrets to Git. Use Sealed Secrets, SOPS, or the External Secrets Operator syncing from a vault (AWS Secrets Manager, HashiCorp Vault).</li>
<li>Size limit is about 1 MiB per object.</li>
</ul>

<h3>Example / Real-World Use</h3>

```bash
kubectl create configmap app-config --from-literal=LOG_LEVEL=INFO
kubectl create secret generic db-secret --from-literal=password='change-me'
```

```yaml
# in the container spec
envFrom:
- configMapRef: { name: app-config }
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef: { name: db-secret, key: password }
```

<p>For Spring Boot, env vars like <code>SPRING_DATASOURCE_PASSWORD</code> override properties automatically, or mount <code>application.yaml</code> from a ConfigMap.</p>

<h3>When to Use</h3>
<ul>
<li>ConfigMap for URLs, feature flags, log levels; Secret for passwords, tokens, TLS keys.</li>
<li>Prefer mounted files for secrets to reduce leakage through process listings and logs.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes/">Learn more: Kubernetes</a></p>

</details>

---

## 15. What are Kubernetes Network Policies and how to implement them?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A <strong>NetworkPolicy</strong> is a namespaced resource that controls which pods can talk to which (ingress and egress) using label selectors, namespaces, and IP blocks, at L3/L4. By default all pod traffic is allowed.</p>

<h3>Why / Context</h3>
<ul>
<li>Policies are only <strong>enforced if the CNI plugin supports them</strong> (Calico, Cilium, Antrea). With a CNI that does not, they are silently ignored.</li>
<li>A pod becomes isolated for a direction once any policy selects it for that direction; then only listed traffic is allowed. Policies are additive (union).</li>
<li>Typical pattern: default-deny, then allow specific flows.</li>
<li>Remember to allow DNS egress (port 53) when denying egress.</li>
<li>Standard NetworkPolicy has no L7 rules (no HTTP paths) or deny rules; CNI-specific or service mesh policies add these.</li>
</ul>

<h3>Example / Real-World Use</h3>

```yaml
# 1. Deny all ingress in the namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: default-deny-ingress, namespace: prod }
spec:
  podSelector: {}
  policyTypes: [Ingress]
---
# 2. Only the API pods may reach the database on 5432
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: db-allow-api, namespace: prod }
spec:
  podSelector: { matchLabels: { app: db } }
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector: { matchLabels: { app: api } }
    ports:
    - { protocol: TCP, port: 5432 }
```

<h3>When to Use</h3>
<ul>
<li>Multi-tenant clusters, compliance zones, limiting blast radius if a pod is compromised.</li>
<li>Roll out in a test namespace first; verify with a debug pod and <code>curl</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-advanced/">Learn more: Kubernetes Advanced Topics</a></p>

</details>

---

## 16. How to implement service mesh (Istio) in Kubernetes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Install Istio's control plane (<code>istiod</code>), enable sidecar injection for a namespace, restart workloads so each pod gets an Envoy proxy, then configure traffic, security and observability through Istio CRDs.</p>

<h3>Why / Context</h3>
<ul>
<li>The mesh moves cross-cutting concerns out of application code: mTLS, retries, timeouts, circuit breaking, traffic splitting, telemetry.</li>
<li><strong>Data plane:</strong> Envoy sidecars (or ambient mode with per-node ztunnel, no sidecars). <strong>Control plane:</strong> istiod distributes config and certificates.</li>
<li>Key resources: <code>VirtualService</code> (routing rules), <code>DestinationRule</code> (subsets, load balancing, circuit breaker), <code>Gateway</code> (edge), <code>PeerAuthentication</code> (mTLS), <code>AuthorizationPolicy</code>.</li>
<li>Costs: extra CPU/memory per pod, an added latency hop, operational complexity.</li>
</ul>

<h3>Example / Real-World Use</h3>

```bash
istioctl install --set profile=default
kubectl label namespace prod istio-injection=enabled
kubectl rollout restart deployment -n prod
```

```yaml
# 90/10 canary split (subsets v1/v2 are defined in a DestinationRule)
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata: { name: api }
spec:
  hosts: [api]
  http:
  - route:
    - { destination: { host: api, subset: v1 }, weight: 90 }
    - { destination: { host: api, subset: v2 }, weight: 10 }
```

<p>Enforce strict mTLS with a <code>PeerAuthentication</code> of mode <code>STRICT</code>; visualise traffic with Kiali, metrics via Prometheus, traces via Jaeger.</p>

<h3>When to Use</h3>
<ul>
<li>Many microservices needing uniform mTLS, canary releases and per-service telemetry.</li>
<li>Skip for a few services; an Ingress plus a client library (e.g. Resilience4j) is simpler.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-advanced/">Learn more: Kubernetes Advanced Topics</a></p>

</details>

---

## 17. What are Custom Resource Definitions (CRDs) in Kubernetes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A <strong>CRD</strong> registers a new resource type with the Kubernetes API server, so you can create, store (in etcd) and query your own objects with <code>kubectl</code> like built-in ones. The objects are <strong>Custom Resources (CRs)</strong>.</p>

<h3>Why / Context</h3>
<ul>
<li>A CRD alone only stores data. Behavior comes from a <strong>controller</strong> that watches the CRs and reconciles real state to the desired spec. CRD + controller = <strong>Operator pattern</strong>.</li>
<li>Schema is defined with OpenAPI v3 for validation; CRDs support versions, a status subresource, and printer columns.</li>
<li>Examples: Istio <code>VirtualService</code>, cert-manager <code>Certificate</code>, Prometheus Operator <code>ServiceMonitor</code>, Argo CD <code>Application</code>.</li>
<li>Controllers are written with client-go, Kubebuilder or Operator SDK (Java options: Fabric8, Java Operator SDK).</li>
</ul>

<h3>Example / Real-World Use</h3>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata: { name: backups.example.com }
spec:
  group: example.com
  scope: Namespaced
  names: { kind: Backup, plural: backups, singular: backup }
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              schedule: { type: string }
```

<p>After applying, <code>kubectl get backups</code> works; a Backup operator would watch these and run the jobs.</p>

<h3>When to Use</h3>
<ul>
<li>Encode domain-specific, declarative workflows (managed databases, certificates, platform APIs).</li>
<li>Avoid for plain config (use a ConfigMap) or when a Helm chart is enough. Gotcha: deleting a CRD deletes all its custom resources.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-advanced/">Learn more: Kubernetes Advanced Topics</a></p>

</details>

---

## 18. What are commonly used Linux commands and scripting techniques for deploying and maintaining Java applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Core toolkit: process and service control (<code>systemctl</code>, <code>ps</code>, <code>kill</code>), log inspection (<code>tail -f</code>, <code>grep</code>, <code>awk</code>, <code>sed</code>, <code>journalctl</code>), resources (<code>top</code>, <code>df</code>, <code>free</code>, <code>ss</code>), JVM tools (<code>jps</code>, <code>jstack</code>, <code>jcmd</code>, <code>jmap</code>), scheduling (<code>cron</code>) and safe Bash scripts.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Task</th><th>Command</th></tr>
<tr><td>Run app as a service</td><td><code>systemctl start|stop|status|restart app</code>, <code>journalctl -u app -f</code></td></tr>
<tr><td>Find errors in logs</td><td><code>grep -E "ERROR|Exception" app.log | tail -50</code></td></tr>
<tr><td>Count by field</td><td><code>awk '{print $9}' access.log | sort | uniq -c | sort -rn</code></td></tr>
<tr><td>Bulk edit config</td><td><code>sed -i 's/^port=.*/port=8081/' app.properties</code></td></tr>
<tr><td>Thread dump</td><td><code>jcmd &lt;pid&gt; Thread.print</code> or <code>jstack &lt;pid&gt;</code></td></tr>
<tr><td>Port/disk checks</td><td><code>ss -tlnp | grep 8080</code>, <code>df -h</code>, <code>du -sh *</code></td></tr>
</table>

<h3>Example / Real-World Use</h3>

```bash
#!/usr/bin/env bash
set -euo pipefail                     # fail fast on errors, unset vars, pipe failures

APP=app; NEW=/tmp/$APP-1.2.jar; DEST=/opt/$APP

cp "$DEST/$APP.jar" "$DEST/$APP.jar.bak"   # keep rollback copy
cp "$NEW" "$DEST/$APP.jar"
sudo systemctl restart "$APP"

for i in {1..30}; do                  # health check with timeout
  curl -fs http://localhost:8080/actuator/health >/dev/null && { echo "UP"; exit 0; }
  sleep 2
done
echo "Health check failed, rolling back"
cp "$DEST/$APP.jar.bak" "$DEST/$APP.jar"; sudo systemctl restart "$APP"; exit 1
```

<p>Minimal <code>systemd</code> unit: <code>ExecStart=/usr/bin/java -Xmx512m -jar /opt/app/app.jar</code>, <code>User=app</code>, <code>Restart=on-failure</code>.</p>

<h3>When to Use</h3>
<ul>
<li>Quote variables, use <code>set -euo pipefail</code>, make scripts idempotent, and never hardcode secrets.</li>
<li>Beyond a few dozen lines or multiple hosts, prefer CI/CD or Ansible over ad hoc scripts.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/ci-cd/">Learn more: CI/CD</a></p>

</details>

---

## 19. How would you automate log rotation or backups on a Linux server?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use <strong>logrotate</strong> (run daily by cron or a systemd timer) for logs, and <strong>cron</strong> or systemd timers to run a backup script that archives, ships off-host, prunes old copies, and is verified by test restores.</p>

<h3>Why / Context</h3>
<ul>
<li>Prefer logging to stdout and letting journald or the container runtime handle rotation; use logrotate for plain files.</li>
<li><code>copytruncate</code> avoids restarting the app but can lose a few lines; apps that reopen logs on a signal are cleaner. Logback and Log4j2 can also rotate by size/time internally.</li>
<li>Backups: the 3-2-1 rule (3 copies, 2 media, 1 off-site), encryption, retention, alerting on job failure, and regular restore tests.</li>
<li>Databases need consistent dumps (<code>pg_dump</code>, <code>mysqldump --single-transaction</code>), not a raw copy of a live data directory.</li>
</ul>

<h3>Example / Real-World Use</h3>

```bash
# /etc/logrotate.d/app
/var/log/app/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}

# crontab -e   (02:30 daily; flock prevents overlapping runs)
30 2 * * * flock -n /var/lock/backup.lock /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# backup.sh
set -euo pipefail
STAMP=$(date +%F)
pg_dump mydb | gzip > /backups/mydb-$STAMP.sql.gz
aws s3 cp /backups/mydb-$STAMP.sql.gz s3://my-backup-bucket/db/
find /backups -name '*.sql.gz' -mtime +7 -delete   # local retention
```

<p>Test a config with <code>logrotate -d /etc/logrotate.d/app</code> (debug, dry run).</p>

<h3>When to Use</h3>
<ul>
<li>Single servers and VMs. On Kubernetes or managed databases, use cluster logging, CronJobs, and managed snapshots instead.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/ci-cd/">Learn more: CI/CD</a></p>

</details>

---

## 20. What scripting challenges have you encountered in production?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Frame it as problem, cause, fix, lesson. The recurring production issues with scripts are listed below; pick one or two you have actually lived through.</p>

<h3>Why / Context</h3>
<table>
<tr><th>Challenge</th><th>Cause</th><th>Fix</th></tr>
<tr><td>Script silently continues after a failure</td><td>No error handling</td><td><code>set -euo pipefail</code>, explicit exit codes, <code>trap</code> for cleanup</td></tr>
<tr><td>Overlapping cron runs corrupt data</td><td>Job takes longer than the interval</td><td><code>flock</code>, lock files, idempotent steps</td></tr>
<tr><td>Works manually, fails in cron</td><td>Minimal <code>PATH</code>/env, different working dir</td><td>Absolute paths, set env in script, log output</td></tr>
<tr><td>Deleted wrong files</td><td>Unquoted or empty variable (<code>rm -rf $DIR/*</code>)</td><td>Quote, use <code>${VAR:?}</code>, dry-run first</td></tr>
<tr><td>Breaks on spaces or special characters</td><td>Word splitting, parsing <code>ls</code> output</td><td>Quote everything, <code>find -print0 | xargs -0</code></td></tr>
<tr><td>Secrets in scripts or shell history</td><td>Hardcoded credentials</td><td>Secret manager, env injection, restricted permissions</td></tr>
<tr><td>Flaky deploys</td><td>Not idempotent, no health check, no rollback</td><td>Health-check loop, keep previous artifact, move logic into CI/CD</td></tr>
<tr><td>Unmaintainable</td><td>Hundreds of lines of Bash</td><td>Switch to Python/Java or Ansible, add ShellCheck and tests</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```bash
TMP=$(mktemp)
trap 'echo "failed at line $LINENO"; rm -f "$TMP"' ERR EXIT
: "${DEPLOY_DIR:?DEPLOY_DIR must be set}"   # abort if unset or empty
```

<h3>When to Use</h3>
<ul>
<li>Interview tip: end with what you changed afterward (monitoring, alerting on job failure, code review for scripts, ShellCheck in CI).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/ci-cd/">Learn more: CI/CD</a></p>

</details>

---

## 21. Which AWS services are commonly used to host and run backend APIs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Choose by how much infrastructure you want to manage: EC2 (VMs), Elastic Beanstalk or App Runner (managed platform), ECS/Fargate or EKS (containers), or Lambda with API Gateway (serverless). Put a load balancer or API Gateway in front, and RDS/DynamoDB behind.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Service</th><th>Model</th><th>Good for</th><th>Trade-off</th></tr>
<tr><td>EC2 + ALB + Auto Scaling</td><td>You manage VMs</td><td>Full control, legacy apps</td><td>Patching, capacity management</td></tr>
<tr><td>Elastic Beanstalk</td><td>PaaS over EC2</td><td>Quick deploy of a Spring Boot jar</td><td>Less flexibility</td></tr>
<tr><td>ECS on Fargate</td><td>Serverless containers</td><td>Containerized APIs without cluster management</td><td>AWS-specific, per-vCPU cost</td></tr>
<tr><td>EKS</td><td>Managed Kubernetes</td><td>K8s portability, large microservice estates</td><td>Complexity, control plane cost</td></tr>
<tr><td>Lambda + API Gateway</td><td>Functions</td><td>Spiky or low traffic, event-driven</td><td>Cold starts (SnapStart helps Java), time limits</td></tr>
<tr><td>App Runner</td><td>Managed container/source service</td><td>Simplest container-to-URL path</td><td>Fewer knobs</td></tr>
</table>

<h3>Example / Real-World Use</h3>
<p>Typical production stack: Route 53, CloudFront or API Gateway, ALB, ECS Fargate tasks in private subnets across 2+ AZs, RDS (Multi-AZ) or DynamoDB, ElastiCache, Secrets Manager, IAM task roles, CloudWatch logs/alarms, ECR for images, and CI/CD pushing new task definitions.</p>

<h3>When to Use</h3>
<ul>
<li>Start with Fargate or Beanstalk for a typical Spring Boot API; pick Lambda for sporadic traffic; EKS if the team already runs Kubernetes.</li>
<li>Follow-ups: security groups and VPC design, autoscaling policies, zero-downtime deploys, cost.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/aws/">Learn more: AWS</a></p>

</details>

---

**See also:** [Docker & Containers](../../devops/docker.md), [Kubernetes](../../devops/kubernetes.md), [CI/CD](../../devops/ci-cd.md), [Interview Preparation](../index.md)
