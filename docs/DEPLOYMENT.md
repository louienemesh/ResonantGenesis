# Deployment Guide

Complete guide for deploying ResonantGenesis to production.

## Deployment Options

| Option | Best For | Complexity |
|--------|----------|------------|
| Cloud Hosted | Most users | None |
| Self-Hosted | Enterprise, privacy | Medium |
| Hybrid | Custom requirements | High |

## Cloud Hosted (Recommended)

Use our managed service at [resonantgenesis.xyz](https://resonantgenesis.xyz). No infrastructure setup required.

## Self-Hosted Deployment

### Prerequisites

- **Server**: 4+ CPU cores, 16GB+ RAM, 100GB+ SSD
- **OS**: Ubuntu 22.04 LTS (recommended)
- **Docker**: 24.0+
- **Docker Compose**: 2.20+
- **Domain**: With DNS configured
- **SSL Certificate**: Let's Encrypt or custom

### Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Nginx (SSL/Proxy)                     │
├─────────────────────────────────────────────────────────┤
│  /api/*  →  FastAPI Backend (8000)                      │
│  /api/v1/node/*  →  Blockchain Node (8081)              │
│  /*  →  Static Frontend                                  │
└─────────────────────────────────────────────────────────┘
         │              │              │
    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
    │ Backend │    │  Node   │    │Frontend │
    │ FastAPI │    │ Rust    │    │ React   │
    └────┬────┘    └─────────┘    └─────────┘
         │
    ┌────┴────┐    ┌─────────┐
    │PostgreSQL│    │  Redis  │
    └─────────┘    └─────────┘
```

### Step 1: Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version
```

### Step 2: Clone Repository

```bash
# Create deployment directory
sudo mkdir -p /opt/resonantgenesis
sudo chown $USER:$USER /opt/resonantgenesis
cd /opt/resonantgenesis

# Clone repository
git clone https://github.com/louienemesh/ResonantGenesis.git .
```

### Step 3: Configure Environment

```bash
# Copy environment templates
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit backend configuration
nano backend/.env
```

**Backend `.env` configuration:**

```bash
# Database
DATABASE_URL=postgresql://resonant:your_secure_password@postgres:5432/resonantgenesis
REDIS_URL=redis://redis:6379

# Security
JWT_SECRET=generate_a_secure_random_string_here
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# API Keys (optional - users can bring their own)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Blockchain
BASE_RPC_URL=https://mainnet.base.org
BASE_SEPOLIA_RPC_URL=https://sepolia.base.org

# Email (optional)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASSWORD=your_smtp_password

# Storage
UPLOAD_DIR=/app/uploads
MAX_UPLOAD_SIZE=10485760
```

**Frontend `.env` configuration:**

```bash
VITE_API_URL=https://your-domain.com/api/v1
VITE_NODE_URL=https://your-domain.com/api/v1/node
VITE_WS_URL=wss://your-domain.com/ws
```

### Step 4: Docker Compose Setup

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: resonant
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: resonantgenesis
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U resonant"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: always
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    restart: always
    env_file:
      - ./backend/.env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - uploads:/app/uploads
    expose:
      - "8000"

  node:
    build:
      context: ./node
      dockerfile: Dockerfile
    restart: always
    env_file:
      - ./node/.env
    expose:
      - "8081"

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
      args:
        VITE_API_URL: ${VITE_API_URL}
        VITE_NODE_URL: ${VITE_NODE_URL}
    restart: always
    volumes:
      - frontend_dist:/app/dist

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - frontend_dist:/var/www/frontend:ro
      - certbot_www:/var/www/certbot:ro
    depends_on:
      - backend
      - node
      - frontend

volumes:
  postgres_data:
  redis_data:
  uploads:
  frontend_dist:
  certbot_www:
```

### Step 5: Nginx Configuration

Create `nginx/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=60r/m;

    # Upstream servers
    upstream backend {
        server backend:8000;
    }

    upstream node {
        server node:8081;
    }

    # HTTP redirect to HTTPS
    server {
        listen 80;
        server_name your-domain.com;

        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    # HTTPS server
    server {
        listen 443 ssl http2;
        server_name your-domain.com;

        # SSL certificates
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        # SSL settings
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Strict-Transport-Security "max-age=31536000" always;

        # API routes
        location /api/v1/node/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://node/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://backend/api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # WebSocket
        location /ws {
            proxy_pass http://backend/ws;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
        }

        # Frontend static files
        location / {
            root /var/www/frontend;
            try_files $uri $uri/ /index.html;

            # Cache static assets
            location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
                expires 1y;
                add_header Cache-Control "public, immutable";
            }
        }
    }
}
```

### Step 6: SSL Certificate

**Option A: Let's Encrypt (recommended)**

```bash
# Install certbot
sudo apt install certbot

# Get certificate
sudo certbot certonly --webroot -w /var/www/certbot -d your-domain.com

# Copy certificates
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem nginx/ssl/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem nginx/ssl/

# Auto-renewal cron job
echo "0 0 * * * certbot renew --quiet && docker-compose restart nginx" | sudo crontab -
```

**Option B: Custom certificate**

```bash
# Copy your certificates
cp your-cert.pem nginx/ssl/fullchain.pem
cp your-key.pem nginx/ssl/privkey.pem
```

### Step 7: Database Migration

```bash
# Start database first
docker-compose -f docker-compose.prod.yml up -d postgres redis

# Wait for healthy status
docker-compose -f docker-compose.prod.yml ps

# Run migrations
docker-compose -f docker-compose.prod.yml run --rm backend alembic upgrade head
```

### Step 8: Deploy

```bash
# Build and start all services
docker-compose -f docker-compose.prod.yml up -d --build

# Check status
docker-compose -f docker-compose.prod.yml ps

# View logs
docker-compose -f docker-compose.prod.yml logs -f
```

### Step 9: Verify Deployment

```bash
# Check API health
curl https://your-domain.com/api/v1/health

# Check node status
curl https://your-domain.com/api/v1/node/status

# Check frontend
curl -I https://your-domain.com
```

## Maintenance

### Updating

```bash
# Pull latest changes
git pull origin main

# Rebuild and restart
docker-compose -f docker-compose.prod.yml up -d --build

# Run any new migrations
docker-compose -f docker-compose.prod.yml run --rm backend alembic upgrade head
```

### Backups

```bash
# Database backup
docker-compose -f docker-compose.prod.yml exec postgres pg_dump -U resonant resonantgenesis > backup.sql

# Restore
cat backup.sql | docker-compose -f docker-compose.prod.yml exec -T postgres psql -U resonant resonantgenesis
```

### Monitoring

```bash
# View logs
docker-compose -f docker-compose.prod.yml logs -f backend

# Resource usage
docker stats

# Health checks
curl https://your-domain.com/api/v1/health
```

### Scaling

For high availability, consider:

1. **Load Balancer**: AWS ALB, Cloudflare, nginx upstream
2. **Database**: Managed PostgreSQL (RDS, Cloud SQL)
3. **Cache**: Managed Redis (ElastiCache, Memorystore)
4. **CDN**: Cloudflare, CloudFront for static assets

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose -f docker-compose.prod.yml logs backend

# Check configuration
docker-compose -f docker-compose.prod.yml config
```

### Database connection failed

```bash
# Verify database is running
docker-compose -f docker-compose.prod.yml ps postgres

# Check connection
docker-compose -f docker-compose.prod.yml exec postgres psql -U resonant -c "SELECT 1"
```

### SSL certificate issues

```bash
# Verify certificate
openssl s_client -connect your-domain.com:443 -servername your-domain.com

# Check nginx config
docker-compose -f docker-compose.prod.yml exec nginx nginx -t
```

## Security Checklist

- [ ] Strong database password
- [ ] Unique JWT secret
- [ ] SSL/TLS enabled
- [ ] Firewall configured (ports 80, 443 only)
- [ ] Regular backups enabled
- [ ] Log monitoring configured
- [ ] Rate limiting enabled
- [ ] Security headers configured

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
