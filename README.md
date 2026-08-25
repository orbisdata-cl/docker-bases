# docker-bases

Production-ready base Docker images for Orbis Data projects.

## Images

| Image | Base | Stack |
|-------|------|-------|
| `orbisdata/node-base:lts` | `node:lts-alpine` | Express, NestJS |
| `orbisdata/node-nextjs:lts` | `node:lts-alpine` | Next.js standalone |
| `orbisdata/node-static:lts` | `nginx:alpine` | React (static build) |
| `orbisdata/python-base:lts` | `python:3.12-slim` | FastAPI (Uvicorn), Flask (Gunicorn) |
| `orbisdata/java-base:lts` | `eclipse-temurin:21-jre-alpine` | Spring Boot fat JAR |

## Principles

- **No dev tools in production** — only what the app needs to run
- **Non-root user** — all images run as an unprivileged user
- **Minimal surface** — Alpine or slim variants to reduce vulnerabilities
- **LTS only** — always pinned to the latest LTS release of each runtime
- **Multi-stage builds** — build artifacts never leak into the final image

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
