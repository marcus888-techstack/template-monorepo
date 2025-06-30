# Deployment Guide

## Overview
This guide covers deployment strategies for the Bakery Website, including local development with Docker, staging environments, and production deployment options.

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
git clone https://github.com/yourusername/bakery-website.git
cd bakery-website
```

2. **Build Docker images**
```bash
# Build frontend image
cd apps/frontend
docker build -t bakery_frontend:latest .

# Build backend image
cd ../backend
docker build -t bakery_backend:latest .
```

3. **Start services**
```bash
# From project root
docker-compose up -d
```

This starts:
- Nginx reverse proxy (port 80)
- PostgreSQL database (port 5432)
- pgAdmin for database management (port 5050)
- Redis for caching (port 6379)
- FastAPI backend (port 5000)
- React frontend (port 5173)

4. **Initialize the database**
```bash
cd apps/backend
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
    container_name: bakery_postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-bakery_db}
      POSTGRES_USER: ${POSTGRES_USER:-bakery_user}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-bakery_pass}
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    volumes:
      - ./docker/volumes/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-bakery_user} -d ${POSTGRES_DB:-bakery_db}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - bakery_network

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: bakery_pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-admin@bakery.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
    ports:
      - "${PGADMIN_PORT:-5050}:80"
    volumes:
      - ./docker/volumes/pgadmin:/var/lib/pgadmin
    depends_on:
      - postgres
    networks:
      - bakery_network

  redis:
    image: redis:7-alpine
    container_name: bakery_redis
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - ./docker/volumes/redis:/data
    networks:
      - bakery_network

  backend:
    image: bakery_backend:latest
    container_name: bakery_backend
    ports:
      - "${BACKEND_PORT:-5000}:5000"
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER:-bakery_user}:${POSTGRES_PASSWORD:-bakery_pass}@postgres:5432/${POSTGRES_DB:-bakery_db}
      REDIS_URL: redis://redis:6379
    volumes:
      - ./apps/backend:/app
      - ./docker/volumes/uploads:/app/uploads
    depends_on:
      - postgres
      - redis
    networks:
      - bakery_network

  frontend:
    image: bakery_frontend:latest
    container_name: bakery_frontend
    ports:
      - "${FRONTEND_PORT:-5173}:5173"
    environment:
      VITE_API_URL: http://localhost:${BACKEND_PORT:-5000}/api
    volumes:
      - ./apps/frontend:/app
    depends_on:
      - backend
    networks:
      - bakery_network

  nginx:
    image: nginx:alpine
    container_name: bakery_nginx
    ports:
      - "${NGINX_PORT:-80}:80"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./docker/nginx/conf.d:/etc/nginx/conf.d:ro
    depends_on:
      - frontend
      - backend
    networks:
      - bakery_network

networks:
  bakery_network:
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

### Dockerfile - Frontend
```dockerfile
# frontend/Dockerfile
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

### nginx.conf
```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # SPA routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

## Environment Variables

### Backend (.env)
```env
# Database
DATABASE_URL="postgresql://bakery_user:bakery_pass@localhost:5432/bakery_db"

# Server
PORT=5000
ENVIRONMENT=development

# Security
SECRET_KEY="your-secret-key-here"
CORS_ORIGINS=["http://localhost:5173", "http://localhost:3000"]

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

### Frontend (.env.local)
```env
# API Configuration
VITE_API_URL=http://localhost:5000/api

# Clerk
VITE_CLERK_PUBLISHABLE_KEY=pk_test_your-clerk-key

# Features
VITE_ENABLE_ANALYTICS=false
```

### Production Environment Variables
```env
# Additional production settings
NODE_ENV=production
ENVIRONMENT=production

# Database (use connection pooling)
DATABASE_URL="postgresql://user:pass@db-host:5432/bakery_db?sslmode=require"

# Security
SECRET_KEY="<generate-strong-secret>"
ALLOWED_HOSTS=["bakery.yourdomain.com", "www.bakery.yourdomain.com"]

# File Storage (S3 or similar)
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"
S3_BUCKET_NAME="bakery-uploads"

# Monitoring
SENTRY_DSN="https://your-sentry-dsn"
```

## Production Deployment

### Option 1: Railway.app (Recommended for simplicity)

1. **Backend Deployment**
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
cd backend
railway init

# Link PostgreSQL
railway add postgresql

# Deploy
railway up
```

