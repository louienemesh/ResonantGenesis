# Docker Deployment Guide

Complete guide to Docker deployment, Docker Compose, and containerization best practices for ResonantGenesis.

## Overview

ResonantGenesis provides official Docker images for easy deployment and development. This guide covers Docker setup, Docker Compose configurations, and best practices for running ResonantGenesis in containers.

## Docker Images

### Official Images

| Image | Description | Size |
|-------|-------------|------|
| `resonantgenesis/api` | API server | ~500MB |
| `resonantgenesis/worker` | Background worker | ~450MB |
| `resonantgenesis/session` | Session handler | ~480MB |
| `resonantgenesis/all-in-one` | Complete stack | ~800MB |

### Image Tags

| Tag | Description |
|-----|-------------|
| `latest` | Latest stable release |
| `1.0.0` | Specific version |
| `1.0` | Latest patch of minor version |
| `main` | Latest development build |
| `sha-abc123` | Specific commit |

## Quick Start

### Pull Images

```bash
# Pull latest images
docker pull resonantgenesis/api:latest
docker pull resonantgenesis/worker:latest
docker pull resonantgenesis/session:latest
```

### Run Single Container

```bash
# Run API server
docker run -d \
  --name resonant-api \
  -p 8000:8000 \
  -e API_KEY=your_api_key \
  -e DATABASE_URL=postgresql://user:pass@host:5432/db \
  -e REDIS_URL=redis://host:6379 \
  resonantgenesis/api:latest
```

### All-in-One Development

```bash
# Run complete stack for development
docker run -d \
  --name resonant-dev \
  -p 8000:8000 \
  -p 5432:5432 \
  -p 6379:6379 \
  -v resonant-data:/data \
  resonantgenesis/all-in-one:latest
```

## Docker Compose

### Basic Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    image: resonantgenesis/api:latest
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://resonant:password@postgres:5432/resonant
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://resonant:password@rabbitmq:5672
      - API_KEY=${API_KEY}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    restart: unless-stopped

  worker:
    image: resonantgenesis/worker:latest
    environment:
      - DATABASE_URL=postgresql://resonant:password@postgres:5432/resonant
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://resonant:password@rabbitmq:5672
    depends_on:
      - api
    restart: unless-stopped
    deploy:
      replicas: 3

  session:
    image: resonantgenesis/session:latest
    environment:
      - DATABASE_URL=postgresql://resonant:password@postgres:5432/resonant
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://resonant:password@rabbitmq:5672
    depends_on:
      - api
    restart: unless-stopped
    deploy:
      replicas: 5

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_USER=resonant
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=resonant
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U resonant"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  rabbitmq:
    image: rabbitmq:3-management
    environment:
      - RABBITMQ_DEFAULT_USER=resonant
      - RABBITMQ_DEFAULT_PASS=password
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "check_running"]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
  rabbitmq-data:
```

### Production Setup

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  api:
    image: resonantgenesis/api:1.0.0
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - RABBITMQ_URL=${RABBITMQ_URL}
      - API_KEY=${API_KEY}
      - LOG_LEVEL=INFO
      - ENVIRONMENT=production
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  worker:
    image: resonantgenesis/worker:1.0.0
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - RABBITMQ_URL=${RABBITMQ_URL}
      - LOG_LEVEL=INFO
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 1G
      restart_policy:
        condition: on-failure

  session:
    image: resonantgenesis/session:1.0.0
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - RABBITMQ_URL=${RABBITMQ_URL}
      - LOG_LEVEL=INFO
    deploy:
      replicas: 10
      resources:
        limits:
          cpus: '8'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 2G
      restart_policy:
        condition: on-failure

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - api
    deploy:
      replicas: 2
    restart: unless-stopped
```

### Development Setup

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - .:/app
      - /app/__pycache__
    environment:
      - DATABASE_URL=postgresql://resonant:password@postgres:5432/resonant
      - REDIS_URL=redis://redis:6379
      - DEBUG=true
      - RELOAD=true
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=resonant
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=resonant
    volumes:
      - postgres-dev:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  postgres-dev:
```

## Dockerfile

### API Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim as builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# Production image
FROM python:3.11-slim

WORKDIR /app

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -r resonant && useradd -r -g resonant resonant

# Copy wheels and install
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*

# Copy application
COPY --chown=resonant:resonant . .

# Switch to non-root user
USER resonant

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Multi-Stage Build

```dockerfile
# Dockerfile.multistage
# Build stage
FROM python:3.11-slim as build

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

