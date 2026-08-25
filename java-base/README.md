# java-base

Base image for **Spring Boot** applications packaged as a fat JAR.

**Image:** `orbisdata/java-base:lts`  
**Base:** `eclipse-temurin:21-jre-alpine` (JRE only — no JDK in production)  
**User:** `appuser` (non-root, uid 1001)  
**Entrypoint:** `dumb-init` (proper PID 1 and signal handling)

## What's included

- Eclipse Temurin 21 JRE (Alpine) — smallest JRE with long-term support
- `dumb-init` — prevents zombie processes and handles SIGTERM correctly (important for graceful shutdown in ECS/EKS)
- `curl` — for health checks in docker-compose
- Non-root `appuser`

## Why JRE and not JDK?

The JDK includes the compiler, debugger, and tools not needed at runtime. The JRE is ~200MB smaller and has a smaller attack surface. Build your JAR in CI, copy only the artifact into this image.

## Usage in your project

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1 — build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /build
COPY . .
RUN ./mvnw package -DskipTests --no-transfer-progress

# Stage 2 — run
FROM orbisdata/java-base:lts AS runner
COPY --from=builder --chown=appuser:appuser /build/target/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## JVM tuning for containers

Spring Boot 3+ and Java 21 are container-aware by default — the JVM reads cgroup memory limits automatically. Still, it's good practice to set explicit flags:

```dockerfile
CMD ["java", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseContainerSupport", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
```

Or via environment variable in docker-compose / ECS task definition:

```yaml
environment:
  - JAVA_OPTS=-XX:MaxRAMPercentage=75.0 -XX:+UseContainerSupport
```

```dockerfile
CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## Graceful shutdown

Spring Boot handles `SIGTERM` for graceful shutdown by default (since 2.3). Ensure your `application.properties` includes:

```properties
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `JAVA_OPTS` | — | JVM flags injected at runtime |
| `SPRING_PROFILES_ACTIVE` | — | e.g. `production`, `staging` |
| `SERVER_PORT` | `8080` | Override the default port |
