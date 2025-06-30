# Full-Stack Web Application Template

A modern, full-stack web application template featuring a two-part architecture: a marketing landing site and a main application with integrated admin dashboard. Built with React, FastAPI, TypeScript, Prisma, and PostgreSQL.

![Application Preview](./docs/images/preview.png)

## 🚀 Features

### Two-Part Architecture
- **Landing Site** (`example.com`): Marketing-focused single-page site
- **Web Application** (`app.example.com`): Main application with integrated admin
- **Shared Backend**: Unified FastAPI backend serving both applications
- **Monorepo Structure**: Organized code with shared packages

### Core Features
- **Modern Tech Stack**: React with Vite, FastAPI, TypeScript, and Tailwind CSS
- **Authentication**: Clerk authentication with JWT verification
- **Content Management**: Browse items by categories, featured content showcase
- **User Collections**: Persistent user selections with real-time updates
- **Responsive Design**: Mobile-first approach with beautiful animations
- **Admin Dashboard**: Integrated admin panel at `/admin` routes
- **Database**: PostgreSQL with Prisma ORM (Python)
- **Docker Support**: Easy development setup with Docker Compose
- **Auto Documentation**: Swagger UI and ReDoc for API exploration

## 📋 Prerequisites

- Node.js 18+ and npm/yarn/pnpm (for frontend)
- Python 3.11+ (for backend)
- Docker and Docker Compose (for database)
- Git

## 🛠️ Tech Stack

### Landing Site (React + Vite)
- **React 18** - UI library
- **Vite** - Lightning-fast build tool
- **TypeScript** - Type safety
- **shadcn/ui** - Accessible and customizable component library
- **Tailwind CSS** - Utility-first CSS
- **Framer Motion** - Smooth animations

### Web Application (React + Vite)
- **React 18** - UI library
- **Vite** - Build tool and dev server
- **TypeScript** - Type safety
- **shadcn/ui** - Accessible and customizable component library
- **Tailwind CSS** - Utility-first CSS
- **React Router v6** - Client-side routing
- **Clerk React** - Authentication UI components
- **Framer Motion** - Animations
- **React Hook Form + Zod** - Form handling and validation
- **TanStack Query** - Data fetching and caching
- **Zustand** - State management
- **Axios** - HTTP client

### Backend (FastAPI)
- **FastAPI** - Modern Python web framework
- **Pydantic** - Data validation and settings
- **Prisma** - Database ORM (Python client)
- **PostgreSQL** - Database
- **Python-Jose[cryptography]** - JWT token verification
- **HTTPX** - HTTP client for JWKS fetching
- **Python-Multipart** - File uploads
- **Pillow** - Image processing and optimization
- **Uvicorn** - ASGI server
- **Python-dotenv** - Environment variables

### Shared Packages
- **@[project]/ui** - Reusable UI components (built on shadcn/ui)
- **@[project]/utils** - Common utilities
- **@[project]/types** - Shared TypeScript types

### DevOps
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration
- **Nginx** - Reverse proxy for subdomain routing
- **pnpm** - Efficient package management
- **Turborepo** - Monorepo build system (optional)
- **ESLint + Prettier** - Code quality
- **Husky** - Git hooks

## 🚀 Quick Start

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/[your-project].git
cd [your-project]
```

### 2. Install dependencies

#### Using pnpm (Recommended for monorepo)
```bash
# Install all dependencies
pnpm install
```

#### Or install individually

##### Landing Site
```bash
cd apps/landing
npm install
```

##### Web Application
```bash
cd apps/web
npm install
```

##### Backend
```bash
cd apps/backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Set up environment variables
```bash
# Copy all environment files
cp .env.example .env
cp apps/landing/.env.example apps/landing/.env.local
cp apps/web/.env.example apps/web/.env.local
cp apps/backend/.env.example apps/backend/.env
```

Landing Site `.env.local`:
```env
VITE_API_URL=http://localhost:5000/api
VITE_APP_URL=http://localhost:5173
```

Web Application `.env.local`:
```env
VITE_API_URL=http://localhost:5000/api
VITE_CLERK_PUBLISHABLE_KEY=pk_test_Y2xlcmsuZGV2JA  # Clerk dev mode key
```

