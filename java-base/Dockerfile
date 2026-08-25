# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jre-alpine AS base

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init curl

WORKDIR /app

# Create non-root user
RUN addgroup --gid 1001 appuser && \
    adduser --uid 1001 --ingroup appuser --disabled-password --no-create-home appuser

USER appuser

ENTRYPOINT ["dumb-init", "--"]
