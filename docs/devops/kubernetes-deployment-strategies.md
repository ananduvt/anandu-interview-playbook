# Kubernetes Deployment Strategies

Advanced deployment strategies for zero-downtime updates, testing, and rollback capabilities in Kubernetes.

---

## Rolling Update (Default, Zero Downtime)

Gradually replace old pods with new ones while maintaining availability.

```
Initial: [v1] [v1] [v1]  (3 replicas)

Step 1:  [v2] [v1] [v1]  (1 updated)
Step 2:  [v2] [v2] [v1]  (2 updated)
Step 3:  [v2] [v2] [v2]  (complete)

Pros: ✅ Zero downtime, automatic rollback on failure
Cons: ❌ Brief mixed versions, no instant cutover
Time: 3-10 minutes typical
```

**Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1              # Allow 1 extra pod (total 4)
      maxUnavailable: 0        # Keep all pods ready (zero downtime)
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v2.0
      # Readiness probe ensures pod is healthy before traffic
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
```

**Kubectl commands:**
```bash
kubectl apply -f deployment.yaml           # Trigger update
kubectl rollout status deployment/app      # Watch progress
kubectl rollout history deployment/app     # See all versions
kubectl rollout undo deployment/app        # Rollback to previous
kubectl rollout undo deployment/app --to-revision=3  # Rollback to specific
```

---

## Blue-Green Deployment (Instant Cutover)

Maintain two identical production environments and switch instantly between them.

```
Blue (v1):   [v1] [v1] [v1]  (production, receiving traffic)
Green (v2):  [v2] [v2] [v2]  (staging, fully deployed, no traffic)

Testing on green...
All checks pass...

Switch traffic: Service.selector points from blue → green
Result: Instant cutover, blue still running for instant rollback

Pros: ✅ Instant cutover, instant rollback, binary switch
Cons: ❌ 2x resources during deployment, no gradual testing
Time: Instant switch, 2x resource footprint
```

**Implementation:**

```yaml
# Blue Deployment (v1.0)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080

---

# Green Deployment (v2.0)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        ports:
        - containerPort: 8080

---

# Service: Points to blue (production)
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
    version: blue    # Change to "green" to switch traffic
  ports:
  - port: 80
    targetPort: 8080
```

**Workflow:**
```bash
# 1. Deploy green (v2.0), but service still points to blue
kubectl apply -f app-green.yaml

# 2. Run integration tests against green
kubectl port-forward deployment/app-green 8080:8080
# Test against localhost:8080

# 3. If tests pass, switch traffic to green
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'

# 4. If issues detected, instant rollback to blue
kubectl patch service app-service -p '{"spec":{"selector":{"version":"blue"}}}'

# 5. Keep blue running for N hours (or days), then delete
kubectl delete deployment app-blue
```

**Advantages over Rolling Update:**
- ✅ Zero risk: complete green environment tested before switch
- ✅ Instant rollback if production issues
- ✅ No mixed versions during update

---

## Canary Deployment (Gradual Traffic Shift)

Gradually shift traffic from old to new version while monitoring metrics.

```
Initial: [v1: 95%] ← 95% of traffic
         [v2: 5%]  ← 5% of traffic (canary - small test group)

Monitor error rate, latency...
If good:
  Step 1:  [v1: 80%, v2: 20%]
  Step 2:  [v1: 50%, v2: 50%]
  Step 3:  [v1: 0%, v2: 100%]  ← Complete

If error rate > threshold:
  Automatic rollback: [v1: 100%, v2: 0%]

Pros: ✅ Gradual rollout, detect issues early with real traffic
Cons: ❌ Requires service mesh or traffic splitting tool, 1-2 hour rollout
Time: 30-120 minutes typical
```

**Simple Canary (without Service Mesh):**

```yaml
# Deploy both versions
# v1 has 95% of replicas, v2 has 5%
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v1
spec:
  replicas: 19        # 95% traffic (19 out of 20)
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v2
spec:
  replicas: 1         # 5% traffic (1 out of 20)
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: app
        image: myapp:v2.0

---

# Service load-balances across both (by replica count)
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp          # No version filter: hits both v1 and v2
  ports:
  - port: 80
    targetPort: 8080
```

**Advanced Canary (with Flagger + Istio):**

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  progressDeadlineSeconds: 3600
  service:
    port: 80
  analysis:
    interval: 1m                        # Check metrics every 1 min
    threshold: 5                        # Rollback if > 5% error rate
    maxWeight: 50                       # Max 50% traffic to canary
    stepWeight: 10                      # Increase 10% per interval
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99                         # Must be ≥ 99% success rate
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500                        # Max 500ms latency
      interval: 30s
    webhooks:
    - name: integration-tests
      url: http://flagger-loadtester/
      timeout: 30s
      metadata:
        type: smoke
        cmd: "curl -sd 'test' http://app-canary/health | grep ok"
    - name: load-tests
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://app-canary/"
```

