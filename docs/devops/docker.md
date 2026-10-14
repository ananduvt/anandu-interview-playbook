# Docker & Containers

## Why containers
Package an app + its dependencies into a portable, isolated unit that runs the same everywhere.
Lighter than VMs (share the host kernel; no full guest OS).

## Containers vs VMs
| | Container | VM |
|--|-----------|----|
| Isolation | process-level (namespaces/cgroups) | full OS (hypervisor) |
| Overhead | low, fast start | heavy, slow start |
| Size | MBs | GBs |

## Core concepts
- **Image** — immutable template (layers); built from a **Dockerfile**.
- **Container** — a running instance of an image.
- **Registry** — stores images (Docker Hub, ECR, Nexus).
- **Volume** — persistent storage outside the container's writable layer.
- **Network** — bridge/host/overlay for container communication.

## Dockerfile essentials
```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```
- **Layer caching** — order instructions least- to most-frequently changing (deps before code).
- **Multi-stage builds** — build in one stage, copy only artifacts into a slim runtime image.
- Run as **non-root**; keep images minimal (distroless/alpine); pin versions.

## Common commands
`docker build -t app .` · `docker run -p 8080:8080 app` · `docker ps` · `docker logs` · `docker exec -it <id> sh` · `docker compose up`.

## Docker Compose
Declare multi-container apps (app + db + cache) in `docker-compose.yml` for local dev/integration.

---

## Java-Specific Containerization

### Dockerfile for Java Application

#### Simple Approach (Single-stage)
```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Problems:**
- Large image size (~400MB with full JRE)
- Includes unnecessary build tools
- Not optimized for production

#### Better: Multi-Stage Build

```dockerfile
# Stage 1: Build
FROM eclipse-temurin:21 AS builder
WORKDIR /build
COPY pom.xml .
COPY src src
RUN mvn clean package -DskipTests

# Stage 2: Runtime (lightweight)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Benefits:**
- Small final image (~150MB instead of 400MB)
- Build artifacts not included
- Faster layer caching

#### Best: Distroless + JVM Tuning

```dockerfile
# Multi-stage with JVM optimization
FROM eclipse-temurin:21 AS builder
WORKDIR /build
COPY . .
RUN mvn clean package -DskipTests

# Use distroless for minimal image
FROM gcr.io/distroless/java21-debian11
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar

# JVM flags for containers
ENV JAVA_TOOL_OPTIONS="-XX:+UseG1GC -XX:MaxRAMPercentage=75.0"

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**distroless advantages:**
- Only Java runtime + app (no shell, no package managers)
- Image size: ~50-80MB
- Smaller attack surface
- Cannot `docker exec` into container (no shell)

### JVM Flags for Containers

```dockerfile
# Recognize container memory limits (Java 10+)
ENV JAVA_TOOL_OPTIONS="-XX:+UseContainerSupport"

# Set heap automatically
ENV JAVA_TOOL_OPTIONS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
# If container has 1GB RAM, heap = 750MB

# Use G1GC for consistent latency
ENV JAVA_TOOL_OPTIONS="-XX:+UseContainerSupport -XX:+UseG1GC -XX:MaxGCPauseMillis=100"

# Virtual threads (Java 21+)
ENV JAVA_TOOL_OPTIONS="-XX:+UseContainerSupport --enable-preview"
```

**Why container-aware:**
```
OLD: java -Xmx2G (fixed heap, ignores container limits)
NEW: java -XX:MaxRAMPercentage=75.0 (uses container's memory)
```

### Layer Caching Optimization

```dockerfile
# ❌ Bad: Changes to source invalidate all layers below
FROM eclipse-temurin:21
WORKDIR /app
COPY . .                           # Copies everything
RUN mvn clean package -DskipTests

# ✅ Better: Dependencies cached, only app code rebuilt
FROM eclipse-temurin:21
WORKDIR /app
COPY pom.xml pom.xml               # Layer 1: Dependencies (stable)
RUN mvn dependency:resolve         # Layer 2: Download deps (cached)
COPY src src                       # Layer 3: Source code (changes frequently)
RUN mvn clean package -DskipTests  # Layer 4: Build (rebuilt if src changes)
```

**Cache busting for frequent builds:**
```dockerfile
# Add timestamp to force rebuild
ARG BUILD_DATE
FROM eclipse-temurin:21
LABEL build_date=${BUILD_DATE}
...
```

### Running Java in Kubernetes

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        image: myrepo/api-service:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        env:
        - name: JAVA_TOOL_OPTIONS
          value: "-XX:MaxRAMPercentage=75.0 -XX:+UseG1GC"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Building & Pushing Images

```bash
# Build image
docker build -t myrepo/api-service:1.0 .

# Run locally
docker run -p 8080:8080 -e JAVA_TOOL_OPTIONS="-Xmx512m" myrepo/api-service:1.0

# Push to registry
docker push myrepo/api-service:1.0

# Kubernetes deployment
kubectl apply -f deployment.yaml
kubectl rollout status deployment/api-service
```

### Docker Compose: Spring Boot + MySQL

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/mydb
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=secret
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=mydb
    volumes:
      - db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

**Run locally:**
```bash
docker-compose up  # Starts app + db
docker-compose logs -f app  # View logs
docker-compose down  # Cleanup
```

### Container Security Best Practices

```dockerfile
# Run as non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Use read-only root filesystem where possible
SECURITY_OPTIONS='--read-only'

# Scan image for vulnerabilities
# docker scan myimage:latest

# Use specific base image tags (not 'latest')
FROM eclipse-temurin:21.0.1-jre-alpine  # ✅ Specific version
FROM eclipse-temurin:21-jre              # ❌ Floating tag

# Pin dependencies
RUN apt-get update && apt-get install -y \
    specific-lib=1.2.3 \  # ✅ Pin version
    another-lib           # ❌ Floating version
```

### Troubleshooting Container Issues

```bash
# Check container logs
docker logs <container-id>

# View container processes
docker exec <container-id> ps aux

# Check memory/CPU usage
docker stats

# Inspect environment variables
docker exec <container-id> env

# Test JVM settings
docker run --rm openjdk:21-jre java -XshowSettings:vm

# View image layers
docker history myimage:1.0
```

### Image Size Comparison

```
Dockerfile approach          | Image Size
─────────────────────────────|───────────
FROM openjdk:21 (full JDK)   | 900MB ❌
FROM openjdk:21-jre          | 400MB
FROM alpine + openjdk:21-jre | 150MB
FROM distroless/java21       | 50-80MB ✅
```

**Best practice:** Use distroless + multi-stage for production.
