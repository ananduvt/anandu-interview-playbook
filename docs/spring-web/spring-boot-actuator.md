# Spring Boot Actuator & Health Checks

Production monitoring: metrics, health checks, endpoints for observability.

---

## What is Spring Boot Actuator?

**Spring Boot Actuator** — Monitoring and management endpoints to inspect and manage your application at runtime.

### Key Features
- **Metrics**: Track application performance (CPU, memory, HTTP requests, database)
- **Health checks**: Endpoint status, database connectivity, disk space
- **Auditable events**: Track user actions, logins, configuration changes
- **Custom endpoints**: Create app-specific monitoring endpoints

### Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

## Built-in Endpoints

### Health Check (`/actuator/health`)
Simple health status:
```json
{
  "status": "UP"
}
```

Detailed health (include `management.endpoints.web.exposure.include=health` + `management.endpoint.health.show-details=always`):
```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 1099511627776,
        "free": 537637302272,
        "threshold": 10485760,
        "exists": true
      }
    },
    "livenessState": { "status": "LIVE" },
    "readinessState": { "status": "READY" }
  }
}
```

### Metrics (`/actuator/metrics`)
List all metrics:
```json
{
  "names": [
    "http.server.requests",
    "jvm.memory.used",
    "jvm.gc.pause",
    "process.uptime",
    "system.cpu.usage",
    "logback.events"
  ]
}
```

Specific metric (e.g., `/actuator/metrics/http.server.requests`):
```json
{
  "name": "http.server.requests",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 1024.0
    },
    {
      "statistic": "TOTAL_TIME",
      "value": 45000.0
    },
    {
      "statistic": "MAX",
      "value": 1234.0
    }
  ],
  "baseUnit": "seconds"
}
```

### Info (`/actuator/info`)
Application metadata:
```json
{
  "app": {
    "name": "My API",
    "version": "1.0.0"
  },
  "build": {
    "version": "1.0.0",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Environment (`/actuator/env`)
Active configuration properties (be careful not to expose secrets!):
```json
{
  "propertySources": [
    {
      "name": "systemProperties",
      "properties": {
        "java.version": "17.0.1",
        "os.name": "Linux"
      }
    }
  ]
}
```

### Loggers (`/actuator/loggers`)
View and modify log levels at runtime:
```json
{
  "levels": ["OFF", "ERROR", "WARN", "INFO", "DEBUG", "TRACE"],
  "loggers": {
    "ROOT": { "configuredLevel": null, "effectiveLevel": "INFO" },
    "com.example.app": { "configuredLevel": "DEBUG", "effectiveLevel": "DEBUG" }
  }
}
```

Change log level:
```bash
curl -X POST http://localhost:8080/actuator/loggers/com.example.app \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel":"DEBUG"}'
```

### Other Endpoints
| Endpoint | Purpose |
|----------|---------|
| `/actuator/httptrace` | HTTP request/response history |
| `/actuator/threaddump` | JVM thread dump |
| `/actuator/heapdump` | Heap dump (for memory analysis) |
| `/actuator/mappings` | All endpoint mappings |
| `/actuator/caches` | Cache statistics |
| `/actuator/prometheus` | Prometheus-format metrics |

---

## Configuration

### Expose Endpoints
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,info,loggers  # Whitelist specific endpoints
        # include: "*"  # Expose all (not recommended in production)
        exclude: env,shutdown  # Exclude sensitive endpoints
  
  endpoint:
    health:
      show-details: always  # Show detailed health info
      # show-details: when-authorized  # Only for authenticated users
    
    loggers:
      enabled: true
    
    shutdown:
      enabled: false  # Don't expose shutdown endpoint

  # Custom endpoint base path
  endpoints:
    web:
      base-path: /monitoring  # Endpoints at /monitoring/health, /monitoring/metrics
      path-mapping:
        health: status  # Rename /health to /status
```

### Application Info
```yaml
info:
  app:
    name: My Invoice API
    description: REST API for invoice management
    version: @project.version@  # From pom.xml
  company:
    name: ACME Corp
    website: https://acme.com
```

### Custom Metrics
```java
import io.micrometer.core.instrument.MeterRegistry;

@Component
public class CustomMetrics {
    private final MeterRegistry meterRegistry;
    
    public CustomMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    // Counter: Increments on events
    public void recordOrderCreated() {
        meterRegistry.counter("orders.created").increment();
    }
    
    // Gauge: Track current value
    public void registerActiveConnections() {
        meterRegistry.gauge("connections.active", () -> 
            getActiveConnectionCount());
    }
    
    // Timer: Measure duration
    public void recordProcessingTime() {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            processData();
        } finally {
            sample.stop(Timer.builder("data.processing.time")
                .tag("operation", "import")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry));
        }
    }
}
```

