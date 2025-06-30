# 🥐 Delicious Bakery Website

A modern, full-stack e-commerce website for a bakery built with React, FastAPI, TypeScript, Prisma, and PostgreSQL. Features a beautiful UI inspired by the Figma design with product catalog, shopping cart, and order management.

![Bakery Website Preview](./docs/images/preview.png)

## 🚀 Features

- **Modern Tech Stack**: React with Vite, FastAPI, TypeScript, and Tailwind CSS
- **Authentication**: Clerk authentication in dev mode (zero configuration required)
- **Product Management**: Browse products by categories, featured items showcase
- **Shopping Cart**: Persistent cart with real-time updates
- **Responsive Design**: Mobile-first approach with beautiful animations
- **Admin Panel**: Manage products, orders, and users
- **Database**: PostgreSQL with Prisma ORM (Python)
- **Docker Support**: Easy development setup with Docker Compose
- **Auto Documentation**: Swagger UI and ReDoc for API exploration

## 📋 Prerequisites

- Node.js 18+ and npm/yarn/pnpm (for frontend)
- Python 3.11+ (for backend)
- Docker and Docker Compose (for database)
- Git

## 🛠️ Tech Stack

### Frontend (React)
- **React 18** - UI library
- **Vite** - Build tool and dev server
- **TypeScript** - Type safety
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

### DevOps
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration
- **ESLint + Prettier** - Code quality
- **Husky** - Git hooks

## 🚀 Quick Start

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/bakery-website.git
cd bakery-website
```

### 2. Install dependencies

#### Frontend
```bash
cd apps/frontend
npm install
```

#### Backend
```bash
cd apps/backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Set up environment variables
```bash
# In frontend directory
cd apps/frontend
cp .env.example .env.local

# In backend directory
cd apps/backend
cp .env.example .env
```

Frontend `.env.local`:
```env
VITE_API_URL=http://localhost:5000/api
VITE_CLERK_PUBLISHABLE_KEY=pk_test_Y2xlcmsuZGV2JA  # Clerk dev mode key
```

Backend `.env`:
```env
DATABASE_URL="postgresql://bakery_user:bakery_pass@localhost:5432/bakery_db"
PORT=5000
CLERK_JWKS_URL=https://your-app.clerk.accounts.dev/.well-known/jwks.json
```

### 4. Build Docker images
```bash
# Build frontend image
cd apps/frontend
docker build -t bakery_frontend:latest .

# Build backend image
cd ../backend
docker build -t bakery_backend:latest .
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

#### Terminal 1 - Backend
```bash
cd apps/backend
source venv/bin/activate  # On Windows: venv\Scripts\activate
uvicorn main:app --reload --port 5000
```

#### Terminal 2 - Frontend
```bash
cd apps/frontend
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) to see the application.
Visit [http://localhost:5000/docs](http://localhost:5000/docs) for API documentation (Swagger UI).
Visit [http://localhost:5000/redoc](http://localhost:5000/redoc) for alternative API documentation.

## 📁 Project Structure

```
bakery-website/
├── apps/
│   ├── frontend/          # React application
│   │   ├── src/
│   │   │   ├── components/    # React components
│   │   │   ├── pages/        # Page components
│   │   │   ├── hooks/        # Custom hooks
│   │   │   ├── services/     # API services
│   │   │   ├── store/        # Zustand stores
│   │   │   ├── utils/        # Utilities
│   │   │   └── types/        # TypeScript types
│   │   ├── public/           # Static assets
│   │   └── index.html        # Entry HTML
│   └── backend/           # FastAPI application
│       ├── app/
│       │   ├── api/          # API endpoints
│       │   ├── core/         # Core configuration
│       │   ├── models/       # Pydantic models
│       │   ├── services/     # Business logic
│       │   ├── utils/        # Utilities
│       │   └── middleware/   # FastAPI middleware
│       ├── prisma/           # Database schema
│       └── main.py           # Application entry point
├── docker/                # Docker configurations
│   ├── nginx/            # Nginx reverse proxy
│   └── volumes/          # Persistent data storage
├── docs/                  # Documentation
└── docker-compose.yml     # Docker orchestration
```

## 📝 Available Scripts

### Frontend
- `npm run dev` - Start development server
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
cd apps/frontend && docker build -t bakery_frontend:latest .
cd ../backend && docker build -t bakery_backend:latest .
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
- React frontend (port 5173)
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
cd apps/frontend && docker build -t bakery_frontend:latest .
cd ../backend && docker build -t bakery_backend:latest .

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

### Clerk Authentication
Clerk is used only on the frontend for user authentication. The backend verifies JWT tokens without Clerk SDK.

Frontend (Dev Mode):
```env
VITE_CLERK_PUBLISHABLE_KEY=pk_test_Y2xlcmsuZGV2JA
```

Backend JWT Verification:
```env
CLERK_JWKS_URL=https://your-app.clerk.accounts.dev/.well-known/jwks.json
```

Note: In dev mode, you can use the test publishable key. In production, create a Clerk account and update both the publishable key and JWKS URL.

## 📚 Documentation

- [Technical Specification](./docs/SPECIFICATION.md) - Detailed technical requirements
- [Database Schema](./docs/DATABASE.md) - Database design and ERD
- [API Documentation](./docs/API.md) - API endpoints reference
- [Architecture](./docs/ARCHITECTURE.md) - System design and architecture
- [UI Specification](./docs/UI-SPEC.md) - Design system and components
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

### Quick Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/yourusername/bakery-website)

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Design inspired by [Bakery Website UI Figma](https://www.figma.com/design/3qSw3ALIPtBzR3NWr0PR3h/)
- Icons from [Heroicons](https://heroicons.com/)
- Images from [Unsplash](https://unsplash.com/)

## 📞 Support

For support, email support@deliciousbakery.com or open an issue in this repository.