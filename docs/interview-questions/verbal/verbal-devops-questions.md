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

<p>📚 <a href="/anandu-interview-playbook/devops/docker.md">Learn more: Docker & Containers</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/docker.md">Learn more: Docker Images & Containers</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes.md">Learn more: Kubernetes</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes.md">Learn more: Kubernetes Pods & Deployments</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes-deployment-strategies.md">Learn more: Deployment Strategies</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/kubernetes.md">Learn more: Kubernetes & GitOps</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/ci-cd.md">Learn more: CI/CD Pipelines</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/devops/observability.md">Learn more: Monitoring & Observability</a></p>

</details>

---

**See also:** [Docker & Containers](../../devops/docker.md), [Kubernetes](../../devops/kubernetes.md), [CI/CD](../../devops/ci-cd.md), [Interview Preparation](../index.md)