**Workflow:**
```bash
# 1. Update image in Deployment
kubectl set image deployment/app app=myapp:v2.0

# 2. Flagger detects change and starts canary analysis
kubectl get canary -w

# 3. Monitor progress
kubectl get canary app -o yaml | grep status

# 4. If analysis passes, promotion is automatic
# If fails, Flagger automatically rolls back
```

---

## Shadow Deployment (Zero-Risk Testing)

Mirror production traffic to new version (no impact) for testing before promotion.

```
Production: [v1] ← Real traffic → Production response
            ↓
Mirror:     [v2] ← Same traffic → Test response (ignored)

Compare v1 vs v2 responses: errors, latency, differences
- If v2 looks good: promote to canary/rolling
- If v2 has issues: discard (zero production impact)

Pros: ✅ Test with real traffic, zero impact if bad, early issue detection
Cons: ❌ Requires 2x resources, slow feedback (need sufficient traffic)
Risk: None (shadow doesn't affect production)
```

**Implementation (Istio VirtualService):**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: app
spec:
  hosts:
  - app
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: app
        subset: v1
      weight: 100                   # All traffic to v1
    mirror:
      host: app
      subset: v2                    # Mirror to v2
    mirrorPercent: 100              # Mirror 100% of traffic
    mirrorPercentage:
      value: 100.0
  timeout: 30s
  retries:
    attempts: 3
    perTryTimeout: 10s

---

# Define versions (subsets)
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: app
spec:
  host: app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Workflow:**
```bash
# 1. Deploy v2 (shadowed)
kubectl set image deployment/app app=myapp:v2.0

# 2. Mirror 100% of v1 traffic to v2
# v2 responses are collected but not sent to clients

# 3. Compare logs/metrics between v1 and v2
kubectl logs -l version=v2 | grep error

# 4. If v2 looks good, switch to canary or rolling update
# If bad issues found, just delete v2 (zero impact)
```

---

## Comparison Table

| Strategy | Cutover | Risk | Rollback Speed | Testing | Resource | Time |
|----------|---------|------|----------------|---------|----------|------|
| **Rolling** | Gradual (5-10 min) | Low (monitored) | Automatic | Limited | Same | 3-10 min |
| **Blue-Green** | Instant | Very Low (full test) | Instant (keep old) | Full environment | 2x | Minutes |
| **Canary** | Gradual (30-120 min) | Low (monitored, auto-rollback) | Automatic | Real traffic | 1.2x | 30-120 min |
| **Shadow** | N/A (testing only) | None (zero impact) | N/A | Real traffic | 2x | Hours |

---

## Choosing a Strategy

| Scenario | Recommendation | Reason |
|----------|-----------------|--------|
| **Standard deployments** | Rolling Update | Default, simple, automated |
| **High-confidence releases** | Blue-Green | Complete testing before cutover |
| **New features in production** | Canary | Detect issues with real users |
| **Risky changes (DB migrations)** | Shadow | Zero-impact testing |
| **Emergency hotfixes** | Blue-Green | Instant rollback if issues |
| **Performance-critical** | Canary | Monitor latency before full rollout |

---

## Rollback Scenarios

```bash
# Rolling update auto-rollback on readiness probe failure
# (Pod fails health check → stays in old version)

# Manual rollback
kubectl rollout undo deployment/app

# Rollback to specific revision
kubectl rollout undo deployment/app --to-revision=3

# Pause rollout (mid-way)
kubectl rollout pause deployment/app

# Resume rollout
kubectl rollout resume deployment/app
```

---

## Interview Talking Points

- **Rolling vs Blue-Green?** Rolling: gradual, no extra resources; Blue-Green: instant, 2x resources
- **Why Canary?** Detect issues with real traffic before full rollout; automatic rollback
- **Shadow deployment use?** Test risky changes (schema updates, algorithm changes) with zero impact
- **Combination strategies?** Blue-green for initial cutover + canary for gradual validation = "canary-blue-green"
- **Readiness probe importance?** Ensures pod is healthy before traffic; rolling update won't proceed if probes fail
- **Kubectl rollout undo?** Revert to previous Deployment revision (reuses old ReplicaSet)
- **Zero-downtime guarantee?** maxUnavailable: 0 + readiness probe = no requests to unhealthy pods

---

**See also:** [Kubernetes Fundamentals](kubernetes.md), [DevOps Overview](index.md)