2. **Frontend Deployment (Vercel)**
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd frontend
vercel --prod
```

### Option 2: DigitalOcean App Platform

1. **Create app.yaml**
```yaml
name: bakery-website
region: nyc
services:
  - name: backend
    github:
      repo: yourusername/bakery-website
      branch: main
      deploy_on_push: true
    source_dir: backend
    environment_slug: python
    build_command: pip install -r requirements.txt && prisma generate
    run_command: uvicorn main:app --host 0.0.0.0 --port 8080
    http_port: 8080
    instance_count: 1
    instance_size_slug: basic-xxs
    envs:
      - key: DATABASE_URL
        scope: RUN_TIME
        value: ${bakery-db.DATABASE_URL}
      - key: CLERK_JWKS_URL
        scope: RUN_TIME
        value: "your-clerk-jwks-url"

  - name: frontend
    github:
      repo: yourusername/bakery-website
      branch: main
      deploy_on_push: true
    source_dir: frontend
    environment_slug: node-js
    build_command: npm install && npm run build
    output_dir: dist
    routes:
      - path: /

databases:
  - name: bakery-db
    engine: PG
    version: "15"
```

### Option 3: AWS Deployment

#### Backend (ECS + Fargate)
```bash
# Build and push Docker image
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_URI
docker build -t bakery-backend backend/
docker tag bakery-backend:latest $ECR_URI/bakery-backend:latest
docker push $ECR_URI/bakery-backend:latest

# Update ECS service
aws ecs update-service --cluster bakery-cluster --service bakery-backend --force-new-deployment
```

#### Frontend (S3 + CloudFront)
```bash
# Build frontend
cd frontend
npm run build

# Sync to S3
aws s3 sync dist/ s3://bakery-frontend-bucket --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id $DISTRIBUTION_ID --paths "/*"
```

### Option 4: Kubernetes Deployment

#### backend-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bakery-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bakery-backend
  template:
    metadata:
      labels:
        app: bakery-backend
    spec:
      containers:
      - name: backend
        image: your-registry/bakery-backend:latest
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: bakery-secrets
              key: database-url
        - name: CLERK_JWKS_URL
          value: "your-clerk-jwks-url"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: bakery-backend-service
spec:
  selector:
    app: bakery-backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

## CI/CD Pipeline

### GitHub Actions

#### .github/workflows/deploy.yml
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        working-directory: ./backend
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        working-directory: ./backend
        run: pytest

  deploy-backend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Railway
        run: |
          npm install -g @railway/cli
          railway up
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        run: |
          npm install -g vercel
          vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
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

1. **Database Indexes**
```sql
-- Add indexes for common queries
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_featured ON products(featured) WHERE featured = true;
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
```

2. **Caching Strategy**
```python
from functools import lru_cache
import redis

redis_client = redis.from_url(os.getenv("REDIS_URL"))

@lru_cache(maxsize=100)
async def get_categories():
    # Cache categories for 1 hour
    cached = redis_client.get("categories")
    if cached:
        return json.loads(cached)
    
    categories = await prisma.category.find_many()
    redis_client.setex("categories", 3600, json.dumps(categories))
    return categories
```

3. **CDN Configuration**
- Use Cloudflare or AWS CloudFront
- Cache static assets (images, CSS, JS)
- Set appropriate cache headers

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

1. **Database Migrations**
```bash
# Rollback last migration
prisma migrate resolve --rolled-back

# Apply specific migration
prisma migrate deploy
```

2. **Application Rollback**
- Keep 3 previous versions
- Use blue-green deployment
- Test rollback procedures regularly

3. **Quick Rollback Script**
```bash
#!/bin/bash
# rollback.sh
PREVIOUS_VERSION=$1
docker pull your-registry/bakery-backend:$PREVIOUS_VERSION
kubectl set image deployment/bakery-backend backend=your-registry/bakery-backend:$PREVIOUS_VERSION
kubectl rollout status deployment/bakery-backend
```

## Troubleshooting

### Common Issues

1. **Database Connection Issues**
- Check DATABASE_URL format
- Verify network connectivity
- Check connection pool settings

2. **CORS Errors**
- Verify CORS_ORIGINS in backend
- Check API_URL in frontend

3. **File Upload Issues**
- Check file permissions
- Verify upload directory exists
- Check file size limits

4. **Authentication Issues**
- Verify CLERK_JWKS_URL
- Check token expiration
- Validate CORS for auth requests

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