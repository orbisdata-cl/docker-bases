# node-static

Base image for **React** applications (static builds served via nginx).

**Image:** `orbisdata/node-static:lts`  
**Base:** `nginx:alpine`  
**User:** `nginx` (non-root)  
**Port:** `8080`

## What's included

- nginx Alpine — minimal footprint
- Hardened config: gzip, cache headers, security headers, SPA fallback
- Non-root nginx user
- nginx version hidden (`server_tokens off`)

## Usage in your project

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1 — build React app
FROM node:lts-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2 — serve with nginx
FROM orbisdata/node-static:lts AS runner
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 8080
```

> If your build output folder is `build/` instead of `dist/`, adjust the `COPY` path accordingly.

## SPA routing

The nginx config includes a fallback to `index.html` for all unmatched routes — React Router works out of the box.

## Cache behavior

| Asset type | Cache |
|-----------|-------|
| JS, CSS, fonts, images | 1 year, immutable |
| HTML (index.html) | No cache (default nginx behavior) |

## Security headers included

- `X-Frame-Options: SAMEORIGIN`
- `X-Content-Type-Options: nosniff`
- `X-XSS-Protection: 1; mode=block`
- `Referrer-Policy: strict-origin-when-cross-origin`

## Environment variables

React environment variables must be injected **at build time** (not runtime) since the output is static HTML/JS.

```dockerfile
FROM node:lts-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG REACT_APP_API_URL
ENV REACT_APP_API_URL=$REACT_APP_API_URL
RUN npm run build
```

Pass at build time:

```bash
docker build --build-arg REACT_APP_API_URL=https://api.example.com .
```
