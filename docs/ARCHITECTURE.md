# System Architecture

## Overview
The Bakery Website follows a modern microservices architecture with a clear separation between frontend and backend. The system uses React for the client-side application and FastAPI for the server-side API, with PostgreSQL as the primary data store.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
├─────────────────────────────────────────────────────────────────┤
│                    React SPA (Vite + TypeScript)                 │
│                         Port: 5173                               │
│  ┌──────────────┬───────────────┬────────────────┬────────────┐ │
│  │   Pages      │  Components   │     Hooks      │   Store    │ │
│  │              │               │                │  (Zustand) │ │
│  └──────────────┴───────────────┴────────────────┴────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ HTTPS (Axios)
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                           │
├─────────────────────────────────────────────────────────────────┤
│                    FastAPI Application                           │
│                         Port: 5000                               │
│  ┌──────────────┬───────────────┬────────────────┬────────────┐ │
│  │   Routers    │  Dependencies │   Middleware   │   Models   │ │
│  │              │    (Auth)     │    (CORS)      │ (Pydantic) │ │
│  └──────────────┴───────────────┴────────────────┴────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Business Logic Layer                       │
├─────────────────────────────────────────────────────────────────┤
│                         Service Classes                          │
│  ┌──────────────┬───────────────┬────────────────┬────────────┐ │
│  │   Product    │     Cart      │     Order      │    User    │ │
│  │   Service    │   Service     │    Service     │  Service   │ │
│  └──────────────┴───────────────┴────────────────┴────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ Prisma ORM
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Data Layer                               │
├─────────────────────────────────────────────────────────────────┤
│                      PostgreSQL Database                         │
│                         Port: 5432                               │
│  ┌──────────────┬───────────────┬────────────────┬────────────┐ │
│  │    Users     │   Products    │    Orders      │    Cart    │ │
│  │              │  Categories   │  OrderItems    │ CartItems  │ │
│  └──────────────┴───────────────┴────────────────┴────────────┘ │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      External Services                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┬───────────────┬────────────────────────────┐ │
│  │    Clerk     │  File Storage │     Email Service          │ │
│  │ (Frontend    │   (Local)     │    (Future: SMTP)          │ │
│  │   Auth)      │               │                            │ │
│  └──────────────┴───────────────┴────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Component Architecture

### Frontend (React)

```
frontend/
├── src/
│   ├── components/           # Reusable UI components
│   │   ├── common/          # Generic components (Button, Input, etc.)
│   │   ├── layout/          # Layout components (Header, Footer)
│   │   ├── products/        # Product-specific components
│   │   ├── cart/           # Shopping cart components
│   │   └── admin/          # Admin panel components
│   │
│   ├── pages/              # Page-level components
│   │   ├── Home.tsx
│   │   ├── Products.tsx
│   │   ├── ProductDetail.tsx
│   │   ├── Cart.tsx
│   │   ├── Checkout.tsx
│   │   └── Admin/
│   │
│   ├── hooks/              # Custom React hooks
│   │   ├── useAuth.ts      # Clerk authentication
│   │   ├── useCart.ts      # Cart operations
│   │   ├── useProducts.ts  # Product data fetching
│   │   └── useOrders.ts    # Order management
│   │
│   ├── services/           # API communication layer
│   │   ├── api.ts         # Axios instance configuration
│   │   ├── auth.ts        # Authentication services
│   │   ├── products.ts    # Product API calls
│   │   ├── cart.ts        # Cart API calls
│   │   └── orders.ts      # Order API calls
│   │
│   ├── store/             # State management (Zustand)
│   │   ├── authStore.ts   # User authentication state
│   │   ├── cartStore.ts   # Shopping cart state
│   │   └── uiStore.ts     # UI state (modals, notifications)
│   │
│   ├── utils/             # Utility functions
│   │   ├── constants.ts   # App constants
│   │   ├── helpers.ts     # Helper functions
│   │   └── validators.ts  # Form validation
│   │
│   └── types/             # TypeScript type definitions
│       ├── models.ts      # Data models
│       ├── api.ts         # API types
│       └── components.ts  # Component prop types
```

