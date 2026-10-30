# syntax=docker/dockerfile:1
FROM node:24-alpine AS base

# Install dumb-init for proper signal handling and process management
RUN apk add --no-cache dumb-init

WORKDIR /app

ENV NODE_ENV=production \
    NPM_CONFIG_UPDATE_NOTIFIER=false \
    NPM_CONFIG_FUND=false

# Run as non-root user (built into node:alpine)
USER node

ENTRYPOINT ["dumb-init", "--"]
