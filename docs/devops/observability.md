# Observability

The three pillars: **metrics, logs, traces** — to answer *what* broke, *why*, and *where*.

## Metrics
- Numeric time-series (counters, gauges, histograms) — request rate, error rate, latency (p50/p95/p99), saturation.
- **Micrometer** (Java facade) → **Prometheus** (scrape + store) → **Grafana** (dashboards + alerts).
- The **RED** method (Rate, Errors, Duration) for services; **USE** (Utilization, Saturation, Errors) for resources.

## Logging
- Structured (JSON) logs with levels; ship to **ELK/OpenSearch** or **Splunk** via stdout + collector.
- **Correlation id** in **MDC** to tie logs of one request together across services.
- Never log secrets/PII — mask sensitive fields.

## Distributed tracing
- Follow one request across services via a propagated **trace id / span id** (W3C traceparent).
- Tools: **OpenTelemetry** (standard), Jaeger, Zipkin, Tempo. Spring: Micrometer Tracing.
- Reveals cross-service latency and the critical path in a fan-out.

## Alerting & SLOs
- **SLI** (measured signal) → **SLO** (target, e.g. 99.9%) → **SLA** (contract). Error budgets guide releases.
- Alert on symptoms (user-facing SLOs) not just causes; avoid alert fatigue.

## Health checks
Liveness (restart), readiness (route traffic), startup — exposed via Spring Boot **Actuator** (`/actuator/health`).