### Backend (FastAPI)

```
backend/
├── app/
│   ├── api/               # API endpoints
│   │   ├── __init__.py
│   │   ├── deps.py       # Common dependencies
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── auth.py        # Authentication endpoints
│   │       ├── products.py    # Product endpoints
│   │       ├── categories.py  # Category endpoints
│   │       ├── cart.py        # Cart endpoints
│   │       ├── orders.py      # Order endpoints
│   │       └── uploads.py     # File upload endpoints
│   │
│   ├── core/              # Core configuration
│   │   ├── __init__.py
│   │   ├── config.py     # Settings management
│   │   ├── security.py   # Security utilities
│   │   └── deps.py       # Dependency injection
│   │
│   ├── models/           # Pydantic models
│   │   ├── __init__.py
│   │   ├── user.py       # User schemas
│   │   ├── product.py    # Product schemas
│   │   ├── cart.py       # Cart schemas
│   │   ├── order.py      # Order schemas
│   │   └── common.py     # Common schemas
│   │
│   ├── services/         # Business logic
│   │   ├── __init__.py
│   │   ├── auth.py       # Authentication service
│   │   ├── product.py    # Product service
│   │   ├── cart.py       # Cart service
│   │   ├── order.py      # Order service
│   │   └── email.py      # Email service
│   │
│   ├── middleware/       # FastAPI middleware
│   │   ├── __init__.py
│   │   ├── auth.py       # Authentication middleware
│   │   ├── cors.py       # CORS configuration
│   │   └── logging.py    # Request logging
│   │
│   └── utils/           # Utility functions
│       ├── __init__.py
│       ├── files.py     # File handling
│       ├── images.py    # Image processing
│       └── validators.py # Custom validators
│
├── prisma/              # Database schema
│   └── schema.prisma
│
├── scripts/             # Utility scripts
│   ├── __init__.py
│   └── seed.py         # Database seeding
│
├── uploads/            # File uploads directory
│   └── products/
│
├── tests/              # Test files
│   ├── __init__.py
│   ├── conftest.py    # Test configuration
│   └── api/           # API tests
│
├── main.py            # FastAPI app initialization
├── requirements.txt   # Python dependencies
└── .env              # Environment variables
```

## Data Flow

### 1. Authentication Flow
```
User Login Request
    │
    ▼
React App ──────► Clerk Auth ──────► JWT Token Issued
    │                                        │
    │                                        ▼
    └────────────────────────────► FastAPI (verify JWT)
                                            │
                                            ▼
                                    Extract User Info
                                            │
                                            ▼
                                     Database Query
```

### 2. Product Browsing Flow
```
User Browse Products
    │
    ▼
React Component ──────► API Service ──────► FastAPI Router
    │                                              │
    │                                              ▼
    │                                       Product Service
    │                                              │
    │                                              ▼
    │                                       Prisma Query
    │                                              │
    │                                              ▼
    └────────────── Response ◄─────────── PostgreSQL
```

### 3. Order Processing Flow
```
User Checkout
    │
    ▼
Cart Store ──────► Order API ──────► FastAPI Router
    │                                       │
    │                                       ▼
    │                                Order Service
    │                                       │
    │                                       ├──► Validate Cart
    │                                       ├──► Calculate Total
    │                                       ├──► Create Order
    │                                       └──► Clear Cart
    │                                              │
    └──────── Order Confirmation ◄───────────────┘
```

## Security Architecture

### Authentication & Authorization
- **Clerk (Frontend Only)**: Handles user authentication and UI
- **JWT Verification**: FastAPI validates tokens using JWKS
- **Role-Based Access**: Admin vs Customer permissions from JWT claims
- **Token Validation**: python-jose with RS256 algorithm

### Data Protection
- **HTTPS**: All communications encrypted
- **Input Validation**: Pydantic models validate all inputs
- **SQL Injection Prevention**: Prisma parameterized queries
- **XSS Protection**: React's built-in escaping
- **CORS Configuration**: Restricted origins

