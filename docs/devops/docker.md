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
