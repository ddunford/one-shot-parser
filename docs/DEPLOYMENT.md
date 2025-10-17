# Deployment Strategy - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## Deployment Overview

**Target Environment**: Local development server
**URL**: https://cvparse.demosrv.uk
**Infrastructure**: Docker + Traefik reverse proxy
**Deployment Method**: Manual (docker compose)

---

## Infrastructure Architecture

```
Internet (HTTPS)
      │
      ▼
┌─────────────────────────────┐
│  Traefik Reverse Proxy      │
│  (External - traefik_       │
│   demosrv network)          │
│  - Let's Encrypt certs      │
│  - Load balancing           │
│  - Health checks            │
└─────────────────────────────┘
      │
      ├─────────────────┬──────────────────┐
      ▼                 ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Frontend    │  │  Backend     │  │  Ollama      │
│  (React)     │  │  (Express)   │  │  (External)  │
│  Port: 3000  │  │  Port: 3001  │  │  :11434      │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Key Components**:
- **Traefik**: Already running externally, handles HTTPS/routing
- **Frontend**: React SPA served via Vite dev server (dev) or nginx (prod)
- **Backend**: Express API with file upload handling
- **Ollama**: External LLM service at inference.lan:11434

---

## Environment Stages

### Development (Local)

**Purpose**: Hot reload, fast iteration

**URL**: https://cvparse.demosrv.uk

**Characteristics**:
- Volume mounts for hot reload
- Development build (non-minified)
- Debug logging enabled
- API keys from .env.local
- No resource limits

**Start Command**:
```bash
docker compose up
```

---

### Production (Same Server)

**Purpose**: Optimized, stable release

**URL**: https://cvparse.demosrv.uk

**Characteristics**:
- Production builds (minified, optimized)
- No volume mounts (compiled code)
- Resource limits applied
- Health checks enabled
- Error logging only

**Start Command**:
```bash
docker compose -f docker-compose.prod.yml up -d
```

---

## Docker Configuration

### docker-compose.yml (Development)

```yaml
version: '3.9'

name: cvparse-dev

