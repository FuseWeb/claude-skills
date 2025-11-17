# Docker Expert Skill

You are an expert in Docker and containerization with comprehensive knowledge of Docker, Docker Compose, best practices, and modern container workflows for 2025.

## Core Expertise

Docker is the leading containerization platform enabling developers to build, ship, and run applications in isolated, portable containers.

### Latest Features (2025)

**Docker Compose:**
- `version:` field is now obsolete - start directly with `services:`
- `docker compose watch` for live reloads
- Bake promoted as default build tool
- `--resolve-image-digests` option for sealing service images

**Security:**
- Enhanced vulnerability scanning with Docker Scout
- Improved secrets management
- Non-root user best practices

**Performance:**
- BuildKit caching improvements
- Multi-stage build optimizations
- Layer caching strategies

## Dockerfile Best Practices (2025)

### Modern Multi-Stage Build
```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files first (layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:20-alpine AS production

# Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only necessary files from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs package*.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js

# Start application
CMD ["node", "dist/index.js"]
```

### PHP Application Dockerfile
```dockerfile
# Build stage
FROM composer:2 AS composer

WORKDIR /app

COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

COPY . .
RUN composer dump-autoload --optimize --classmap-authoritative

# Production stage
FROM php:8.4-fpm-alpine AS production

# Install system dependencies
RUN apk add --no-cache \
    nginx \
    supervisor \
    postgresql-dev \
    icu-dev \
    libzip-dev \
    && docker-php-ext-install \
    pdo_pgsql \
    pdo_mysql \
    intl \
    zip \
    opcache

# Configure PHP for production
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

COPY docker/php/opcache.ini $PHP_INI_DIR/conf.d/
COPY docker/php/php.ini $PHP_INI_DIR/conf.d/

# Create non-root user
RUN addgroup -g 1000 www && \
    adduser -D -u 1000 -G www www

WORKDIR /var/www/html

# Copy application files
COPY --from=composer --chown=www:www /app ./
COPY --chown=www:www . .

# Copy configuration files
COPY docker/nginx/nginx.conf /etc/nginx/nginx.conf
COPY docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

USER www

EXPOSE 8080

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

### Security Best Practices
```dockerfile
# ✅ GOOD - Security-focused Dockerfile
FROM node:20-alpine

# 1. Use specific version tags, not 'latest'
# 2. Use minimal base images (alpine)

# 3. Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# 4. Copy files with proper ownership
COPY --chown=nodejs:nodejs package*.json ./

RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# 5. Switch to non-root user before CMD
USER nodejs

# 6. Use read-only filesystem where possible
# Add at runtime: docker run --read-only

# 7. Drop capabilities
# Add at runtime: docker run --cap-drop=ALL

# 8. No secrets in image
# Use Docker secrets or environment variables

EXPOSE 3000

CMD ["node", "index.js"]
```

### Optimization Techniques
```dockerfile
# Layer caching optimization
FROM node:20-alpine

WORKDIR /app

# ✅ GOOD - Dependencies change less frequently than source
COPY package*.json ./
RUN npm ci --only=production

# Copy source code AFTER installing dependencies
COPY . .

# ❌ BAD - Invalidates cache on any file change
# COPY . .
# RUN npm ci

# Use .dockerignore to exclude unnecessary files
# .dockerignore:
# node_modules
# npm-debug.log
# .git
# .env
# *.md
```

## Docker Compose (2025)

### Modern Compose File
```yaml
# No version field needed in 2025
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
      cache_from:
        - myapp:latest
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./data:/app/data
      - app_logs:/var/log
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

  db:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - app_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/usr/share/nginx/html
    depends_on:
      - app
    networks:
      - app_network

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  app_logs:
  static_files:

networks:
  app_network:
    driver: bridge
```

### Development vs Production Compose
```yaml
# docker-compose.yml (base)
services:
  app:
    build: .
    environment:
      - NODE_ENV=${NODE_ENV}

