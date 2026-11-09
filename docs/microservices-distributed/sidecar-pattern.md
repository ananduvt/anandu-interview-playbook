# Sidecar Pattern

A **sidecar** is a helper process or container deployed alongside the main application, sharing its lifecycle, network namespace and (optionally) volumes. It adds a cross-cutting capability without changing application code or language. In Kubernetes it is an extra container in the same Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: orders
spec:
  containers:
    - name: app
      image: example/orders:1.0
      volumeMounts:
        - { name: logs, mountPath: /var/log/app }
    - name: log-shipper            # sidecar
      image: fluent/fluent-bit:3.0
      volumeMounts:
        - { name: logs, mountPath: /var/log/app, readOnly: true }
  volumes:
    - name: logs
      emptyDir: {}
```

Recent Kubernetes versions support **native sidecars**: an init container with `restartPolicy: Always`. It starts before the app containers, runs for the Pod's life and stops after them, which fixes the startup-ordering and Job-completion problems of plain sidecar containers.

## Common Sidecar Uses

| Use | What the sidecar does |
|-----|-----------------------|
| Service mesh proxy | Envoy handles mTLS, retries, routing, telemetry (see [Kubernetes Advanced](../devops/kubernetes-advanced.md)) |
| Logging | Tails log files and ships them (for example Fluent Bit) |
| Config / secret refresh | Pulls config from a store into a shared volume and signals the app (for example Vault Agent, config reloaders) |
| Metrics / tracing | Exporter or OpenTelemetry collector next to the app |
| Ambassador | Proxy for outbound calls to remote services |
| Adapter | Normalises output (metrics or log format) for the platform |
| Database proxy | Local proxy that handles auth and TLS to a managed database |

## Sidecar vs Ambassador vs Adapter

| Pattern | Direction | Purpose |
|---------|-----------|---------|
| Sidecar | Any | Extends the app with a general capability |
| Ambassador | App to outside | App talks to `localhost`; the ambassador handles discovery, retries, auth, sharding |
| Adapter | Outside to app | Makes a non-conforming app look standard (for example exposes Prometheus metrics for a legacy app) |

Ambassador and adapter are specialised sidecars.

## Trade-offs

| Pros | Cons |
|------|------|
| Language agnostic, no library per language | Extra CPU and memory per Pod, multiplied by replicas |
| Independent upgrade and ownership (platform team) | Added latency hop for proxies |
| Isolation: a sidecar crash does not corrupt app memory | More moving parts, startup and shutdown ordering |
| Consistent behaviour across services | Debugging spans two containers |

Alternatives: a shared library (simple, but one per language and upgrades are coupled), a node-level **DaemonSet** agent (cheaper per node, less isolated), or a sidecar-less mesh mode such as Istio ambient.

## Common Interview Questions

- **Sidecar vs library?** A sidecar is language neutral and independently deployed; a library has no extra hop or resource cost.
- **Sidecar vs DaemonSet?** A sidecar is per Pod with Pod-level isolation; a DaemonSet agent is shared per node.
- **How does the app talk to a sidecar?** Via `localhost` or a shared volume, since containers in a Pod share the network namespace and can share volumes.
- **What is the ambassador pattern?** A sidecar proxying the app's outbound calls to a remote service.
