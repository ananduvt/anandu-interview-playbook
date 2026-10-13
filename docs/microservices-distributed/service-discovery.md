# Service Discovery

Service discovery is the automatic detection and registration of services in a distributed system. Instead of hardcoding service locations, services register themselves and clients discover them dynamically.

---

## Problem: Static Service Locations

```java
// ❌ Bad: Hardcoded URLs break when services move
@Configuration
public class PaymentClientConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    public String getPaymentServiceUrl() {
        return "http://payment-service-prod-1:8080";  // Fixed IP/hostname
    }
}
```

**Issues:**
- Services move (scaling, failures, deployments)
- No awareness of instance health
- Manual configuration maintenance
- Difficult to handle load balancing

---

## Solution: Service Discovery

```
Service A (registers itself)
    ↓
Discovery Server (Eureka, Consul, etcd)
    ↑
Service B (queries "where is Service A?")
    ↓
Discovery Server responds with: [ip:port list]
    ↓
Service B uses endpoint dynamically
```

---

## Service Discovery Patterns

### 1. **Client-Side Discovery**

Client queries registry → gets service instances → chooses one → calls it

```java
@Service
public class PaymentClientService {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    public PaymentResponse chargeCard(Order order) {
        // Query registry for all "payment-service" instances
        List<ServiceInstance> instances = 
            discoveryClient.getInstances("payment-service");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("Payment service down");
        }
        
        // Client chooses an instance (round-robin, random, etc.)
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri().toString();
        
        // Call the service
        return restTemplate.postForObject(
            url + "/api/charge",
            order,
            PaymentResponse.class
        );
    }
}
```

**Pros:**
- Client has control over load balancing logic
- Can implement smart routing (latency-aware, circuit breaker)

**Cons:**
- Client logic is complex
- Every client must implement service discovery

---

### 2. **Server-Side Discovery**

Client calls a load balancer → load balancer queries registry → routes request

```
Client
  ↓ (calls load balancer)
API Gateway / Load Balancer
  ↓ (queries registry for service location)
Discovery Server (Eureka, Consul)
  ↓ (responds with available instances)
Load Balancer
  ↓ (routes to chosen instance)
Service A / Service B / Service C
```

**Pros:**
- Client simplicity (just call load balancer URL)
- Centralized routing logic

**Cons:**
- Load balancer becomes single point of failure
- Requires stable load balancer endpoint

---

## Spring Cloud Eureka (Client-Side Discovery)

### Setup

**Dependencies:**
```xml
<!-- Server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<!-- Clients -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### Eureka Server Configuration

```yaml
# Eureka Server (application.yml)
spring:
  application:
    name: eureka-server

server:
  port: 8761

eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
  server:
    enableSelfPreservation: false  # Dev only; prod should be true
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### Eureka Client (Service Registration)

```yaml
# application.yml
spring:
  application:
    name: payment-service

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instanceId: ${spring.cloud.client.hostname}:${spring.application.name}:${server.port}
    preferIpAddress: true
    leaseRenewalIntervalInSeconds: 30  # Heartbeat every 30s
    leaseExpirationDurationInSeconds: 90  # Remove if no heartbeat for 90s
```

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/api")
public class PaymentController {
    
    @PostMapping("/charge")
    public PaymentResponse charge(@RequestBody Order order) {
        // Service logic
        return new PaymentResponse(order.getId(), "SUCCESS");
    }
}
```

### Eureka Client (Service Discovery)

```java
@Service
public class OrderService {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    private RestTemplate restTemplate;
    
    public Order createOrder(Order order) {
        // Discover payment service
        ServiceInstance paymentService = 
            discoveryClient.getInstances("payment-service")
                .stream()
                .findAny()
                .orElseThrow(() -> new ServiceUnavailableException("Payment service"));
        
        String paymentUrl = paymentService.getUri().toString();
        
        // Charge order
        PaymentResponse payment = restTemplate.postForObject(
            paymentUrl + "/api/charge",
            order,
            PaymentResponse.class
        );
        
        if ("SUCCESS".equals(payment.getStatus())) {
            order.setStatus("CONFIRMED");
        }
        
        return order;
    }
}
```

### Better: Spring Cloud LoadBalancer

Instead of manual DiscoveryClient calls, use `@LoadBalanced` RestTemplate:

```java
@Configuration
public class ClientConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    
    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;
    
    public Order createOrder(Order order) {
        // Use service name directly; load balancer resolves it
        PaymentResponse payment = restTemplate.postForObject(
            "http://payment-service/api/charge",  // No IP/port!
            order,
            PaymentResponse.class
        );
        
        order.setStatus("CONFIRMED");
        return order;
    }
}
```

---

## Service Discovery Frameworks

| Framework | Type | Status | Approach | Use Case |
|-----------|------|--------|----------|----------|
| **Eureka** | Library | ⚠️ Maintenance | Client-side discovery, registry | Netflix stack, Spring Cloud |
| **Consul** | Standalone | ✅ Active | Server + client-side, KV store | Polyglot microservices, multi-cloud |
| **etcd** | Standalone | ✅ Active | Distributed key-value store | Kubernetes, CoreOS, cloud-native |
| **Zookeeper** | Standalone | ⚠️ Legacy | Distributed coordination | Hadoop, Kafka, older systems |
| **Kubernetes DNS** | Built-in | ✅ Native | Server-side via DNS | Container-native services |

---

## Detailed Comparison

### **Eureka (Netflix)**
```java
// ✅ Pros
- Spring Cloud native integration
- Easy to set up with Spring Boot
- Client-side load balancing

