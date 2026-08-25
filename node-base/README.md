# node-base

Base image for **Express** and **NestJS** applications.

**Image:** `orbisdata/node-base:lts`  
**Base:** `node:lts-alpine`  
**User:** `node` (non-root)  
**Entrypoint:** `dumb-init` (proper PID 1 and signal handling)

## What's included

- Node.js LTS (Alpine)
- `dumb-init` — prevents zombie processes and handles SIGTERM correctly
- `NODE_ENV=production` pre-set

## What's NOT included

No dev dependencies, no build tools, no shell extras. Only what's needed to run a Node.js app.

## Usage in your project

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1 — install dependencies
FROM orbisdata/node-base:lts AS deps
COPY package*.json ./
RUN npm ci --omit=dev

# Stage 2 — final image
FROM orbisdata/node-base:lts AS runner
COPY --from=deps /app/node_modules ./node_modules
COPY --chown=node:node . .

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## NestJS example

```dockerfile
# syntax=docker/dockerfile:1

FROM orbisdata/node-base:lts AS builder
USER root
RUN apk add --no-cache python3 make g++
USER node
COPY package*.json ./
RUN npm ci
COPY --chown=node:node . .
RUN npm run build

FROM orbisdata/node-base:lts AS runner
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `production` | Node environment |
| `PORT` | — | Set in your app or docker-compose |
