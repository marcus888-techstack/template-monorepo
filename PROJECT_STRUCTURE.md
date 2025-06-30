# Project Structure

## Overview
This project follows a modern two-part architecture pattern commonly used for medium-sized SaaS applications, with a separate landing/marketing site and a main application that includes admin functionality.

```
[project-name]/
│
├── apps/                          # All applications (monorepo structure)
│   ├── landing/                   # Marketing/Landing site (React + Vite)
│   │   ├── src/                  # Source code
│   │   │   ├── components/       # Landing page components
│   │   │   │   ├── Hero.tsx     # Hero section
│   │   │   │   ├── Features.tsx # Features section
│   │   │   │   ├── Pricing.tsx  # Pricing section
│   │   │   │   └── CTA.tsx      # Call-to-action
│   │   │   ├── App.tsx          # Main app component
│   │   │   ├── main.tsx         # Entry point
│   │   │   └── index.css        # Global styles
│   │   ├── public/              # Static assets
│   │   ├── index.html           # HTML template
│   │   ├── .env.example         # Environment variables template
│   │   ├── vite.config.ts       # Vite configuration
│   │   ├── package.json         # Dependencies
│   │   └── tsconfig.json        # TypeScript config
│   │
│   ├── web/                       # Main application (React + Vite)
│   │   ├── src/                  # Source code
│   │   │   ├── components/       # React components
│   │   │   │   ├── common/      # Shared UI components
│   │   │   │   ├── layout/      # App layout components
│   │   │   │   ├── dashboard/   # Dashboard components
│   │   │   │   ├── content/     # Content-related components
│   │   │   │   ├── collections/ # Collection components
│   │   │   │   └── admin/       # Admin-specific components
│   │   │   ├── pages/           # Page components
│   │   │   │   ├── dashboard/   # User dashboard pages
│   │   │   │   ├── content/     # Content pages
│   │   │   │   ├── settings/    # User settings
│   │   │   │   └── admin/       # Admin dashboard pages
│   │   │   │       ├── Dashboard.tsx      # Admin overview
│   │   │   │       ├── ItemsManagement.tsx # Content management
│   │   │   │       ├── UsersManagement.tsx # User management
│   │   │   │       └── Activities.tsx      # Activity tracking
│   │   │   ├── guards/          # Route protection
│   │   │   │   ├── AuthGuard.tsx    # Authentication guard
│   │   │   │   └── AdminGuard.tsx   # Admin role guard
│   │   │   ├── hooks/           # Custom hooks
│   │   │   ├── services/        # API services
│   │   │   ├── store/           # State management (Zustand)
│   │   │   ├── utils/           # Utilities
│   │   │   ├── types/           # TypeScript types
│   │   │   ├── App.tsx          # App root component
│   │   │   └── main.tsx         # Entry point
│   │   ├── public/              # Static assets
│   │   ├── .env.example         # Environment variables template
│   │   ├── .dockerignore        # Docker ignore patterns
│   │   ├── Dockerfile           # Production Docker configuration
│   │   ├── Dockerfile.dev       # Development Docker configuration
│   │   ├── package.json         # Dependencies
│   │   ├── tsconfig.json        # TypeScript config
│   │   └── vite.config.ts       # Vite configuration
│   │
│   └── backend/                   # FastAPI application
│       ├── app/                  # Application code
│       │   ├── api/             # API endpoints
│       │   │   └── v1/          # API version 1
│       │   │       ├── public/  # Public endpoints
│       │   │       ├── auth/    # Authentication endpoints
│       │   │       └── admin/   # Admin-only endpoints
│       │   ├── core/            # Core configuration
│       │   ├── models/          # Pydantic models
│       │   ├── services/        # Business logic
│       │   ├── middleware/      # Middleware
│       │   └── utils/           # Utilities
│       ├── prisma/              # Database schema
│       │   └── schema.prisma    # Prisma schema
│       ├── scripts/             # Utility scripts
│       │   └── seed.py          # Database seeding
│       ├── tests/               # Test files
│       ├── .env.example         # Environment variables template
│       ├── .dockerignore        # Docker ignore patterns
│       ├── Dockerfile           # Production Docker configuration
│       ├── Dockerfile.dev       # Development Docker configuration
│       ├── main.py              # Application entry point
│       ├── requirements.txt     # Python dependencies
│       └── pyproject.toml       # Python project config
│
├── packages/                      # Shared packages (monorepo)
│   ├── ui/                       # Shared UI components
│   │   ├── src/
│   │   │   ├── components/      # Reusable components
│   │   │   ├── hooks/           # Shared hooks
│   │   │   └── styles/          # Shared styles
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── utils/                    # Shared utilities
│   │   ├── src/
│   │   │   ├── formatters/      # Data formatters
│   │   │   ├── validators/      # Validation functions
│   │   │   └── helpers/         # Helper functions
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── types/                    # Shared TypeScript types
│       ├── src/
│       │   ├── models/          # Data models
│       │   ├── api/             # API types
│       │   └── common/          # Common types
│       ├── package.json
│       └── tsconfig.json
│
├── docker/                        # Docker-related configurations
│   ├── nginx/                    # Nginx configuration
│   │   ├── nginx.conf           # Main Nginx configuration
│   │   └── conf.d/              # Site-specific configs
│   │       ├── landing.conf     # Landing site config
│   │       └── app.conf         # App + API config
│   ├── postgres/                # PostgreSQL configurations (if needed)
│   └── volumes/                 # Docker volumes (git-ignored)
│       ├── postgres/            # PostgreSQL data
│       ├── redis/               # Redis data
│       ├── pgadmin/             # pgAdmin configuration
│       └── uploads/             # File uploads
│
├── docs/                          # Project documentation
│   ├── API.md                    # API endpoints documentation
│   ├── ARCHITECTURE.md           # System architecture and design
│   ├── DATABASE.md               # Database schema and ERD
│   ├── DEPLOYMENT.md             # Deployment guide
│   ├── SPECIFICATION.md          # Technical specifications
│   ├── UI-SPEC.md                # UI/UX design specifications
│   ├── ADMIN_DASHBOARD.md        # Admin dashboard documentation
│   └── APPLICATION_STRUCTURE.md  # Application structure guide
│
├── .env.example                   # Docker Compose environment template
├── .gitignore                     # Git ignore patterns
├── docker-compose.yml             # Main Docker Compose configuration
├── docker-compose.dev.yml         # Development overrides
├── docker-compose.prod.yml        # Production overrides
├── turbo.json                     # Turborepo configuration (optional)
├── pnpm-workspace.yaml           # PNPM workspace config (or npm/yarn)
├── PROJECT_STRUCTURE.md           # This file
└── README.md                      # Main project documentation
```

