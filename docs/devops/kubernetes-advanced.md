# Kubernetes Advanced Topics

Helm package management, service mesh (Istio), advanced networking, and production patterns.

---

## Helm Charts

**Helm** — Package manager for Kubernetes; templating and deployment automation.

### Why Helm?

**Problem:** Deploying applications to Kubernetes requires multiple YAML files
```
deployment.yaml
service.yaml
configmap.yaml
secret.yaml
ingress.yaml
```

Managing versions, dependencies, and configurations across environments is complex.

**Solution:** Helm Charts
- Single command to deploy entire application stack
- Reusable, versioned packages
- Environment-specific values
- Dependency management

### Chart Structure

```
my-app/
├── Chart.yaml                 # Chart metadata (name, version, description)
├── values.yaml                # Default configuration values
├── values-dev.yaml            # Development overrides
├── values-prod.yaml           # Production overrides
├── templates/
│   ├── deployment.yaml        # Template files (use {{ .Values.x }})
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl           # Template functions
│   └── NOTES.txt              # Post-install notes
├── Chart.lock                 # Dependency versions (auto-generated)
├── charts/                    # Dependent charts
└── README.md
```

### Helm Commands

```bash
# Create new chart
helm create my-app

# Validate chart
helm lint my-app

# See what will be deployed (dry-run)
helm install my-app my-app/ --dry-run --debug

# Install chart
helm install my-app my-app/

# Deploy with custom values
helm install my-app my-app/ -f values-prod.yaml

# Upgrade existing release
helm upgrade my-app my-app/ -f values-prod.yaml

# Rollback to previous version
helm rollback my-app 1

# List installed releases
helm list

# Get release details
helm status my-app
helm get values my-app
helm get manifest my-app

# Uninstall release
helm uninstall my-app

# Repository management
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm search repo nginx  # Find charts
helm pull stable/nginx  # Download chart
```

### Chart.yaml

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for my application
type: application
version: 1.0.0           # Chart version (increment for releases)
appVersion: "1.0"        # Application version
maintainers:
  - name: DevTeam
    email: dev@example.com
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```

### values.yaml

```yaml
# Default values for my-app
replicaCount: 3

image:
  repository: my-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

nodeSelector:
  workload: general

tolerations: []

affinity: {}

# Database config
postgresql:
  enabled: true
  auth:
    username: app
    password: secret
    database: mydb
```

### deployment.yaml Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
        env:
        - name: DATABASE_URL
          value: "postgresql://{{ .Values.postgresql.auth.username }}:{{ .Values.postgresql.auth.password }}@postgresql:5432/{{ .Values.postgresql.auth.database }}"
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### Helm Hooks

Execute commands at specific release lifecycle events:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-db-migrate
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"  # Run before main install
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: db-migrate
        image: my-app:{{ .Values.appVersion }}
        command: ["/app/migrate.sh"]
      restartPolicy: Never
```

**Hook Types:**
- `pre-install`, `post-install` — Before/after first install
- `pre-upgrade`, `post-upgrade` — Before/after upgrade
- `pre-delete`, `post-delete` — Before/after uninstall
- `pre-rollback`, `post-rollback` — Before/after rollback
- `test` — Run tests

---

## Service Mesh (Istio)

**Service Mesh** — Dedicated infrastructure layer for managing service-to-service communication.

### What Problems Does Istio Solve?

**Without Istio:**
- Service-to-service communication logic scattered in application code
- Hard to implement retry, timeout, circuit breaker consistently
- Monitoring latency/errors requires application instrumentation
- Mutual TLS (mTLS) setup is complex
- Canary deployments require custom logic

**With Istio:**
- Declarative traffic management (separate from apps)
- Automatic resilience (retries, timeouts, circuit breaker)
- Distributed tracing & monitoring out of the box
- Automatic mTLS encryption
- Traffic shifting for canary/A-B deployments

### Istio Architecture

```
┌─────────────────────────────────────────────┐
│           Control Plane                      │
│  ┌──────────────┐  ┌──────────────┐         │
│  │ Istiod       │  │ Cert Manager │         │
│  │ (config mgmt)│  │ (security)   │         │
│  └──────────────┘  └──────────────┘         │
└────────────────────┬────────────────────────┘
                     │
      ┌──────────────┴──────────────┐
      │ (xDS protocol)              │
      ▼                             ▼
  ┌─────────────┐           ┌─────────────┐
  │  Service A  │           │  Service B  │
  │ (w/ Envoy   │           │ (w/ Envoy   │
  │  sidecar)   │           │  sidecar)   │
  └─────────────┘           └─────────────┘
```

**Components:**
- **Istiod**: Control plane (configuration, certificate management)
- **Envoy**: Sidecar proxy in each pod (intercepts traffic, applies policies)
- **xDS**: Protocol for control plane → proxy communication

### Installation

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.16.0
export PATH=$PWD/bin:$PATH

# Install Istio
istioctl install --set profile=demo -y

# Enable sidecar injection for namespace
kubectl label namespace default istio-injection=enabled

# Verify installation
kubectl get pods -n istio-system
```

### Traffic Management

#### VirtualService

Routes traffic to different service versions:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app  # Service name
  http:
  - match:
    - uri:
        prefix: /v2  # Route /v2/* to v2
    route:
    - destination:
        host: my-app
        subset: v2
      weight: 100
  - route:  # Default: route to v1
    - destination:
        host: my-app
        subset: v1
      weight: 100
```