COPY . .

# Test stage
FROM build as test

RUN pip install --user pytest pytest-cov
RUN pytest tests/ --cov=app

# Production stage
FROM python:3.11-slim as production

WORKDIR /app

RUN apt-get update && apt-get install -y \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r resonant && useradd -r -g resonant resonant

COPY --from=build /root/.local /home/resonant/.local
COPY --from=build /app .

ENV PATH=/home/resonant/.local/bin:$PATH

USER resonant

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Environment Variables

### Required Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `API_KEY` | Platform API key |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_LEVEL` | INFO | Logging level |
| `LOG_FORMAT` | json | Log format |
| `WORKERS` | 4 | Number of workers |
| `TIMEOUT` | 60 | Request timeout |
| `MAX_CONNECTIONS` | 100 | Max DB connections |

### Environment File

```bash
# .env
DATABASE_URL=postgresql://resonant:password@localhost:5432/resonant
REDIS_URL=redis://localhost:6379
RABBITMQ_URL=amqp://resonant:password@localhost:5672
API_KEY=your_api_key_here
LOG_LEVEL=INFO
ENVIRONMENT=production
```

## Volume Management

### Named Volumes

```yaml
volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  uploads:
    driver: local
```

### Bind Mounts

```yaml
services:
  api:
    volumes:
      - ./config:/app/config:ro
      - ./logs:/app/logs
      - ./uploads:/app/uploads
```

### Volume Backup

```bash
# Backup PostgreSQL data
docker run --rm \
  -v resonant_postgres-data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/postgres-$(date +%Y%m%d).tar.gz /data

# Restore
docker run --rm \
  -v resonant_postgres-data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar xzf /backup/postgres-20260221.tar.gz -C /
```

## Networking

### Custom Network

```yaml
networks:
  resonant-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16

services:
  api:
    networks:
      resonant-net:
        ipv4_address: 172.28.0.10
```

### External Network

```yaml
networks:
  external-net:
    external: true

services:
  api:
    networks:
      - external-net
      - default
```

## Health Checks

### Container Health Check

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Dependency Health

```yaml
services:
  api:
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
```

## Logging

### JSON Logging

```yaml
services:
  api:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
        labels: "service,environment"
```

### Fluentd Logging

```yaml
services:
  api:
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: resonant.api
```

### Loki Logging

```yaml
services:
  api:
    logging:
      driver: loki
      options:
        loki-url: "http://loki:3100/loki/api/v1/push"
        loki-batch-size: "400"
```

## Security

### Non-Root User

```dockerfile
RUN groupadd -r resonant && useradd -r -g resonant resonant
USER resonant
```

### Read-Only Filesystem

```yaml
services:
  api:
    read_only: true
    tmpfs:
      - /tmp
      - /app/cache
```

### Security Options

```yaml
services:
  api:
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

## Commands

### Basic Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f api

# Scale service
docker-compose up -d --scale worker=5

# Rebuild images
docker-compose build --no-cache

# Execute command in container
docker-compose exec api python manage.py migrate
```

### Maintenance Commands

```bash
# Prune unused resources
docker system prune -a

# View resource usage
docker stats

# Inspect container
docker inspect resonant-api

# Copy files
docker cp resonant-api:/app/logs ./logs
```

## Best Practices

### For Images

1. **Use specific tags** - Avoid `latest` in production
2. **Multi-stage builds** - Reduce image size
3. **Non-root user** - Security best practice
4. **Health checks** - Enable container health monitoring
5. **Minimize layers** - Combine RUN commands

### For Compose

1. **Use .env files** - Externalize configuration
2. **Named volumes** - Persist data properly
3. **Depends_on with condition** - Proper startup order
4. **Resource limits** - Prevent resource exhaustion
5. **Restart policies** - Handle failures

### For Production

1. **Use secrets** - Docker secrets for sensitive data
2. **Enable logging** - Centralized log collection
3. **Monitor containers** - Resource and health monitoring
4. **Regular updates** - Keep images updated
5. **Backup volumes** - Regular data backups

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs resonant-api

# Check container status
docker ps -a

# Inspect container
docker inspect resonant-api
```

### Network Issues

```bash
# List networks
docker network ls

# Inspect network
docker network inspect resonant_default

# Test connectivity
docker exec resonant-api ping postgres
```

### Performance Issues

```bash
# Check resource usage
docker stats

# Check disk usage
docker system df

# Check container processes
docker top resonant-api
```

---

**Need Docker help?** Contact infrastructure@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
