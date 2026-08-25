# syntax=docker/dockerfile:1
FROM nginx:alpine AS base

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Add hardened nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Run as non-root
RUN chown -R nginx:nginx /var/cache/nginx /var/log/nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 8080