## Architecture Overview

### Two-Part Architecture
1. **Landing Site** (`[domain].com`)
   - Marketing and conversion-focused
   - Simple single-page landing
   - React + Vite for consistency
   - Optimized for fast loading

2. **Web Application** (`app.[domain].com`)
   - Main user application
   - Admin dashboard integrated at `/admin`
   - React + Vite for fast development
   - Protected routes for authenticated users

### Shared Resources
- **Backend API**: Serves both landing and web app
- **Packages**: Shared code between applications
- **Database**: Single database for all data

## Directory Purposes

### `/apps`
Contains all applications in a monorepo structure:
- **landing**: Next.js marketing website
- **web**: React + Vite main application
- **backend**: FastAPI backend serving both

### `/packages`
Shared code between applications:
- **@[project]/ui**: Reusable UI components
- **@[project]/utils**: Common utilities
- **@[project]/types**: Shared TypeScript types

### `/docker`
All Docker-related configurations:
- **nginx/**: Reverse proxy with subdomain routing
- **volumes/**: Persistent data storage

### `/docs`
Comprehensive project documentation

## URL Structure

### Landing Site
```
[domain].com/                    # Single-page landing
[domain].com/#features           # Features section (anchor)
[domain].com/#pricing            # Pricing section (anchor)
[domain].com/#contact            # Contact section (anchor)
```

### Web Application
```
app.[domain].com/                # App dashboard
app.[domain].com/login           # Authentication
app.[domain].com/dashboard       # User dashboard
app.[domain].com/settings        # User settings
app.[domain].com/content         # Content management

# Admin Routes (protected)
app.[domain].com/admin           # Admin dashboard
app.[domain].com/admin/users     # User management
app.[domain].com/admin/content   # Content management
app.[domain].com/admin/analytics # Analytics
```

### API Endpoints
```
app.[domain].com/api/v1/         # API base
app.[domain].com/api/docs        # API documentation
```

## Development Workflow

### Initial Setup
```bash
# Clone repository
git clone [repository-url]
cd [project-name]

# Install dependencies (using pnpm)
pnpm install

# Copy environment files
cp .env.example .env
cp apps/landing/.env.example apps/landing/.env.local
cp apps/web/.env.example apps/web/.env.local
cp apps/backend/.env.example apps/backend/.env
```

### Running Applications

#### Using Docker (Recommended)
```bash
# Build images
docker-compose build

# Start all services
docker-compose up -d

# Access applications
# Landing: http://localhost:3000
# Web App: http://localhost:5173
# Backend: http://localhost:5000
```

#### Manual Development
```bash
# Terminal 1 - Landing site
cd apps/landing
pnpm dev

# Terminal 2 - Web application
cd apps/web
pnpm dev

# Terminal 3 - Backend
cd apps/backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload

# Terminal 4 - Database (if not using Docker)
docker run -p 5432:5432 -e POSTGRES_PASSWORD=password postgres:15
```

### Building for Production
```bash
# Build all applications
pnpm build

# Or build individually
cd apps/landing && pnpm build
cd apps/web && pnpm build
cd apps/backend && docker build -t [project]-backend .
```

## Deployment Strategy

### Landing Site
- Deploy to Vercel/Netlify (optimized for Next.js)
- Configure domain: `[domain].com`
- Enable CDN and edge caching

### Web Application
- Deploy to cloud provider (AWS/GCP/Azure)
- Configure subdomain: `app.[domain].com`
- Set up SSL certificates

### Backend API
- Deploy as containerized service
- Configure at: `app.[domain].com/api`
- Set up database and Redis

## Package Management

### Using Shared Packages
```typescript
// In apps/web/package.json
{
  "dependencies": {
    "@[project]/ui": "workspace:*",
    "@[project]/utils": "workspace:*",
    "@[project]/types": "workspace:*"
  }
}

// Usage in code
import { Button } from '@[project]/ui';
import { formatDate } from '@[project]/utils';
import { User } from '@[project]/types';
```

### Creating New Shared Components
```bash
# Add to UI package
cd packages/ui/src/components
# Create new component

# Build package
cd packages/ui
pnpm build

# Use in apps
pnpm install
```

## Benefits of This Structure

1. **Performance Optimization**
   - Landing site can be fully static/CDN-served
   - App bundle doesn't include marketing assets
   - Separate optimization strategies

2. **Development Efficiency**
   - Clear separation of concerns
   - Shared code through packages
   - Independent deployment cycles

3. **Scalability**
   - Easy to extract admin to separate app later
   - Can add more apps as needed
   - Natural microservices evolution path

4. **SEO & Marketing**
   - Landing site optimized for search engines
   - Fast page loads for better conversion
   - A/B testing without affecting app

5. **Security**
   - App code not exposed to public visitors
   - Separate authentication boundaries
   - Admin routes additionally protected