services:
  backend:
    build:
      context: ./src/backend
      dockerfile: Dockerfile
      target: development
    container_name: cvparse-backend-dev
    init: true
    stop_signal: SIGTERM
    volumes:
      - ./src/backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=3001
      - OLLAMA_URL=http://inference.lan:11434
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - AWS_REGION=${AWS_REGION:-us-east-1}
    ports:
      - "3001:3001"
    networks:
      - traefik_demosrv
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cvparse-api.rule=Host(`cvparse.demosrv.uk`) && PathPrefix(`/api`)"
      - "traefik.http.routers.cvparse-api.entrypoints=websecure"
      - "traefik.http.routers.cvparse-api.tls.certresolver=letsencrypt"
      - "traefik.http.services.cvparse-api.loadbalancer.server.port=3001"
    healthcheck:
      test: ["CMD", "wget", "--spider", "--quiet", "http://localhost:3001/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 40s

  frontend:
    build:
      context: ./src/frontend
      dockerfile: Dockerfile
      target: development
    container_name: cvparse-frontend-dev
    init: true
    stop_signal: SIGTERM
    volumes:
      - ./src/frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - VITE_API_URL=https://cvparse.demosrv.uk/api
    networks:
      - traefik_demosrv
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cvparse.rule=Host(`cvparse.demosrv.uk`)"
      - "traefik.http.routers.cvparse.entrypoints=websecure"
      - "traefik.http.routers.cvparse.tls.certresolver=letsencrypt"
      - "traefik.http.services.cvparse.loadbalancer.server.port=3000"
    healthcheck:
      test: ["CMD", "wget", "--spider", "--quiet", "http://localhost:3000"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 40s

networks:
  traefik_demosrv:
    external: true
```

---

### Dockerfiles

#### Backend Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# ===========================================
# Stage 1: Development
# ===========================================
FROM node:20-alpine AS development

WORKDIR /app

RUN apk add --no-cache git curl

COPY package.json package-lock.json ./
RUN npm install

COPY . .

EXPOSE 3001

CMD ["npm", "run", "dev"]

# ===========================================
# Stage 2: Production Dependencies
# ===========================================
FROM node:20-alpine AS deps

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --only=production && npm cache clean --force

# ===========================================
# Stage 3: Builder
# ===========================================
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# ===========================================
# Stage 4: Production
# ===========================================
FROM node:20-alpine AS production

RUN apk upgrade --no-cache && \
    addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
  CMD wget --spider --quiet http://localhost:3001/api/health || exit 1

CMD ["node", "dist/index.js"]
```

#### Frontend Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# ===========================================
# Stage 1: Development
# ===========================================
FROM node:20-alpine AS development

WORKDIR /app

RUN apk add --no-cache git curl

COPY package.json package-lock.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]

# ===========================================
# Stage 2: Builder
# ===========================================
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# ===========================================
# Stage 3: Production
# ===========================================
FROM nginx:alpine AS production

COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget --spider --quiet http://localhost:80 || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

---

## Deployment Workflow

### Initial Setup

1. **Clone Repository**:
```bash
git clone <repo-url> /opt/workspaces/cvparse.demosrv.uk
cd /opt/workspaces/cvparse.demosrv.uk
```

2. **Configure Environment**:
```bash
cp .env.example .env
nano .env  # Add API keys
```

3. **Verify Traefik**:
```bash
docker ps | grep traefik
# Should show traefik container running
```

4. **Start Services**:
```bash
docker compose up -d
```

5. **Verify Deployment**:
```bash
curl https://cvparse.demosrv.uk/api/health
```

---

### Update Deployment

```bash
# Pull latest code
cd /opt/workspaces/cvparse.demosrv.uk
git pull origin main

# Rebuild and restart
docker compose down
docker compose up -d --build

# Verify
curl https://cvparse.demosrv.uk/api/health
```

---

### Rollback

```bash
# Revert to previous commit
git log --oneline  # Find commit hash
git checkout <previous-commit-hash>

# Rebuild
docker compose up -d --build
```

---

## Environment Variables

### .env File

```bash
# LLM Providers
OLLAMA_URL=http://inference.lan:11434
OPENAI_API_KEY=sk-...
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1

# Application
NODE_ENV=development
PORT=3001
LOG_LEVEL=info

# Security
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX=10
FILE_SIZE_LIMIT=5242880  # 5MB

# Frontend
VITE_API_URL=https://cvparse.demosrv.uk/api
```

**Security**:
- Never commit .env to Git
- Use `.env.example` as template
- Production: Use Docker secrets or external secret management

---

## Monitoring & Health Checks

### Health Check Endpoint

**URL**: https://cvparse.demosrv.uk/api/health

**Response**:
```json
{
  "status": "healthy",
  "timestamp": "2025-10-16T14:30:00Z",
  "uptime": 86400,
  "providers": {
    "ollama": { "status": "online", "latency": 120 },
    "openai": { "status": "unconfigured" },
    "bedrock": { "status": "unconfigured" }
  }
}
```

### Monitoring Checks

**Automated Checks** (every 30s):
- Container health (Docker healthchecks)
- API responsiveness (health endpoint)
- LLM provider availability

**Manual Checks**:
```bash
# Check containers
docker ps

# Check logs
docker compose logs -f backend
docker compose logs -f frontend

# Check resource usage
docker stats

# Check Traefik routing
curl -H "Host: cvparse.demosrv.uk" https://cvparse.demosrv.uk/api/health
```

---

## Backup & Recovery

### What to Backup

1. **Configuration**: .env, docker-compose.yml
2. **Custom Scripts**: deployment scripts
3. **Documentation**: CLAUDE.md, .autoflow/

**No Database**: No data backups needed (stateless)

### Backup Command

```bash
# Backup configuration
tar -czf cvparse-config-$(date +%Y%m%d).tar.gz \
  .env docker-compose.yml CLAUDE.md .autoflow/
```

### Disaster Recovery

1. **Complete Server Failure**:
   - Provision new server
   - Install Docker + Docker Compose
   - Restore configuration from backup
   - Clone repository
   - Start services: `docker compose up -d`

2. **Service Crash**:
   - View logs: `docker compose logs backend`
   - Restart service: `docker compose restart backend`
   - If persistent: rebuild `docker compose up -d --build`

**Recovery Time Objective (RTO)**: <30 minutes

**Recovery Point Objective (RPO)**: N/A (stateless, no data loss)

---

## Scaling Strategy

### Vertical Scaling (Current)

**Current Resources**:
- 2 CPU cores
- 4GB RAM
- Suitable for <100 concurrent users

**Upgrade Path**:
- 4 CPU cores
- 8GB RAM
- Suitable for <500 concurrent users

### Horizontal Scaling (Future)

**Strategy**: Multiple backend instances behind Traefik

```yaml
services:
  backend:
    # ... config ...
    deploy:
      replicas: 3
```

**Command**:
```bash
docker compose up -d --scale backend=3
```

**Load Balancing**: Traefik handles automatically (round-robin)

**Stateless Benefit**: Trivial to scale horizontally

---

## Troubleshooting

### Common Issues

**1. Container Won't Start**
```bash
# Check logs
docker compose logs backend

# Common causes:
# - Missing .env file
# - Invalid environment variables
# - Port already in use
```

**2. Cannot Connect to API**
```bash
# Verify containers running
docker ps

# Check Traefik routing
docker logs traefik | grep cvparse

# Test directly (bypass Traefik)
curl http://localhost:3001/api/health
```

**3. LLM Provider Unreachable**
```bash
# Test Ollama directly
curl http://inference.lan:11434/api/tags

# Check network connectivity
docker exec cvparse-backend-dev ping inference.lan
```

**4. Hot Reload Not Working**
```bash
# Verify volume mounts
docker inspect cvparse-backend-dev | grep Mounts

# Restart container
docker compose restart backend
```

---

## CI/CD (Future)

### Proposed GitHub Actions Workflow

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v3

      - name: Run tests
        run: npm test

      - name: Build and deploy
        run: |
          docker compose down
          docker compose up -d --build

      - name: Health check
        run: |
          sleep 10
          curl --fail https://cvparse.demosrv.uk/api/health

      - name: Notify
        if: failure()
        run: echo "Deployment failed!"
```

---

## Maintenance Windows

**Scheduled Maintenance**: None required (stateless)

**Updates**: Can be performed anytime with minimal downtime (<2 minutes)

**Zero-Downtime Deployment** (Future):
1. Start new containers
2. Wait for health checks to pass
3. Traefik automatically routes to new containers
4. Stop old containers

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 2
**Owner**: DevOps/Engineering Team