#### DestinationRule

Defines how to load balance traffic:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app  # Service FQDN
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

#### Gateway & VirtualService (Ingress)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-app-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "app.example.com"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app-routes
spec:
  hosts:
  - "app.example.com"
  gateways:
  - my-app-gateway
  http:
  - match:
    - uri:
        prefix: /api/v2
    route:
    - destination:
        host: my-app
        port:
          number: 8080
        subset: v2
  - route:
    - destination:
        host: my-app
        port:
          number: 8080
        subset: v1
```

### Canary Deployment

Gradually shift traffic to new version:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome.*"  # Route Chrome users to v2
    route:
    - destination:
        host: my-app
        subset: v2
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 90  # 90% to v1
    - destination:
        host: my-app
        subset: v2
      weight: 10  # 10% to v2
```

**Progressive rollout:**
- Day 1: 95% v1, 5% v2
- Day 2: 80% v1, 20% v2
- Day 3: 50% v1, 50% v2
- Day 4: Full v2

### Resilience Patterns

#### Retry

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure
    route:
    - destination:
        host: my-app
```

#### Timeout

```yaml
http:
- timeout: 10s  # Request timeout
  route:
  - destination:
      host: my-app
```

#### Circuit Breaker

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    outlierDetection:
      consecutive5xxErrors: 5      # Eject after 5 errors
      interval: 30s                 # Check every 30s
      baseEjectionTime: 30s         # Eject for 30s
      maxEjectionPercent: 50        # Max 50% of instances ejected
      minRequestVolume: 5           # Minimum requests before checking
```

### Security (mTLS)

Enable automatic mutual TLS:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT  # Require mTLS for all traffic

---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-my-app
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-app
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
```

### Observability

#### Distributed Tracing

Automatically trace requests across services:

```bash
# Port-forward to Jaeger UI
kubectl port-forward -n istio-system svc/jaeger 16686:16686

# Access at http://localhost:16686
```

#### Monitoring Dashboards

```bash
# Port-forward to Kiali (service mesh UI)
kubectl port-forward -n istio-system svc/kiali 20000:20000

# Access at http://localhost:20000
# Shows:
# - Service topology
# - Request rates, errors, latency
# - Traffic flow
# - Circuit breaker status
```

#### Prometheus Metrics

```bash
# Istio automatically exports Prometheus metrics
# Access at http://prometheus:9090

# Query examples:
# - request_count: Sum of all requests
# - request_duration_milliseconds: Latency distribution
# - request_errors: Error count
```

### Comparison: Istio vs Linkerd

| Feature | Istio | Linkerd |
|---------|-------|---------|
| **Complexity** | Complex | Simple |
| **Resource Usage** | High (large control plane) | Low (lightweight) |
| **Performance** | Slower (more features) | Faster |
| **mTLS** | ✅ Automatic | ✅ Automatic |
| **Observability** | ✅ Built-in (Kiali, Jaeger) | ✅ Built-in (web dashboard) |
| **Traffic Management** | ✅ Advanced (VirtualService) | ✅ Basic |
| **Canary Deployments** | ✅ Native support | ⚠️ Manual |
| **Use Case** | Enterprise, complex | Lightweight, Kubernetes-native |

**Choose Istio if:**
- Complex traffic management needed (canary, A/B testing, traffic shaping)
- Advanced observability required
- Enterprise support needed

**Choose Linkerd if:**
- Lightweight, "works out of the box"
- Minimal overhead
- Kubernetes-first philosophy

---

## Advanced Networking

### Network Policies

Restrict traffic between pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### DNS & Service Discovery

Kubernetes provides built-in DNS:

```bash
# Service DNS format
<service-name>.<namespace>.svc.cluster.local

# Examples:
my-app.default.svc.cluster.local
postgresql.databases.svc.cluster.local

# Kubernetes automatically updates DNS when services scale
```

---

## Production Patterns

### Blue-Green Deployment with Istio

```yaml
# Blue (current) - 100% traffic
# Green (new) - 0% traffic
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: blue
      weight: 100
    - destination:
        host: my-app
        subset: green
      weight: 0

# When green is ready, switch traffic
# $ kubectl patch vs my-app --type merge -p '
# spec:
#   http[0]:
#     route:
#     - destination:
#         host: my-app
#         subset: blue
#       weight: 0
#     - destination:
#         host: my-app
#         subset: green
#       weight: 100'
```

### Cost Optimization

```yaml
# Pod Disruption Budget (ensure availability during node maintenance)
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app

---
# Vertical Pod Autoscaler (optimize resource requests)
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Automatically update pod requests
```

---

## Best Practices

✅ **Do's**
- Use Helm for templating and versioning
- Implement service mesh for complex environments
- Use network policies to restrict traffic
- Enable mTLS for sensitive services
- Implement proper RBAC (role-based access control)
- Use resource limits to prevent resource starvation
- Monitor with Prometheus + Grafana or Kiali

❌ **Don'ts**
- Don't use Helm for simple deployments (overcomplicated)
- Don't deploy service mesh without understanding it
- Don't skip network policies (security risk)
- Don't use plain HTTP in production (no mTLS)
- Don't ignore pod resource limits
- Don't skip monitoring and alerting

---

**See also:** [Kubernetes Fundamentals](kubernetes.md), [Kubernetes Deployment Strategies](kubernetes-deployment-strategies.md), [DevOps Observability](observability.md)

