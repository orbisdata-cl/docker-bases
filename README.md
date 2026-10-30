# docker-bases

Production-ready base Docker images for Orbis Data projects.

## Images

| Image | Base | Stack |
|-------|------|-------|
| `ghcr.io/orbisdata-cl/node-base:lts` | `node:22-alpine` | Express, NestJS |
| `ghcr.io/orbisdata-cl/node-nextjs:lts` | `node:22-alpine` | Next.js standalone |
| `ghcr.io/orbisdata-cl/node-static:lts` | `nginx:1.27-alpine` | React (static build) |
| `ghcr.io/orbisdata-cl/python-base:lts` | `python:3.12-slim` | FastAPI (Uvicorn), Flask (Gunicorn) |
| `ghcr.io/orbisdata-cl/java-base:lts` | `eclipse-temurin:21-jre-alpine` | Spring Boot fat JAR |

## Principles

- **No dev tools in production** — only what the app needs to run
- **Non-root user** — all images run as an unprivileged user
- **Minimal surface** — Alpine or slim variants to reduce vulnerabilities
- **LTS & explicit versions** — pinned to specific LTS releases for predictability
- **Multi-stage builds** — build artifacts never leak into the final image
- **Multi-arch support** — native `linux/amd64` and `linux/arm64` (Graviton/Apple Silicon)
- **Proper PID 1 handling** — `dumb-init` configured across runtime containers for graceful shutdown

## Updating a base image

1. Edit the `Dockerfile` in the relevant folder
2. Open a PR to `main`
3. CI rebuilds, tests, and pushes the new image to DockerHub
4. Projects inherit the update on their next deploy — no changes needed per project

## Using in your project

Each folder has a `README.md` with copy-paste `Dockerfile` examples for that stack.

## Versioning

Images are tagged with:
- `lts` — always the latest stable build (recommended)
- `lts-YYYYMMDD` — date-pinned build for reproducibility
