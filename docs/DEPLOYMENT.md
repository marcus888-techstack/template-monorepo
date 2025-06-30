# Deployment Guide

## Overview
This guide covers deployment strategies for the two-part architecture project, including local development with Docker, staging environments, and production deployment options. The project consists of a landing site, web application, and unified backend API, each requiring specific deployment considerations.

## Table of Contents
1. [Development Environment](#development-environment)
2. [Docker Configuration](#docker-configuration)
3. [Environment Variables](#environment-variables)
4. [Production Deployment](#production-deployment)
5. [CI/CD Pipeline](#cicd-pipeline)
6. [Monitoring & Maintenance](#monitoring--maintenance)

## Development Environment

### Prerequisites
- Docker Desktop installed
- Python 3.11+ for backend
- Node.js 18+ for frontend
- PostgreSQL client tools (optional)

### Local Setup with Docker Compose

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/[project-name].git
cd [project-name]
```

2. **Install monorepo dependencies**
```bash
# Using pnpm (recommended)
pnpm install

# Or using npm workspaces
npm install
```

3. **Build Docker images**
```bash
# Build landing site image
cd apps/landing
docker build -t [project]_landing:latest .

# Build web application image
cd ../web
docker build -t [project]_web:latest .

# Build backend image
cd ../backend
docker build -t [project]_backend:latest .
```

3. **Start services**
```bash
# From project root
docker-compose up -d
```

This starts:
- Nginx reverse proxy (port 80)
  - Routes example.local to landing site
  - Routes app.example.local to web application
- PostgreSQL database (port 5432)
- pgAdmin for database management (port 5050)
- Redis for caching (port 6379)
- FastAPI backend (port 5000)
- Landing site (port 3000)
- Web application (port 5173)

4. **Configure local domains (optional)**
```bash
# Add to /etc/hosts for local subdomain testing
127.0.0.1 example.local
127.0.0.1 app.example.local
```

5. **Initialize the database**
```bash
cd apps/backend
prisma generate
prisma migrate dev
python -m scripts.seed  # Optional: Add sample data
```

## Docker Configuration

### Docker Organization

All Docker configurations are organized under the `docker/` directory:

```
docker/
├── nginx/                  # Nginx reverse proxy config
│   ├── nginx.conf         # Main configuration
│   └── conf.d/           # Site configurations
├── postgres/             # PostgreSQL configs (optional)
└── volumes/              # Persistent data (git-ignored)
    ├── postgres/        # Database files
    ├── redis/          # Cache data
    ├── pgadmin/        # pgAdmin settings
    └── uploads/        # User uploaded files
```

### docker-compose.yml (Runtime Only)
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: ${PROJECT_NAME:-project}_postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-project_db}
      POSTGRES_USER: ${POSTGRES_USER:-project_user}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-project_pass}
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    volumes:
      - ./docker/volumes/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-project_user} -d ${POSTGRES_DB:-project_db}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - project_network

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: ${PROJECT_NAME:-project}_pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-admin@example.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
    ports:
      - "${PGADMIN_PORT:-5050}:80"
    volumes:
      - ./docker/volumes/pgadmin:/var/lib/pgadmin
    depends_on:
      - postgres
    networks:
      - project_network

  redis:
    image: redis:7-alpine
    container_name: ${PROJECT_NAME:-project}_redis
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - ./docker/volumes/redis:/data
    networks:
      - project_network

  backend:
    image: ${PROJECT_NAME:-project}_backend:latest
    container_name: ${PROJECT_NAME:-project}_backend
    ports:
      - "${BACKEND_PORT:-5000}:5000"
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER:-project_user}:${POSTGRES_PASSWORD:-project_pass}@postgres:5432/${POSTGRES_DB:-project_db}
      REDIS_URL: redis://redis:6379
      CORS_ORIGINS: '["http://localhost:3000","http://localhost:5173","http://example.local","http://app.example.local"]'
    volumes:
      - ./apps/backend:/app
      - ./docker/volumes/uploads:/app/uploads
    depends_on:
      - postgres
      - redis
    networks:
      - project_network

  landing:
    image: ${PROJECT_NAME:-project}_landing:latest
    container_name: ${PROJECT_NAME:-project}_landing
    ports:
      - "${LANDING_PORT:-3000}:3000"
    environment:
      VITE_API_URL: http://localhost:${BACKEND_PORT:-5000}/api
      VITE_APP_URL: http://localhost:${WEB_PORT:-5173}
    volumes:
      - ./apps/landing:/app
    depends_on:
      - backend
    networks:
      - project_network

  web:
    image: ${PROJECT_NAME:-project}_web:latest
    container_name: ${PROJECT_NAME:-project}_web
    ports:
      - "${WEB_PORT:-5173}:5173"
    environment:
      VITE_API_URL: http://localhost:${BACKEND_PORT:-5000}/api
      VITE_CLERK_PUBLISHABLE_KEY: ${CLERK_PUBLISHABLE_KEY:-pk_test_Y2xlcmsuZGV2JA}
    volumes:
      - ./apps/web:/app
    depends_on:
      - backend
    networks:
      - project_network

  nginx:
    image: nginx:alpine
    container_name: ${PROJECT_NAME:-project}_nginx
    ports:
      - "${NGINX_PORT:-80}:80"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./docker/nginx/conf.d:/etc/nginx/conf.d:ro
    depends_on:
      - landing
      - web
      - backend
    networks:
      - project_network

networks:
  project_network:
    driver: bridge
```

### Dockerfile - Backend
```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Generate Prisma client
RUN prisma generate

# Create uploads directory
RUN mkdir -p uploads/products

# Expose port
EXPOSE 5000

# Run the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000"]
```

### Dockerfile - Landing Site
```dockerfile
# apps/landing/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Dockerfile - Web Application
```dockerfile
# apps/web/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration

#### Main nginx.conf
```nginx
# docker/nginx/nginx.conf
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip Settings
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss application/rss+xml application/atom+xml image/svg+xml;

    # Include site configurations
    include /etc/nginx/conf.d/*.conf;
}
```

#### Site configurations
```nginx
# docker/nginx/conf.d/landing.conf
server {
    listen 80;
    server_name localhost example.local;

    location / {
        proxy_pass http://landing:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# docker/nginx/conf.d/app.conf
server {
    listen 80;
    server_name app.localhost app.example.local;

    location / {
        proxy_pass http://web:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Environment Variables

### Root (.env)
```env
# Project
PROJECT_NAME=myproject

# Database
POSTGRES_DB=project_db
POSTGRES_USER=project_user
POSTGRES_PASSWORD=secure_password_here

# Services Ports
LANDING_PORT=3000
WEB_PORT=5173
BACKEND_PORT=5000
NGINX_PORT=80

# Clerk (for web app)
CLERK_PUBLISHABLE_KEY=pk_test_your-clerk-key
```

### Backend (.env)
```env
# Database
DATABASE_URL="postgresql://project_user:project_pass@localhost:5432/project_db"

# Server
PORT=5000
ENVIRONMENT=development

# Security
SECRET_KEY="your-secret-key-here"
CORS_ORIGINS=["http://localhost:3000", "http://localhost:5173", "http://example.local", "http://app.example.local"]

# Clerk JWT Verification
CLERK_JWKS_URL="https://your-app.clerk.accounts.dev/.well-known/jwks.json"

# File Upload
UPLOAD_DIR="/app/uploads"
MAX_FILE_SIZE=5242880  # 5MB

# Redis (optional)
REDIS_URL="redis://localhost:6379"

# Email (future)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT=587
SMTP_USER="your-email@gmail.com"
SMTP_PASSWORD="your-app-password"
```

### Landing Site (.env.local)
```env
# API Configuration
VITE_API_URL=http://localhost:5000/api

# Web App URL
VITE_APP_URL=http://localhost:5173

# Features
VITE_ENABLE_ANALYTICS=false
VITE_GA_TRACKING_ID=
```

### Web Application (.env.local)
```env
# API Configuration
VITE_API_URL=http://localhost:5000/api

# Clerk
VITE_CLERK_PUBLISHABLE_KEY=pk_test_your-clerk-key

# Features
VITE_ENABLE_ANALYTICS=false
VITE_ADMIN_EMAIL=admin@example.com
```

### Production Environment Variables
```env
# Additional production settings
NODE_ENV=production
ENVIRONMENT=production

# Database (use connection pooling)
DATABASE_URL="postgresql://user:pass@db-host:5432/project_db?sslmode=require&pool_timeout=0"

# Security
SECRET_KEY="<generate-strong-secret>"
ALLOWED_HOSTS=["yourdomain.com", "www.yourdomain.com", "app.yourdomain.com"]
CORS_ORIGINS=["https://yourdomain.com", "https://app.yourdomain.com"]

# File Storage (S3 or similar)
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"
S3_BUCKET_NAME="project-uploads"
S3_REGION="us-east-1"

# CDN
CDN_URL="https://cdn.yourdomain.com"

# Monitoring
SENTRY_DSN="https://your-sentry-dsn"
SENTRY_ENVIRONMENT="production"
```

## Production Deployment

### Deployment Architecture
- **Landing Site**: Deploy to Vercel/Netlify (optimized for static sites)
- **Web Application**: Deploy to cloud provider (AWS/GCP/Azure)
- **Backend API**: Deploy with web application or separately
- **Database**: Managed PostgreSQL service

### Option 1: Vercel + Railway (Recommended for simplicity)

#### 1. Landing Site Deployment (Vercel)
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy landing site
cd apps/landing
vercel --prod

# Configure custom domain
# In Vercel dashboard: Add yourdomain.com
```

#### 2. Web Application Deployment (Vercel)
```bash
# Deploy web application
cd apps/web
vercel --prod

# Configure subdomain
# In Vercel dashboard: Add app.yourdomain.com
```

#### 3. Backend Deployment (Railway)
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
cd apps/backend
railway init

# Add services
railway add postgresql
railway add redis

# Deploy
railway up

# Get deployment URL and update frontend env vars
```

### Option 2: DigitalOcean App Platform

#### Complete app.yaml
```yaml
name: project-platform
region: nyc
services:
  # Landing Site
  - name: landing
    github:
      repo: yourusername/project-name
      branch: main
      deploy_on_push: true
    source_dir: apps/landing
    environment_slug: node-js
    build_command: npm install && npm run build
    output_dir: dist
    routes:
      - path: /
    domains:
      - domain: yourdomain.com
        type: PRIMARY

  # Web Application
  - name: web
    github:
      repo: yourusername/project-name
      branch: main
      deploy_on_push: true
    source_dir: apps/web
    environment_slug: node-js
    build_command: npm install && npm run build
    output_dir: dist
    envs:
      - key: VITE_API_URL
        scope: BUILD_TIME
        value: "https://app.yourdomain.com/api"
      - key: VITE_CLERK_PUBLISHABLE_KEY
        scope: BUILD_TIME
        value: "your-production-clerk-key"
    routes:
      - path: /
    domains:
      - domain: app.yourdomain.com
        type: ALIAS

  # Backend API
  - name: backend
    github:
      repo: yourusername/project-name
      branch: main
      deploy_on_push: true
    source_dir: apps/backend
    environment_slug: python
    build_command: pip install -r requirements.txt && prisma generate
    run_command: uvicorn main:app --host 0.0.0.0 --port 8080
    http_port: 8080
    instance_count: 2
    instance_size_slug: professional-xs
    envs:
      - key: DATABASE_URL
        scope: RUN_TIME
        value: ${project-db.DATABASE_URL}
      - key: REDIS_URL
        scope: RUN_TIME
        value: ${project-redis.REDIS_URL}
      - key: CORS_ORIGINS
        scope: RUN_TIME
        value: '["https://yourdomain.com","https://app.yourdomain.com"]'
      - key: CLERK_JWKS_URL
        scope: RUN_TIME
        value: "your-production-clerk-jwks-url"
    routes:
      - path: /api

databases:
  - name: project-db
    engine: PG
    version: "15"
    size: db-s-1vcpu-1gb
    num_nodes: 1

  - name: project-redis
    engine: REDIS
    version: "7"
```

### Option 3: AWS Deployment

#### Landing Site (S3 + CloudFront)
```bash
# Build landing site
cd apps/landing
npm run build

# Create S3 bucket for landing
aws s3 mb s3://yourdomain-landing
aws s3 website s3://yourdomain-landing --index-document index.html --error-document index.html

# Sync files
aws s3 sync dist/ s3://yourdomain-landing --delete

# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name yourdomain-landing.s3.amazonaws.com \
  --default-root-object index.html
```

#### Web Application (ECS + ALB)
```bash
# Build and push web app image
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_URI
cd apps/web
docker build -t project-web .
docker tag project-web:latest $ECR_URI/project-web:latest
docker push $ECR_URI/project-web:latest

# Create ECS task definition
aws ecs register-task-definition --cli-input-json file://web-task-definition.json

# Create ECS service
aws ecs create-service \
  --cluster project-cluster \
  --service-name project-web \
  --task-definition project-web:1 \
  --desired-count 2 \
  --launch-type FARGATE
```

#### Backend API (ECS + Fargate)
```bash
# Build and push backend image
cd apps/backend
docker build -t project-backend .
docker tag project-backend:latest $ECR_URI/project-backend:latest
docker push $ECR_URI/project-backend:latest

# Deploy to ECS
aws ecs update-service --cluster project-cluster --service project-backend --force-new-deployment
```

### Option 4: Kubernetes Deployment

#### Namespace and ConfigMap
```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: project-prod
---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: project-config
  namespace: project-prod
data:
  API_URL: "https://app.yourdomain.com/api"
  APP_URL: "https://app.yourdomain.com"
  LANDING_URL: "https://yourdomain.com"
```

#### Landing Deployment
```yaml
# k8s/landing-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: landing
  namespace: project-prod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: landing
  template:
    metadata:
      labels:
        app: landing
    spec:
      containers:
      - name: landing
        image: your-registry/project-landing:latest
        ports:
        - containerPort: 80
        env:
        - name: VITE_API_URL
          valueFrom:
            configMapKeyRef:
              name: project-config
              key: API_URL
---
apiVersion: v1
kind: Service
metadata:
  name: landing-service
  namespace: project-prod
spec:
  selector:
    app: landing
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

#### Web App Deployment
```yaml
# k8s/web-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: project-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: your-registry/project-web:latest
        ports:
        - containerPort: 80
        env:
        - name: VITE_API_URL
          valueFrom:
            configMapKeyRef:
              name: project-config
              key: API_URL
        - name: VITE_CLERK_PUBLISHABLE_KEY
          valueFrom:
            secretKeyRef:
              name: project-secrets
              key: clerk-publishable-key
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: project-prod
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

#### Ingress Configuration
```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: project-ingress
  namespace: project-prod
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - yourdomain.com
    - app.yourdomain.com
    secretName: project-tls
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: landing-service
            port:
              number: 80
  - host: app.yourdomain.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

## CI/CD Pipeline

### GitHub Actions

#### Monorepo CI/CD
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  # Test all applications
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install pnpm
        run: npm install -g pnpm
        
      - name: Install dependencies
        run: pnpm install
        
      - name: Run tests
        run: pnpm test
        
      - name: Build all apps
        run: pnpm build

  # Deploy landing site
  deploy-landing:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        run: |
          cd apps/landing
          npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_LANDING_PROJECT_ID }}

  # Deploy web application
  deploy-web:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        run: |
          cd apps/web
          npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_WEB_PROJECT_ID }}

  # Deploy backend
  deploy-backend:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Test backend
        working-directory: ./apps/backend
        run: |
          pip install -r requirements.txt
          pip install pytest
          pytest
      
      - name: Deploy to Railway
        working-directory: ./apps/backend
        run: |
          npm install -g @railway/cli
          railway up
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

#### Preview Deployments
```yaml
# .github/workflows/preview.yml
name: Preview Deployment

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy Landing Preview
        run: |
          cd apps/landing
          npx vercel --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_LANDING_PROJECT_ID }}
          
      - name: Deploy Web Preview
        run: |
          cd apps/web
          npx vercel --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_WEB_PROJECT_ID }}
          
      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: 'Preview deployments ready! 🚀'
            })
```

## Monitoring & Maintenance

### Health Checks

Backend health endpoint:
```python
@app.get("/health")
async def health_check():
    try:
        # Check database
        await prisma.connect()
        db_status = "healthy"
    except:
        db_status = "unhealthy"
    
    return {
        "status": "healthy" if db_status == "healthy" else "degraded",
        "timestamp": datetime.utcnow(),
        "services": {
            "database": db_status,
            "cache": check_redis_health(),
            "storage": check_storage_health()
        }
    }
```

### Monitoring Setup

1. **Application Monitoring (Sentry)**
```python
import sentry_sdk

sentry_sdk.init(
    dsn=os.getenv("SENTRY_DSN"),
    environment=os.getenv("ENVIRONMENT", "production"),
    traces_sample_rate=0.1
)
```

2. **Uptime Monitoring**
- Use UptimeRobot or Pingdom
- Monitor endpoints:
  - `GET /health` - Application health
  - `GET /api/products` - API functionality
  - Frontend URL - User accessibility

3. **Log Aggregation**
```python
# Configure structured logging
import structlog

logger = structlog.get_logger()

# Log important events
logger.info("order_created", order_id=order.id, user_id=user.id, total=order.total)
```

### Backup Strategy

1. **Database Backups**
```bash
# Daily backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
docker exec bakery_postgres pg_dump -U bakery_user bakery_db > backup_$DATE.sql
tar -czf backup_$DATE.tar.gz backup_$DATE.sql docker/volumes/postgres/
aws s3 cp backup_$DATE.tar.gz s3://bakery-backups/db/
rm backup_$DATE.sql backup_$DATE.tar.gz
```

2. **File Backups**
```bash
# Backup all volumes
tar -czf volumes_backup_$(date +%Y%m%d).tar.gz docker/volumes/
aws s3 cp volumes_backup_*.tar.gz s3://bakery-backups/volumes/

# Sync uploads to S3
aws s3 sync ./docker/volumes/uploads s3://bakery-backups/uploads/ --delete
```

3. **Volume Restore**
```bash
# Restore from backup
tar -xzf volumes_backup_20240101.tar.gz
docker-compose down
docker-compose up -d
```

### Performance Optimization

#### 1. Database Optimization
```sql
-- Indexes for landing site queries
CREATE INDEX idx_items_featured ON items(featured) WHERE featured = true;
CREATE INDEX idx_items_category_featured ON items(category_id, featured);

-- Indexes for web app queries
CREATE INDEX idx_activities_user_status ON activities(user_id, status);
CREATE INDEX idx_collection_items_user ON collection_items(collection_id);

-- Indexes for admin queries
CREATE INDEX idx_activities_created ON activities(created_at DESC);
CREATE INDEX idx_contact_submissions_created ON contact_submissions(created_at DESC);
```

#### 2. Caching Strategy
```python
from functools import lru_cache
import redis
import json

redis_client = redis.from_url(os.getenv("REDIS_URL"))

# Cache for landing site (longer TTL)
@lru_cache(maxsize=100)
async def get_featured_items():
    cache_key = "landing:featured_items"
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    items = await prisma.item.find_many(
        where={"featured": True, "available": True},
        include={"category": True},
        take=6
    )
    # Cache for 6 hours
    redis_client.setex(cache_key, 21600, json.dumps(items))
    return items

# Cache for web app (shorter TTL)
@lru_cache(maxsize=100)
async def get_user_collection(user_id: str):
    cache_key = f"user:{user_id}:collection"
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    collection = await prisma.collection.find_unique(
        where={"userId": user_id},
        include={"items": {"include": {"item": True}}}
    )
    # Cache for 5 minutes
    redis_client.setex(cache_key, 300, json.dumps(collection))
    return collection
```

#### 3. CDN Configuration

**Landing Site CDN Rules:**
```nginx
# Cloudflare Page Rules
*.yourdomain.com/assets/*
  Cache Level: Cache Everything
  Edge Cache TTL: 1 month

*.yourdomain.com/images/*
  Cache Level: Cache Everything
  Edge Cache TTL: 1 week
  Polish: Lossy
  WebP: On
```

**Web App CDN Rules:**
```nginx
# Different caching for app
app.yourdomain.com/assets/*
  Cache Level: Cache Everything
  Edge Cache TTL: 1 week

app.yourdomain.com/api/*
  Cache Level: Bypass
  
app.yourdomain.com/*
  Cache Level: No Query String
  Edge Cache TTL: 1 hour
```

### Maintenance Tasks

1. **Regular Updates**
```bash
# Update dependencies monthly
cd backend
pip list --outdated
pip install --upgrade -r requirements.txt

cd ../frontend
npm outdated
npm update
```

2. **Database Maintenance**
```sql
-- Weekly maintenance
VACUUM ANALYZE;
REINDEX DATABASE bakery_db;
```

3. **Security Scanning**
```bash
# Security audit
cd backend
pip install safety
safety check

cd ../frontend
npm audit
npm audit fix
```

## Rollback Strategy

### 1. Vercel Rollback (Landing & Web)
```bash
# List deployments
vercel ls

# Rollback to previous deployment
vercel rollback [deployment-url]

# Or use Vercel dashboard for instant rollback
```

### 2. Database Rollback
```bash
# View migration history
cd apps/backend
prisma migrate status

# Rollback last migration
prisma migrate resolve --rolled-back

# Create rollback migration
prisma migrate dev --name rollback_feature_x
```

### 3. Backend Rollback (Railway)
```bash
# Railway automatically keeps previous deployments
railway rollback

# Or rollback to specific deployment
railway deployments list
railway rollback [deployment-id]
```

### 4. Emergency Rollback Script
```bash
#!/bin/bash
# emergency-rollback.sh
set -e

echo "Starting emergency rollback..."

# Rollback landing site
cd apps/landing
vercel rollback --yes

# Rollback web app
cd ../web
vercel rollback --yes

# Rollback backend
cd ../backend
railway rollback

echo "Rollback complete!"
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] Environment variables configured
- [ ] Database migrations tested
- [ ] SSL certificates ready
- [ ] Domain DNS configured
- [ ] Monitoring setup

### Landing Site
- [ ] Custom domain configured
- [ ] Analytics tracking added
- [ ] SEO meta tags updated
- [ ] Sitemap generated
- [ ] Performance optimized

### Web Application
- [ ] Subdomain configured
- [ ] Clerk production keys
- [ ] Admin user created
- [ ] Error tracking enabled
- [ ] Rate limiting configured

### Backend API
- [ ] Database connection pooling
- [ ] Redis cache configured
- [ ] CORS origins updated
- [ ] File storage setup
- [ ] Email service configured

## Troubleshooting

### Common Issues

#### 1. Subdomain Routing Issues
```bash
# Check DNS propagation
dig app.yourdomain.com

# Verify nginx configuration
docker exec project_nginx nginx -t

# Check nginx logs
docker logs project_nginx
```

#### 2. CORS Errors Between Apps
```python
# Backend: Ensure all origins are listed
CORS_ORIGINS = [
    "https://yourdomain.com",
    "https://app.yourdomain.com",
    "http://localhost:3000",  # Landing dev
    "http://localhost:5173",  # Web dev
]
```

#### 3. Authentication Issues
- Verify CLERK_JWKS_URL matches your Clerk app
- Check web app has correct publishable key
- Ensure cookies work across subdomains

#### 4. Database Connection Issues
```bash
# Test connection from backend container
docker exec project_backend python -c "from prisma import Prisma; p = Prisma(); p.connect()"

# Check PostgreSQL logs
docker logs project_postgres
```

### Debug Mode
```python
# Enable debug logging
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Add request logging
@app.middleware("http")
async def log_requests(request: Request, call_next):
    logger.debug(f"{request.method} {request.url}")
    response = await call_next(request)
    logger.debug(f"Status: {response.status_code}")
    return response
```