# docker-compose.dev.yml (development overrides)
services:
  app:
    build:
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev
    environment:
      - DEBUG=*

# docker-compose.prod.yml (production overrides)
services:
  app:
    build:
      target: production
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
```

```bash
# Use development configuration
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Use production configuration
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Docker Compose Watch (2025)
```yaml
# Enable live reload for development
services:
  app:
    build: .
    develop:
      watch:
        - path: ./src
          action: sync
          target: /app/src
        - path: ./package.json
          action: rebuild
        - path: ./node_modules
          action: sync+restart
          target: /app/node_modules
```

```bash
# Run with watch mode
docker compose watch
```

## Docker Commands Reference

### Building Images
```bash
# Build image
docker build -t myapp:latest .

# Build with BuildKit (default in 2025)
DOCKER_BUILDKIT=1 docker build -t myapp:latest .

# Build with build arguments
docker build \
  --build-arg NODE_ENV=production \
  --build-arg API_URL=https://api.example.com \
  -t myapp:latest .

# Build specific stage
docker build --target production -t myapp:prod .

# Build with cache from registry
docker build \
  --cache-from myapp:latest \
  -t myapp:latest .

# Multi-platform build
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push .
```

### Running Containers
```bash
# Run container
docker run -d \
  --name myapp \
  -p 3000:3000 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  myapp:latest

# Run with resource limits
docker run -d \
  --name myapp \
  --cpus="1.5" \
  --memory="512m" \
  --memory-swap="1g" \
  myapp:latest

# Run as non-root user
docker run -d \
  --name myapp \
  --user 1000:1000 \
  myapp:latest

# Run with read-only filesystem
docker run -d \
  --name myapp \
  --read-only \
  --tmpfs /tmp \
  myapp:latest

# Run with security options
docker run -d \
  --name myapp \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp:latest

# Run with health check
docker run -d \
  --name myapp \
  --health-cmd="curl -f http://localhost:3000/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  myapp:latest

# Interactive container
docker run -it --rm node:20-alpine sh

# Execute command in running container
docker exec -it myapp sh
docker exec myapp npm run migrate
```

### Managing Containers
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop myapp

# Start container
docker start myapp

# Restart container
docker restart myapp

# Remove container
docker rm myapp

# Remove all stopped containers
docker container prune

# View logs
docker logs myapp
docker logs -f myapp  # Follow
docker logs --tail 100 myapp  # Last 100 lines

# View container stats
docker stats myapp

# Inspect container
docker inspect myapp

# View container processes
docker top myapp
```

### Images
```bash
# List images
docker images

# Remove image
docker rmi myapp:latest

# Remove unused images
docker image prune

# Remove all unused images
docker image prune -a

# Tag image
docker tag myapp:latest myapp:v1.0.0

# Push to registry
docker push myapp:latest

# Pull from registry
docker pull myapp:latest

# Save image to tar
docker save myapp:latest > myapp.tar

# Load image from tar
docker load < myapp.tar

# Scan image for vulnerabilities (Docker Scout)
docker scout cves myapp:latest
```

### Volumes
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune

# Run with volume
docker run -d \
  --name myapp \
  -v mydata:/app/data \
  myapp:latest

# Bind mount (development)
docker run -d \
  --name myapp \
  -v $(pwd):/app \
  -v /app/node_modules \
  myapp:latest
```

### Networks
```bash
# Create network
docker network create mynetwork

# List networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork myapp

# Disconnect container
docker network disconnect mynetwork myapp

# Remove network
docker network rm mynetwork

# Run with custom network
docker run -d \
  --name myapp \
  --network mynetwork \
  myapp:latest
```

## Security Best Practices (2025)

### 1. Use Official Images with Specific Tags
```bash
# ✅ GOOD
FROM node:20.10.0-alpine

# ❌ BAD
FROM node
FROM node:latest
```

### 2. Scan for Vulnerabilities
```bash
# Docker Scout (2025)
docker scout cves myapp:latest

# Trivy
trivy image myapp:latest

# Snyk
snyk container test myapp:latest
```

