# Project Structure

```
bakery-website/
│
├── apps/                          # All applications (monorepo structure)
│   ├── frontend/                  # React application
│   │   ├── src/                  # Source code
│   │   │   ├── components/       # React components
│   │   │   ├── pages/           # Page components
│   │   │   ├── hooks/           # Custom hooks
│   │   │   ├── services/        # API services
│   │   │   ├── store/           # State management
│   │   │   ├── utils/           # Utilities
│   │   │   ├── types/           # TypeScript types
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
├── docker/                        # Docker-related configurations
│   ├── nginx/                    # Nginx configuration
│   │   ├── nginx.conf           # Main Nginx configuration
│   │   └── conf.d/              # Additional Nginx configs
│   ├── postgres/                # PostgreSQL configurations (if needed)
│   └── volumes/                 # Docker volumes (git-ignored)
│       ├── postgres/            # PostgreSQL data
│       ├── redis/               # Redis data
│       ├── pgadmin/             # pgAdmin configuration
│       └── uploads/             # File uploads
│
├── packages/                      # Shared packages (future use)
│   └── shared/                   # Shared utilities/types
│
├── docs/                          # Project documentation
│   ├── API.md                    # API endpoints documentation
│   ├── ARCHITECTURE.md           # System architecture and design
│   ├── DATABASE.md               # Database schema and ERD
│   ├── DEPLOYMENT.md             # Deployment guide
│   ├── SPECIFICATION.md          # Technical specifications
│   └── UI-SPEC.md                # UI/UX design specifications
│
├── .env.example                   # Docker Compose environment template
├── .gitignore                     # Git ignore patterns
├── docker-compose.yml             # Main Docker Compose configuration
├── docker-compose.dev.yml         # Development overrides
├── docker-compose.prod.yml        # Production overrides
├── PROJECT_STRUCTURE.md           # This file
└── README.md                      # Main project documentation
```

## Directory Purposes

### `/apps`
Contains all applications in a monorepo structure:
- **frontend**: React + Vite + TypeScript application
- **backend**: FastAPI + Python application

### `/docker`
All Docker-related configurations:
- **nginx/**: Nginx reverse proxy configuration
- **postgres/**: PostgreSQL custom configurations (if needed)
- **volumes/**: Persistent data storage (git-ignored)
  - PostgreSQL database files
  - Redis cache data
  - pgAdmin configuration
  - Uploaded files

### `/docs`
Comprehensive project documentation:
- Technical specifications
- API documentation
- Database design
- Architecture diagrams
- Deployment guides
- UI/UX specifications

### Root Level Files
- **Docker Compose files**: Orchestrate all services (runtime only, no build configs)
- **.env.example**: Template for Docker Compose environment variables
- **README.md**: Main entry point for project documentation

## Getting Started

1. Clone the repository
2. Copy `.env.example` to `.env`
3. Run `docker-compose up -d` to start all services
4. Follow the setup instructions in README.md

## Development Workflow

### Using Docker (Recommended)
```bash
# Note: Images must be built separately before running docker-compose
# Build frontend image
cd apps/frontend
docker build -t bakery_frontend:latest .

# Build backend image
cd apps/backend
docker build -t bakery_backend:latest .

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Clean volumes (careful - deletes all data)
rm -rf docker/volumes/*
```

### Manual Setup
```bash
# Frontend
cd apps/frontend
npm install
npm run dev

# Backend (in another terminal)
cd apps/backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

## Volume Management

All persistent data is stored in `docker/volumes/`:
- **postgres/**: Database files
- **redis/**: Cache data
- **pgadmin/**: pgAdmin settings
- **uploads/**: User uploaded files

To backup volumes:
```bash
tar -czf backup.tar.gz docker/volumes/
```

To restore volumes:
```bash
tar -xzf backup.tar.gz
```