Backend `.env`:
```env
DATABASE_URL="postgresql://[db_user]:[db_pass]@localhost:5432/[db_name]"
PORT=5000
CLERK_JWKS_URL=https://your-app.clerk.accounts.dev/.well-known/jwks.json
```

### 4. Build Docker images
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

### 5. Start the database
```bash
# From project root
docker-compose up -d postgres
```

### 6. Generate Prisma Client and Run migrations
```bash
cd apps/backend
prisma generate --schema=./prisma/schema.prisma
prisma migrate dev --schema=./prisma/schema.prisma
```

### 7. Seed the database (optional)
```bash
cd apps/backend
python -m scripts.seed
```

### 8. Start the development servers

#### Using pnpm (All apps)
```bash
# From project root
pnpm dev
```

#### Or start individually

##### Terminal 1 - Backend
```bash
cd apps/backend
source venv/bin/activate  # On Windows: venv\Scripts\activate
uvicorn main:app --reload --port 5000
```

##### Terminal 2 - Landing Site
```bash
cd apps/landing
npm run dev  # Runs on port 3000
```

##### Terminal 3 - Web Application
```bash
cd apps/web
npm run dev  # Runs on port 5173
```

#### Access Points
- Landing Site: [http://localhost:3000](http://localhost:3000)
- Web Application: [http://localhost:5173](http://localhost:5173)
- Admin Dashboard: [http://localhost:5173/admin](http://localhost:5173/admin)
- API Documentation: [http://localhost:5000/docs](http://localhost:5000/docs)
- Alternative API Docs: [http://localhost:5000/redoc](http://localhost:5000/redoc)

## 📁 Project Structure

```
[your-project]/
├── apps/                          # All applications (monorepo)
│   ├── landing/                   # Marketing/Landing site
│   │   ├── src/
│   │   │   ├── components/        # Landing page components
│   │   │   ├── App.tsx
│   │   │   └── main.tsx
│   │   ├── public/                # Static assets
│   │   └── package.json
│   ├── web/                       # Main application
│   │   ├── src/
│   │   │   ├── components/        # React components
│   │   │   ├── pages/            # Page components
│   │   │   │   └── admin/        # Admin dashboard
│   │   │   ├── hooks/            # Custom hooks
│   │   │   ├── services/         # API services
│   │   │   ├── store/            # Zustand stores
│   │   │   └── types/            # TypeScript types
│   │   └── package.json
│   └── backend/                   # FastAPI application
│       ├── app/
│       │   ├── api/              # API endpoints
│       │   ├── core/             # Core configuration
│       │   ├── models/           # Pydantic models
│       │   └── services/         # Business logic
│       ├── prisma/               # Database schema
│       └── main.py               # Entry point
├── packages/                      # Shared packages
│   ├── ui/                       # Reusable components
│   ├── utils/                    # Common utilities
│   └── types/                    # Shared TypeScript types
├── docker/                        # Docker configurations
│   ├── nginx/                    # Nginx reverse proxy
│   └── volumes/                  # Persistent data
├── docs/                          # Documentation
├── docker-compose.yml             # Docker orchestration
└── pnpm-workspace.yaml           # PNPM workspace config
```

## 📝 Available Scripts

### Monorepo Scripts (from root)
- `pnpm dev` - Start all development servers
- `pnpm build` - Build all applications
- `pnpm lint` - Run ESLint on all projects
- `pnpm format` - Format all code with Prettier

### Landing Site
- `npm run dev` - Start development server (port 3000)
- `npm run build` - Build for production
- `npm run preview` - Preview production build

### Web Application
- `npm run dev` - Start development server (port 5173)
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier

### Backend
- `uvicorn main:app --reload` - Start development server
- `uvicorn main:app` - Start production server
- `pytest` - Run tests
- `black .` - Format code
- `ruff check .` - Lint code
- `prisma generate` - Generate Prisma client
- `prisma migrate dev` - Run database migrations
- `prisma db push` - Push schema changes
- `prisma studio` - Open Prisma Studio
- `python -m scripts.seed` - Seed database

## 🐳 Docker Development

### Quick Start with Docker

1. **Copy environment file**
```bash
cp .env.example .env
```

2. **Build Docker images**
```bash
# Build all images
cd apps/landing && docker build -t [project]_landing:latest .
cd ../web && docker build -t [project]_web:latest .
cd ../backend && docker build -t [project]_backend:latest .
```

3. **Start all services**
```bash
# From project root
docker-compose up -d
```

This will start:
- PostgreSQL database (port 5432)
- pgAdmin database GUI (port 5050)
- Redis cache (port 6379)
- FastAPI backend (port 5000)
- Landing site (port 3000)
- Web application (port 5173)
- Nginx reverse proxy (port 80)

4. **Run database migrations**
```bash
docker-compose exec backend prisma migrate dev
```

5. **Seed the database (optional)**
```bash
docker-compose exec backend python -m scripts.seed
```

### Volume Management

All persistent data is stored in `docker/volumes/`:
- PostgreSQL data: `docker/volumes/postgres/`
- Redis data: `docker/volumes/redis/`
- pgAdmin config: `docker/volumes/pgadmin/`
- File uploads: `docker/volumes/uploads/`

### Docker Commands

```bash
# Build images first (required)
cd apps/landing && docker build -t [project]_landing:latest .
cd ../web && docker build -t [project]_web:latest .
cd ../backend && docker build -t [project]_backend:latest .

# Start services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Stop services
docker-compose down

# Clean all volume data (careful!)
rm -rf docker/volumes/*

# Backup volumes
tar -czf backup-$(date +%Y%m%d).tar.gz docker/volumes/

# Restore volumes
tar -xzf backup-20240101.tar.gz

# Run with development overrides
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Run with production settings
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

## 🔧 Configuration

### Database
Configure your database connection in backend `.env`:
```env
DATABASE_URL="postgresql://username:password@localhost:5432/database_name"
```

### Architecture Decision
This project uses a two-part architecture pattern:

1. **Landing Site** (`example.com`)
   - Marketing-focused single-page site
   - Optimized for SEO and conversion
   - Built with React + Vite for consistency
   - Can be deployed to Vercel/Netlify

2. **Web Application** (`app.example.com`)
   - Main user application
   - Integrated admin dashboard at `/admin`
   - Protected routes for authenticated users
   - Deployed to cloud provider

### Clerk Authentication
Clerk is used in the web application for user authentication. The backend verifies JWT tokens without Clerk SDK.

Web Application:
```env
VITE_CLERK_PUBLISHABLE_KEY=pk_test_Y2xlcmsuZGV2JA
```

Backend JWT Verification:
```env
CLERK_JWKS_URL=https://your-app.clerk.accounts.dev/.well-known/jwks.json
```

Note: The landing site doesn't require authentication. In production, create a Clerk account and update both the publishable key and JWKS URL.

## 📚 Documentation

- [Technical Specification](./docs/SPECIFICATION.md) - Detailed technical requirements
- [Database Schema](./docs/DATABASE.md) - Database design and ERD
- [API Documentation](./docs/API.md) - API endpoints reference
- [Architecture](./docs/ARCHITECTURE.md) - System design and architecture
- [UI Specification](./docs/UI-SPEC.md) - Design system and components
- [Admin Dashboard](./docs/ADMIN_DASHBOARD.md) - Admin panel documentation
- [Deployment Guide](./docs/DEPLOYMENT.md) - Production deployment instructions

## 🧪 Testing

```bash
# Run unit tests
npm run test

# Run e2e tests
npm run test:e2e

# Run with coverage
npm run test:coverage
```

## 🚀 Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for detailed deployment instructions.

### Quick Deploy

#### Landing Site (Vercel)
[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/yourusername/[your-project]&root-directory=apps/landing)

#### Web Application (Vercel)
[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/yourusername/[your-project]&root-directory=apps/web)

#### Backend (Railway)
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/yourusername/[your-project])

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Icons from [Heroicons](https://heroicons.com/)
- Images from [Unsplash](https://unsplash.com/)

## 📞 Support

For support, email support@[your-domain].com or open an issue in this repository.