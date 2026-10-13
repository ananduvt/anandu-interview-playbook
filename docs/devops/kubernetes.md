# Kubernetes Fundamentals

Container orchestration platform: automated deployment, scaling, self-healing, networking, and service mesh integration for microservices at scale.

---

## Kubernetes Architecture

### Control Plane (Master)

```
┌─────────────────────────────────────┐
│         Control Plane (Master)      │
├─────────────────────────────────────┤
│ API Server       (kube-apiserver)   │ ← REST API, cluster gateway
│ Etcd             (data store)       │ ← Cluster state (immutable log)
│ Scheduler        (kube-scheduler)   │ ← Assigns pods to nodes
│ Controller Mgr   (kube-controller)  │ ← Runs controllers (Deployment, HPA)
└─────────────────────────────────────┘
```

### Nodes (Workers)

```
┌─────────────────────────────────────┐
│      Node (Worker Machine)          │
├─────────────────────────────────────┤
│ Kubelet              (agent)        │ ← Runs pods, reports status
│ Kube-proxy           (networking)   │ ← iptables rules, service routing
│ Container Runtime    (Docker/CRI)   │ ← Manages containers
│ ┌─────────────────────────────────┐ │
│ │ Pod (1+ containers + storage)   │ │
│ │ ┌──────────┬──────────────────┐ │ │
│ │ │ Container│ Shared network   │ │ │
│ │ └──────────┴──────────────────┘ │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

---

## Core Objects

### Pod (Smallest deployable unit)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Deployment (Declarative pods with rolling updates)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: default
spec:
  replicas: 3                    # Keep 3 replicas running
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                # Max 1 extra pod during update
      maxUnavailable: 0          # Min pods that must be ready (zero downtime)
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
        image: nginx:1.22
        ports:
        - containerPort: 80
```

### Service (Stable network endpoint)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer             # Expose externally
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80                     # External port
    targetPort: 80               # Pod port
```

**Service Types:**
- `ClusterIP`: Internal DNS (default)
- `NodePort`: Expose on node IP:port
- `LoadBalancer`: Cloud load balancer (AWS ELB, GCP LB)
- `ExternalName`: DNS CNAME redirect

### Ingress (L7 HTTP routing)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-cert
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

### ConfigMap & Secret (Externalized config)

```yaml
# ConfigMap: non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "postgres.default.svc.cluster.local"
  log.level: "INFO"

# Secret: sensitive data (base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: dXNlcm5hbWU=         # base64(username)
  password: cGFzc3dvcmQ=         # base64(password)
```

### Other Core Objects

- **ReplicaSet**: Keeps N pod replicas running (managed by Deployment)
- **StatefulSet**: Stable pod identity/persistent storage (databases)
- **DaemonSet**: One pod per node (logging, monitoring)
- **Job/CronJob**: Batch/scheduled workloads
- **Namespace**: Logical isolation (dev, staging, prod)

---

## Health Checks & Probes

### Liveness Probe (Is pod alive?)

Restart pod if check fails.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30           # Wait 30s before checking
  periodSeconds: 10                 # Check every 10s
  failureThreshold: 3               # Restart after 3 failures
```

### Readiness Probe (Is pod ready for traffic?)

Remove from load balancer if check fails.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Startup Probe (Slow startup app?)

Allow time for slow-starting containers (Java, .NET).

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30              # 30 * 10s = 5min max startup time
  periodSeconds: 10
```

---

## Scaling Strategies

### Horizontal Pod Autoscaler (HPA)

Scale pod replicas based on metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 3                    # Minimum pods
  maxReplicas: 10                   # Maximum pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70      # Scale up if CPU > 70%
```

### Vertical Pod Autoscaler (VPA)

Right-size resource requests/limits.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  updatePolicy:
    updateMode: "Auto"              # Auto-apply recommendations
```

---

## Resource Management

### Requests & Limits

```yaml
resources:
  requests:
    memory: "128Mi"                 # Minimum guaranteed
    cpu: "250m"                     # 250 millicores (0.25 CPU)
  limits:
    memory: "256Mi"                 # Maximum allowed
    cpu: "500m"                     # Hard cap
```

**QoS Classes:**
1. **Guaranteed**: requests == limits (highest priority)
2. **Burstable**: requests < limits (medium priority)
3. **BestEffort**: no requests/limits (lowest priority, first to evict)

---

## Deployment Strategies

For detailed deployment strategies including Rolling Update, Blue-Green, Canary, and Shadow deployments, see [Kubernetes Deployment Strategies](kubernetes-deployment-strategies.md).

---

## GitOps with Argo CD

Declarative, Git-driven cluster management.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/config
    targetRevision: main
    path: k8s/apps/my-app         # Manifests in git
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true                 # Delete resources removed from git
      selfHeal: true              # Sync if cluster drifts
```

**Workflow:**
1. Push manifest to Git → main branch
2. Argo CD detects change (polling or webhook)
3. Applies to cluster automatically
4. Git is single source of truth

---

## AWS EKS (Elastic Kubernetes Service)

Managed Kubernetes on AWS.

```yaml
# Pod assumes IAM role (IRSA: IAM Roles for Service Accounts)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/app-role

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      serviceAccountName: app-sa   # Pod assumes IAM role
      containers:
      - name: app
        image: myapp:1.0
```

**Benefits:**
- AWS manages control plane (API, etcd, scheduler)
- Auto-patch nodes
- Integration with IAM, EC2, RDS, S3

---

## Common kubectl Commands

```bash
# View resources
kubectl get pods                        # List pods
kubectl get svc                         # List services
kubectl get deploy                      # List deployments
kubectl describe pod <name>             # Detailed info
kubectl logs <pod>                      # Container logs
kubectl logs -f <pod>                   # Stream logs

# Apply/update resources
kubectl apply -f deployment.yaml        # Create/update
kubectl delete -f deployment.yaml       # Delete

# Port forwarding
kubectl port-forward svc/nginx 8080:80  # Local 8080 → Pod 80

# Execute commands
kubectl exec -it <pod> -- /bin/bash     # Shell into pod

# Rollout management
kubectl rollout status deployment/app   # Watch rollout progress
kubectl rollout undo deployment/app     # Rollback to previous
kubectl rollout history deployment/app  # See versions

# Debugging
kubectl describe node <node>            # Node status
kubectl top nodes                       # Node resource usage
kubectl top pod                         # Pod resource usage
```

---

## Interview Talking Points

- **Why Kubernetes?** Automate deployment, scaling, rolling updates, self-healing, multi-cloud
- **Pod vs Container?** Pod: smallest unit (1+ containers); shares network namespace (IP, ports)
- **Deployment vs Pod?** Deployment: declarative, rolling updates, replicas; Pod: static
- **Service types?** ClusterIP (internal), NodePort (node:port), LoadBalancer (cloud LB)
- **HPA vs VPA?** HPA: add/remove pods (horizontal); VPA: increase/decrease resources (vertical)
- **Probes?** Liveness (restart if unhealthy), Readiness (remove from LB), Startup (slow apps)
- **GitOps benefit?** Git as source of truth; automatic sync; audit trail; declarative infrastructure
- **IRSA?** IAM Roles for Service Accounts (AWS); pod assumes IAM role without credentials

---

**See also:** [Kubernetes Deployment Strategies](kubernetes-deployment-strategies.md), [DevOps Overview](index.md), [Docker & Containers](docker.md), [CI/CD](ci-cd.md)
