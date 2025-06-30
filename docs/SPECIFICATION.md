# Technical Specification - [Project Name]

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Decision](#architecture-decision)
3. [Functional Requirements](#functional-requirements)
4. [Non-Functional Requirements](#non-functional-requirements)
5. [User Stories](#user-stories)
6. [Technical Architecture](#technical-architecture)
7. [API Specification](#api-specification)
8. [Security Requirements](#security-requirements)
9. [Performance Requirements](#performance-requirements)

## Project Overview

### Description
A modern two-part web platform consisting of a marketing landing site and a full-featured web application. The landing site focuses on visitor conversion, while the web application provides authenticated users with content browsing, collection management, and data processing capabilities. The system includes an integrated admin panel within the web application for content and user management.

### Goals
- **Landing Site**: Maximize visitor conversion with optimized performance
- **Web Application**: Provide intuitive experience for authenticated users
- **Admin Integration**: Enable efficient content management within the main app
- **Shared Infrastructure**: Leverage unified backend and shared components
- **Security**: Ensure secure authentication and data protection
- **Performance**: Deliver fast, responsive experience across all devices

### Technology Stack

#### Landing Site
- **Framework**: React 18 with Vite
- **UI Components**: shadcn/ui
- **Styling**: Tailwind CSS
- **Animations**: Framer Motion
- **Deployment**: Vercel/Netlify (static hosting)

#### Web Application
- **Framework**: React 18 with TypeScript, Vite
- **UI Components**: shadcn/ui
- **Routing**: React Router v6
- **State Management**: Zustand
- **Data Fetching**: TanStack Query
- **Authentication**: Clerk
- **Forms**: React Hook Form + Zod (integrated with shadcn/ui form components)
- **Styling**: Tailwind CSS

#### Backend (Shared)
- **Framework**: FastAPI with Python
- **ORM**: Prisma
- **Database**: PostgreSQL
- **Cache**: Redis
- **Authentication**: JWT verification (from Clerk)

#### Infrastructure
- **Monorepo**: pnpm workspaces
- **Shared Packages**: @project/ui (shadcn/ui based), @project/utils, @project/types
- **Containerization**: Docker
- **Orchestration**: Docker Compose
- **Reverse Proxy**: Nginx (subdomain routing)

## Architecture Decision

### Two-Part Architecture
The project is structured as two separate applications:

1. **Landing Site** (`yourdomain.com`)
   - Public-facing marketing site
   - Optimized for SEO and conversion
   - Minimal JavaScript for fast loading
   - Static site generation capabilities
   - No authentication required

2. **Web Application** (`app.yourdomain.com`)
   - Full-featured SPA for authenticated users
   - Integrated admin dashboard at `/admin`
   - Protected routes and role-based access
   - Rich interactive features
   - Real-time updates

### Benefits of This Approach
- **Performance**: Landing site loads faster without app bundle
- **SEO**: Better search engine optimization for marketing content
- **Security**: Application code not exposed to public visitors
- **Deployment**: Independent deployment cycles
- **Scalability**: Can scale landing and app separately

## Functional Requirements

### 1. User Management

#### 1.1 Authentication
- **Sign Up**: Users can create accounts using email/password or social providers
- **Sign In**: Users can authenticate using credentials or social login
- **Sign Out**: Users can securely log out from their session
- **Password Reset**: Users can reset forgotten passwords via email
- **Profile Management**: Users can update their profile information

#### 1.2 User Roles
- **User**: Can browse content, manage collections, perform actions
- **Admin**: Full access to content management, data processing, and user administration

### 2. Landing Site Features

#### 2.1 Homepage Sections
- **Hero Section**: Eye-catching banner with CTA
- **Features Showcase**: Key platform benefits
- **Pricing/Plans**: Service tiers (if applicable)
- **Testimonials**: Customer reviews
- **FAQ Section**: Common questions
- **Contact Form**: Lead capture
- **Newsletter Signup**: Email list building

#### 2.2 Performance Features
- **Static Generation**: Pre-rendered pages
- **Image Optimization**: WebP with fallbacks
- **Lazy Loading**: Progressive image loading
- **Minimal JavaScript**: Fast initial load

### 3. Content Management

#### 3.1 Public Content Display (Both Sites)
- **Featured Items**: Highlighted on landing and web app
- **Category Navigation**: Browse by content type
- **Public API**: Accessible without authentication

#### 3.2 Authenticated Content (Web App Only)
- **Full Catalog**: All items with pagination
- **Advanced Search**: Filter by multiple criteria
- **Detailed Views**: Complete item information
- **User Interactions**: Add to collections

### 4. User Collections (Web App Only)

#### 4.1 Collection Operations
- **Add to Collection**: Add items with quantity/options selection
- **Update Selection**: Modify item properties in collection
- **Remove Items**: Delete items from collection
- **Collection Persistence**: Collections saved for authenticated users
- **Guest Collections**: Session-based for non-authenticated users

#### 4.2 Collection Features
- Real-time updates
- Validation checks
- Apply modifiers/rules
- Collection summary display
- Quick actions from any page

### 5. Data Processing (Web App Only)

#### 5.1 Processing Workflow
- **Data Review**: Display summary before confirmation
- **User Information**: Collect necessary details
- **Process Confirmation**: Generate reference number and send confirmation
- **Activity History**: Users can view past activities

#### 5.2 Process Status
- Pending, Processing, Completed, Cancelled
- Email notifications for status changes
- Activity tracking for users
- Real-time status updates

### 6. Admin Panel (Integrated in Web App)

#### 6.1 Access Control
- **Route Protection**: `/admin/*` routes require admin role
- **Role Verification**: Checked at both frontend and backend
- **Seamless Integration**: Same authentication as main app

#### 6.2 Content Management
- **CRUD Operations**: Create, read, update, delete content items
- **Media Upload**: Upload and manage images/files
- **Bulk Operations**: Update multiple items at once
- **Status Management**: Track item availability
- **Featured Selection**: Mark items for homepage display

#### 6.3 Data Management
- **Data Dashboard**: View all activities with filters
- **Status Updates**: Change process status
- **Data Details**: View complete activity information
- **Export Data**: Download activity reports

#### 6.4 User Management
- **User List**: View all registered users
- **Role Management**: Assign admin privileges
- **User Activity**: Track user activities and engagement
- **Contact Submissions**: View landing site inquiries

### 7. Cross-Application Features

#### 7.1 Shared Components (via packages)
- **UI Components**: Buttons, cards, modals
- **Utilities**: Formatters, validators
- **Type Definitions**: Consistent data models

#### 7.2 Navigation Flow
- **Landing → Web**: "Get Started" CTA
- **Web → Landing**: Logo link to marketing site
- **Deep Linking**: Preserve user intent

#### 7.3 Data Consistency
- **Shared Database**: Single source of truth
- **Public API**: Consistent data access
- **Cache Strategy**: Optimized for each use case

## Non-Functional Requirements

### 1. Performance
- Page load time < 3 seconds
- API response time < 200ms for simple queries
- Support 1000+ concurrent users
- Image optimization and lazy loading

### 2. Security
- HTTPS encryption for all communications
- Secure authentication with Clerk
- Input validation and sanitization
- SQL injection prevention with Prisma
- XSS protection
- CSRF protection

### 3. Usability
- Responsive design for all screen sizes
- Intuitive navigation
- Clear error messages
- Loading states for async operations
- Accessibility compliance (WCAG 2.1 AA)

### 4. Reliability
- 99.9% uptime
- Automated backups
- Error logging and monitoring
- Graceful error handling

### 5. Scalability
- Horizontal scaling capability
- Database optimization
- Caching strategy
- CDN for static assets

## User Stories

### User Stories

1. **As a user, I want to browse content by category so that I can find what I'm looking for quickly.**
   - Acceptance Criteria:
     - Categories are clearly visible
     - Content loads within 2 seconds
     - Can switch between categories without page reload

2. **As a user, I want to add items to my collection so that I can manage multiple selections.**
   - Acceptance Criteria:
     - "Add" button is prominent
     - Collection updates without page reload
     - Visual feedback when item is added

3. **As a user, I want to view my activity history so that I can review past actions.**
   - Acceptance Criteria:
     - History shows all past activities
     - Can view activity details
     - Can repeat actions with one click

### Admin Stories

1. **As an admin, I want to add new content so that I can update the platform.**
   - Acceptance Criteria:
     - Form with all content fields
     - Media upload functionality
     - Preview before saving

2. **As an admin, I want to manage user activities so that I can process user requests.**
   - Acceptance Criteria:
     - Dashboard shows pending activities
     - Can update activity status
     - Can view user details

## Technical Implementation Details

### Monorepo Structure
```
apps/
├── landing/         # Marketing site
├── web/            # Main application + admin
└── backend/        # Shared API

packages/
├── ui/             # Shared components
├── utils/          # Common utilities
└── types/          # TypeScript types
```

### Admin Dashboard Implementation
The admin dashboard remains integrated within the web application:
- **Location**: `apps/web/src/pages/admin/*`
- **Route Guards**: `AdminGuard` component
- **Shared Auth**: Same Clerk instance as main app
- **Benefits**: Code reuse, single deployment

## Technical Architecture

### Frontend Architecture

#### Landing Site Structure
```
apps/landing/src/
├── components/
│   ├── Hero.tsx           # Hero section
│   ├── Features.tsx       # Features showcase
│   ├── Pricing.tsx        # Pricing tiers
│   ├── Testimonials.tsx   # Customer reviews
│   ├── FAQ.tsx            # Frequently asked questions
│   └── ContactForm.tsx    # Contact form
├── App.tsx                # Main app
└── main.tsx               # Entry point
```

#### Web Application Structure
```
apps/web/src/
├── components/
│   ├── common/            # Shared UI components
│   ├── layout/            # App layout components
│   ├── content/           # Content components
│   ├── collections/       # Collection components
│   └── admin/            # Admin components
├── pages/
│   ├── dashboard/         # User pages
│   └── admin/            # Admin pages
├── guards/                # Route protection
├── hooks/                 # Custom hooks
├── services/              # API services
├── store/                 # Zustand stores
└── types/                 # TypeScript types
```

### Backend Architecture
```
backend/
├── app/
│   ├── api/              # API route endpoints
│   ├── core/             # Core configuration
│   ├── models/           # Pydantic models
│   ├── services/         # Business logic
│   ├── middleware/       # FastAPI middleware
│   └── utils/            # Utility functions
├── prisma/
│   ├── schema.prisma     # Database schema
│   └── migrations/       # Database migrations
```

### Database Schema

#### Core Tables (Shared)
- Users table (web app users via Clerk)
- Items table (content for both sites)
- Categories table (navigation structure)

#### Web Application Tables
- Activities table (user activities)
- ActivityItems table (activity details)
- Collections table (user collections)
- CollectionItems table (collection items)

#### Landing Site Tables
- ContactSubmissions table (form submissions)
- NewsletterSubscriptions table (email signups)

## API Specification

### Public Endpoints (No Auth Required)
- `GET /api/public/items/featured` - Featured items for homepage
- `GET /api/public/categories` - All categories
- `POST /api/public/contact` - Submit contact form
- `POST /api/public/newsletter` - Newsletter signup

### Authenticated Endpoints (Web App)
- `GET /api/items` - List all items
- `GET /api/items/:id` - Get item details
- `GET /api/categories/:slug/items` - Items by category
- `GET /api/collections` - User's collection
- `POST /api/collections/items` - Add to collection
- `PUT /api/collections/items/:id` - Update collection item
- `DELETE /api/collections/items/:id` - Remove from collection
- `GET /api/activities` - User's activities
- `POST /api/activities` - Create activity

### Admin Endpoints (Admin Role Required)
- `POST /api/items` - Create item
- `PUT /api/items/:id` - Update item
- `DELETE /api/items/:id` - Delete item
- `GET /api/admin/dashboard` - Dashboard stats
- `GET /api/admin/users` - List all users
- `PUT /api/admin/users/:id/role` - Update user role
- `PUT /api/activities/:id/status` - Update activity status
- `GET /api/admin/contact-submissions` - View submissions

## Security Requirements

### Authentication & Authorization

#### Landing Site
- No authentication required
- Rate limiting on contact forms
- CSRF protection on forms
- Honeypot fields for bot prevention

#### Web Application
- Clerk authentication required
- JWT tokens for API access
- Role-based access control (USER/ADMIN)
- Session management via Clerk

#### Backend
- JWT verification using JWKS
- Role extraction from token claims
- Admin middleware for protected routes
- API key for service-to-service (future)

### Data Protection
- HTTPS enforced on all domains
- Database encryption at rest
- Sensitive data hashing (future passwords)
- PII data isolation and audit trails

### CORS Configuration
```python
CORS_ORIGINS = [
    "https://yourdomain.com",      # Landing
    "https://app.yourdomain.com",  # Web app
    # Development origins
    "http://localhost:3000",
    "http://localhost:5173"
]
```

## Performance Requirements

### Landing Site Performance
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 3s
- **Lighthouse Score**: > 90
- **Core Web Vitals**: All green
- **Static Asset Caching**: 1 year
- **CDN Distribution**: Global edge nodes

### Web Application Performance
- **Initial Load**: < 3s
- **Route Changes**: < 200ms
- **API Response**: < 300ms (p95)
- **Collection Updates**: < 100ms
- **Search Results**: < 500ms

### Backend Performance
- **Database Queries**: < 50ms
- **Concurrent Users**: 1000+
- **Request Rate**: 100 req/s
- **Cache Hit Rate**: > 80%

### Optimization Strategies

#### Landing Site
- Static generation for all pages
- Image optimization pipeline
- Critical CSS inlining
- Minimal JavaScript bundle
- Preconnect to API domain

#### Web Application  
- Route-based code splitting
- Component lazy loading
- Virtual scrolling for lists
- Optimistic UI updates
- Service worker caching

#### Shared
- WebP images with fallbacks
- Brotli compression
- HTTP/2 push for critical resources
- Database query optimization
- Redis caching strategy

## Deployment Strategy

### Multi-Environment Architecture

#### Development
- **Landing**: `http://localhost:3000`
- **Web App**: `http://localhost:5173`
- **Backend**: `http://localhost:5000`
- **Database**: Local PostgreSQL
- **Tools**: Docker Compose, hot reload

#### Staging
- **Landing**: `staging.yourdomain.com`
- **Web App**: `app-staging.yourdomain.com`
- **Backend**: Shared with web app
- **Database**: Staging PostgreSQL
- **Purpose**: Pre-production testing

#### Production
- **Landing**: `yourdomain.com` (Vercel/Netlify)
- **Web App**: `app.yourdomain.com` (Cloud provider)
- **Backend**: `app.yourdomain.com/api`
- **Database**: Managed PostgreSQL (RDS/CloudSQL)
- **CDN**: CloudFlare for all domains

### Deployment Workflow

1. **Landing Site**
   - Push to main branch
   - Vercel auto-deploys
   - Instant global distribution
   
2. **Web Application**
   - Push to main branch
   - CI/CD builds Docker image
   - Deploy to cloud provider
   - Health checks before switch
   
3. **Backend API**
   - Same as web application
   - Database migrations first
   - Rolling deployment

### Infrastructure as Code
```terraform
# Example Terraform structure
modules/
├── landing/        # S3 + CloudFront
├── web-app/        # ECS/K8s + ALB
├── backend/        # Shared with web-app
├── database/       # RDS PostgreSQL
└── networking/     # VPC, subnets, security groups
```