### File Upload Security
- **File Type Validation**: Only images allowed
- **Size Limits**: Maximum 5MB per file
- **Filename Sanitization**: Prevent directory traversal
- **Storage Isolation**: Uploads in separate directory

## Performance Optimization

### Frontend Optimization
- **Code Splitting**: Dynamic imports for routes
- **Lazy Loading**: Images and components
- **Caching**: TanStack Query for API responses
- **Bundle Optimization**: Vite tree-shaking
- **Image Optimization**: WebP format, responsive images

### Backend Optimization
- **Async Operations**: FastAPI async endpoints
- **Database Indexing**: Optimized queries
- **Connection Pooling**: Prisma connection management
- **Response Caching**: Redis (future implementation)
- **Static File Serving**: Nginx in production

### Database Optimization
- **Indexed Columns**: Fast lookups
- **Query Optimization**: Efficient JOINs
- **Connection Pooling**: Managed by Prisma
- **Data Pagination**: Limit result sets

## Scalability Considerations

### Horizontal Scaling
```
                    Load Balancer
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   FastAPI Instance  FastAPI Instance  FastAPI Instance
        │                │                │
        └────────────────┼────────────────┘
                         │
                    PostgreSQL
                    (Primary)
                         │
                ┌────────┴────────┐
                │                 │
           Read Replica      Read Replica
```

### Caching Strategy
- **API Response Cache**: Redis for frequently accessed data
- **Static Assets**: CDN for images and files
- **Database Query Cache**: PostgreSQL query caching
- **Session Storage**: Redis for user sessions

### Future Enhancements
1. **Microservices Split**: Separate order processing service
2. **Message Queue**: RabbitMQ for async operations
3. **Search Service**: Elasticsearch for product search
4. **Analytics Service**: Separate analytics database
5. **CDN Integration**: CloudFlare for global distribution

## Deployment Architecture

### Docker Organization
```
docker/
├── nginx/                  # Nginx configuration
│   ├── nginx.conf         # Main Nginx config
│   └── conf.d/           # Additional configs
├── postgres/             # PostgreSQL configs (optional)
└── volumes/              # Persistent data (git-ignored)
    ├── postgres/        # Database files
    ├── redis/          # Cache data
    ├── pgadmin/        # pgAdmin settings
    └── uploads/        # User uploads
```

### Development Environment
```
Docker Compose (Runtime Only)
├── nginx:alpine          # Reverse proxy
├── postgres:15-alpine    # Database
├── redis:7-alpine       # Cache
├── pgadmin4:latest      # DB management
├── bakery_backend       # FastAPI app (pre-built image)
└── bakery_frontend      # React app (pre-built image)
```

All services use bind mounts for volumes:
- `./docker/volumes/postgres:/var/lib/postgresql/data`
- `./docker/volumes/redis:/data`
- `./docker/volumes/pgadmin:/var/lib/pgadmin`
- `./docker/volumes/uploads:/app/uploads`

### Production
```
Cloud Provider (AWS/GCP/Azure)
├── Application Load Balancer
├── ECS/Kubernetes Cluster
│   ├── FastAPI Containers (Auto-scaling)
│   └── React App (Static hosting)
├── RDS PostgreSQL (Multi-AZ)
├── ElastiCache Redis
├── S3/Cloud Storage (File uploads)
└── CloudFront CDN
```

## Monitoring & Logging

### Application Monitoring
- **APM**: DataDog or New Relic
- **Error Tracking**: Sentry
- **Uptime Monitoring**: Pingdom
- **Performance Metrics**: Custom dashboards

### Logging Strategy
- **Structured Logging**: JSON format
- **Centralized Logs**: ELK Stack
- **Log Levels**: DEBUG, INFO, WARNING, ERROR
- **Request Tracing**: Correlation IDs

### Health Checks
```python
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "database": check_database(),
        "storage": check_storage(),
        "external_services": check_external_services()
    }
```