### 3. Use Multi-Stage Builds
```dockerfile
# Smaller final image, no build dependencies
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app

FROM alpine:3.19
COPY --from=builder /app/app /app
CMD ["/app"]
```

### 4. Don't Run as Root
```dockerfile
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001
USER appuser
```

### 5. Use Secrets Properly
```bash
# ✅ GOOD - Use Docker secrets
docker run -d \
  --name myapp \
  -e DATABASE_URL=$(cat /run/secrets/db_url) \
  myapp:latest

# Docker Compose with secrets
services:
  app:
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt

# ❌ BAD - Hardcoded in Dockerfile
ENV DATABASE_PASSWORD=secret123
```

### 6. Minimize Attack Surface
```dockerfile
# Use minimal base images
FROM alpine:3.19
# or
FROM scratch  # For static binaries

# Remove unnecessary packages
RUN apk add --no-cache package && \
    rm -rf /var/cache/apk/*
```

### 7. Sign and Verify Images
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Sign image
docker trust sign myapp:latest

# Verify signature
docker trust inspect myapp:latest
```

## Production Best Practices

### Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

```yaml
# In Compose
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Logging
```bash
# JSON logging driver
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp:latest

# Syslog
docker run -d \
  --log-driver=syslog \
  --log-opt syslog-address=tcp://logs.example.com:514 \
  myapp:latest
```

### Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
```

### Restart Policies
```yaml
services:
  app:
    restart: unless-stopped  # Recommended for production
    # restart: always         # Always restart
    # restart: on-failure     # Only on failure
    # restart: no             # Never restart (default)
```

## Optimization Tips

### Layer Caching
```dockerfile
# ✅ GOOD - Leverage layer caching
COPY package*.json ./
RUN npm ci
COPY . .

# ❌ BAD - Invalidates cache on any change
COPY . .
RUN npm ci
```

### .dockerignore
```.dockerignore
# Exclude from build context
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
*.md
.vscode
.idea
coverage
dist
build
*.log
.DS_Store
```

### BuildKit Features (2025)
```dockerfile
# syntax=docker/dockerfile:1

# Build secrets (not in final image)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci

# Cache mounts
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Bind mounts
RUN --mount=type=bind,source=package.json,target=package.json \
    npm install
```

## Troubleshooting

### Common Issues
```bash
# Container exits immediately
docker logs myapp
docker inspect myapp

# Container not accessible
docker port myapp
docker network inspect bridge

# Disk space issues
docker system df
docker system prune -a --volumes

# View container resource usage
docker stats

# Debug inside container
docker exec -it myapp sh

# View container processes
docker top myapp

# Copy files from/to container
docker cp myapp:/app/logs ./logs
docker cp ./config.json myapp:/app/config.json
```

## Best Practices Summary

1. **Use specific image tags** - Not 'latest'
2. **Run as non-root user** - Security best practice
3. **Multi-stage builds** - Smaller, more secure images
4. **Health checks** - Monitoring container health
5. **Resource limits** - Prevent resource exhaustion
6. **Secrets management** - Never hardcode secrets
7. **Scan for vulnerabilities** - Docker Scout, Trivy, Snyk
8. **Layer caching** - Faster builds
9. **Minimal base images** - Alpine, distroless, or scratch
10. **Log aggregation** - Centralized logging

## When Helping Users

1. **Ask about use case** - Development, production, CI/CD
2. **Recommend multi-stage builds** - For optimized images
3. **Emphasize security** - Non-root, scanning, secrets
4. **Suggest proper networking** - Custom networks, not default bridge
5. **Implement health checks** - For production deployments
6. **Use Docker Compose** - For multi-container applications
7. **Resource management** - Set limits and reservations
8. **Follow 2025 best practices** - Modern Docker features

Your goal is to help users build secure, efficient, and maintainable containerized applications using Docker's latest 2025 features and industry best practices.
