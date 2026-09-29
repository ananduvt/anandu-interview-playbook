# Kubernetes

Container **orchestration** — automates deployment, scaling, healing, and networking of containers across a cluster.

## Architecture
- **Control plane**: API server, **etcd** (cluster state store), scheduler, controller manager.
- **Nodes** (workers): **kubelet** (agent), **kube-proxy** (networking), container runtime.

## Core objects
- **Pod** — smallest deployable unit; one or more containers sharing network/storage.
- **ReplicaSet** — keeps N pod replicas running.
- **Deployment** — declarative pods + rolling updates/rollbacks (manages ReplicaSets).
- **Service** — stable network endpoint / load balancing for pods (ClusterIP, NodePort, LoadBalancer).
- **Ingress** — L7 HTTP routing into the cluster.
- **ConfigMap / Secret** — externalized config / sensitive data.
- **Namespace** — logical isolation.
- **StatefulSet** — stable identity/storage for stateful apps; **DaemonSet** — one pod per node; **Job/CronJob** — batch.

## Scaling & health
- **HPA** (Horizontal Pod Autoscaler) — scale pods on CPU/memory/custom metrics.
- **Liveness / readiness / startup probes** — restart unhealthy pods; route traffic only when ready.
- **Requests/limits** — resource guarantees + caps.

## Deployment strategies
Rolling update (default), recreate; blue-green / canary via labels or a service mesh / Argo Rollouts.

## GitOps
**Argo CD** / Flux sync the cluster to declarative manifests in git (single source of truth).

## On AWS
**EKS** = managed Kubernetes; **IRSA** lets pods assume IAM roles via a projected service-account token.

## Common commands
`kubectl get pods/svc/deploy` · `kubectl describe` · `kubectl logs` · `kubectl apply -f` · `kubectl rollout status/undo`.
