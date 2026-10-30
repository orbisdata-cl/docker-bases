# syntax=docker/dockerfile:1
FROM node:24-alpine AS base

RUN apk add --no-cache dumb-init

WORKDIR /app

ENV NODE_ENV=production \
    NEXT_TELEMETRY_DISABLED=1 \
    NPM_CONFIG_UPDATE_NOTIFIER=false \
    NPM_CONFIG_FUND=false

USER node

ENTRYPOINT ["dumb-init", "--"]