### Custom Health Indicator
```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class ExternalServiceHealth implements HealthIndicator {
    
    @Override
    public Health health() {
        try {
            boolean serviceUp = checkExternalService();
            if (serviceUp) {
                return Health.up()
                    .withDetail("external-api", "responding")
                    .build();
            } else {
                return Health.down()
                    .withDetail("external-api", "not responding")
                    .build();
            }
        } catch (Exception e) {
            return Health.outOfService()
                .withException(e)
                .build();
        }
    }
    
    private boolean checkExternalService() {
        // Call external API or check resource
        return true;
    }
}
```

Result in `/actuator/health`:
```json
{
  "status": "UP",
  "components": {
    "externalServiceHealth": {
      "status": "UP",
      "details": {
        "external-api": "responding"
      }
    }
  }
}
```

---

## Liveness & Readiness Probes

For Kubernetes deployments:

### Liveness Probe
"Is my app still running?" — Kill and restart if unhealthy
```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### Readiness Probe
"Can I send traffic to this instance?" — Remove from service if not ready
```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Enable Probes
```yaml
management:
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

### Custom Readiness State
```java
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.ReadinessState;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
public class DataInitService {
    private final ApplicationEventPublisher publisher;
    
    public DataInitService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }
    
    @EventListener
    public void onApplicationReady(ApplicationReadyEvent event) {
        try {
            initializeData();
            // Mark app as ACCEPTING_TRAFFIC
            publisher.publishEvent(
                new AvailabilityChangeEvent<>(
                    this, ReadinessState.ACCEPTING_TRAFFIC));
        } catch (Exception e) {
            // Mark app as REFUSING_TRAFFIC until data loads
            publisher.publishEvent(
                new AvailabilityChangeEvent<>(
                    this, ReadinessState.REFUSING_TRAFFIC));
        }
    }
}
```

---

## Integration with Prometheus & Grafana

### Prometheus Format Metrics
Enable Micrometer Prometheus:
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Access metrics at `/actuator/prometheus`:
```
# HELP http_server_requests_seconds_max ...
# TYPE http_server_requests_seconds_max gauge
http_server_requests_seconds_max{exception="None",method="GET",status="200",uri="/api/users"} 0.123
```

### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-app'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/actuator/prometheus'
```

### Grafana Dashboard
Query Prometheus metrics in Grafana:
- `http_server_requests_seconds_count` — Request count
- `http_server_requests_seconds_sum` — Total request time
- `jvm_memory_used_bytes` — Memory usage
- `process_uptime_seconds` — App uptime

---

## Security Considerations

### 1. Protect Sensitive Endpoints
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,info  # Only safe endpoints
```

### 2. Require Authentication
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/actuator/health").permitAll()  // Public health
                .antMatchers("/actuator/**").hasRole("ADMIN")  // Restrict others
                .anyRequest().authenticated()
                .and()
            .httpBasic();
        return http.build();
    }
}
```

### 3. Mask Sensitive Info
```yaml
management:
  endpoint:
    env:
      enabled: false  # Don't expose environment variables
    
    health:
      show-details: when-authorized  # Only for authenticated users
```

### 4. Audit Logs
```java
@Component
public class ActuatorAuditLog {
    private final AuditEventRepository repository;
    
    @EventListener
    public void onActuatorAccess(HttpServletRequest request) {
        if (request.getRequestURI().startsWith("/actuator")) {
            repository.add(new AuditEvent(
                "ADMIN",
                "ACTUATOR_ACCESS",
                Map.of("endpoint", request.getRequestURI())
            ));
        }
    }
}
```

---

## Troubleshooting

### Endpoints Not Accessible
**Problem:** 404 on `/actuator/health`
**Solution:** Add actuator dependency and expose in config
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
```

### Health Shows DOWN Unexpectedly
**Problem:** Custom health indicator returns DOWN
**Solution:** Check the custom health component details
```bash
curl http://localhost:8080/actuator/health
# Look for DOWN components and fix the underlying issue
```

### Metrics Missing
**Problem:** Expected metrics not in `/actuator/metrics`
**Solution:** Ensure Micrometer is on classpath (included in spring-boot-starter-actuator)

---

## Best Practices

✅ **Do's**
- Use health checks for deployment readiness (Kubernetes)
- Export metrics to monitoring system (Prometheus, Datadog)
- Implement custom metrics for business KPIs
- Authenticate actuator endpoints in production
- Monitor key metrics (request latency, error rate, heap memory)
- Use alarms on critical metrics

❌ **Don'ts**
- Don't expose all endpoints (`include: "*"`) in production
- Don't log sensitive data (passwords, API keys) in health/metrics
- Don't ignore failed health checks
- Don't query metrics endpoint frequently (performance impact)
- Don't rely on in-memory metrics (restart = data loss) — use external monitoring

---

**See also:** [Spring Boot Advanced](spring-boot-advanced.md), [Observability](../devops/observability.md), [Kubernetes](../devops/kubernetes.md)