// ❌ Cons
- Maintenance mode (Netflix moved to Kubernetes)
- Less suitable for polyglot environments
- Requires Java
```

### **Consul (HashiCorp)**
```java
// ✅ Pros
- HTTP/DNS interfaces (language-agnostic)
- Built-in health checks and KV store
- Server-side discovery via DNS
- Multi-datacenter support

// ❌ Cons
- More complex to operate
- Heavier than Eureka
```

### **Kubernetes DNS (Native)**
```yaml
# In Kubernetes, services auto-discover via DNS
# Pod can call another service directly:
http://payment-service.default.svc.cluster.local:8080

# ✅ Pros
- Zero configuration (declarative)
- Native container orchestration
- Service health managed by K8s

# ❌ Cons
- Tightly coupled to Kubernetes
- Not suitable for non-containerized deployments
```

---

## Health Checks & Heartbeat

Service discovery relies on health information to avoid routing to dead instances.

### Eureka Health Check

```yaml
eureka:
  instance:
    leaseRenewalIntervalInSeconds: 30  # Heartbeat interval
    leaseExpirationDurationInSeconds: 90  # TTL before removal
```

```java
@Component
public class HealthCheck {
    
    @GetMapping("/actuator/health")
    public HealthStatus health() {
        return new HealthStatus("UP", "Database: UP, Cache: UP");
    }
}
```

### Consul Health Check

```json
{
  "id": "payment-service-1",
  "name": "payment-service",
  "tags": ["payment", "production"],
  "address": "192.168.1.100",
  "port": 8080,
  "check": {
    "http": "http://localhost:8080/health",
    "interval": "10s",
    "timeout": "5s"
  }
}
```

---

## Failure Scenarios & Mitigation

| Scenario | Impact | Mitigation |
|----------|--------|-----------|
| **Service crashes** | Registry still lists it (stale entry) | Health checks + TTL + circuit breaker |
| **Registry down** | New services can't register | Cache discovered services locally, fallback IPs |
| **Network partition** | Service isolated, registry can't reach it | Quorum-based removal (Consul), self-preservation (Eureka) |
| **Client cache stale** | Routes to dead instance | Combine with retry + circuit breaker |

### Resilient Discovery Pattern

```java
@Service
public class ResilientPaymentClient {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @CircuitBreaker(name = "payment")
    @Retry(name = "payment")
    public PaymentResponse charge(Order order) {
        ServiceInstance instance = discoveryClient
            .getInstances("payment-service")
            .stream()
            .findAny()
            .orElseThrow(() -> new ServiceUnavailableException("No healthy instances"));
        
        try {
            return restTemplate.postForObject(
                instance.getUri() + "/api/charge",
                order,
                PaymentResponse.class
            );
        } catch (Exception e) {
            // Circuit breaker + retry will handle
            throw e;
        }
    }
}
```

---

## Interview Talking Points

- **Why service discovery?** Eliminate hardcoded URLs; support dynamic scaling, failover, rolling updates
- **Client vs. server discovery?** Client-side = flexible routing, server-side = simpler clients
- **Eureka vs. Kubernetes DNS?** Eureka for legacy/polyglot; K8s DNS for containerized
- **Health checks importance?** Prevent routing to dead instances; combine with circuit breaker
- **Failure handling?** Use retry + circuit breaker + fallback; never trust discovery alone
- **Scalability?** Service discovery is a bottleneck; cache results, use hierarchical registries

---

**See also:** [Spring Cloud Implementation](spring-cloud.md), [Microservices Principles](principles.md)
