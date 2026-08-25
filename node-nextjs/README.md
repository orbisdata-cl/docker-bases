# node-nextjs

Base image for **Next.js** applications using [standalone output](https://nextjs.org/docs/pages/api-reference/next-config-js/output).

**Image:** `orbisdata/node-nextjs:lts`  
**Base:** `node:lts-alpine`  
**User:** `node` (non-root)  
**Entrypoint:** `dumb-init`

## Requirement

Your `next.config.js` must enable standalone output:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
};

module.exports = nextConfig;
```

This tells Next.js to produce a self-contained `/.next/standalone` folder with only what's needed to run — no `node_modules` in the final image.

## Usage in your project

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1 — install and build
FROM node:lts-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2 — final image
FROM orbisdata/node-nextjs:lts AS runner

# Copy standalone output
COPY --from=builder --chown=node:node /app/.next/standalone ./
# Copy static assets
COPY --from=builder --chown=node:node /app/.next/static ./.next/static
# Copy public folder
COPY --from=builder --chown=node:node /app/public ./public

EXPOSE 3000
ENV PORT=3000 HOSTNAME=0.0.0.0

CMD ["node", "server.js"]
```

## Why standalone?

Without standalone, the final image would need the full `node_modules` folder (~300MB+).  
With standalone, Next.js traces only the dependencies actually used and bundles them — final image is typically **under 100MB**.

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `production` | Node environment |
| `NEXT_TELEMETRY_DISABLED` | `1` | Disables Next.js telemetry |
| `PORT` | `3000` | Set in CMD or docker-compose |
| `HOSTNAME` | `0.0.0.0` | Required to bind inside container |
