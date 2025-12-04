
# Use Node.js 18 Alpine as base image for smaller size
FROM node:18-alpine AS base

# Install dependencies needed for other packages
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    supervisor \
    wget \
    libc6-compat \
    && rm -rf /var/cache/apk/*

# Build-time toggles (used by next.config and Next itself)
ARG SKIP_LINT_AND_TYPECHECK=1
ENV SKIP_LINT_AND_TYPECHECK=$SKIP_LINT_AND_TYPECHECK
ENV NEXT_DISABLE_ESLINT=1
ENV NEXT_DISABLE_TYPECHECK=1
ENV NEXT_TELEMETRY_DISABLED=1
# Give Node more heap to avoid OOM during build
ENV NODE_OPTIONS=--max-old-space-size=4096

# Create app directory
WORKDIR /app

COPY ./akshayapatra .

# Install dependencies
RUN npm install

# Build the Next.js application (lint + typecheck skipped)
RUN npm run build

# Create supervisor configuration to manage Next.js
RUN mkdir -p /etc/supervisor/conf.d
COPY <<EOF /etc/supervisor/conf.d/supervisord.conf
[supervisord]
nodaemon=true
user=root
logfile=/var/log/supervisor/supervisord.log
pidfile=/var/run/supervisord.pid

[program:nextjs]
command=npm start
directory=/app
autostart=true
autorestart=true
stderr_logfile=/var/log/supervisor/nextjs.err.log
stdout_logfile=/var/log/supervisor/nextjs.out.log
environment=NODE_ENV=production
EOF

# Create necessary directories
RUN mkdir -p /var/log/supervisor /var/run

# Expose ports
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

# Start supervisor to manage Next.js service
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
